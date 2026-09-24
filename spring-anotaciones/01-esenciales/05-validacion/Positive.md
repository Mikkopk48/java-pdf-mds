# @Positive

> Nivel: esencial · Categoría: validación / restricciones

## 1. Nombre
`@Positive`

## 2. Paquete
`jakarta.validation.constraints`

## 3. Framework / librería
Jakarta Bean Validation (desde 2.0). Implementación: Hibernate Validator.

## 4. Propósito
Exigir que un número sea **estrictamente mayor que cero**. `null` es válido.

## 5. Target
`METHOD`, `FIELD`, `ANNOTATION_TYPE`, `CONSTRUCTOR`, `PARAMETER`, `TYPE_USE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`, `@Repeatable(Positive.List.class)`).

## 7. Atributos
`message`, `groups`, `payload`.

## 8. Valores por defecto
`message = "{jakarta.validation.constraints.Positive.message}"` → *"debe ser mayor que 0"*.

## 9. Quién la procesa
Hibernate Validator (`PositiveValidatorForBigDecimal`, `...ForInteger`, `...ForDouble`, etc.).

## 10. Cuándo se procesa
Al disparar la validación.

## 11. Efecto observable
`importe = 0` o `-5` → 400 `debe ser mayor que 0`.

## 12. Qué ocurre internamente
Compara con cero según el tipo (`BigDecimal.signum() > 0`, `valor > 0`…). Soporta `BigDecimal`, `BigInteger`, `byte`, `short`, `int`, `long`, `float`, `double` y wrappers.

## 13. Relación con otras anotaciones
`@PositiveOrZero`, `@Negative`, `@NegativeOrZero`, `@Min(1)`, `@DecimalMin(value = "0", inclusive = false)` (equivalente para decimales), `@Digits`.

## 14. Dependencias necesarias
`spring-boot-starter-validation`.

## 15. Alternativas
`@DecimalMin(value = "0.00", inclusive = false)`; value objects de dominio (`Dinero`) que no admiten valores no positivos en su constructor.

## 16. Limitaciones
- No limita la precisión (`0.0000001` es positivo): combínalo con `@Digits`.
- No impone mínimos de negocio (importe mínimo de transferencia).

## 17. Errores comunes
- Aceptar `0.001` en un importe en pesos: pasa `@Positive` pero no se puede representar en céntimos.
- Olvidar `@NotNull`.

## 18. Ejemplo básico
```java
public record LineaPedido(@NotNull @Positive Integer cantidad) {}
```

## 19. Ejemplo real
**FinTech — recarga de billetera**:
```java
public record RecargaRequest(
        @NotNull @Positive @Digits(integer = 9, fraction = 2) BigDecimal importe,
        @NotNull Currency moneda,
        @NotBlank String medioPagoId) {}
```
Con `@Positive` + `@Digits(fraction = 2)`: rechaza `0`, `-100`, `10.555`.

**Otro dominio — inventario**: `@Positive int unidades` en una entrada de stock.

## 20. Qué ocurre si la elimino
Se aceptan importes negativos o cero: una "recarga" de `-500` podría convertirse en un **débito** si la lógica no se defiende, un clásico fallo de seguridad en sistemas de pagos.
