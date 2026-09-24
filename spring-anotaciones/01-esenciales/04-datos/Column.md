# @Column

> Nivel: esencial · Categoría: datos / JPA

## 1. Nombre
`@Column`

## 2. Paquete
`jakarta.persistence`

## 3. Framework / librería
Jakarta Persistence (JPA). Implementación: Hibernate ORM.

## 4. Propósito
Personalizar el **mapeo de un atributo a una columna**: nombre, nulabilidad, longitud, precisión, si participa en `INSERT`/`UPDATE`, etc.

## 5. Target
`ElementType.METHOD`, `ElementType.FIELD`.

## 6. Retention
`RetentionPolicy.RUNTIME`.

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `name` | `String` | Nombre de la columna. |
| `unique` | `boolean` | Crea restricción única (DDL). |
| `nullable` | `boolean` | Permite `NULL` (DDL). |
| `insertable` | `boolean` | Si se incluye en el `INSERT`. |
| `updatable` | `boolean` | Si se incluye en el `UPDATE`. |
| `columnDefinition` | `String` | Fragmento SQL literal para el tipo (DDL). |
| `table` | `String` | Tabla (si la entidad usa `@SecondaryTable`). |
| `length` | `int` | Longitud para `VARCHAR`. |
| `precision` | `int` | Dígitos totales para `DECIMAL/NUMERIC`. |
| `scale` | `int` | Decimales para `DECIMAL/NUMERIC`. |
| `options`, `comment`, `check`, `secondPrecision` | varios | (JPA 3.2+) Opciones extra de DDL, comentario, `CHECK` de columna, precisión de fracciones de segundo. |

## 8. Valores por defecto
| Atributo | Default |
|---|---|
| `name` | `""` → nombre del atributo con la estrategia de nombres (`fechaValor` → `fecha_valor`) |
| `unique` | `false` |
| `nullable` | `true` |
| `insertable` / `updatable` | `true` |
| `columnDefinition`, `table` | `""` |
| `length` | `255` |
| `precision` / `scale` | `0` (Hibernate usa el default del dialecto, p. ej. `numeric(38,2)`) |

## 9. Quién la procesa
Hibernate al construir el metamodelo (`PropertyBinder`, `ColumnsBuilder`).

## 10. Cuándo se procesa
Arranque (metamodelo, DDL si `ddl-auto`). `insertable`/`updatable` influyen en cada SQL generado.

## 11. Efecto observable
- Nombres de columnas en el SQL.
- Con `ddl-auto`, tipos y restricciones en la tabla creada.
- Con `updatable = false`, los cambios en ese atributo **no se guardan** (sin error).

## 12. Qué ocurre internamente
1. Hibernate crea un objeto `Column` en su modelo relacional con nombre, tipo SQL (según el tipo Java y el dialecto), longitud, precisión…
2. Las sentencias `INSERT`/`UPDATE` precompiladas incluyen o excluyen la columna según `insertable`/`updatable`.
3. `nullable`, `unique`, `length` **no se validan en Java**: solo afectan al DDL y (con `ddl-auto=validate`) a la validación de esquema. Hibernate Validator puede derivar `@NotNull`/`@Size` al DDL, pero no al revés.

## 13. Relación con otras anotaciones
- `@Id`, `@Version`, `@Enumerated`, `@Lob`, `@Convert`, `@Temporal` (obsoleto con `java.time`).
- Bean Validation (`@NotNull`, `@Size`) para validar en Java; `@Column` para el esquema.
- `@JoinColumn` es su equivalente para claves foráneas.
- `@AttributeOverride` para renombrar columnas de `@Embeddable`.

## 14. Dependencias necesarias
`spring-boot-starter-data-jpa`.

## 15. Alternativas
- Omitirla y usar la convención de nombres.
- `@JdbcTypeCode`, `@ColumnTransformer` (Hibernate) para casos especiales.

## 16. Limitaciones
- `nullable = false` no evita que Java tenga `null`; la BD lanzará el error al hacer `flush`.
- `columnDefinition` ata el código a un motor concreto.
- `precision`/`scale` solo aplican a `BigDecimal`/`BigInteger`.

## 17. Errores comunes
- `BigDecimal` sin `precision/scale` para dinero → la columna queda con 2 decimales por defecto y se redondean tipos de cambio o intereses con 4‑6 decimales.
- Pensar que `length = 50` valida la longitud en Java (usa `@Size`).
- `updatable = false` en un campo que sí debería cambiar → cambios que "desaparecen" en silencio.
- `double` para importes.

## 18. Ejemplo básico
```java
@Column(name = "correo", nullable = false, unique = true, length = 120)
private String email;
```

## 19. Ejemplo real
**FinTech — préstamo con importes, tasas y campos inmutables**:
```java
@Entity
@Table(name = "prestamos")
public class Prestamo {

    @Id @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private Long id;

    @Column(name = "numero_contrato", nullable = false, unique = true, length = 20, updatable = false)
    private String numeroContrato;

    @Column(name = "capital", nullable = false, precision = 19, scale = 4, updatable = false)
    private BigDecimal capital;

    @Column(name = "tna", nullable = false, precision = 9, scale = 6)   // 0.452500 = 45,25 %
    private BigDecimal tasaNominalAnual;

    @Column(name = "saldo_pendiente", nullable = false, precision = 19, scale = 4)
    private BigDecimal saldoPendiente;

    @Column(name = "creado_en", nullable = false, updatable = false)
    private Instant creadoEn;

    @Column(name = "cliente_id", nullable = false, updatable = false)
    private Long clienteId;

    protected Prestamo() {}
}
```
**Otro dominio — e‑commerce**: `@Column(columnDefinition = "jsonb")` para atributos variables de producto en PostgreSQL (junto con `@JdbcTypeCode(SqlTypes.JSON)` de Hibernate).

## 20. Qué ocurre si la elimino
El atributo sigue persistiéndose (JPA mapea todos los atributos por defecto), pero con nombre implícito y valores por defecto: si la columna real se llama distinto → `column "x" does not exist`; con `ddl-auto`, los importes pierden precisión (escala por defecto) y los campos inmutables pasan a ser actualizables.
