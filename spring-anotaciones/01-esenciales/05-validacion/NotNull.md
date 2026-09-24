# @NotNull

> Nivel: esencial · Categoría: validación / restricciones

## 1. Nombre
`@NotNull`

## 2. Paquete
`jakarta.validation.constraints`

> No confundir con `org.springframework.lang.NonNull`, `org.jspecify.annotations.NonNull` o `lombok.NonNull`, que no son restricciones de Bean Validation.

## 3. Framework / librería
Jakarta Bean Validation. Implementación: Hibernate Validator.

## 4. Propósito
Exigir que el valor **no sea `null`**. Acepta cualquier tipo. No comprueba contenido (una cadena vacía `""` es válida).

## 5. Target
`METHOD`, `FIELD`, `ANNOTATION_TYPE`, `CONSTRUCTOR`, `PARAMETER`, `TYPE_USE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`, `@Repeatable(NotNull.List.class)`, `@Constraint(validatedBy = {})`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `message` | `String` | Mensaje o clave de mensaje. |
| `groups` | `Class<?>[]` | Grupos a los que pertenece la restricción. |
| `payload` | `Class<? extends Payload>[]` | Metadatos para el cliente de la validación (p. ej. severidad). |

## 8. Valores por defecto
| Atributo | Default |
|---|---|
| `message` | `"{jakarta.validation.constraints.NotNull.message}"` → en español: *"no debe ser nulo"* |
| `groups` | `{}` (grupo `Default`) |
| `payload` | `{}` |

## 9. Quién la procesa
Hibernate Validator (`NotNullValidator`), invocado por `@Valid`/`@Validated` o por `Validator.validate()`. Hibernate ORM también la tiene en cuenta: con `hibernate.validator.apply_to_ddl=true` (por defecto) genera `NOT NULL` en el DDL, y valida antes de `persist/update` (evento `pre-insert`) si Bean Validation está presente.

## 10. Cuándo se procesa
Cuando se dispara la validación (controlador, servicio `@Validated`, binding de `@ConfigurationProperties`, eventos de JPA antes de insertar/actualizar).

## 11. Efecto observable
Error `campo: no debe ser nulo` y respuesta 400 (en controlador).

## 12. Qué ocurre internamente
El motor lee la anotación de los metadatos de la clase, instancia `NotNullValidator` y llama a `isValid(valor)` → `valor != null`. El mensaje se interpola con `MessageInterpolator` usando `ValidationMessages.properties` (y la localización de la petición en Spring MVC).

## 13. Relación con otras anotaciones
- `@NotBlank` (texto), `@NotEmpty` (colecciones y texto) son más estrictas.
- Casi todas las demás restricciones consideran `null` **válido**: combínalas con `@NotNull`.
- `@Column(nullable = false)` es su equivalente de esquema.

## 14. Dependencias necesarias
`spring-boot-starter-validation`.

## 15. Alternativas
`@NotBlank`/`@NotEmpty`; `Objects.requireNonNull` en constructores de dominio; tipos primitivos (nunca nulos, pero toman `0`/`false` por defecto).

## 16. Limitaciones
No comprueba vacío ni contenido.

## 17. Errores comunes
- Usar `@NotNull` en `String` esperando que rechace `""`.
- Importar `lombok.NonNull` o `org.springframework.lang.NonNull` por autocompletado del IDE.
- Poner `@NotNull` en un `int` primitivo (nunca es null; el cliente que omite el campo envía 0 sin error).

## 18. Ejemplo básico
```java
public record Pedido(@NotNull Long clienteId, @NotNull LocalDate fecha) {}
```

## 19. Ejemplo real
**FinTech — orden de compra de acciones**:
```java
public record OrdenBolsaRequest(
        @NotNull TipoOrden tipo,                    // COMPRA / VENTA
        @NotNull @Pattern(regexp = "^[A-Z]{1,6}$") String ticker,
        @NotNull @Positive Integer cantidad,        // Integer (no int) para detectar omisión
        @NotNull ModalidadPrecio modalidad,         // MERCADO / LIMITE
        @Positive BigDecimal precioLimite) {        // opcional: solo con LIMITE (validador a nivel de clase)
}
```
**Otro dominio — salud**: `@NotNull LocalDate fechaNacimiento` en la ficha de un paciente.

## 20. Qué ocurre si la elimino
Se aceptan valores `null`: pueden causar `NullPointerException` en la lógica o `ConstraintViolation`/`DataIntegrityViolationException` al guardar (500 en lugar de 400).
