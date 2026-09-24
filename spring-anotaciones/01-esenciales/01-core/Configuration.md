# @Configuration

> Nivel: esencial · Categoría: core / configuración Java

## 1. Nombre
`@Configuration`

## 2. Paquete
`org.springframework.context.annotation`

## 3. Framework / librería
Spring Framework (`spring-context`).

## 4. Propósito
Declarar una clase como **fuente de definiciones de beans**. Sus métodos `@Bean` producen objetos que Spring gestiona. Es el reemplazo del antiguo XML (`<beans>`). Además, por defecto la clase se envuelve en un proxy CGLIB para que llamar a un método `@Bean` desde otro devuelva **siempre el mismo singleton**.

## 5. Target
`ElementType.TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`, meta‑anotada con `@Component`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` | `String` | Nombre del bean de la propia clase de configuración (alias de `@Component.value`). |
| `proxyBeanMethods` | `boolean` | `true` = modo "full": subclase CGLIB que intercepta llamadas entre métodos `@Bean`. `false` = modo "lite": sin proxy, cada llamada directa crea un objeto nuevo. |
| `enforceUniqueMethods` | `boolean` | (Spring 6.0+) Si es `true`, prohíbe métodos `@Bean` sobrecargados con el mismo nombre. |

## 8. Valores por defecto
| Atributo | Default |
|---|---|
| `value` | `""` |
| `proxyBeanMethods` | `true` |
| `enforceUniqueMethods` | `true` |

## 9. Quién la procesa
- `ConfigurationClassPostProcessor` → `ConfigurationClassParser` (lee `@Bean`, `@Import`, `@ComponentScan`, `@PropertySource`…) → `ConfigurationClassBeanDefinitionReader` (registra las definiciones).
- `ConfigurationClassEnhancer` genera la subclase CGLIB en modo full.

## 10. Cuándo se procesa
Arranque, en `invokeBeanFactoryPostProcessors`: primero se parsea y registra; el *enhancement* CGLIB ocurre en `postProcessBeanFactory` del mismo post‑procesador, antes de instanciar ningún bean.

## 11. Efecto observable
- Los beans de sus métodos `@Bean` están disponibles para inyección.
- El bean de la clase de configuración se llama `MiConfig$$SpringCGLIB$$0` si inspeccionas su `getClass()` (modo full).

## 12. Qué ocurre internamente
1. El parser marca la clase como *full* (con `proxyBeanMethods = true`) o *lite*.
2. Para cada método `@Bean` se registra un `BeanDefinition` con `factoryBeanName = miConfig` y `factoryMethodName = metodo`.
3. En modo full, CGLIB crea una subclase cuyo `BeanMethodInterceptor` intercepta cada método `@Bean`: si el bean ya existe en el contenedor lo devuelve; si no, llama al método real. Así `a()` llamando a `b()` no crea un segundo `b`.
4. Por eso la clase no puede ser `final` y los métodos `@Bean` no pueden ser `private`/`final` en modo full.

## 13. Relación con otras anotaciones
- Contiene métodos `@Bean`.
- Se combina con `@Import`, `@ComponentScan`, `@PropertySource`, `@Profile`, `@Conditional…`, `@EnableXxx`.
- `@SpringBootConfiguration` y `@AutoConfiguration` son variantes.
- `@TestConfiguration` es la versión para tests.

## 14. Dependencias necesarias
`spring-context` (en todos los starters).

## 15. Alternativas
- `@Component` con métodos `@Bean` (modo lite: sin proxy).
- XML (`@ImportResource`), casi en desuso.
- Registro funcional (`ApplicationContextInitializer`, `BeanRegistrar` en Spring 7).

## 16. Limitaciones
- Modo full: requiere CGLIB, clase no final, constructor no privado; arranque algo más lento y no ideal para imágenes nativas (por eso Boot usa `proxyBeanMethods = false` en sus autoconfiguraciones).
- Con `proxyBeanMethods = false` NO debes llamar a un método `@Bean` desde otro: inyecta el bean como parámetro.

## 17. Errores comunes
- Llamar a métodos `@Bean` entre sí en una clase `@Component` o con `proxyBeanMethods = false` → se crean instancias duplicadas (dos pools de conexiones, dos clientes HTTP).
- Declarar la clase `final` (Kotlin: clases son `final` por defecto; se necesita el plugin `kotlin-spring`).
- Poner un método `@Bean` `static` que no sea un `BeanFactoryPostProcessor`: no se intercepta.

## 18. Ejemplo básico
```java
@Configuration
public class RelojConfig {
    @Bean
    public Clock clock() {
        return Clock.systemUTC();
    }
}
```

## 19. Ejemplo real
**FinTech — clientes HTTP hacia el proveedor de pagos y el servicio de tipos de cambio**, en modo lite (recomendado):
```java
@Configuration(proxyBeanMethods = false)
public class ClientesExternosConfig {

    @Bean
    RestClient pasarelaPagosClient(RestClient.Builder builder, PasarelaProperties props) {
        return builder
            .baseUrl(props.baseUrl())
            .defaultHeader("X-Api-Key", props.apiKey())
            .requestInterceptor(new IdempotencyKeyInterceptor())
            .build();
    }

    @Bean
    TipoCambioClient tipoCambioClient(RestClient.Builder builder) {
        return new TipoCambioClient(builder.baseUrl("https://fx.interno").build());
    }

    @Bean
    Clock relojContable() {
        return Clock.system(ZoneId.of("America/Argentina/Cordoba"));
    }
}
```
**Otro dominio — IoT**: configuración de un cliente MQTT con reconexión automática.

## 20. Qué ocurre si la elimino
Los métodos `@Bean` dejan de procesarse (la clase ni siquiera es bean) → todo lo que dependía de esos beans falla al arrancar con `NoSuchBeanDefinitionException`. Si la reemplazas por `@Component`, los beans se registran pero en modo lite: cuidado con llamadas entre métodos.
