# @Table

> Nivel: esencial · Categoría: datos / JPA

## 1. Nombre
`@Table`

## 2. Paquete
`jakarta.persistence`

## 3. Framework / librería
Jakarta Persistence (JPA). Implementación: Hibernate ORM.

## 4. Propósito
Especificar la **tabla principal** a la que se mapea una entidad: nombre, esquema, catálogo, restricciones únicas e índices (estos últimos solo se usan al generar el DDL).

## 5. Target
`ElementType.TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME`.

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `name` | `String` | Nombre de la tabla. |
| `catalog` | `String` | Catálogo (en MySQL equivale a la base de datos). |
| `schema` | `String` | Esquema (PostgreSQL, Oracle…). |
| `uniqueConstraints` | `UniqueConstraint[]` | Restricciones únicas (para DDL). |
| `indexes` | `Index[]` | Índices (para DDL). |
| `check` | `CheckConstraint[]` | (JPA 3.2+) Restricciones `CHECK` a nivel de tabla. |
| `comment` | `String` | (JPA 3.2+) Comentario de la tabla en el DDL. |
| `options` | `String` | (JPA 3.2+) Fragmento SQL extra al final del `CREATE TABLE`. |

## 8. Valores por defecto
Todo vacío → tabla con el nombre de la entidad, transformado por la **estrategia de nombres** de Spring Boot (`CamelCaseToUnderscoresNamingStrategy`): `MovimientoCuenta` → `movimiento_cuenta`. Esquema y catálogo por defecto de la conexión.

## 9. Quién la procesa
Hibernate al construir el metamodelo (`AnnotationBinder`/`EntityBinder`), aplicando la `PhysicalNamingStrategy` y la `ImplicitNamingStrategy` configuradas por Spring Boot.

## 10. Cuándo se procesa
Arranque, al crear el `EntityManagerFactory`. Índices y constraints solo si Hibernate genera esquema (`ddl-auto`) o si exportas el DDL.

## 11. Efecto observable
El SQL generado usa `schema.nombre_tabla` (visible con `spring.jpa.show-sql=true` o `logging.level.org.hibernate.SQL=debug`).

## 12. Qué ocurre internamente
1. Hibernate calcula el nombre lógico (el de `name` o el implícito).
2. Aplica la estrategia física (Boot: minúsculas y guiones bajos). Si quieres el nombre literal, usa comillas: `@Table(name = "\"Cuentas\"")` o `spring.jpa.properties.hibernate.globally_quoted_identifiers=true`.
3. Registra un objeto `Table` en el modelo relacional de Hibernate, que se usa para generar SQL y, si procede, DDL.

## 13. Relación con otras anotaciones
- Requiere `@Entity`.
- `@UniqueConstraint`, `@Index`, `@CheckConstraint` (3.2) como atributos.
- `@SecondaryTable` para repartir una entidad en varias tablas.
- `@Column(table = ...)` para columnas en tabla secundaria.

## 14. Dependencias necesarias
`spring-boot-starter-data-jpa`.

## 15. Alternativas
- Omitirla y dejar el nombre implícito.
- Configurar una `PhysicalNamingStrategy` propia (prefijos por módulo, p. ej. `trf_`).
- Gestionar índices y constraints en migraciones Flyway/Liquibase (lo recomendable en producción).

## 16. Limitaciones
- `uniqueConstraints`/`indexes` **no se validan** en tiempo de ejecución si no generas el esquema: son solo documentación para el DDL.
- Palabras reservadas (`user`, `order`, `group`) como nombre de tabla rompen el SQL si no se escapan.

## 17. Errores comunes
- Entidad `User` o `Order` sin `@Table` → `syntax error at or near "user"`/`"order"`.
- Esperar mayúsculas exactas (`@Table(name = "CuentasCliente")`) y encontrarse `cuentas_cliente` por la estrategia de nombres.
- Declarar índices solo en `@Table` y usar Flyway: el índice nunca se crea.

## 18. Ejemplo básico
```java
@Entity
@Table(name = "usuarios")   // "user" es palabra reservada en PostgreSQL
public class Usuario { ... }
```

## 19. Ejemplo real
**FinTech — movimientos en esquema `ledger`** con índices pensados para las consultas de extracto:
```java
@Entity
@Table(
    name = "movimientos",
    schema = "ledger",
    uniqueConstraints = @UniqueConstraint(name = "uk_mov_idempotency", columnNames = "idempotency_key"),
    indexes = {
        @Index(name = "ix_mov_cuenta_fecha", columnList = "cuenta_id, fecha_valor DESC"),
        @Index(name = "ix_mov_referencia", columnList = "referencia_externa")
    }
)
public class Movimiento {
    @Id private UUID id;
    @Column(name = "idempotency_key", nullable = false, updatable = false) private UUID idempotencyKey;
    @Column(name = "cuenta_id", nullable = false) private Long cuentaId;
    @Column(name = "fecha_valor", nullable = false) private LocalDate fechaValor;
    @Column(precision = 19, scale = 4, nullable = false) private BigDecimal importe;
    @Column(name = "referencia_externa", length = 64) private String referenciaExterna;
    protected Movimiento() { }
}
```
Las mismas restricciones deben existir en la migración Flyway `V12__ledger_movimientos.sql`.

**Otro dominio — multi‑esquema**: un ERP con esquemas `ventas`, `compras`, `rrhh`.

## 20. Qué ocurre si la elimino
La entidad se mapea a la tabla con el nombre implícito (`movimiento`, esquema por defecto). Si la tabla real se llama distinto → `relation "movimiento" does not exist` en la primera consulta (o en el arranque si `ddl-auto=validate`).
