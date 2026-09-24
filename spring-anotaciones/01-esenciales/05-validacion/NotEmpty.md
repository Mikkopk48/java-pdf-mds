# @NotEmpty

> Nivel: esencial · Categoría: validación / restricciones

## 1. Nombre
`@NotEmpty`

## 2. Paquete
`jakarta.validation.constraints`

## 3. Framework / librería
Jakarta Bean Validation (desde 2.0). Implementación: Hibernate Validator.

## 4. Propósito
Exigir que el valor **no sea null ni esté vacío**. Tipos soportados: `CharSequence` (longitud > 0), `Collection` (tamaño > 0), `Map` (tamaño > 0) y arrays (longitud > 0).

## 5. Target
`METHOD`, `FIELD`, `ANNOTATION_TYPE`, `CONSTRUCTOR`, `PARAMETER`, `TYPE_USE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`, `@Repeatable(NotEmpty.List.class)`).

## 7. Atributos
`message`, `groups`, `payload`.

## 8. Valores por defecto
`message = "{jakarta.validation.constraints.NotEmpty.message}"` → *"no debe estar vacío"*; `groups = {}`; `payload = {}`.

## 9. Quién la procesa
Hibernate Validator (`NotEmptyValidatorForCharSequence`, `...ForCollection`, `...ForMap`, `...ForArray`).

## 10. Cuándo se procesa
Al disparar la validación.

## 11. Efecto observable
`{"lineas": []}` → 400 `lineas: no debe estar vacío`.

## 12. Qué ocurre internamente
Según el tipo, selecciona el validador adecuado y comprueba `!= null && size/length > 0`.

## 13. Relación con otras anotaciones
- En texto, `@NotBlank` es más estricta (rechaza espacios).
- `@Size(min = 1)` es equivalente pero **acepta null**.
- En colecciones se suele combinar con `@Size(max = ...)` y `List<@Valid Elemento>`.

## 14. Dependencias necesarias
`spring-boot-starter-validation`.

## 15. Alternativas
`@NotNull @Size(min = 1)`; `@NotBlank` para texto.

## 16. Limitaciones
No valida los elementos internos (una lista con `null` dentro pasa); usa `List<@NotNull Item>`.

## 17. Errores comunes
- Usarla en `String` esperando que rechace `"  "`.
- No limitar el tamaño máximo → peticiones con millones de elementos.

## 18. Ejemplo básico
```java
public record Encuesta(@NotEmpty List<@NotBlank String> respuestas) {}
```

## 19. Ejemplo real
**FinTech — solicitud de firma conjunta** (una cuenta de empresa requiere al menos un firmante adicional):
```java
public record SolicitudFirmaRequest(
        @NotNull UUID operacionId,
        @NotEmpty @Size(max = 5) Set<@NotBlank String> firmantesRequeridos,
        @NotEmpty Map<@NotBlank String, @NotBlank String> metadatos) {}
```
**Otro dominio — carrito de compras**: `@NotEmpty List<@Valid ItemCarrito> items` antes de confirmar el pedido.

## 20. Qué ocurre si la elimino
Se aceptan listas vacías: lotes de pago sin líneas, pedidos sin productos, solicitudes de firma sin firmantes; la lógica posterior debe defenderse o generará registros sin sentido.
