# @ConfigurationProperties

> Nivel: esencial · Categoría: configuración externa

## 1. Nombre
`@ConfigurationProperties`

## 2. Paquete
`org.springframework.boot.context.properties`

## 3. Framework / librería
Spring Boot (`spring-boot`).

## 4. Propósito
Enlazar (*bind*) un **grupo de propiedades** con un prefijo común (`app.pagos.*`) a un objeto Java tipado: una clase o un `record`. Es la forma recomendada de leer configuración en Spring Boot.

## 5. Target
`ElementType.TYPE`, `ElementType.METHOD` (sobre un método `@Bean`).

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` / `prefix` | `String` | Prefijo de las propiedades (`"app.pagos"`). En formato kebab-case canónico. |
| `ignoreInvalidFields` | `boolean` | Si `true`, ignora valores que no se pueden convertir (en vez de fallar). |
| `ignoreUnknownFields` | `boolean` | Si `false`, falla si hay propiedades bajo el prefijo que no existen en la clase. |

## 8. Valores por defecto
| Atributo | Default |
|---|---|
| `prefix` | `""` |
| `ignoreInvalidFields` | `false` |
| `ignoreUnknownFields` | `true` |

## 9. Quién la procesa
- Registro de la clase como bean: `@EnableConfigurationProperties` / `@ConfigurationPropertiesScan` (o `@Component`).
- Enlace: `ConfigurationPropertiesBindingPostProcessor` → `ConfigurationPropertiesBinder` → `Binder` (API de binding de Boot).
- Para `record`/constructor: `ConfigurationPropertiesBeanRegistrar` registra una definición que crea el objeto ya enlazado (*constructor binding*).
- Metadatos para el IDE: `spring-boot-configuration-processor` (procesador de anotaciones en compilación).

## 10. Cuándo se procesa
Arranque, en `postProcessBeforeInitialization` del bean (JavaBean) o al instanciarlo (constructor binding), antes de que otros beans lo usen.

## 11. Efecto observable
- El objeto tiene los valores del YAML/entorno ya convertidos (`Duration`, `DataSize`, `BigDecimal`, `List`, `Map`, objetos anidados…).
- Aparece en `/actuator/configprops` (con valores sensibles enmascarados).
- Autocompletado de propiedades en IntelliJ/VS Code si se usa el procesador.

## 12. Qué ocurre internamente
1. `Binder` recorre los `ConfigurationPropertySource`s (adaptadores sobre los `PropertySource` del `Environment`).
2. Aplica **relaxed binding**: `app.pagos.max-reintentos`, `app.pagos.maxReintentos`, `APP_PAGOS_MAXREINTENTOS` apuntan a la misma propiedad.
3. Si la clase tiene un único constructor con parámetros (o es un `record`), usa `ValueObjectBinder` (inmutable). Si no, `JavaBeanBinder` con setters.
4. Convierte tipos con el `ConversionService` de Boot (`ApplicationConversionService`), que entiende `10s`, `5MB`, etc.
5. Si la clase tiene `@Validated`, valida con Bean Validation y falla el arranque si hay errores.

## 13. Relación con otras anotaciones
- `@EnableConfigurationProperties`, `@ConfigurationPropertiesScan`: registran la clase.
- `@Validated` + `@NotNull`, `@Min`…: validación al arrancar.
- `@DefaultValue`, `@Name`, `@ConstructorBinding`, `@DurationUnit`, `@DataSizeUnit`, `@NestedConfigurationProperty`, `@DeprecatedConfigurationProperty` (nivel avanzado).
- Alternativa a `@Value`.

## 14. Dependencias necesarias
- `spring-boot` (en todos los starters).
- Opcional pero recomendado:
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
  <optional>true</optional>
</dependency>
```

## 15. Alternativas
`@Value` por cada propiedad; inyectar `Environment`; `Binder.get(env).bind("prefijo", Clase.class)` de forma programática.

## 16. Limitaciones
- No admite SpEL (`#{}`) en los valores.
- Poner solo `@ConfigurationProperties` sin registrar la clase no hace nada.
- Con constructor binding no hay setters: no se puede modificar en tiempo de ejecución (ventaja en la práctica).
- El prefijo debe estar en kebab-case en minúsculas (`app.miPrefijo` es inválido → error de arranque).

## 17. Errores comunes
- Olvidar `@EnableConfigurationProperties`/`@ConfigurationPropertiesScan` → `No qualifying bean of type ...Properties`.
- Clase JavaBean sin setters → los valores quedan a `null` sin error.
- Anotar además con `@Component` un `record` con constructor binding → conflicto (se crea como componente sin binding por constructor).
- Esperar que falle con claves mal escritas: con `ignoreUnknownFields = true` (default) se ignoran en silencio.
- Tipos primitivos para valores opcionales (`int` queda `0`, no `null`).

## 18. Ejemplo básico
```yaml
app:
  correo:
    remitente: no-reply@ejemplo.com
    reintentos: 3
```
```java
@ConfigurationProperties("app.correo")
public record CorreoProperties(String remitente, int reintentos) {}

@SpringBootApplication
@ConfigurationPropertiesScan
public class App { ... }
```

## 19. Ejemplo real
**FinTech — configuración de la pasarela de pagos, validada al arrancar** (si falta la API key, la app no arranca en lugar de fallar con el primer pago):
```yaml
app:
  pasarela:
    base-url: https://api.psp.example.com
    api-key: ${PSP_API_KEY}
    timeout: 3s
    comision-porcentaje: 1.25
    monedas-soportadas: [ARS, USD, EUR]
    reintentos:
      maximo: 3
      espera: 500ms
```
```java
@Validated
@ConfigurationProperties("app.pasarela")
public record PasarelaProperties(
        @NotNull URI baseUrl,
        @NotBlank String apiKey,
        @DefaultValue("5s") Duration timeout,
        @DecimalMin("0.0") @DecimalMax("10.0") BigDecimal comisionPorcentaje,
        @NotEmpty Set<Currency> monedasSoportadas,
        @Valid Reintentos reintentos) {

    public record Reintentos(@Min(0) @Max(10) int maximo, Duration espera) {}
}
```
**Otro dominio — SaaS**: `TenantProperties` con un `Map<String, TenantConfig>` para definir límites por cliente.

## 20. Qué ocurre si la elimino
- La clase ya no se enlaza: si seguía registrada como bean, sus campos quedan con valores por defecto de Java (`null`, `0`).
- Si era un `record` registrado vía scan, deja de ser bean → falla la inyección.
- Pierdes la validación de arranque y el autocompletado del IDE.
