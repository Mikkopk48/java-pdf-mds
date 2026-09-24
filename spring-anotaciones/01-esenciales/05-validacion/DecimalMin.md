# @DecimalMin

> Nivel: esencial · Categoría: validación / restricciones

## 1. Nombre
`@DecimalMin`

## 2. Paquete
`jakarta.validation.constraints`

## 3. Framework / librería
Jakarta Bean Validation. Implementación: Hibernate Validator.

## 4. Propósito
Exigir que un número sea mayor (o igual) que un **mínimo decimal** expresado como `String`, lo que permite límites con decimales exactos (`"0.01"`). `null` es válido.

## 5. Target
`METHOD`, `FIELD`, `ANNOTATION_TYPE`, `CONSTRUCTOR`, `PARAMETER`, `TYPE_USE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`, `@Repeatable(DecimalMin.List.class)`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` | `String` | Mínimo, en formato `BigDecimal`. **Obligatorio.** |
| `inclusive` | `boolean` | Si el mínimo está permitido (`>=`) o no (`>`). |
| `message`, `groups`, `payload` | | Comunes. |

## 8. Valores por defecto
`inclusive = true`. `message = "{jakarta.validation.constraints.DecimalMin.message}"` → *"debe ser mayor que o igual a {value}"* (o *"mayor que"* si no es inclusivo).

## 9. Quién la procesa
Hibernate Validator (`DecimalMinValidatorForBigDecimal`, `...ForNumber`, `...ForCharSequence`…).

## 10. Cuándo se procesa
Al disparar la validación. El `String` se parsea a `BigDecimal` al inicializar el validador (valor mal formado → `IllegalArgumentException` en la primera validación).

## 11. Efecto observable
`0.005` con `@DecimalMin("0.01")` → 400.

## 12. Qué ocurre internamente
`new BigDecimal(value)` y comparación `compareTo` con el valor (convertido a `BigDecimal`), respetando `inclusive`.

## 13. Relación con otras anotaciones
`@DecimalMax`, `@Min` (solo enteros), `@Positive`, `@Digits`.

## 14. Dependencias necesarias
`spring-boot-starter-validation`.

## 15. Alternativas
`@Positive`, `@Min`, validador propio con mínimos que dependen de la moneda o del producto.

## 16. Limitaciones
- Valor fijo en tiempo de compilación.
- Admite *placeholders* de mensajes, pero no propiedades de configuración en `value`.

## 17. Errores comunes
- Escribir `@DecimalMin("0,01")` (coma) → formato inválido.
- Olvidar `inclusive = false` cuando el mínimo no debe permitirse (tasa > 0).

## 18. Ejemplo básico
```java
public record Donacion(@NotNull @DecimalMin("1.00") BigDecimal importe) {}
```

## 19. Ejemplo real
**FinTech — plazo fijo y tasas**:
```java
public record PlazoFijoRequest(
        @NotNull @DecimalMin("1000.00") @Digits(integer = 15, fraction = 2) BigDecimal capital,
        @NotNull @Min(30) Integer dias,
        @NotNull @DecimalMin(value = "0.0", inclusive = false) @DecimalMax("2.0")
        BigDecimal tnaSolicitada) {}   // 0 < TNA <= 200 %
```
**Otro dominio — marketplace**: `@DecimalMin("0.50")` como precio mínimo de un producto digital.

## 20. Qué ocurre si la elimino
Se aceptan importes por debajo del mínimo operativo (plazos fijos de 1 peso, tasas de 0 %) que la lógica o el core bancario rechazarán más tarde con un error menos claro.
