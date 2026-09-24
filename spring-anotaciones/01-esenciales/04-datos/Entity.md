# @Entity

> Nivel: esencial · Categoría: datos / JPA

## 1. Nombre
`@Entity`

## 2. Paquete
`jakarta.persistence` (en Spring Boot 2.x: `javax.persistence`)

## 3. Framework / librería
Jakarta Persistence (JPA). Implementación por defecto en Spring Boot: **Hibernate ORM**.

## 4. Propósito
Declarar que una clase es una **entidad persistente**: sus instancias se corresponden con filas de una tabla y el `EntityManager` puede guardarlas, cargarlas y rastrear sus cambios.

## 5. Target
`ElementType.TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `name` | `String` | Nombre de la entidad en consultas JPQL (`SELECT c FROM Cuenta c`). No es el nombre de la tabla. |

## 8. Valores por defecto
`name = ""` → nombre simple de la clase.

## 9. Quién la procesa
- Descubrimiento: Spring Boot escanea los paquetes de `@AutoConfigurationPackage` (o `@EntityScan`) y construye `PersistenceManagedTypes`, que se pasan a `LocalContainerEntityManagerFactoryBean`.
- Metamodelo y mapeo: el proveedor JPA (Hibernate: `MetadataSources` → `MetadataBuildingProcess`).

## 10. Cuándo se procesa
Arranque, al crear el `EntityManagerFactory` (autoconfiguración `HibernateJpaAutoConfiguration`). En tiempo de ejecución, cada operación del `EntityManager`.

## 11. Efecto observable
- Si `spring.jpa.hibernate.ddl-auto=create/update`, Hibernate crea/modifica la tabla.
- Puedes usarla en repositorios Spring Data (`JpaRepository<Cuenta, Long>`) y en JPQL.
- Log de arranque: `Initialized JPA EntityManagerFactory for persistence unit 'default'`.

## 12. Qué ocurre internamente
1. Hibernate construye el **metamodelo**: por cada entidad, sus atributos, tipos, id, relaciones y la tabla asociada.
2. Genera y cachea las sentencias SQL básicas (insert, update, delete, select por id).
3. En ejecución, las entidades cargadas o persistidas quedan **gestionadas** en el contexto de persistencia (caché de primer nivel): Hibernate guarda una copia (*snapshot*) y al hacer `flush` compara (*dirty checking*) para generar los `UPDATE` necesarios.
4. Puede aplicar *bytecode enhancement* o proxies (subclases generadas) para la carga perezosa: por eso la clase no puede ser `final`.

## 13. Relación con otras anotaciones
- Obligatoria junto con `@Id` (o `@EmbeddedId`).
- `@Table`, `@Column`, `@GeneratedValue`, `@Version`, relaciones (`@OneToMany`…), callbacks (`@PrePersist`…).
- `@EntityScan` para escanear entidades fuera del paquete principal.
- `@MappedSuperclass` y `@Embeddable` son conceptos hermanos (no son entidades).

## 14. Dependencias necesarias
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<!-- + driver: postgresql, mysql-connector-j, ojdbc11, h2 (tests)... -->
```

## 15. Alternativas
- Spring Data JDBC (`@Table` de `org.springframework.data.relational`), más simple, sin *lazy loading* ni *dirty checking*.
- jOOQ, MyBatis, `JdbcClient` para SQL explícito.
- Spring Data MongoDB (`@Document`) para documentos.

## 16. Limitaciones / requisitos
- Necesita **constructor sin argumentos** `public` o `protected`.
- La clase y los métodos persistentes no deben ser `final`.
- Debe ser clase de primer nivel (no interfaz ni enum; los `record` **no** pueden ser entidades porque son finales e inmutables).
- `equals`/`hashCode` deben diseñarse con cuidado (id generado es `null` antes de persistir).

## 17. Errores comunes
- Entidad fuera del paquete escaneado → `Not a managed type: class com.x.Cuenta`.
- Mezclar `javax.persistence` y `jakarta.persistence` tras migrar a Boot 3 → la entidad no se reconoce.
- Usar Lombok `@Data` en entidades: `equals/hashCode/toString` recorren relaciones lazy → consultas inesperadas o `StackOverflowError`.
- Exponer la entidad directamente en la API.
- `ddl-auto=update` en producción (usar Flyway/Liquibase).

## 18. Ejemplo básico
```java
@Entity
public class Producto {
    @Id @GeneratedValue
    private Long id;
    private String nombre;
    private BigDecimal precio;

    protected Producto() { }            // requerido por JPA
    public Producto(String nombre, BigDecimal precio) { this.nombre = nombre; this.precio = precio; }
    // getters
}
```

## 19. Ejemplo real
**FinTech — cuenta bancaria** con invariantes de dominio (el saldo solo cambia mediante métodos de negocio) y bloqueo optimista:
```java
@Entity(name = "Cuenta")
@Table(name = "cuentas", uniqueConstraints = @UniqueConstraint(name = "uk_cuentas_iban", columnNames = "iban"))
public class Cuenta {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "cuentas_seq")
    @SequenceGenerator(name = "cuentas_seq", sequenceName = "cuentas_seq", allocationSize = 50)
    private Long id;

    @Column(nullable = false, length = 34, updatable = false)
    private String iban;

    @Column(nullable = false, precision = 19, scale = 4)
    private BigDecimal saldo = BigDecimal.ZERO;

    @Column(nullable = false, length = 3)
    private String moneda;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private EstadoCuenta estado = EstadoCuenta.ACTIVA;

    @Version
    private long version;                            // evita "lost updates" concurrentes

    protected Cuenta() { }

    public Cuenta(String iban, String moneda) { this.iban = iban; this.moneda = moneda; }

    public void debitar(BigDecimal importe) {
        if (estado != EstadoCuenta.ACTIVA) throw new CuentaBloqueadaException(iban);
        if (saldo.compareTo(importe) < 0) throw new SaldoInsuficienteException(saldo, importe);
        saldo = saldo.subtract(importe);
    }

    public void acreditar(BigDecimal importe) { saldo = saldo.add(importe); }

    @Override public boolean equals(Object o) {
        return o instanceof Cuenta c && id != null && id.equals(c.id);
    }
    @Override public int hashCode() { return getClass().hashCode(); }
}
```
**Otro dominio — educación**: `Alumno`, `Curso`, `Inscripcion` como entidades con relaciones.

## 20. Qué ocurre si la elimino
La clase deja de ser gestionada por JPA:
- Los repositorios fallan al arrancar: `Not a managed type: class ...Cuenta`.
- Las consultas JPQL fallan: `Cuenta is not mapped`.
- Las demás anotaciones JPA de la clase (`@Id`, `@Column`) se ignoran.
