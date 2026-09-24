# @Min

> Nivel: esencial · Categoría: validación / restricciones

## 1. Nombre
`@Min`

## 2. Paquete
`jakarta.validation.constraints`

## 3. Framework / librería
Jakarta Bean Validation. Implementación: Hibernate Validator.

## 4. Propósito
Exigir que un número sea **mayor o igual** que un valor mínimo **entero** (`long`). `null` es válido.

## 5. Target
`METHOD`, `FIELD`, `ANNOTATION_TYPE`, `CONSTRUCTOR`, `PARAMETER`, `TYPE_USE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`, `@Repeatable(Min.List.class)`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` | `long` | Mínimo permitido (inclusive). **Obligatorio.** |
| `message`, `groups`, `payload` | | Comunes. |

## 8. Valores por defecto
`value`: sin default. `message = "{jakarta.validation.constraints.Min.message}"` → *"debe ser mayor que o igual a {value}"*.

## 9. Quién la procesa
Hibernate Validator (`MinValidatorForNumber`, `...ForBigDecimal`, `...ForLong`, etc.). Hibernate ORM puede añadir un `CHECK` en el DDL.

## 10. Cuándo se procesa
Al disparar la validación.

## 11. Efecto observable
`cuotas = 0` con `@Min(1)` → 400 `debe ser mayor que o igual a 1`.

## 12. Qué ocurre internamente
Compara el número con `value`. Tipos soportados por la especificación: `BigDecimal`, `BigInteger`, `byte`, `short`, `int`, `long` y sus wrappers. Hibernate Validator añade soporte para `CharSequence` numéricas y otros `Number` (en `double`/`float` puede haber errores de redondeo).

## 13. Relación con otras anotaciones
`@Max`, `@Positive`, `@PositiveOrZero`, `@DecimalMin` (para mínimos decimales), `@Range` (Hibernate).

## 14. Dependencias necesarias
`spring-boot-starter-validation`.

## 15. Alternativas
`@DecimalMin("0.01")` cuando el mínimo tiene decimales; `@Positive` para "> 0"; `@Range(min, max)`.

## 16. Limitaciones
El mínimo es un `long`: `@Min(0.01)` no compila. Para importes con céntimos usa `@DecimalMin`.

## 17. Errores comunes
- Usarla en `BigDecimal` importes esperando mínimo 0,01 → `@Min(1)` rechaza 0,50.
- Olvidar `@NotNull` → `null` pasa.

## 18. Ejemplo básico
```java
public record Paginacion(@Min(0) int pagina, @Min(1) @Max(100) int tamanio) {}
```

## 19. Ejemplo real
**FinTech — simulación de préstamo**:
```java
public record SimulacionPrestamoRequest(
        @NotNull @DecimalMin("10000.00") BigDecimal capital,
        @NotNull @Min(3) @Max(72) Integer plazoMeses,          // de 3 a 72 cuotas
        @NotNull @Min(1) @Max(28) Integer diaVencimiento) {}   // evita meses sin día 29-31
```
**Otro dominio — hotel**: `@Min(1) int huespedes` en una reserva.

## 20. Qué ocurre si la elimino
Se aceptan valores por debajo del mínimo (0 cuotas → división por cero al calcular la cuota; página negativa → error en la consulta).
