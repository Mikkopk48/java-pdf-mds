# @ExceptionHandler

> Nivel: esencial · Categoría: web / manejo de errores

## 1. Nombre
`@ExceptionHandler`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`; ejecutado por `spring-webmvc`/`spring-webflux`).

## 4. Propósito
Declarar un **método que maneja excepciones** lanzadas por los handlers: convierte una excepción en una respuesta HTTP controlada (código, cuerpo, cabeceras). Dentro de un controlador aplica solo a ese controlador; dentro de un `@ControllerAdvice`, a todos.

## 5. Target
`ElementType.METHOD`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`). Meta‑anotada con `@Reflective`.

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` (alias `exception` desde 6.2) | `Class<? extends Throwable>[]` | Excepciones que maneja. Si se omite, se deducen de los parámetros del método. |
| `produces` | `String[]` | (Spring 6.2+) Tipos de contenido que produce; permite handlers distintos para JSON y HTML. |

## 8. Valores por defecto
`value = {}` (se infiere de la firma), `produces = {}`.

## 9. Quién la procesa
`ExceptionHandlerExceptionResolver` (MVC), primero de la cadena de `HandlerExceptionResolver`s. Usa `ExceptionHandlerMethodResolver` para indexar los métodos por tipo de excepción.

## 10. Cuándo se procesa
- Indexado: al arrancar (se cachean los métodos por clase).
- Ejecución: cuando un handler (o la resolución de argumentos, validación, etc.) lanza una excepción durante la petición.

## 11. Efecto observable
En lugar de un 500 genérico, el cliente recibe la respuesta que devuelve el método (p. ej. 422 con un `ProblemDetail`).

## 12. Qué ocurre internamente
1. `DispatcherServlet.processHandlerException()` recorre los resolvers.
2. `ExceptionHandlerExceptionResolver` busca primero en la clase del controlador que falló y luego en los `@ControllerAdvice` aplicables (ordenados por `@Order`).
3. Elige el método cuya excepción declarada sea la **más cercana** en la jerarquía (`ExceptionDepthComparator`); también mira las causas (`getCause()`).
4. Invoca el método: admite parámetros como la excepción, `HttpServletRequest`, `WebRequest`, `Locale`, `HandlerMethod`…
5. El retorno se procesa como un handler normal (`ResponseEntity`, `ProblemDetail`, objeto + `@ResponseBody`, vista…).

## 13. Relación con otras anotaciones
- `@ControllerAdvice` / `@RestControllerAdvice` para manejo global.
- `@ResponseStatus` en el método handler para fijar el estado.
- Clase base `ResponseEntityExceptionHandler` para las excepciones estándar de Spring MVC.

## 14. Dependencias necesarias
`spring-boot-starter-web` / `webflux`.

## 15. Alternativas
- `@ResponseStatus` en la excepción.
- `ResponseStatusException` / `ErrorResponseException`.
- `HandlerExceptionResolver` propio.
- Personalizar `/error` (`ErrorAttributes`, `ErrorController`).

## 16. Limitaciones
- Solo captura excepciones que ocurren **dentro del flujo de Spring MVC** (handler, argumentos, retorno). Las excepciones de **filtros** (incluida Spring Security antes del `DispatcherServlet`) no llegan aquí.
- Excepciones lanzadas mientras se escribe el cuerpo ya comprometido no pueden cambiar el estado.

## 17. Errores comunes
- Esperar que capture errores de autenticación de Spring Security (se manejan con `AuthenticationEntryPoint`/`AccessDeniedHandler`).
- Manejar `Exception` genérica y ocultar errores de validación (que tienen su propio handler más específico).
- Devolver el `stackTrace` o el mensaje interno al cliente (fuga de información).
- Dos handlers para la misma excepción en el mismo advice → error `Ambiguous @ExceptionHandler method mapped`.

## 18. Ejemplo básico
```java
@RestController
public class ProductoController {

    @GetMapping("/productos/{id}")
    public Producto get(@PathVariable long id) { return servicio.buscar(id); }

    @ExceptionHandler(ProductoNoEncontrado.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public Map<String, String> noEncontrado(ProductoNoEncontrado ex) {
        return Map.of("error", ex.getMessage());
    }
}
```

## 19. Ejemplo real
**FinTech — errores de negocio con `ProblemDetail` (RFC 9457)**:
```java
@ExceptionHandler(SaldoInsuficienteException.class)
public ProblemDetail saldoInsuficiente(SaldoInsuficienteException ex) {
    ProblemDetail pd = ProblemDetail.forStatusAndDetail(
            HttpStatus.UNPROCESSABLE_ENTITY, "El saldo disponible no cubre la operación");
    pd.setType(URI.create("https://api.neobank.example/errores/saldo-insuficiente"));
    pd.setTitle("Saldo insuficiente");
    pd.setProperty("disponible", ex.disponible());
    pd.setProperty("solicitado", ex.solicitado());
    return pd;
}

@ExceptionHandler({CannotAcquireLockException.class, PessimisticLockingFailureException.class})
public ResponseEntity<ProblemDetail> cuentaBloqueada() {
    ProblemDetail pd = ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT,
            "La cuenta está procesando otra operación. Reintente.");
    return ResponseEntity.status(HttpStatus.CONFLICT)
            .header(HttpHeaders.RETRY_AFTER, "2")
            .body(pd);
}
```
**Otro dominio — reservas de hotel**: `@ExceptionHandler(HabitacionNoDisponibleException.class)` → 409 con fechas alternativas.

## 20. Qué ocurre si la elimino
La excepción sigue su camino: si tiene `@ResponseStatus` se aplica ese código; si no, llega al `/error` de Spring Boot como **500** con el cuerpo genérico (`timestamp`, `status`, `error`, `path`).
