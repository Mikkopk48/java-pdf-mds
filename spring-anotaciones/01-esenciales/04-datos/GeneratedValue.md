# @GeneratedValue

> Nivel: esencial · Categoría: datos / JPA

## 1. Nombre
`@GeneratedValue`

## 2. Paquete
`jakarta.persistence`

## 3. Framework / librería
Jakarta Persistence (JPA). Implementación: Hibernate ORM.

## 4. Propósito
Indicar que el valor de la **clave primaria se genera automáticamente** y con qué estrategia (secuencia, columna identidad, tabla, UUID).

## 5. Target
`ElementType.METHOD`, `ElementType.FIELD`.

## 6. Retention
`RetentionPolicy.RUNTIME`.

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `strategy` | `GenerationType` | `AUTO`, `IDENTITY`, `SEQUENCE`, `TABLE`, `UUID` (JPA 3.1+). |
| `generator` | `String` | Nombre de un generador declarado con `@SequenceGenerator`/`@TableGenerator`. |

## 8. Valores por defecto
| Atributo | Default |
|---|---|
| `strategy` | `GenerationType.AUTO` |
| `generator` | `""` |

Con Hibernate 6 y `AUTO`: para tipos numéricos usa **SEQUENCE** con una secuencia `<entidad>_seq` e incremento **50**; para `UUID` usa generación de UUID.

## 9. Quién la procesa
Hibernate: crea un `IdentifierGenerator`/`Generator` (`SequenceStyleGenerator`, `IdentityGenerator`, `UuidGenerator`, `TableGenerator`) para la entidad.

## 10. Cuándo se procesa
- Configuración: arranque.
- Generación: al hacer `persist()` (SEQUENCE/UUID/TABLE) o al ejecutar el `INSERT` (IDENTITY).

## 11. Efecto observable
Tras `repository.save(entidad)`, `entidad.getId()` tiene valor. Con SEQUENCE verás `select nextval('cuentas_seq')` en los logs, pero solo una vez cada 50 inserciones.

## 12. Qué ocurre internamente
- **IDENTITY**: la BD genera el id en el `INSERT`. Hibernate debe ejecutar el `INSERT` **inmediatamente** al persistir para conocer el id → **desactiva el batching JDBC de inserts**.
- **SEQUENCE**: Hibernate pide `nextval` y con el optimizador *pooled* reserva un bloque de `allocationSize` ids en memoria; los inserts se pueden agrupar en batch.
- **TABLE**: simula una secuencia con una tabla y bloqueos; lento, casi nunca recomendable.
- **UUID**: genera el UUID en Java, sin ida a la BD.

## 13. Relación con otras anotaciones
- Requiere `@Id`.
- `@SequenceGenerator` (nombre de secuencia, `allocationSize`, `initialValue`) y `@TableGenerator`.
- Hibernate: `@UuidGenerator` (permite elegir UUID v7 ordenable por tiempo en versiones recientes), `@GenericGenerator` (deprecado).

## 14. Dependencias necesarias
`spring-boot-starter-data-jpa` + driver de la base de datos.

## 15. Alternativas
- Asignar el id en la aplicación (UUID, ULID, Snowflake) sin `@GeneratedValue`.
- Claves naturales (código ISO de país, IBAN) cuando son estables.

## 16. Limitaciones
- `IDENTITY` impide batch inserts (problema en cargas masivas).
- `SEQUENCE` requiere que la BD soporte secuencias (MySQL no las tiene: Hibernate usará `TABLE` con AUTO o hay que usar IDENTITY).
- Con `allocationSize = 50`, los ids tienen **huecos** tras reinicios; nunca uses el id como número correlativo de negocio (p. ej. número de factura).
- El `allocationSize` debe coincidir con el `INCREMENT BY` de la secuencia real; si no, Hibernate falla en validación o genera duplicados.

## 17. Errores comunes
- Migraciones con `CREATE SEQUENCE ... INCREMENT BY 1` y la entidad con `allocationSize` por defecto (50) → `The increment size of the [x_seq] sequence is set to [50] in the entity mapping while the associated database sequence increment size is [1]`.
- Usar ids consecutivos para numeración fiscal (facturas) — la normativa exige correlatividad sin huecos; se necesita un contador de negocio aparte.
- IDENTITY en tablas con millones de inserciones por lote.

## 18. Ejemplo básico
```java
@Entity
public class Cliente {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

## 19. Ejemplo real
**FinTech — asientos contables con inserción masiva en batch** (cierre diario de millones de movimientos) en PostgreSQL:
```java
@Entity
@Table(name = "asientos", schema = "ledger")
public class Asiento {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "asientos_gen")
    @SequenceGenerator(name = "asientos_gen", sequenceName = "ledger.asientos_seq", allocationSize = 100)
    private Long id;
    // ...
}
```
```sql
-- Flyway V20__asientos.sql
CREATE SEQUENCE ledger.asientos_seq INCREMENT BY 100;
```
```yaml
spring:
  jpa:
    properties:
      hibernate:
        jdbc.batch_size: 100
        order_inserts: true
```
La **numeración contable correlativa** (nº de asiento visible para auditoría) se lleva aparte, en una columna asignada dentro de la transacción de cierre.

**Otro dominio — IoT**: lecturas de sensores con `GenerationType.UUID` generadas en muchos nodos sin coordinación.

## 20. Qué ocurre si la elimino
El id deja de generarse: al guardar una entidad nueva con id `null`, Hibernate lanza `IdentifierGenerationException: Identifier of entity '...' must be manually assigned before calling 'persist()'`.
