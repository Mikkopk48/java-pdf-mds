# @Bean

> Nivel: esencial · Categoría: core / configuración Java

## 1. Nombre
`@Bean`

## 2. Paquete
`org.springframework.context.annotation`

## 3. Framework / librería
Spring Framework (`spring-context`).

## 4. Propósito
Indicar que un **método produce un bean** que Spring debe registrar y gestionar. Es la forma de convertir en beans objetos de clases que no controlas (librerías externas) o que requieren construcción compleja.

## 5. Target
`ElementType.METHOD`, `ElementType.ANNOTATION_TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` / `name` | `String[]` | Nombre del bean; si hay varios, el primero es el nombre y el resto alias. |
| `autowireCandidate` | `boolean` | Si el bean puede inyectarse por tipo en otros beans. |
| `defaultCandidate` | `boolean` | (6.2+) Si `false`, solo se inyecta cuando se pide explícitamente con `@Qualifier`/nombre. |
| `bootstrap` | `Bean.Bootstrap` | (6.2+) `DEFAULT` o `BACKGROUND` (inicialización en segundo plano). |
| `initMethod` | `String` | Método a invocar tras la inyección. |
| `destroyMethod` | `String` | Método a invocar al cerrar el contexto. |
| `autowire` | `Autowire` | **Deprecado** desde 5.1: autowiring por nombre/tipo de propiedades. |

## 8. Valores por defecto
| Atributo | Default |
|---|---|
| `value`/`name` | `{}` → nombre del método |
| `autowireCandidate` | `true` |
| `defaultCandidate` | `true` |
| `bootstrap` | `Bootstrap.DEFAULT` |
| `initMethod` | `""` (ninguno) |
| `destroyMethod` | `"(inferred)"` → llama automáticamente a `close()` o `shutdown()` públicos si existen |

## 9. Quién la procesa
`ConfigurationClassParser` detecta los métodos y `ConfigurationClassBeanDefinitionReader` registra un `BeanDefinition` por método. En la creación, `ConstructorResolver.instantiateUsingFactoryMethod` invoca el método.

## 10. Cuándo se procesa
Registro: arranque (`invokeBeanFactoryPostProcessors`). Invocación del método: al crear el singleton (`finishBeanFactoryInitialization`) o la primera vez que se pide si es `@Lazy`.

## 11. Efecto observable
El objeto retornado está disponible para inyección bajo el nombre del método. Si tiene `close()`, se cierra al apagar la app (útil para pools y clientes).

## 12. Qué ocurre internamente
1. Los parámetros del método se resuelven como dependencias (igual que un constructor): por tipo, con `@Qualifier`, `@Value`, `ObjectProvider`, etc.
2. Se invoca el método. En una `@Configuration` full, la llamada pasa por el interceptor CGLIB.
3. Se aplican `BeanPostProcessor`s al objeto devuelto (puede acabar envuelto en proxies).
4. Se registra el callback de destrucción según `destroyMethod`.

## 13. Relación con otras anotaciones
- Vive en clases `@Configuration` (modo full) o `@Component` (modo lite).
- Se combina con `@Primary`, `@Qualifier`, `@Scope`, `@Lazy`, `@Profile`, `@ConditionalOnMissingBean`, `@DependsOn`, `@Order`, `@Description`, `@Fallback`.
- `@ConfigurationProperties` puede ir sobre un método `@Bean` para enlazar propiedades al objeto devuelto.

## 14. Dependencias necesarias
`spring-context`.

## 15. Alternativas
- `@Component` en la clase (si es tuya).
- `FactoryBean<T>` para lógica de creación reutilizable.
- Registro programático (`registerBean`, `BeanRegistrar`).

## 16. Limitaciones
- Si el tipo de retorno es una interfaz, Spring solo conoce ese tipo hasta que crea el bean (afecta a `@ConditionalOnMissingBean` y a búsquedas por tipo tempranas). Declara el tipo más específico posible.
- Métodos `private` o `final` no funcionan en modo full.
- `static` solo tiene sentido para `BeanFactoryPostProcessor`/`PropertySourcesPlaceholderConfigurer`.

## 17. Errores comunes
- `destroyMethod` inferido cerrando algo que no debía (p. ej. un `ExecutorService` compartido que otro componente gestiona) → pon `destroyMethod = ""`.
- Dos `@Bean` del mismo tipo sin `@Primary`/`@Qualifier` → `NoUniqueBeanDefinitionException`.
- Dos métodos `@Bean` con el mismo nombre en distintas configuraciones: uno sobrescribe al otro; Spring Boot lo prohíbe por defecto (`spring.main.allow-bean-definition-overriding=false`) → `BeanDefinitionOverrideException`.

## 18. Ejemplo básico
```java
@Configuration
public class JsonConfig {
    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper().findAndRegisterModules();
    }
}
```

## 19. Ejemplo real
**FinTech — dos pools de hilos**: uno para liquidación de pagos (crítico) y otro para notificaciones (best effort):
```java
@Configuration(proxyBeanMethods = false)
public class EjecutoresConfig {

    @Bean(name = "liquidacionExecutor", destroyMethod = "shutdown")
    @Primary
    ThreadPoolTaskExecutor liquidacionExecutor() {
        var ex = new ThreadPoolTaskExecutor();
        ex.setCorePoolSize(8);
        ex.setMaxPoolSize(8);
        ex.setQueueCapacity(1000);
        ex.setThreadNamePrefix("liq-");
        ex.setWaitForTasksToCompleteOnShutdown(true); // no perder liquidaciones en un redeploy
        ex.initialize();
        return ex;
    }

    @Bean(name = "notificacionesExecutor", defaultCandidate = false)
    ThreadPoolTaskExecutor notificacionesExecutor() {
        var ex = new ThreadPoolTaskExecutor();
        ex.setCorePoolSize(2);
        ex.setThreadNamePrefix("notif-");
        ex.initialize();
        return ex;
    }
}
```
**Otro dominio — streaming**: un `@Bean` que construye el cliente de S3 para subir vídeos.

## 20. Qué ocurre si la elimino
El método pasa a ser un método Java normal que nadie llama: el objeto no existe en el contexto. Todo bean que lo necesite falla al arrancar, o, si la dependencia era opcional (`ObjectProvider`), la funcionalidad queda desactivada en silencio.
