# @DeleteMapping

> Nivel: esencial · Categoría: web / enrutamiento

## 1. Nombre
`@DeleteMapping`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`), desde 4.3.

## 4. Propósito
Atajo de `@RequestMapping(method = RequestMethod.DELETE)`: mapea peticiones **DELETE** para **eliminar** (o dar de baja) un recurso. Es idempotente.

## 5. Target
`ElementType.METHOD`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`). Meta‑anotada con `@RequestMapping(method = RequestMethod.DELETE)`.

## 7. Atributos
Alias de `@RequestMapping`: `name`, `value`/`path`, `params`, `headers`, `consumes`, `produces`, `version` (Spring 7+).

## 8. Valores por defecto
Todos vacíos.

## 9. Quién la procesa
`RequestMappingHandlerMapping`.

## 10. Cuándo se procesa
Arranque y en cada petición.

## 11. Efecto observable
`DELETE /recurso/{id}` llega al método. Respuesta típica: `204 No Content`.

## 12. Qué ocurre internamente
Igual que `@RequestMapping` con `method = DELETE`.

## 13. Relación con otras anotaciones
`@PathVariable`, `@ResponseStatus(HttpStatus.NO_CONTENT)`, `@PreAuthorize` (borrar suele requerir más permisos), `@Transactional` en el servicio.

## 14. Dependencias necesarias
`spring-boot-starter-web` / `webflux`.

## 15. Alternativas
`@PostMapping("/{id}/baja")` o `@PatchMapping` cambiando estado cuando el "borrado" es lógico y tiene reglas de negocio.

## 16. Limitaciones
- Cuerpo en DELETE: permitido pero ignorado por muchos intermediarios; no lo uses para datos obligatorios.
- En sistemas financieros casi nunca se borra físicamente: la normativa exige conservar registros (borrado lógico).

## 17. Errores comunes
- Devolver `404` en el segundo DELETE y que el cliente lo interprete como fallo (discutible; muchos equipos responden `204` siempre).
- Borrado físico de datos que deben conservarse por auditoría.
- Olvidar la autorización a nivel de recurso (un usuario borra recursos de otro: IDOR).

## 18. Ejemplo básico
```java
@DeleteMapping("/notas/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void borrar(@PathVariable long id) {
    servicio.borrar(id);
}
```

## 19. Ejemplo real
**FinTech — revocar un beneficiario de transferencias** (borrado lógico + auditoría, solo el titular):
```java
@DeleteMapping("/api/v1/beneficiarios/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
@PreAuthorize("@beneficiarioSeguridad.esTitular(#id, authentication)")
public void revocar(@PathVariable UUID id, @AuthenticationPrincipal Jwt jwt) {
    beneficiarios.revocar(id, jwt.getSubject()); // marca estado=REVOCADO, fecha y usuario; no borra la fila
}
```
**Otro dominio — redes sociales**: `DELETE /posts/{id}` que elimina una publicación y sus comentarios.

## 20. Qué ocurre si la elimino
`DELETE` a esa ruta → 405 o 404; los clientes no pueden eliminar el recurso.
