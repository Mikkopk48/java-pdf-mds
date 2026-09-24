# @Validated

> Nivel: esencial · Categoría: validación

## 1. Nombre
`@Validated`

## 2. Paquete
`org.springframework.validation.annotation`

## 3. Framework / librería
Spring Framework (`spring-context`), sobre Bean Validation.

## 4. Propósito
Variante de Spring de `@Valid` con dos capacidades extra:
1. **Grupos de validación**: validar solo un subconjunto de restricciones (p. ej. `Alta` vs `Modificacion`).
2. **Validación de métodos**: puesta en una **clase**, activa la validación de parámetros y valores de retorno de sus métodos (servicios, componentes, `@ConfigurationProperties`).

## 5. Target
`ElementType.TYPE`, `ElementType.METHOD`, `ElementType.PARAMETER`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` | `Class<?>[]` | Grupos de validación a aplicar. |

## 8. Valores por defecto
`value = {}` → grupo `jakarta.validation.groups.Default`.

## 9. Quién la procesa
- En parámetros de controlador: los mismos resolvers que `@Valid` (leen los grupos con `ValidationAnnotationUtils.determineValidationHints`).
- En clases: `MethodValidationPostProcessor` (Spring Boot lo registra en `ValidationAutoConfiguration`) crea un proxy AOP con `MethodValidationInterceptor`.
- En `@ConfigurationProperties`: `ConfigurationPropertiesBinder` valida al arrancar.

## 10. Cuándo se procesa
- Proxy: al crear el bean.
- Validación: en cada llamada al método (antes para parámetros, después para el retorno).
- Propiedades: al arrancar.

## 11. Efecto observable
- En servicios: llamar a `servicio.metodo(null)` con `@NotNull` lanza `ConstraintViolationException`.
- En propiedades: la aplicación **no arranca** si la configuración es inválida (`Binding validation errors on app.pasarela`).

## 12. Qué ocurre internamente
1. `MethodValidationPostProcessor` crea un advisor con un *pointcut* sobre clases con `@Validated`.
2. `MethodValidationInterceptor` obtiene los grupos y llama a `ExecutableValidator.validateParameters(...)`.
3. Si hay violaciones → `ConstraintViolationException` (o `MethodValidationException` según la configuración de adaptación de Spring 6.1).
4. Ejecuta el método y, si el retorno tiene restricciones, `validateReturnValue`.

## 13. Relación con otras anotaciones
- `@Valid`: sin grupos, y es la que se usa dentro de objetos para cascada.
- Restricciones con `groups = ...`.
- `@ConfigurationProperties` (validación de configuración).
- `@Service`: se combina para validar contratos de entrada del servicio.

## 14. Dependencias necesarias
`spring-boot-starter-validation`.

## 15. Alternativas
`@Valid` (sin grupos); validación manual con `Validator`; en Spring 6.1+ los controladores validan parámetros con restricciones sin necesidad de `@Validated` en la clase.

## 16. Limitaciones
- Validación de métodos por proxy: no funciona en auto‑invocaciones ni en métodos `private`.
- En una clase de controlador con Spring 6.1+, poner `@Validated` en la clase activa el mecanismo AOP en lugar del integrado, cambiando la excepción lanzada (`ConstraintViolationException` en vez de `HandlerMethodValidationException`).
- Los grupos complican el modelo; úsalos con moderación (a menudo es más claro tener DTOs distintos).

## 17. Errores comunes
- Esperar un 400 cuando se lanza `ConstraintViolationException` desde un servicio: por defecto se convierte en **500**; hay que manejarla en el `@RestControllerAdvice`.
- Poner restricciones en la **implementación** de una interfaz distintas de las de la interfaz → `ConstraintDeclarationException` (regla de Liskov de Bean Validation).
- Usar `@Validated(Grupo.class)` y olvidar que las restricciones sin `groups` están en `Default` y no se evaluarán.

## 18. Ejemplo básico
```java
@Service
@Validated
public class UsuarioService {
    public Usuario buscar(@NotBlank String username) { ... }
}
```

## 19. Ejemplo real
**FinTech — mismo DTO de beneficiario para alta y modificación con grupos**, más validación de contrato en el servicio:
```java
public interface Alta {}
public interface Modificacion {}

public record BeneficiarioRequest(
        @Null(groups = Alta.class) @NotNull(groups = Modificacion.class) UUID id,
        @NotBlank(groups = {Alta.class, Modificacion.class}) String alias,
        @NotBlank(groups = Alta.class) @Pattern(regexp = "^[0-9]{22}$", groups = Alta.class) String cbu) {}

@RestController
@RequestMapping("/api/v1/beneficiarios")
class BeneficiarioController {
    @PostMapping
    BeneficiarioDto alta(@Validated(Alta.class) @RequestBody BeneficiarioRequest req) { ... }

    @PutMapping("/{id}")
    BeneficiarioDto modificar(@PathVariable UUID id,
                              @Validated(Modificacion.class) @RequestBody BeneficiarioRequest req) { ... }
}

@Service
@Validated
class LimiteService {
    public void fijarLimiteDiario(@NotNull UUID clienteId,
                                  @NotNull @DecimalMin("0.00") @DecimalMax("5000000.00") BigDecimal limite) { ... }
}
```
**Otro dominio — configuración de un servicio de correo**: `@Validated @ConfigurationProperties("mail")` que impide arrancar sin host SMTP.

## 20. Qué ocurre si la elimino
- En la clase: los métodos del servicio **dejan de validar** sus parámetros (las anotaciones `@NotNull` en parámetros se ignoran).
- En un parámetro con grupos: no se valida nada (o, si pones `@Valid`, se valida solo `Default`).
- En `@ConfigurationProperties`: la app arranca con configuración inválida y falla más tarde.
