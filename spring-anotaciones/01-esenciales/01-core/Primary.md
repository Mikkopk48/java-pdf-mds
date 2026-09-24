# @Primary

> Nivel: esencial · Categoría: core / inyección de dependencias

## 1. Nombre
`@Primary`

## 2. Paquete
`org.springframework.context.annotation`

## 3. Framework / librería
Spring Framework (`spring-context`).

## 4. Propósito
Marcar un bean como **candidato preferido** cuando hay varios del mismo tipo y el punto de inyección no especifica cuál quiere.

## 5. Target
`ElementType.TYPE`, `ElementType.METHOD`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
No tiene atributos.

## 8. Valores por defecto
No aplica.

## 9. Quién la procesa
- Registro: `AnnotationConfigUtils.processCommonDefinitionAnnotations()` marca `BeanDefinition.setPrimary(true)` (para `@Component` y para métodos `@Bean`).
- Uso: `DefaultListableBeanFactory.determinePrimaryCandidate()` al resolver dependencias.

## 10. Cuándo se procesa
La marca se guarda al registrar la definición (arranque). Se consulta cada vez que se resuelve una dependencia ambigua.

## 11. Efecto observable
Con dos beans `DataSource`, un `@Autowired DataSource ds` recibe el marcado con `@Primary` en lugar de fallar.

## 12. Qué ocurre internamente
En `doResolveDependency`, si `findAutowireCandidates` devuelve más de uno, se llama a `determineAutowireCandidate()`, que en orden:
1. Busca el candidato con `primary = true` (si hay más de uno primario → `NoUniqueBeanDefinitionException: more than one 'primary' bean found`).
2. Si no hay primario, busca el de mayor `@Priority`.
3. Si no, busca coincidencia de nombre con el parámetro/campo.
4. (6.2+) Descarta los marcados con `@Fallback` si queda uno solo no‑fallback.

## 13. Relación con otras anotaciones
- `@Qualifier` tiene prioridad: si se pide explícitamente otro bean, `@Primary` se ignora.
- `@Fallback` (Spring 6.2) es su inverso: "úsame solo si no hay otro".
- En autoconfiguraciones se combina con `@ConditionalOnMissingBean`.

## 14. Dependencias necesarias
`spring-context`.

## 15. Alternativas
`@Qualifier` en cada punto de inyección; `@Fallback`; `@Priority` (jakarta.annotation); inyección por nombre de parámetro.

## 16. Limitaciones
- Solo un primario por tipo.
- No afecta a inyecciones de colecciones (`List<T>` recibe todos).
- Puede ocultar ambigüedades reales y dar lugar a inyecciones incorrectas silenciosas.

## 17. Errores comunes
- Dos `@Primary` del mismo tipo (p. ej. uno tuyo y otro de una librería) → error de ambigüedad.
- Declarar un segundo `DataSource` sin `@Primary` en el original: la autoconfiguración de JPA no sabe cuál usar.
- Usar `@Primary` en tests para sustituir beans cuando lo adecuado es `@MockitoBean` o `@TestBean`.

## 18. Ejemplo básico
```java
@Configuration
class RelojConfig {
    @Bean @Primary Clock utc()   { return Clock.systemUTC(); }
    @Bean Clock local()          { return Clock.systemDefaultZone(); }
}
```

## 19. Ejemplo real
**FinTech — base de datos transaccional + réplica de solo lectura para reporting**:
```java
@Configuration(proxyBeanMethods = false)
class DataSourcesConfig {

    @Bean
    @Primary
    @ConfigurationProperties("app.datasource.core")
    DataSource coreDataSource() {       // JPA, ledger, transferencias
        return DataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties("app.datasource.reporting")
    DataSource reportingDataSource() {  // réplica, consultas pesadas de BI
        return DataSourceBuilder.create().build();
    }

    @Bean
    JdbcClient reportingJdbc(@Qualifier("reportingDataSource") DataSource ds) {
        return JdbcClient.create(ds);
    }
}
```
JPA y `@Transactional` usan automáticamente `coreDataSource`; el reporting pide explícitamente la réplica.

**Otro dominio — mensajería**: `@Primary` en el `EmailSender` real y un `EmailSender` de sandbox para entornos de QA.

## 20. Qué ocurre si la elimino
Cualquier punto de inyección sin calificador de ese tipo falla con `NoUniqueBeanDefinitionException`. En el caso de `DataSource`, la autoconfiguración de JPA/Flyway deja de arrancar.
