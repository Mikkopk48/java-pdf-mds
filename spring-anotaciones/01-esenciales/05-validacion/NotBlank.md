# @NotBlank

> Nivel: esencial · Categoría: validación / restricciones

## 1. Nombre
`@NotBlank`

## 2. Paquete
`jakarta.validation.constraints`

## 3. Framework / librería
Jakarta Bean Validation (desde 2.0). Implementación: Hibernate Validator.

## 4. Propósito
Exigir que un texto (`CharSequence`) **no sea null y contenga al menos un carácter que no sea espacio en blanco**. Rechaza `null`, `""` y `"   "`.

## 5. Target
`METHOD`, `FIELD`, `ANNOTATION_TYPE`, `CONSTRUCTOR`, `PARAMETER`, `TYPE_USE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`, `@Repeatable(NotBlank.List.class)`).

## 7. Atributos
`message`, `groups`, `payload` (ver `@NotNull`).

## 8. Valores por defecto
`message = "{jakarta.validation.constraints.NotBlank.message}"` → *"no debe estar vacío"*; `groups = {}`; `payload = {}`.

## 9. Quién la procesa
Hibernate Validator (`NotBlankValidator`).

## 10. Cuándo se procesa
Al disparar la validación (`@Valid`, `@Validated`, JPA pre‑insert/update, `@ConfigurationProperties`).

## 11. Efecto observable
`{"nombre": "   "}` → 400 con `nombre: no debe estar vacío`.

## 12. Qué ocurre internamente
`isValid`: `valor != null && valor.toString().trim().length() > 0` (usa la definición de espacio en blanco de Java).

## 13. Relación con otras anotaciones
- Implica `@NotNull`; no hace falta combinarlas.
- Complementar con `@Size(max = ...)` y `@Pattern`.
- Solo para texto; en colecciones usa `@NotEmpty`.

## 14. Dependencias necesarias
`spring-boot-starter-validation`.

## 15. Alternativas
`@NotEmpty` (acepta `"   "`), `@Pattern(regexp = "\\S.*")`.

## 16. Limitaciones
- Solo `CharSequence`; en otros tipos → `UnexpectedTypeException: No validator could be found for constraint NotBlank validating type Integer`.
- No recorta el valor: sigue llegando con espacios (normaliza tú, o con un `@InitBinder`/deserializador).

## 17. Errores comunes
- Ponerlo en `Integer`, `LocalDate` o `List` → `UnexpectedTypeException` (500).
- Asumir que también valida longitud máxima.

## 18. Ejemplo básico
```java
public record Contacto(@NotBlank String nombre, @NotBlank @Email String email) {}
```

## 19. Ejemplo real
**FinTech — alta de comercio adquirente**:
```java
public record AltaComercioRequest(
        @NotBlank @Size(max = 100) String razonSocial,
        @NotBlank @Pattern(regexp = "^(20|23|24|27|30|33|34)[0-9]{9}$") String cuit,   // identificador fiscal AR
        @NotBlank @Size(max = 22) String nombreFantasia,       // aparece en el resumen de la tarjeta
        @NotBlank @Pattern(regexp = "^[0-9]{4}$") String mcc) {} // Merchant Category Code
```
**Otro dominio — blog**: `@NotBlank String titulo` en una publicación.

## 20. Qué ocurre si la elimino
Se aceptan nombres vacíos o de solo espacios: datos basura en BD y problemas aguas abajo (p. ej. el nombre en el resumen de tarjeta queda en blanco o lo rechaza la red de tarjetas).
