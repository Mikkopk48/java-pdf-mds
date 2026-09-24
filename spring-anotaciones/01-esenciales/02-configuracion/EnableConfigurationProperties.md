# @EnableConfigurationProperties

> Nivel: esencial · Categoría: configuración externa

## 1. Nombre
`@EnableConfigurationProperties`

## 2. Paquete
`org.springframework.boot.context.properties`

## 3. Framework / librería
Spring Boot (`spring-boot`).

## 4. Propósito
Registrar como beans **clases concretas** anotadas con `@ConfigurationProperties` y activar la infraestructura de binding. Útil cuando no quieres escanear paquetes enteros o en autoconfiguraciones.

## 5. Target
`ElementType.TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`). Meta‑anotada con `@Import(EnableConfigurationPropertiesRegistrar.class)`.

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` | `Class<?>[]` | Clases `@ConfigurationProperties` a registrar como beans. |

## 8. Valores por defecto
`value = {}` → solo activa la infraestructura (post‑procesador de binding), sin registrar clases.

## 9. Quién la procesa
`EnableConfigurationPropertiesRegistrar` (un `ImportBeanDefinitionRegistrar`), que:
- Registra `ConfigurationPropertiesBindingPostProcessor`, `ConfigurationPropertiesBinder` y un `MethodValidationExcludeFilter` (para que la validación de métodos no intercepte los beans de propiedades).
- Registra cada clase de `value` con `ConfigurationPropertiesBeanRegistrar`.

## 10. Cuándo se procesa
Arranque, al procesar clases de configuración (`ConfigurationClassPostProcessor`).

## 11. Efecto observable
Las clases indicadas se pueden inyectar. El nombre del bean sigue el formato `<prefijo>-<nombreCompletoDeClase>` (p. ej. `app.pasarela-com.neobank.PasarelaProperties`).

## 12. Qué ocurre internamente
1. `@Import` dispara el registrar.
2. Se asegura de que la infraestructura de binding existe (solo una vez aunque la anotación aparezca en muchos sitios).
3. Por cada clase: comprueba que tenga `@ConfigurationProperties`, decide si usa constructor binding o JavaBean y registra la `BeanDefinition`.

## 13. Relación con otras anotaciones
- Requiere que la clase destino tenga `@ConfigurationProperties`.
- `@ConfigurationPropertiesScan` la incluye y la amplía con escaneo.
- Muy usada junto a `@AutoConfiguration` en starters propios.
- `@SpringBootApplication` la activa indirectamente (muchas autoconfiguraciones la usan), pero **no** registra tus clases.

## 14. Dependencias necesarias
`spring-boot`.

## 15. Alternativas
`@ConfigurationPropertiesScan` (escaneo), `@Component` en la clase de propiedades (solo JavaBean), o `@Bean` + `@ConfigurationProperties` sobre el método.

## 16. Limitaciones
Hay que listar las clases a mano; en proyectos con muchas propiedades es más cómodo el escaneo.

## 17. Errores comunes
- Pasar una clase que no tiene `@ConfigurationProperties` → `IllegalStateException: No ConfigurationProperties annotation found on '...'`.
- Pensar que registrar la clase dos veces (scan + enable) crea duplicados: Boot lo detecta, pero conviene elegir un solo mecanismo.

## 18. Ejemplo básico
```java
@Configuration
@EnableConfigurationProperties(CorreoProperties.class)
public class CorreoConfig { }
```

## 19. Ejemplo real
**FinTech — starter interno de auditoría** reutilizado por varios microservicios del banco:
```java
@AutoConfiguration
@ConditionalOnClass(AuditoriaPublisher.class)
@EnableConfigurationProperties(AuditoriaProperties.class)
public class AuditoriaAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    AuditoriaPublisher auditoriaPublisher(AuditoriaProperties props, KafkaTemplate<String, EventoAuditoria> kafka) {
        return new AuditoriaPublisher(kafka, props.topic(), props.enmascararCampos());
    }
}

@ConfigurationProperties("neobank.auditoria")
public record AuditoriaProperties(
        @DefaultValue("auditoria.eventos") String topic,
        @DefaultValue({"iban", "numeroTarjeta", "cvv"}) List<String> enmascararCampos) {}
```
**Otro dominio — telemetría**: librería interna que expone `MetricasProperties` a todas las apps de una empresa de logística.

## 20. Qué ocurre si la elimino
Si era el único mecanismo que registraba la clase, al inyectar `AuditoriaProperties` falla el arranque con `NoSuchBeanDefinitionException`.
