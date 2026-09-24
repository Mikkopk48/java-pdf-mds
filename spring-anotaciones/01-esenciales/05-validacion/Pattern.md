# @Pattern

> Nivel: esencial · Categoría: validación / restricciones

## 1. Nombre
`@Pattern`

## 2. Paquete
`jakarta.validation.constraints`

## 3. Framework / librería
Jakarta Bean Validation. Implementación: Hibernate Validator.

## 4. Propósito
Exigir que un texto **coincida completamente** con una expresión regular Java. `null` es válido.

## 5. Target
`METHOD`, `FIELD`, `ANNOTATION_TYPE`, `CONSTRUCTOR`, `PARAMETER`, `TYPE_USE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`, `@Repeatable(Pattern.List.class)`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `regexp` | `String` | Expresión regular. **Obligatorio.** |
| `flags` | `Pattern.Flag[]` | `CASE_INSENSITIVE`, `MULTILINE`, `DOTALL`, `UNICODE_CASE`, `CANON_EQ`, `COMMENTS`, `UNIX_LINES`. |
| `message`, `groups`, `payload` | | Comunes. |

## 8. Valores por defecto
`flags = {}`, `message = "{jakarta.validation.constraints.Pattern.message}"` → *"debe coincidir con \"{regexp}\""*.

## 9. Quién la procesa
Hibernate Validator (`PatternValidator`), que compila la regex **una vez** y usa `matcher.matches()`.

## 10. Cuándo se procesa
Al disparar la validación. La regex se compila al crear el validador (una regex inválida falla la primera vez que se valida esa clase).

## 11. Efecto observable
`"AR12"` en un campo de CBU → 400 con el mensaje (que por defecto muestra la regex: mejor personalizarlo).

## 12. Qué ocurre internamente
`java.util.regex.Pattern.compile(regexp, flags).matcher(valor).matches()` — coincidencia de la **cadena completa** (no hace falta `^` y `$`, aunque no molestan).

## 13. Relación con otras anotaciones
`@NotBlank` para exigir presencia; `@Size` para longitudes (más legible que en la regex); `@Email`.

## 14. Dependencias necesarias
`spring-boot-starter-validation`.

## 15. Alternativas
Validador propio (`@Constraint`) cuando la regla necesita cálculo (dígitos de control de IBAN, CBU, tarjeta Luhn): la regex solo valida la forma, no la integridad. Hibernate Validator trae `@LuhnCheck`, `@Mod11Check`, `@CreditCardNumber`.

## 16. Limitaciones
- Solo `CharSequence`.
- No valida dígitos de control.
- Regex complejas son difíciles de mantener y pueden sufrir *backtracking* catastrófico (ReDoS) con entradas maliciosas.

## 17. Errores comunes
- Olvidar escapar la barra en Java (`"\\d"` y no `"\d"`).
- Dejar el mensaje por defecto: el usuario ve la regex.
- Regex con cuantificadores anidados (`(a+)+`) expuestas a entradas largas → CPU al 100 %.

## 18. Ejemplo básico
```java
public record Telefono(@Pattern(regexp = "^\\+?[0-9]{8,15}$", message = "teléfono no válido") String numero) {}
```

## 19. Ejemplo real
**FinTech — identificadores bancarios** (forma; los dígitos de control se validan con un validador propio):
```java
public record CuentaDestinoRequest(
        @NotBlank
        @Pattern(regexp = "^[0-9]{22}$", message = "El CBU debe tener 22 dígitos")
        String cbu,

        @Pattern(regexp = "^[a-zA-Z0-9.\\-]{6,20}$", message = "Alias CBU inválido")
        String alias,

        @Pattern(regexp = "^[A-Z]{4}[A-Z]{2}[A-Z0-9]{2}([A-Z0-9]{3})?$", message = "BIC/SWIFT inválido")
        String bic,

        @Pattern(regexp = "^[A-Z]{3}$") String moneda) {}   // ISO 4217
```
**Otro dominio — logística**: `@Pattern(regexp = "^[A-Z]{2}[0-9]{9}[A-Z]{2}$")` para números de seguimiento postal S10.

## 20. Qué ocurre si la elimino
Se aceptan formatos arbitrarios; el error aparece al llamar a la red de pagos (rechazo externo) o, peor, se envía dinero a un identificador mal escrito que casualmente existe.
