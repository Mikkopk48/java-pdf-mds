# @Size

> Nivel: esencial · Categoría: validación / restricciones

## 1. Nombre
`@Size`

## 2. Paquete
`jakarta.validation.constraints`

## 3. Framework / librería
Jakarta Bean Validation. Implementación: Hibernate Validator.

## 4. Propósito
Restringir el **tamaño** de un valor entre `min` y `max` (inclusive): longitud de `CharSequence`, tamaño de `Collection`/`Map` o longitud de array. **`null` es válido.**

## 5. Target
`METHOD`, `FIELD`, `ANNOTATION_TYPE`, `CONSTRUCTOR`, `PARAMETER`, `TYPE_USE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`, `@Repeatable(Size.List.class)`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `min` | `int` | Tamaño mínimo (inclusive). |
| `max` | `int` | Tamaño máximo (inclusive). |
| `message`, `groups`, `payload` | | Comunes. |

## 8. Valores por defecto
| Atributo | Default |
|---|---|
| `min` | `0` |
| `max` | `Integer.MAX_VALUE` |
| `message` | `"{jakarta.validation.constraints.Size.message}"` → *"el tamaño debe estar entre {min} y {max}"* |

## 9. Quién la procesa
Hibernate Validator (`SizeValidatorForCharSequence`, `...ForCollection`, `...ForMap`, `...ForArray…`). Hibernate ORM usa `max` para la longitud de columna en el DDL si Bean Validation está activo.

## 10. Cuándo se procesa
Al disparar la validación.

## 11. Efecto observable
Texto de 200 caracteres en un campo `@Size(max = 140)` → 400 `el tamaño debe estar entre 0 y 140`.

## 12. Qué ocurre internamente
Comprueba `min <= longitud <= max` (con `length()` de `CharSequence`, que cuenta unidades UTF‑16: un emoji puede contar como 2).

## 13. Relación con otras anotaciones
- `@NotNull`/`@NotBlank` para excluir null.
- `@Column(length = ...)` debe coincidir con `max`.
- `@Length` de Hibernate Validator (solo texto) es equivalente.

## 14. Dependencias necesarias
`spring-boot-starter-validation`.

## 15. Alternativas
`@Length` (Hibernate Validator), `@Pattern` con cuantificadores (`{3,20}`), `@NotEmpty`.

## 16. Limitaciones
- No aplica a números (para eso `@Min`/`@Max`/`@Digits`).
- La longitud en caracteres UTF‑16 puede no coincidir con la longitud en bytes de la columna (`VARCHAR` en bytes en algunas BDs).

## 17. Errores comunes
- Ponerla en `Integer` → `UnexpectedTypeException`.
- Esperar que rechace `null`.
- `max` distinto del `length` de la columna → el texto pasa la validación y la BD lo trunca o lanza error.

## 18. Ejemplo básico
```java
public record Usuario(@NotBlank @Size(min = 3, max = 20) String username) {}
```

## 19. Ejemplo real
**FinTech — concepto de transferencia SEPA/local**: los estándares de mensajería limitan la longitud del texto libre (en SEPA, 140 caracteres de concepto no estructurado):
```java
public record TransferenciaRequest(
        @NotBlank String ibanOrigen,
        @NotBlank String ibanDestino,
        @NotNull @Positive BigDecimal importe,
        @Size(max = 140) String concepto,                           // opcional, pero acotado
        @Size(max = 35) String referenciaExtremoAExtremo,           // EndToEndId
        @Size(max = 10) List<@Size(max = 70) String> notasInternas) {}
```
**Otro dominio — redes sociales**: `@Size(max = 280) String contenido` en un post.

## 20. Qué ocurre si la elimino
Se aceptan textos o listas de cualquier tamaño: errores de BD por longitud (`value too long for type character varying(140)` → 500), rechazo del mensaje por la red de pagos, o riesgo de peticiones enormes que consumen memoria.
