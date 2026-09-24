# @Email

> Nivel: esencial · Categoría: validación / restricciones

## 1. Nombre
`@Email`

## 2. Paquete
`jakarta.validation.constraints`

## 3. Framework / librería
Jakarta Bean Validation (desde 2.0). Implementación: Hibernate Validator.

## 4. Propósito
Comprobar que un texto tiene **formato de dirección de correo** bien formada. `null` es válido (y una cadena vacía también lo es para Hibernate Validator).

## 5. Target
`METHOD`, `FIELD`, `ANNOTATION_TYPE`, `CONSTRUCTOR`, `PARAMETER`, `TYPE_USE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`, `@Repeatable(Email.List.class)`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `regexp` | `String` | Expresión regular **adicional** que también debe cumplirse. |
| `flags` | `Pattern.Flag[]` | Flags de la regex. |
| `message`, `groups`, `payload` | | Comunes. |

## 8. Valores por defecto
`regexp = ".*"` (sin restricción extra), `flags = {}`, `message = "{jakarta.validation.constraints.Email.message}"` → *"debe ser una dirección de correo electrónico con formato correcto"*.

## 9. Quién la procesa
Hibernate Validator (`EmailValidator`).

## 10. Cuándo se procesa
Al disparar la validación.

## 11. Efecto observable
`"juan@"` → 400. `"juan@localhost"` → **válido** (el estándar permite dominios sin punto).

## 12. Qué ocurre internamente
Divide por la última `@`, valida la parte local y la de dominio con expresiones regulares próximas a la RFC (longitudes, caracteres permitidos, IDN), y después aplica `regexp` si se definió.

## 13. Relación con otras anotaciones
`@NotBlank` (para exigirlo), `@Size(max = 254)`, `@Pattern` para reglas corporativas.

## 14. Dependencias necesarias
`spring-boot-starter-validation`.

## 15. Alternativas
`@Pattern` con regex propia; verificación real mediante envío de un código (la única forma de saber que el correo existe y pertenece al usuario).

## 16. Limitaciones
- No comprueba que el dominio exista ni que el buzón reciba correo.
- Acepta formatos poco habituales pero válidos (`"a b"@x.com`, `user@localhost`).

## 17. Errores comunes
- Asumir que `@Email` rechaza `null` o `""`.
- Considerar el correo verificado solo por pasar la validación.

## 18. Ejemplo básico
```java
public record Suscripcion(@NotBlank @Email String email) {}
```

## 19. Ejemplo real
**FinTech — correo para notificaciones de seguridad**: exige dominio con punto (no `localhost`) y longitud razonable; después se envía un OTP de verificación:
```java
public record ActualizarEmailRequest(
        @NotBlank
        @Email(regexp = "^[^@\\s]+@[^@\\s]+\\.[A-Za-z]{2,}$")
        @Size(max = 254)
        String nuevoEmail) {}
```
**Otro dominio — RRHH**: `@Email(regexp = ".+@miempresa\\.com$")` para aceptar solo correos corporativos.

## 20. Qué ocurre si la elimino
Se guardan correos mal formados; las notificaciones (alertas de fraude, OTP) no se entregan y el problema solo aparece cuando el cliente no recibe un aviso crítico.
