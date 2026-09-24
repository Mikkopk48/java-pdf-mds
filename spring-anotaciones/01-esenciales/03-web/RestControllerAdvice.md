# @RestControllerAdvice

> Nivel: esencial · Categoría: web / manejo de errores REST

## 1. Nombre
`@RestControllerAdvice`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`), desde 4.3.

## 4. Propósito
Versión REST de `@ControllerAdvice`: todos sus métodos `@ExceptionHandler` escriben su retorno como **cuerpo de respuesta** (JSON). Es la forma estándar de centralizar el **manejo global de errores** en una API.

## 5. Target
`ElementType.TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`). Meta‑anotada con `@ControllerAdvice` y `@ResponseBody`.

## 7. Atributos
Alias de `@ControllerAdvice`: `name`, `value`/`basePackages`, `basePackageClasses`, `assignableTypes`, `annotations`.

## 8. Valores por defecto
Todos vacíos → aplica a todos los controladores.

## 9. Quién la procesa
Igual que `@ControllerAdvice` (`ExceptionHandlerExceptionResolver`, `RequestMappingHandlerAdapter`); el `@ResponseBody` hace que `RequestResponseBodyMethodProcessor` serialice el retorno.

## 10. Cuándo se procesa
Descubrimiento al arrancar; ejecución cuando un controlador lanza una excepción.

## 11. Efecto observable
Todas las APIs devuelven errores con un formato uniforme (p. ej. `application/problem+json`).

## 12. Qué ocurre internamente
Ver `@ControllerAdvice` y `@ExceptionHandler`. Si la clase extiende `ResponseEntityExceptionHandler`, hereda handlers para ~15 excepciones estándar de Spring MVC (`MethodArgumentNotValidException`, `HttpMessageNotReadableException`, `NoResourceFoundException`…) que devuelven `ProblemDetail`, y puedes sobrescribir `handleMethodArgumentNotValid`, etc.

## 13. Relación con otras anotaciones
- `@ExceptionHandler`, `@ResponseStatus`, `@Order`.
- Complementa `@Valid` (convierte errores de validación en 400 legibles).
- Con `spring.mvc.problemdetails.enabled=true`, Boot registra su propio advice con `ProblemDetail` para las excepciones estándar.

## 14. Dependencias necesarias
`spring-boot-starter-web` / `webflux`.

## 15. Alternativas
`@ControllerAdvice` + `ResponseEntity`; `@ResponseStatus` en excepciones; `ErrorAttributes` personalizado para el `/error` de Boot.

## 16. Limitaciones
- No captura errores de filtros (Spring Security, filtros propios).
- Si hay varios advices, el orden importa.
- `ResponseEntityExceptionHandler` y el advice de Boot (`problemdetails.enabled`) no deben convivir: manejarían las mismas excepciones.

## 17. Errores comunes
- Handler de `Exception` que devuelve 500 **también** para `MethodArgumentNotValidException` si no se extiende `ResponseEntityExceptionHandler` ni se maneja aparte → los errores de validación salen como 500.
- Loguear como `ERROR` excepciones de negocio esperadas (ruido en alertas).
- Exponer `ex.getMessage()` de excepciones técnicas (SQL, rutas internas).
- No incluir un identificador de correlación para soporte.

## 18. Ejemplo básico
```java
@RestControllerAdvice
public class ErroresApi {
    @ExceptionHandler(IllegalArgumentException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String, String> badRequest(IllegalArgumentException ex) {
        return Map.of("error", ex.getMessage());
    }
}
```

## 19. Ejemplo real
**FinTech — manejo global de errores de una API bancaria**: formato RFC 9457, códigos de negocio estables, detalle de validación por campo y *trace id* para soporte:
```java
@RestControllerAdvice
public class ApiErrorHandler extends ResponseEntityExceptionHandler {

    private static final Logger log = LoggerFactory.getLogger(ApiErrorHandler.class);

    @ExceptionHandler(ReglaNegocioException.class)       // saldo insuficiente, límite excedido, cuenta bloqueada...
    ProblemDetail negocio(ReglaNegocioException ex) {
        ProblemDetail pd = ProblemDetail.forStatusAndDetail(HttpStatus.UNPROCESSABLE_ENTITY, ex.getMessage());
        pd.setProperty("codigo", ex.codigo());           // p. ej. "TRF-003"
        return conTrace(pd);
    }

    @ExceptionHandler(RecursoNoEncontradoException.class)
    ProblemDetail noEncontrado(RecursoNoEncontradoException ex) {
        return conTrace(ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage()));
    }

    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(
            MethodArgumentNotValidException ex, HttpHeaders headers, HttpStatusCode status, WebRequest req) {
        ProblemDetail pd = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, "Datos inválidos");
        pd.setProperty("errores", ex.getBindingResult().getFieldErrors().stream()
            .map(e -> Map.of("campo", e.getField(), "mensaje", String.valueOf(e.getDefaultMessage())))
            .toList());
        return ResponseEntity.badRequest().body(conTrace(pd));
    }

    @ExceptionHandler(Exception.class)
    ProblemDetail inesperado(Exception ex) {
        log.error("Error no controlado", ex);             // detalle solo en logs
        return conTrace(ProblemDetail.forStatusAndDetail(
            HttpStatus.INTERNAL_SERVER_ERROR, "Error interno. Contacte con soporte indicando el traceId."));
    }

    private ProblemDetail conTrace(ProblemDetail pd) {
        pd.setProperty("traceId", MDC.get("traceId"));
        return pd;
    }
}
```
**Otro dominio — marketplace**: advice que traduce errores de proveedores externos (timeouts) a `503` con `Retry-After`.

## 20. Qué ocurre si la elimino
Las excepciones de negocio se convierten en **500** genéricos o en el JSON por defecto de Boot; los errores de validación vuelven al formato estándar (o `ProblemDetail` si está activado en Boot); se pierde la uniformidad y la trazabilidad de errores.
