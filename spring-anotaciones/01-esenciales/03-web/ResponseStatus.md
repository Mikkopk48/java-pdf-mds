# @ResponseStatus

> Nivel: esencial · Categoría: web / respuesta

## 1. Nombre
`@ResponseStatus`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`).

## 4. Propósito
Fijar el **código de estado HTTP** de la respuesta:
- En un **método handler**: el estado de las respuestas exitosas (p. ej. `201 Created`).
- En una **clase de excepción**: el estado que se devuelve cuando esa excepción escapa del controlador (p. ej. `404`).

## 5. Target
`ElementType.TYPE`, `ElementType.METHOD`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `code` / `value` | `HttpStatus` | Código de estado. |
| `reason` | `String` | Motivo. Si se indica, Spring usa `response.sendError(code, reason)`. |

## 8. Valores por defecto
| Atributo | Default |
|---|---|
| `code`/`value` | `HttpStatus.INTERNAL_SERVER_ERROR` (500) |
| `reason` | `""` |

## 9. Quién la procesa
- En métodos: `ServletInvocableHandlerMethod.setResponseStatus()`.
- En excepciones: `ResponseStatusExceptionResolver` (uno de los `HandlerExceptionResolver` por defecto).

## 10. Cuándo se procesa
En cada petición: tras ejecutar el método, o cuando se lanza la excepción anotada.

## 11. Efecto observable
- Método con `@ResponseStatus(CREATED)` → responde 201.
- `throw new CuentaNoEncontradaException()` anotada con `NOT_FOUND` → responde 404 (con el cuerpo de error de Spring Boot o `ProblemDetail` si está activado).

## 12. Qué ocurre internamente
- **Método**: antes de escribir el cuerpo, se llama a `response.setStatus(code)`. Si hay `reason`, se llama a `sendError`, que **descarta el cuerpo** y pasa al manejo de errores del contenedor (`/error`).
- **Excepción**: `ResponseStatusExceptionResolver` busca `@ResponseStatus` en la excepción (y en su causa) con `AnnotatedElementUtils.findMergedAnnotation` y llama a `sendError`. Spring Boot entonces redirige a `BasicErrorController` en `/error`, que genera el JSON de error.

## 13. Relación con otras anotaciones
- `@ExceptionHandler` / `@RestControllerAdvice`: alternativa más flexible (y tienen prioridad: `ExceptionHandlerExceptionResolver` va antes que `ResponseStatusExceptionResolver`).
- `@PostMapping` + `@ResponseStatus(CREATED)` es habitual.
- `ResponseStatusException` (clase) cuando el estado se decide en tiempo de ejecución.

## 14. Dependencias necesarias
`spring-boot-starter-web` / `webflux`.

## 15. Alternativas
- `ResponseEntity.status(...)` (control total por respuesta).
- `throw new ResponseStatusException(HttpStatus.NOT_FOUND, "motivo")`.
- `ErrorResponseException` / `ProblemDetail` (RFC 9457) en Spring 6.

## 16. Limitaciones
- Estado fijo: no depende de la lógica.
- Con `reason` en un método, se pierde el cuerpo de la respuesta.
- En excepciones, acopla la capa de dominio a HTTP.

## 17. Errores comunes
- Usar `reason` en un método de un `@RestController` y no entender por qué la respuesta no trae JSON.
- Anotar excepciones de dominio con códigos HTTP (el dominio no debería conocer HTTP); mejor mapear en un `@RestControllerAdvice`.
- Olvidar que `server.error.include-message` está en `never` por defecto: el `reason` no aparece en el JSON de error de Boot.

## 18. Ejemplo básico
```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ProductoNoEncontrado extends RuntimeException { }

@PostMapping("/productos")
@ResponseStatus(HttpStatus.CREATED)
public Producto crear(@RequestBody Producto p) { return servicio.guardar(p); }
```

## 19. Ejemplo real
**FinTech — excepciones de infraestructura con estado fijo** y un endpoint asíncrono que devuelve `202 Accepted` (la transferencia internacional se procesa después):
```java
@ResponseStatus(HttpStatus.SERVICE_UNAVAILABLE)
public class CoreBancarioNoDisponibleException extends RuntimeException {
    public CoreBancarioNoDisponibleException(Throwable causa) { super("Core bancario no disponible", causa); }
}

@PostMapping("/api/v1/transferencias-internacionales")
@ResponseStatus(HttpStatus.ACCEPTED)
public SolicitudAceptadaDto solicitar(@Valid @RequestBody TransferenciaSwiftRequest req) {
    return swift.encolar(req);   // { "id": "...", "estado": "PENDIENTE_COMPLIANCE" }
}
```
**Otro dominio — e‑commerce**: `@ResponseStatus(HttpStatus.CONFLICT)` en `StockInsuficienteException`.

## 20. Qué ocurre si la elimino
- En el método: vuelve a `200 OK` (los clientes que esperan `201`/`202` pueden fallar).
- En la excepción: si nadie la maneja, se convierte en **500 Internal Server Error**.
