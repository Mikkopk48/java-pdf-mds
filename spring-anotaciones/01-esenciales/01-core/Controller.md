# @Controller

> Nivel: esencial · Categoría: core / web MVC

## 1. Nombre
`@Controller`

## 2. Paquete
`org.springframework.stereotype`

## 3. Framework / librería
Spring Framework (`spring-context`; lo usa `spring-webmvc` y `spring-webflux`).

## 4. Propósito
Marcar una clase como **controlador web**. Spring MVC busca en ella métodos anotados con `@RequestMapping` (y variantes) para atender peticiones HTTP. Por defecto, el valor devuelto se interpreta como **nombre de vista** (Thymeleaf, JSP…), no como cuerpo de la respuesta.

## 5. Target
`ElementType.TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` | `String` | Nombre del bean (`@AliasFor(annotation = Component.class)`). |

## 8. Valores por defecto
`value = ""`.

## 9. Quién la procesa
- Registro: `ClassPathBeanDefinitionScanner`.
- Mapeo de rutas: `RequestMappingHandlerMapping` (MVC) o su equivalente reactivo en WebFlux. Su método `isHandler()` considera handler a cualquier bean con `@Controller` (en versiones antiguas también `@RequestMapping` a nivel de clase).

## 10. Cuándo se procesa
- Registro: arranque.
- Detección de métodos handler: en `afterPropertiesSet()` de `RequestMappingHandlerMapping`, tras crear los beans.
- Invocación: en cada petición HTTP, a través de `DispatcherServlet`.

## 11. Efecto observable
- En los logs en nivel `TRACE` de `org.springframework.web` aparecen los mapeos.
- Un `GET /` devuelve la plantilla HTML cuyo nombre retorna el método.

## 12. Qué ocurre internamente
1. `RequestMappingHandlerMapping` recorre los beans con `@Controller`, inspecciona sus métodos y crea un `RequestMappingInfo` por cada `@RequestMapping`.
2. En cada petición, `DispatcherServlet` pide el handler adecuado, obtiene un `HandlerMethod` y lo ejecuta con `RequestMappingHandlerAdapter`.
3. Los argumentos se resuelven con `HandlerMethodArgumentResolver`s y el valor de retorno con `HandlerMethodReturnValueHandler`s. Para `String`, `ViewNameMethodReturnValueHandler` lo trata como nombre de vista y el `ViewResolver` renderiza la plantilla.

## 13. Relación con otras anotaciones
- `@RestController` = `@Controller` + `@ResponseBody`.
- Usa `@RequestMapping`, `@GetMapping`, `@ModelAttribute`, `@InitBinder`, `@ExceptionHandler`.
- Con `@ResponseBody` en un método concreto, ese método devuelve JSON en lugar de vista.

## 14. Dependencias necesarias
`spring-boot-starter-web` (MVC) o `spring-boot-starter-webflux`. Para vistas: `spring-boot-starter-thymeleaf` u otro motor.

## 15. Alternativas
- `@RestController` para APIs REST.
- Endpoints funcionales: `RouterFunction` + `HandlerFunction` (`RouterFunctions.route()`), sin anotaciones.

## 16. Limitaciones
- Si no hay motor de plantillas, devolver un `String` provoca un error de vista no encontrada (o un bucle de redirección `Circular view path`).
- Pensado para páginas servidas por el servidor; para APIs es más cómodo `@RestController`.

## 17. Errores comunes
- Usar `@Controller` en una API y olvidar `@ResponseBody` → error `Circular view path [x]: would dispatch back to the current handler URL` o 404 de plantilla.
- Poner `@Controller` en una interfaz de cliente HTTP.
- Lógica de negocio en el controlador en vez de delegar en un `@Service`.

## 18. Ejemplo básico
```java
@Controller
public class InicioController {
    @GetMapping("/")
    public String inicio(Model model) {
        model.addAttribute("mensaje", "Bienvenido");
        return "inicio"; // src/main/resources/templates/inicio.html
    }
}
```

## 19. Ejemplo real
**FinTech — portal de banca online** con Thymeleaf que muestra el extracto y descarga un PDF:
```java
@Controller
@RequestMapping("/cuentas")
public class ExtractoController {

    private final ExtractoService extractos;

    public ExtractoController(ExtractoService extractos) { this.extractos = extractos; }

    @GetMapping("/{iban}/extracto")
    public String ver(@PathVariable String iban, @RequestParam YearMonth mes, Model model) {
        model.addAttribute("extracto", extractos.generar(iban, mes));
        return "cuentas/extracto";
    }

    @GetMapping(value = "/{iban}/extracto.pdf", produces = MediaType.APPLICATION_PDF_VALUE)
    @ResponseBody
    public byte[] pdf(@PathVariable String iban, @RequestParam YearMonth mes) {
        return extractos.generarPdf(iban, mes);
    }
}
```
**Otro dominio — e‑commerce**: `CarritoController` que renderiza el carrito y redirige al checkout con `return "redirect:/checkout";`.

## 20. Qué ocurre si la elimino
La clase no es bean ni handler: todas sus rutas responden **404 Not Found** (`No static resource ...` en Boot 3.2+). Si la dejas como `@Component` con `@RequestMapping` en métodos, en Spring 6 tampoco se registra: desde Spring Framework 6.0 hace falta `@Controller` a nivel de clase.
