# @RestController

> Nivel: esencial · Categoría: web / REST

## 1. Nombre
`@RestController`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`), usado por Spring MVC y Spring WebFlux. Desde Spring 4.0.

## 4. Propósito
Declarar un **controlador REST**: cada método handler escribe su valor de retorno directamente en el cuerpo de la respuesta HTTP (normalmente JSON), en vez de resolver una vista.

## 5. Target
`ElementType.TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`). Meta‑anotada con `@Controller` y `@ResponseBody`.

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` | `String` | Nombre del bean (`@AliasFor(annotation = Controller.class)`). |

## 8. Valores por defecto
`value = ""`.

## 9. Quién la procesa
- `ClassPathBeanDefinitionScanner` (registro, por ser `@Component`).
- `RequestMappingHandlerMapping` (mapeo de rutas, por ser `@Controller`).
- `RequestResponseBodyMethodProcessor` (escritura del cuerpo, por el `@ResponseBody` heredado) usando `HttpMessageConverter`s (Jackson para JSON).

## 10. Cuándo se procesa
Registro y mapeo al arrancar; serialización en cada petición.

## 11. Efecto observable
Un método que devuelve un objeto produce una respuesta `200 OK` con `Content-Type: application/json` y el objeto serializado.

## 12. Qué ocurre internamente
1. `DispatcherServlet` recibe la petición y encuentra el `HandlerMethod`.
2. `RequestMappingHandlerAdapter` resuelve argumentos, invoca el método.
3. Como la clase tiene `@ResponseBody`, el `RequestResponseBodyMethodProcessor` gestiona el retorno:
   - Negocia el tipo de contenido (cabecera `Accept` vs `produces`).
   - Elige un `HttpMessageConverter` (`MappingJackson2HttpMessageConverter` en Boot 3; el conversor de Jackson 3 en Boot 4).
   - Escribe el cuerpo y marca la petición como gestionada (no hay vista).
4. `ResponseEntity<T>` permite controlar estado y cabeceras.

## 13. Relación con otras anotaciones
- = `@Controller` + `@ResponseBody`.
- Con `@RequestMapping`, `@GetMapping`, `@PostMapping`…, `@PathVariable`, `@RequestParam`, `@RequestBody`, `@Valid`.
- Errores con `@RestControllerAdvice` + `@ExceptionHandler`.

## 14. Dependencias necesarias
`spring-boot-starter-web` (MVC, Tomcat) o `spring-boot-starter-webflux` (reactivo, Netty). En Boot 4 existe también `spring-boot-starter-webmvc` como nombre explícito.

## 15. Alternativas
- `@Controller` + `@ResponseBody` por método.
- Endpoints funcionales (`RouterFunction`).
- Spring Data REST (expone repositorios sin controladores).

## 16. Limitaciones
- No renderiza vistas (devolver `"inicio"` produce literalmente el texto `inicio`).
- Todo lo que devuelvas se serializa: cuidado con exponer entidades JPA (bucles, *lazy loading*, datos sensibles).

## 17. Errores comunes
- Devolver entidades JPA con relaciones bidireccionales → `StackOverflowError`/JSON infinito o `LazyInitializationException`. Usa DTOs.
- Devolver `null` esperando un 404: se devuelve `200` con cuerpo vacío. Usa `ResponseEntity.notFound()` o lanza una excepción.
- Olvidar `@RequestBody` en un `POST` → el objeto llega vacío (se trata como `@ModelAttribute`).

## 18. Ejemplo básico
```java
@RestController
@RequestMapping("/api/hola")
public class HolaController {
    @GetMapping
    public Map<String, String> hola() {
        return Map.of("mensaje", "Hola mundo");
    }
}
```

## 19. Ejemplo real
**FinTech — API de cuentas**:
```java
@RestController
@RequestMapping("/api/v1/cuentas")
public class CuentaController {

    private final CuentaService cuentas;

    public CuentaController(CuentaService cuentas) { this.cuentas = cuentas; }

    @GetMapping("/{iban}")
    public CuentaDto obtener(@PathVariable String iban) {
        return cuentas.buscar(iban);                      // 404 vía @RestControllerAdvice si no existe
    }

    @GetMapping("/{iban}/saldo")
    public SaldoDto saldo(@PathVariable String iban) {
        return cuentas.saldo(iban);                       // { "disponible": 1520.35, "moneda": "ARS" }
    }

    @PostMapping
    public ResponseEntity<CuentaDto> abrir(@Valid @RequestBody AperturaCuentaRequest req,
                                          UriComponentsBuilder uri) {
        CuentaDto creada = cuentas.abrir(req);
        return ResponseEntity
            .created(uri.path("/api/v1/cuentas/{iban}").build(creada.iban()))
            .body(creada);
    }
}
```
**Otro dominio — logística**: `EnvioController` con `GET /envios/{tracking}` para seguimiento de paquetes.

## 20. Qué ocurre si la elimino
Todas las rutas del controlador responden **404**. Si la cambias por `@Controller` sin `@ResponseBody`, Spring intentará resolver el retorno como vista → error `Circular view path` o plantilla no encontrada (500).
