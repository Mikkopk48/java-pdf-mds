# @PostMapping

> Nivel: esencial · Categoría: web / enrutamiento

## 1. Nombre
`@PostMapping`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`), desde 4.3.

## 4. Propósito
Atajo de `@RequestMapping(method = RequestMethod.POST)`: mapea peticiones **POST**, usadas para **crear recursos** o ejecutar acciones (no idempotentes por naturaleza).

## 5. Target
`ElementType.METHOD`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`). Meta‑anotada con `@RequestMapping(method = RequestMethod.POST)`.

## 7. Atributos
Alias de `@RequestMapping`: `name`, `value`/`path`, `params`, `headers`, `consumes`, `produces`, `version` (Spring 7+).

## 8. Valores por defecto
Todos vacíos. Sin `consumes`, acepta cualquier `Content-Type` para el que exista un conversor.

## 9. Quién la procesa
`RequestMappingHandlerMapping`; el cuerpo lo lee `RequestResponseBodyMethodProcessor` (con `@RequestBody`).

## 10. Cuándo se procesa
Arranque (registro) y en cada petición.

## 11. Efecto observable
`POST /ruta` llega al método. Por defecto responde `200`; para creación lo correcto es `201 Created` con cabecera `Location`.

## 12. Qué ocurre internamente
Igual que `@RequestMapping` con `method = POST`. Si hay `@RequestBody`, el conversor Jackson deserializa el JSON; si hay `@Valid`, se valida antes de invocar el método.

## 13. Relación con otras anotaciones
`@RequestBody`, `@Valid`, `@ResponseStatus(HttpStatus.CREATED)`, `@RequestHeader("Idempotency-Key")`, `@RequestPart` (multipart).

## 14. Dependencias necesarias
`spring-boot-starter-web` / `webflux`.

## 15. Alternativas
`@RequestMapping(method = POST)`; `@PutMapping` si el cliente decide el identificador y la operación es idempotente.

## 16. Limitaciones
- No idempotente: un reintento de red puede duplicar la operación. En pagos hay que implementar idempotencia a mano (clave de idempotencia).
- Con CSRF activado en Spring Security, los POST desde navegador requieren token.

## 17. Errores comunes
- Olvidar `@RequestBody` → los campos llegan `null`.
- Responder `200` con cuerpo vacío al crear.
- 403 inesperado por CSRF en APIs stateless (hay que desactivarlo conscientemente en la `SecurityFilterChain` para APIs con token).
- No manejar reintentos de clientes → pagos duplicados.

## 18. Ejemplo básico
```java
@PostMapping("/tareas")
@ResponseStatus(HttpStatus.CREATED)
public Tarea crear(@Valid @RequestBody NuevaTarea req) {
    return servicio.crear(req);
}
```

## 19. Ejemplo real
**FinTech — iniciar una transferencia con clave de idempotencia**:
```java
@PostMapping(path = "/api/v1/transferencias", consumes = MediaType.APPLICATION_JSON_VALUE)
public ResponseEntity<TransferenciaDto> transferir(
        @RequestHeader("Idempotency-Key") UUID idempotencyKey,
        @Valid @RequestBody TransferenciaRequest req,
        @AuthenticationPrincipal Jwt jwt) {

    // Si la misma clave ya se procesó, devuelve el resultado original en vez de transferir otra vez
    ResultadoIdempotente<TransferenciaDto> r =
        transferencias.ejecutar(idempotencyKey, jwt.getSubject(), req);

    return ResponseEntity
        .status(r.esRepetida() ? HttpStatus.OK : HttpStatus.CREATED)
        .location(URI.create("/api/v1/transferencias/" + r.valor().id()))
        .body(r.valor());
}
```
**Otro dominio — e‑commerce**: `POST /pedidos` que crea el pedido y reserva stock.

## 20. Qué ocurre si la elimino
`POST` a esa ruta → 404, o 405 si existen otros métodos (GET) en la misma ruta.
