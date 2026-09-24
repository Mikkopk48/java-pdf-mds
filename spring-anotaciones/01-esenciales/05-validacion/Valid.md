# @Valid

> Nivel: esencial · Categoría: validación

## 1. Nombre
`@Valid`

## 2. Paquete
`jakarta.validation` (en Boot 2.x: `javax.validation`)

## 3. Framework / librería
Jakarta Bean Validation (especificación). Implementación en Spring Boot: **Hibernate Validator**.

## 4. Propósito
Pedir que se **valide un objeto** (y, en cascada, sus objetos anidados) según las restricciones declaradas en sus campos (`@NotNull`, `@Size`…). Sirve en dos contextos:
- En un **parámetro** de controlador o método: dispara la validación.
- En un **campo** de un objeto: propaga la validación en cascada a ese objeto anidado o a los elementos de una colección.

## 5. Target
`METHOD`, `FIELD`, `CONSTRUCTOR`, `PARAMETER`, `TYPE_USE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
No tiene atributos (no admite grupos; para grupos usa `@Validated`).

## 8. Valores por defecto
No aplica. Valida el grupo `Default`.

## 9. Quién la procesa
- En controladores MVC: `RequestResponseBodyMethodProcessor` / `ModelAttributeMethodProcessor` llaman a `validateIfApplicable()` → `WebDataBinder.validate()` → `LocalValidatorFactoryBean` (adaptador de Spring sobre Hibernate Validator).
- En beans con `@Validated`: `MethodValidationPostProcessor` / `MethodValidationInterceptor`.
- En cascada: el propio motor de Hibernate Validator.

## 10. Cuándo se procesa
- Controladores: al resolver el argumento, **antes** de ejecutar el método.
- Servicios `@Validated`: al invocar el método a través del proxy.

## 11. Efecto observable
Un `POST` con datos inválidos devuelve **400 Bad Request** sin llegar a ejecutar el método. El detalle de errores lo recibe tu `@RestControllerAdvice` como `MethodArgumentNotValidException`.

## 12. Qué ocurre internamente
1. El resolver deserializa el cuerpo.
2. Detecta `@Valid` (o `@Validated`, o cualquier anotación cuyo nombre empiece por "Valid").
3. Llama a `Validator.validate(objeto, grupos)`.
4. Hibernate Validator obtiene (y cachea) los metadatos de restricciones de la clase, evalúa cada `ConstraintValidator` y recorre en cascada los campos marcados con `@Valid`.
5. Los errores se acumulan en un `BindingResult`. Si el método no declara un parámetro `BindingResult` justo después → `MethodArgumentNotValidException`.

## 13. Relación con otras anotaciones
- `@Validated` (Spring): igual pero con grupos y necesaria a nivel de clase para validar parámetros de métodos de servicios.
- Restricciones: `@NotNull`, `@NotBlank`, `@Size`, `@Email`, `@Pattern`, `@Positive`, `@Digits`…
- `@RequestBody`, `@ModelAttribute`, `@RequestPart`.
- `@ConvertGroup` para cambiar de grupo en la cascada.

## 14. Dependencias necesarias
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```
Desde Boot 2.3 **no** viene incluido en `spring-boot-starter-web`.

## 15. Alternativas
- `@Validated` (Spring).
- Validación manual inyectando `jakarta.validation.Validator`.
- Validación en el dominio (constructores que lanzan excepciones), complementaria.

## 16. Limitaciones
- No admite grupos.
- No valida parámetros simples (`@RequestParam @Min(1) int x`) por sí sola en versiones anteriores a Spring 6.1; en 6.1+ la validación de métodos de controladores está integrada si hay restricciones en los parámetros.
- Validación en cascada de colecciones requiere `@Valid` en el campo o en el tipo del elemento (`List<@Valid Item>`).

## 17. Errores comunes
- Sin `spring-boot-starter-validation` → las anotaciones se ignoran **en silencio**.
- Olvidar `@Valid` en objetos anidados → solo se valida el nivel superior.
- Mezclar `javax.validation` y `jakarta.validation` → las restricciones no se reconocen.
- Declarar `BindingResult` y no comprobarlo → el método se ejecuta con datos inválidos.

## 18. Ejemplo básico
```java
public record Registro(@NotBlank String nombre, @Email String email) {}

@PostMapping("/registro")
public void registrar(@Valid @RequestBody Registro r) { ... }
```

## 19. Ejemplo real
**FinTech — pago por lote a proveedores** con validación en cascada de cada línea:
```java
public record LotePagosRequest(
        @NotBlank String cuentaOrigen,
        @NotNull @FutureOrPresent LocalDate fechaEjecucion,
        @NotEmpty @Size(max = 500) List<@Valid LineaPago> lineas) {}

public record LineaPago(
        @NotBlank @Pattern(regexp = "^[A-Z]{2}[0-9]{2}[A-Z0-9]{11,30}$") String ibanDestino,
        @NotNull @Positive @Digits(integer = 12, fraction = 2) BigDecimal importe,
        @NotBlank @Size(max = 140) String concepto) {}

@PostMapping("/api/v1/lotes-pago")
@ResponseStatus(HttpStatus.ACCEPTED)
public LoteAceptadoDto crear(@Valid @RequestBody LotePagosRequest req) {
    return lotes.encolar(req);
}
```
Un error en la línea 37 devuelve `lineas[36].importe: debe ser mayor que 0`.

**Otro dominio — formularios de admisión escolar**: `@Valid` en el tutor legal anidado dentro de la solicitud del alumno.

## 20. Qué ocurre si la elimino
No se valida nada: el método recibe datos inválidos (importes negativos, IBAN vacíos) y los errores aparecen más tarde como excepciones de base de datos (500) o, peor, como datos corruptos guardados.
