# @RequestMapping

> Nivel: esencial · Categoría: web / enrutamiento

## 1. Nombre
`@RequestMapping`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`), usado por Spring MVC y WebFlux.

## 4. Propósito
**Mapear peticiones HTTP** a clases y métodos de controlador según ruta, método HTTP, parámetros, cabeceras y tipos de contenido. Es la anotación base de la que derivan `@GetMapping`, `@PostMapping`, etc.

## 5. Target
`ElementType.TYPE`, `ElementType.METHOD`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`). Meta‑anotada con `@Mapping` y `@Reflective` (para AOT/nativo).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `name` | `String` | Nombre del mapeo (útil para construir URLs con `MvcUriComponentsBuilder`). |
| `value` / `path` | `String[]` | Rutas. Admite variables `{id}`, patrones `*`, `**`, regex `{id:\\d+}`. |
| `method` | `RequestMethod[]` | Métodos HTTP (`GET`, `POST`…). |
| `params` | `String[]` | Condiciones sobre parámetros: `"tipo=pdf"`, `"!debug"`. |
| `headers` | `String[]` | Condiciones sobre cabeceras: `"X-Api-Version=2"`. |
| `consumes` | `String[]` | `Content-Type` aceptados. |
| `produces` | `String[]` | Tipos que produce (se compara con `Accept`). |
| `version` | `String` | (Spring Framework 7+) Versión de API a la que responde este mapeo, si configuras versionado de API. |

## 8. Valores por defecto
Todos vacíos (`""` o `{}`): sin restricción. Sin `method` → acepta **todos** los métodos HTTP.

## 9. Quién la procesa
`RequestMappingHandlerMapping` (MVC) / `org.springframework.web.reactive.result.method.annotation.RequestMappingHandlerMapping` (WebFlux). Crea un `RequestMappingInfo` por método combinando el de la clase y el del método.

## 10. Cuándo se procesa
- Registro de mapeos: al arrancar, tras crear los beans controlador.
- Búsqueda de coincidencia: en cada petición.

## 11. Efecto observable
Las peticiones que cumplen todas las condiciones llegan al método; si la ruta existe pero no el método HTTP → `405 Method Not Allowed`; si no se acepta el `Content-Type` → `415`; si no se puede producir lo pedido en `Accept` → `406`.

## 12. Qué ocurre internamente
1. Al arrancar, combina ruta de clase + ruta de método (`/api/cuentas` + `/{iban}`).
2. Registra cada `RequestMappingInfo` en un `MappingRegistry`.
3. En cada petición, el `DispatcherServlet` pregunta a los `HandlerMapping`: se filtran los mapeos por ruta (con `PathPatternParser`, por defecto desde Boot 2.6), método, params, headers, consumes, produces.
4. Si hay varias coincidencias, se elige la **más específica** (`RequestMappingInfo.compareTo`); si empatan → `IllegalStateException: Ambiguous handler methods`.

## 13. Relación con otras anotaciones
- Atajos: `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`.
- Se usa en clases `@Controller`/`@RestController`.
- Variables de ruta con `@PathVariable`.

## 14. Dependencias necesarias
`spring-boot-starter-web` o `spring-boot-starter-webflux`.

## 15. Alternativas
Atajos `@XxxMapping` (a nivel de método); `RouterFunction` en estilo funcional; `@HttpExchange` para clientes (no para servidores de forma habitual).

## 16. Limitaciones
- A nivel de clase solo tiene sentido `path` (y condiciones comunes); `method` a nivel de clase se hereda pero resulta confuso.
- Con `PathPatternParser`, `**` solo se permite al final del patrón.
- Desde Spring 6 ya no se permite la barra final implícita: `/cuentas/` ≠ `/cuentas`.

## 17. Errores comunes
- Usar `@RequestMapping` sin `method` en un método: responde a GET, POST, DELETE… (riesgo de seguridad: un DELETE puede llegar a un método de consulta).
- Rutas duplicadas en dos controladores → error al arrancar `Ambiguous mapping`.
- Esperar que `/cuentas/` funcione igual que `/cuentas` tras migrar a Spring 6.

## 18. Ejemplo básico
```java
@RestController
@RequestMapping("/api/productos")
public class ProductoController {
    @RequestMapping(method = RequestMethod.GET, path = "/{id}")
    public Producto obtener(@PathVariable long id) { ... }
}
```

## 19. Ejemplo real
**FinTech — descarga de extractos en varios formatos** según `Accept`, sobre la misma ruta:
```java
@RestController
@RequestMapping(path = "/api/v1/cuentas/{iban}/movimientos")
public class MovimientoController {

    @RequestMapping(method = RequestMethod.GET, produces = MediaType.APPLICATION_JSON_VALUE)
    public List<MovimientoDto> json(@PathVariable String iban, @RequestParam LocalDate desde) { ... }

    @RequestMapping(method = RequestMethod.GET, produces = "text/csv")
    public String csv(@PathVariable String iban, @RequestParam LocalDate desde) { ... }

    @RequestMapping(method = RequestMethod.GET, produces = MediaType.APPLICATION_PDF_VALUE,
                    headers = "X-Canal=oficina")          // solo desde terminales de sucursal
    public byte[] pdf(@PathVariable String iban, @RequestParam LocalDate desde) { ... }
}
```
**Otro dominio — API pública versionada (Spring 7)**: `@GetMapping(path = "/productos", version = "2")` junto con la configuración de versionado de API en `WebMvcConfigurer`.

## 20. Qué ocurre si la elimino
- En la clase: los métodos siguen mapeados pero sin prefijo (`/{iban}` en vez de `/api/v1/cuentas/{iban}`), posiblemente colisionando con otros.
- En el método (sin otro atajo): el método deja de ser handler → 404.
