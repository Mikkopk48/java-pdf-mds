# @Repository

> Nivel: esencial · Categoría: core / estereotipos / datos

## 1. Nombre
`@Repository`

## 2. Paquete
`org.springframework.stereotype`

## 3. Framework / librería
Spring Framework (`spring-context`; la traducción de excepciones vive en `spring-tx`/`spring-orm`).

## 4. Propósito
Marcar una clase como **componente de acceso a datos** (DAO / repositorio). Además de registrarla como bean, habilita la **traducción de excepciones**: las excepciones nativas de JPA, Hibernate o JDBC se convierten en la jerarquía unificada `DataAccessException` de Spring.

## 5. Target
`ElementType.TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` | `String` | Nombre del bean (`@AliasFor(annotation = Component.class)`). |

## 8. Valores por defecto
`value = ""` → nombre generado por la clase.

## 9. Quién la procesa
- Registro: `ClassPathBeanDefinitionScanner` (como cualquier `@Component`).
- Traducción de excepciones: `PersistenceExceptionTranslationPostProcessor` (un `BeanPostProcessor`), que Spring Boot registra automáticamente al usar JPA/JDBC. Busca beans con `@Repository` y les envuelve en un proxy con `PersistenceExceptionTranslationAdvisor`.

## 10. Cuándo se procesa
Registro en el escaneo de arranque; el proxy de traducción se crea en `postProcessAfterInitialization` del bean.

## 11. Efecto observable
Si una consulta viola una clave única, en vez de `org.hibernate.exception.ConstraintViolationException` o `SQLIntegrityConstraintViolationException` recibes `DataIntegrityViolationException` (de Spring), independiente de la tecnología.

## 12. Qué ocurre internamente
1. El bean se registra como cualquier componente.
2. `PersistenceExceptionTranslationPostProcessor` detecta la anotación y crea un proxy.
3. Cuando un método lanza `RuntimeException`, el interceptor pregunta a todos los `PersistenceExceptionTranslator` del contexto (p. ej. `HibernateJpaDialect`, `SQLErrorCodeSQLExceptionTranslator` vía `JpaTransactionManager`/`LocalContainerEntityManagerFactoryBean`) si saben traducirla.
4. Si alguno la traduce, se relanza la `DataAccessException` correspondiente.

## 13. Relación con otras anotaciones
- Es un `@Component`.
- Las interfaces de **Spring Data** (`extends JpaRepository`) **no necesitan** `@Repository`: Spring Data las detecta por `@EnableJpaRepositories` (autoconfigurado) y ya aplican traducción de excepciones.
- Suele usarse con `@Transactional`, `@PersistenceContext`, `JdbcTemplate`/`JdbcClient`.

## 14. Dependencias necesarias
- `spring-context` para el estereotipo.
- `spring-boot-starter-data-jpa` o `spring-boot-starter-jdbc` para que exista la traducción de excepciones.

## 15. Alternativas
- Interfaces Spring Data (`JpaRepository`, `CrudRepository`) sin anotación.
- `@Component` si no te interesa la traducción de excepciones.
- Registrar el DAO con `@Bean`.

## 16. Limitaciones
- La traducción solo funciona si hay un `PersistenceExceptionTranslator` en el contexto.
- Al ser un proxy, llamadas internas no pasan por la traducción.
- Proxies basados en clase requieren clases no `final` (CGLIB).

## 17. Errores comunes
- Anotar con `@Repository` una interfaz de Spring Data "por si acaso": redundante (no rompe nada, pero confunde).
- Capturar `SQLException` o `PersistenceException` en el servicio cuando lo que llega es `DataAccessException`.
- Poner lógica de negocio en el repositorio.

## 18. Ejemplo básico
```java
@Repository
public class ClienteDao {
    private final JdbcClient jdbc;
    public ClienteDao(JdbcClient jdbc) { this.jdbc = jdbc; }

    public Optional<String> nombrePorId(long id) {
        return jdbc.sql("SELECT nombre FROM cliente WHERE id = ?")
                   .param(id).query(String.class).optional();
    }
}
```

## 19. Ejemplo real
**FinTech — ledger contable con SQL nativo** por rendimiento (inserción masiva de asientos):
```java
@Repository
public class AsientoContableDao {

    private final JdbcTemplate jdbc;

    public AsientoContableDao(JdbcTemplate jdbc) { this.jdbc = jdbc; }

    public void insertarLote(List<Asiento> asientos) {
        jdbc.batchUpdate("""
            INSERT INTO ledger_entry (id, cuenta, debe, haber, fecha_valor)
            VALUES (?, ?, ?, ?, ?)
            """,
            asientos, 500,
            (ps, a) -> {
                ps.setObject(1, a.id());
                ps.setString(2, a.cuenta());
                ps.setBigDecimal(3, a.debe());
                ps.setBigDecimal(4, a.haber());
                ps.setObject(5, a.fechaValor());
            });
    }
}

// En el servicio:
try {
    dao.insertarLote(lote);
} catch (DuplicateKeyException e) {           // excepción de Spring, no de JDBC
    log.warn("Lote ya contabilizado (idempotencia): {}", loteId);
}
```
**Otro dominio — educación**: `CalificacionDao` que lee notas de un sistema legado vía JDBC.

## 20. Qué ocurre si la elimino
- La clase deja de ser bean → falla la inyección al arrancar.
- Si la cambias por `@Component`, arranca, pero pierdes la traducción: tus `catch (DuplicateKeyException e)` dejan de capturar y aparecen excepciones crudas del driver.
