# @PutMapping

> Nivel: esencial · Categoría: web / enrutamiento

## 1. Nombre
`@PutMapping`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`), desde 4.3.

## 4. Propósito
Atajo de `@RequestMapping(method = RequestMethod.PUT)`: mapea peticiones **PUT**, que **reemplazan por completo** un recurso (o lo crean en una URI conocida). PUT es **idempotente**: enviar la misma petición varias veces deja el mismo resultado.

## 5. Target
`ElementType.METHOD`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`). Meta‑anotada con `@RequestMapping(method = RequestMethod.PUT)`.

## 7. Atributos
Alias de `@RequestMapping`: `name`, `value`/`path`, `params`, `headers`, `consumes`, `produces`, `version` (Spring 7+).

## 8. Valores por defecto
Todos vacíos.

## 9. Quién la procesa
`RequestMappingHandlerMapping`.

## 10. Cuándo se procesa
Arranque y en cada petición.

## 11. Efecto observable
`PUT /recurso/{id}` llega al método. Respuestas típicas: `200 OK` con el recurso, `204 No Content`, o `201 Created` si se creó.

## 12. Qué ocurre internamente
Igual que `@RequestMapping` con `method = PUT`.

## 13. Relación con otras anotaciones
`@PathVariable` (identificador), `@RequestBody` + `@Valid` (representación completa), `@RequestHeader("If-Match")` para control de concurrencia optimista junto a `@Version` de JPA.

## 14. Dependencias necesarias
`spring-boot-starter-web` / `webflux`.

## 15. Alternativas
`@PatchMapping` para cambios parciales; `@PostMapping` para acciones no idempotentes.

## 16. Limitaciones
- Semánticamente exige enviar el recurso completo; campos omitidos deberían quedar a `null`/por defecto.
- Formularios HTML no envían PUT (se necesita `HiddenHttpMethodFilter` y `spring.mvc.hiddenmethod.filter.enabled=true`).

## 17. Errores comunes
- Usar PUT como PATCH (actualizar solo campos presentes) → comportamiento inconsistente entre clientes.
- Implementar PUT de forma no idempotente (p. ej. "sumar saldo").
- No controlar concurrencia: dos clientes sobrescriben cambios del otro (*lost update*).

## 18. Ejemplo básico
```java
@PutMapping("/usuarios/{id}")
public Usuario reemplazar(@PathVariable long id, @Valid @RequestBody UsuarioRequest req) {
    return servicio.reemplazar(id, req);
}
```

## 19. Ejemplo real
**FinTech — actualizar los límites de una tarjeta con control de concurrencia** (ETag = versión de la entidad):
```java
@PutMapping("/api/v1/tarjetas/{id}/limites")
public ResponseEntity<LimitesDto> actualizarLimites(
        @PathVariable UUID id,
        @RequestHeader(HttpHeaders.IF_MATCH) String ifMatch,
        @Valid @RequestBody LimitesRequest req) {

    long versionEsperada = Long.parseLong(ifMatch.replace("\"", ""));
    LimitesDto actualizados = tarjetas.reemplazarLimites(id, versionEsperada, req); // lanza 412 si la versión no coincide
    return ResponseEntity.ok()
        .eTag("\"" + actualizados.version() + "\"")
        .body(actualizados);
}
```
Enviar la misma petición dos veces deja los mismos límites: es idempotente.

**Otro dominio — CMS**: `PUT /paginas/{slug}` que reemplaza el contenido completo de una página.

## 20. Qué ocurre si la elimino
`PUT` a esa ruta → 405 Method Not Allowed (si hay otros handlers en la ruta) o 404.
