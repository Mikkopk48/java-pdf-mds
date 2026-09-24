# @PatchMapping

> Nivel: esencial · Categoría: web / enrutamiento

## 1. Nombre
`@PatchMapping`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`), desde 4.3.

## 4. Propósito
Atajo de `@RequestMapping(method = RequestMethod.PATCH)`: mapea peticiones **PATCH** para **modificaciones parciales** de un recurso (solo los campos enviados).

## 5. Target
`ElementType.METHOD`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`). Meta‑anotada con `@RequestMapping(method = RequestMethod.PATCH)`.

## 7. Atributos
Alias de `@RequestMapping`: `name`, `value`/`path`, `params`, `headers`, `consumes`, `produces`, `version` (Spring 7+).

## 8. Valores por defecto
Todos vacíos.

## 9. Quién la procesa
`RequestMappingHandlerMapping`.

## 10. Cuándo se procesa
Arranque y en cada petición.

## 11. Efecto observable
`PATCH /recurso/{id}` llega al método; solo cambian los campos presentes en el cuerpo.

## 12. Qué ocurre internamente
Igual que `@RequestMapping` con `method = PATCH`. Spring no implementa la semántica de "parcial": eres tú quien decide cómo aplicar el cambio (DTO con campos opcionales, JSON Merge Patch `application/merge-patch+json`, o JSON Patch `application/json-patch+json`).

## 13. Relación con otras anotaciones
`@PathVariable`, `@RequestBody`, `@Valid` (con grupos de validación, ya que los campos son opcionales), `@Version` en la entidad.

## 14. Dependencias necesarias
`spring-boot-starter-web` / `webflux`. Para JSON Patch real, una librería como `json-patch` (com.github.java-json-tools) u otra equivalente.

## 15. Alternativas
`@PutMapping` (reemplazo completo); endpoints de acción `@PostMapping("/{id}/bloquear")` cuando el cambio tiene significado de negocio.

## 16. Limitaciones
- Distinguir "campo no enviado" de "campo enviado como `null`" no es trivial con DTOs normales (usa `Optional`, `JsonNullable` o `Map`).
- No es idempotente por definición (depende de la implementación).
- El cliente HTTP por defecto de Java (`HttpURLConnection`) no soporta PATCH; `RestClient` con JDK HttpClient sí.

## 17. Errores comunes
- Aplicar el DTO parcial y poner a `null` los campos no enviados.
- Validar con `@NotNull` en un DTO de PATCH (rechaza peticiones parciales válidas).
- Permitir cambiar por PATCH campos que no deberían ser modificables (saldo, estado KYC).

## 18. Ejemplo básico
```java
@PatchMapping("/perfil")
public Perfil actualizar(@RequestBody Map<String, Object> cambios) {
    return servicio.aplicar(cambios);
}
```

## 19. Ejemplo real
**FinTech — actualizar datos de contacto del cliente** (solo campos permitidos; el nombre legal no se cambia por aquí porque requiere KYC):
```java
public record ContactoPatch(
        Optional<@Email String> email,
        Optional<@Pattern(regexp = "^\\+?[0-9]{8,15}$") String> telefono,
        Optional<Boolean> aceptaNotificacionesPush) {}

@PatchMapping(path = "/api/v1/clientes/me/contacto", consumes = "application/merge-patch+json")
public ContactoDto actualizarContacto(@AuthenticationPrincipal Jwt jwt,
                                      @Valid @RequestBody ContactoPatch patch) {
    return clientes.actualizarContacto(jwt.getSubject(), patch);
}
```
Si `email` cambia, el servicio dispara la verificación por código antes de hacerlo efectivo.

**Otro dominio — gestión de tareas**: `PATCH /tareas/{id}` con `{ "estado": "HECHA" }`.

## 20. Qué ocurre si la elimino
`PATCH` a esa ruta → 405 o 404; los clientes tendrían que enviar el recurso completo por PUT.
