# @Digits

> Nivel: esencial · Categoría: validación / restricciones

## 1. Nombre
`@Digits`

## 2. Paquete
`jakarta.validation.constraints`

## 3. Framework / librería
Jakarta Bean Validation. Implementación: Hibernate Validator.

## 4. Propósito
Limitar el **número máximo de dígitos** de la parte entera y de la parte decimal de un número. Imprescindible para importes monetarios. `null` es válido.

## 5. Target
`METHOD`, `FIELD`, `ANNOTATION_TYPE`, `CONSTRUCTOR`, `PARAMETER`, `TYPE_USE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`, `@Repeatable(Digits.List.class)`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `integer` | `int` | Máximo de dígitos enteros. **Obligatorio.** |
| `fraction` | `int` | Máximo de dígitos decimales. **Obligatorio.** |
| `message`, `groups`, `payload` | | Comunes. |

## 8. Valores por defecto
`integer` y `fraction` sin default. `message = "{jakarta.validation.constraints.Digits.message}"` → *"valor numérico fuera de rango (se esperaba <{integer} dígitos>.<{fraction} dígitos>)"*.

## 9. Quién la procesa
Hibernate Validator (`DigitsValidatorForNumber`, `DigitsValidatorForCharSequence`). Hibernate ORM la usa para `precision`/`scale` en el DDL (precision = integer + fraction).

## 10. Cuándo se procesa
Al disparar la validación.

## 11. Efecto observable
`10.555` con `@Digits(integer = 12, fraction = 2)` → 400.

## 12. Qué ocurre internamente
Convierte el valor a `BigDecimal` (sin ceros a la derecha: `stripTrailingZeros()`), calcula `precision() - scale()` (enteros) y `scale()` (decimales) y los compara con los límites. Así `10.50` cuenta como 1 decimal.

## 13. Relación con otras anotaciones
`@Positive`, `@DecimalMin`, `@DecimalMax`, `@Column(precision, scale)`.

## 14. Dependencias necesarias
`spring-boot-starter-validation`.

## 15. Alternativas
Value object `Dinero` que normaliza la escala (`setScale(2, RoundingMode.UNNECESSARY)` lanza excepción si hay más decimales); librería JSR‑354 (Moneta, `MonetaryAmount`).

## 16. Limitaciones
- Monedas con distinto número de decimales (JPY 0, ARS/EUR 2, BHD/KWD 3) no se pueden expresar con una sola anotación fija: requiere validador a nivel de clase que use `Currency.getDefaultFractionDigits()`.
- Con `double` la representación binaria puede dar resultados inesperados: usa `BigDecimal`.

## 17. Errores comunes
- No validar decimales en importes → céntimos fraccionarios que descuadran la contabilidad.
- `integer` mayor que lo que admite la columna (`precision - scale`) → error SQL de desbordamiento.

## 18. Ejemplo básico
```java
public record Producto(@Digits(integer = 6, fraction = 2) BigDecimal precio) {}
```

## 19. Ejemplo real
**FinTech — importes y tasas con precisión alineada a la base de datos**:
```java
public record OperacionCambioRequest(
        @NotNull @Positive @Digits(integer = 15, fraction = 2) BigDecimal importeOrigen,  // NUMERIC(17,2)
        @NotNull @Positive @Digits(integer = 6, fraction = 6) BigDecimal tipoCambio,      // 1234.567890
        @NotNull Currency monedaOrigen,
        @NotNull Currency monedaDestino) {}
```
**Otro dominio — laboratorio clínico**: `@Digits(integer = 3, fraction = 1) BigDecimal temperatura`.

## 20. Qué ocurre si la elimino
Se aceptan importes con más decimales que la moneda o la columna: la BD redondea en silencio (diferencias de céntimos en conciliación) o lanza `numeric field overflow` (500).
