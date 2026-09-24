# @Id

> Nivel: esencial · Categoría: datos / JPA

## 1. Nombre
`@Id` (JPA)

## 2. Paquete
`jakarta.persistence`

> No confundir con `org.springframework.data.annotation.Id`, que usan Spring Data JDBC, MongoDB, Redis, etc.

## 3. Framework / librería
Jakarta Persistence (JPA). Implementación: Hibernate ORM.

## 4. Propósito
Marcar el atributo que es la **clave primaria** de la entidad. Determina la identidad del objeto en el contexto de persistencia.

## 5. Target
`ElementType.METHOD`, `ElementType.FIELD`.

## 6. Retention
`RetentionPolicy.RUNTIME`.

## 7. Atributos
No tiene atributos.

## 8. Valores por defecto
No aplica. La columna se llama como el atributo (o como indique `@Column`).

## 9. Quién la procesa
El proveedor JPA (Hibernate) al construir el metamodelo. Spring Data lo usa vía `JpaEntityInformation` para `findById`, `existsById` y para decidir si `save()` hace `persist` o `merge`.

## 10. Cuándo se procesa
Arranque (metamodelo) y en cada operación de persistencia.

## 11. Efecto observable
- `find(Cuenta.class, 42L)` busca por esa columna.
- La **posición** de `@Id` (campo o getter) define el **tipo de acceso** de toda la entidad: acceso por campo o por propiedad.

## 12. Qué ocurre internamente
1. Hibernate crea el `IdentifierProperty` y el `EntityPersister` usa la clave para `SELECT ... WHERE id = ?`.
2. El contexto de persistencia es un mapa `EntityKey(tipo, id) → instancia`: dos `find` con el mismo id en la misma transacción devuelven **la misma instancia**.
3. `SimpleJpaRepository.save()` llama a `entityInformation.isNew(entity)`: si el id es `null` (o hay `@Version` nulo) → `persist`; si no → `merge` (que hace un `SELECT` previo).

## 13. Relación con otras anotaciones
- `@GeneratedValue` para generar el valor.
- `@EmbeddedId` / `@IdClass` para claves compuestas.
- `@MapsId` para compartir la clave con una relación.
- `@Version` influye en la detección de "nuevo".
- `Persistable<ID>` (Spring Data) para controlar `isNew()` con ids asignados manualmente.

## 14. Dependencias necesarias
`spring-boot-starter-data-jpa`.

## 15. Alternativas
`@EmbeddedId` (clave compuesta como objeto), `@IdClass` (clave compuesta con varios `@Id`), `@NaturalId` de Hibernate como identificador natural adicional.

## 16. Limitaciones
- Toda entidad necesita exactamente un identificador (simple o compuesto).
- Tipos recomendados: `Long`, `UUID`, `String`; evita primitivos (`long` vale `0`, no `null`, y confunde la detección de nuevo).
- No debe cambiar una vez persistido.

## 17. Errores comunes
- Entidad sin `@Id` → `No identifier specified for entity`.
- Usar `long` primitivo con `@GeneratedValue`: con valor `0`, Spring Data puede considerarla existente y hacer `merge`.
- Asignar UUIDs manualmente sin `Persistable` → cada `save()` hace un `SELECT` extra (merge) antes del `INSERT`.
- Poner `@Id` en el getter y otras anotaciones en campos → Hibernate ignora las de los campos (acceso por propiedad).

## 18. Ejemplo básico
```java
@Entity
public class Categoria {
    @Id
    private String codigo;   // clave natural asignada: "ELEC", "HOGAR"
    private String nombre;
    protected Categoria() {}
}
```

## 19. Ejemplo real
**FinTech — pago con UUID generado en la aplicación** (útil para idempotencia y sistemas distribuidos) evitando el `SELECT` extra de `merge`:
```java
@Entity
@Table(name = "pagos")
public class Pago implements Persistable<UUID> {

    @Id
    private UUID id;                       // lo genera la app (UUID v7 ordenable por tiempo)

    @Column(nullable = false, precision = 19, scale = 4)
    private BigDecimal importe;

    @Transient
    private boolean nuevo = true;

    protected Pago() { }

    public Pago(UUID id, BigDecimal importe) { this.id = id; this.importe = importe; }

    @Override public UUID getId() { return id; }
    @Override public boolean isNew() { return nuevo; }

    @PostLoad @PostPersist
    void marcarNoNuevo() { this.nuevo = false; }
}
```
**Otro dominio — catálogo**: `Pais` con `@Id String codigoIso` (`"AR"`, `"ES"`).

## 20. Qué ocurre si la elimino
Hibernate no puede construir el metamodelo y la aplicación **no arranca**: `org.hibernate.AnnotationException: Entity 'com.neobank.Pago' has no identifier (every '@Entity' class must declare or inherit at least one '@Id' or '@EmbeddedId' property)`.
