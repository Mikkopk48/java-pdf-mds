# @Max

> Nivel: esencial · Categoría: validación / restricciones

## 1. Nombre
`@Max`

## 2. Paquete
`jakarta.validation.constraints`

## 3. Framework / librería
Jakarta Bean Validation. Implementación: Hibernate Validator.

## 4. Propósito
Exigir que un número sea **menor o igual** que un valor máximo **entero** (`long`). `null` es válido.

## 5. Target
`METHOD`, `FIELD`, `ANNOTATION_TYPE`, `CONSTRUCTOR`, `PARAMETER`, `TYPE_USE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`, `@Repeatable(Max.List.class)`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` | `long` | Máximo permitido (inclusive). **Obligatorio.** |
| `message`, `groups`, `payload` | | Comunes. |

## 8. Valores por defecto
`value`: sin default. `message = "{jakarta.validation.constraints.Max.message}"` → *"debe ser menor que o igual a {value}"*.

## 9. Quién la procesa
Hibernate Validator (`MaxValidatorForNumber`, `...ForBigDecimal`, etc.).

## 10. Cuándo se procesa
Al disparar la validación.

## 11. Efecto observable
`tamanio = 5000` con `@Max(100)` → 400.

## 12. Qué ocurre internamente
Compara el número con `value` según su tipo. Mismos tipos soportados que `@Min`.

## 13. Relación con otras anotaciones
`@Min`, `@DecimalMax`, `@Negative`, `@Range` (Hibernate).

## 14. Dependencias necesarias
`spring-boot-starter-validation`.

## 15. Alternativas
`@DecimalMax("99999.99")` para máximos decimales; `@Range`.

## 16. Limitaciones
Máximo entero (`long`); límites que cambian por cliente o por configuración no pueden expresarse con un valor fijo en la anotación (usa un validador propio o lógica de negocio).

## 17. Errores comunes
- Poner límites de negocio variables (límite de transferencia por cliente) como `@Max` fijo en el DTO.
- No limitar tamaños de página → consultas que devuelven millones de filas.

## 18. Ejemplo básico
```java
public record Valoracion(@Min(1) @Max(5) int estrellas) {}
```

## 19. Ejemplo real
**FinTech — protección de endpoints de consulta** y límites técnicos (no de negocio):
```java
@RestController
@RequestMapping("/api/v1/movimientos")
class MovimientosController {

    @GetMapping
    Page<MovimientoDto> listar(@RequestParam String iban,
                               @RequestParam(defaultValue = "0") @Min(0) int pagina,
                               @RequestParam(defaultValue = "50") @Min(1) @Max(200) int tamanio) {
        return servicio.listar(iban, PageRequest.of(pagina, tamanio));
    }
}

public record PlazoFijoRequest(
        @NotNull @DecimalMin("1000.00") BigDecimal capital,
        @NotNull @Min(30) @Max(365) Integer dias) {}          // regulación: mínimo 30 días
```
> En Spring 6.1+ las restricciones en `@RequestParam` se validan sin `@Validated` en la clase y generan `HandlerMethodValidationException` (400).

**Otro dominio — educación**: `@Max(10) int nota`.

## 20. Qué ocurre si la elimino
Valores por encima del máximo se aceptan: páginas de 1 millón de filas (memoria y latencia), plazos fuera de lo regulado.
