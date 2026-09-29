## 🗄️ JPA y Hibernate

[⬆️ Volver al índice](./README.md)

---

## 📑 Contenidos de esta sección

### 🔹 Conceptos Fundamentales
1. [JPA vs Hibernate](#1-cuál-es-la-diferencia-entre-jpa-y-hibernate)
2. [Ciclo de vida de Entity](#2-cuál-es-el-ciclo-de-vida-de-una-entity)
3. [LAZY vs EAGER fetching](#3-cuál-es-la-diferencia-entre-lazy-y-eager-fetching)
4. [Problema N+1](#4-qué-es-el-problema-n1-y-cómo-solucionarlo)
5. [@Transactional](#5-qué-hace-transactional-y-cómo-funciona)
6. [Paginación](#6-cómo-manejas-grandes-volúmenes-de-datos-paginación)
7. [Proyecciones](#7-qué-son-las-proyecciones-en-spring-data)

---

### 🔹 Conceptos Fundamentales

#### 1. ¿Cuál es la diferencia entre JPA y Hibernate?

**Respuesta:**

**JPA (Java Persistence API)**:
- Es una **especificación** (conjunto de interfaces y reglas).
- Define **qué** debe hacer un ORM, no **cómo**.
- Parte de Java EE (ahora Jakarta EE).
- Las aplicaciones codifican contra la API JPA (portabilidad).

**Hibernate**:
- Es la **implementación** más popular de JPA.
- Proveedor que hace el trabajo real: genera SQL, gestiona caché, etc.
- Tiene características propias más allá de JPA.
- Otras implementaciones: EclipseLink, OpenJPA.

**Analogía:**
- JPA = JDBC (interfaz estándar)
- Hibernate = Driver MySQL/PostgreSQL (implementación concreta)

```java
// Código usando JPA (portable entre implementaciones)
import javax.persistence.Entity;
import javax.persistence.EntityManager;

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}

// Hibernate-specific (si necesitas features extra)
import org.hibernate.Session;
import org.hibernate.annotations.Cache;

@Entity
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)  // Feature de Hibernate
public class User { /* ... */ }
```

**Best Practice:** Usa anotaciones JPA estándar siempre que sea posible. Usa features de Hibernate solo cuando sea necesario.

---

#### 2. ¿Cuál es el ciclo de vida de una Entity?

**Respuesta:**

Las **entidades** atraviesan diferentes estados gestionados por el `EntityManager`/`Session`:

**Estados:**

```
┌─────────────┐
│  Transient  │  new User() → Objeto nuevo, sin ID
└──────┬──────┘
       │ persist()
       ▼
┌─────────────┐
│ Persistent  │  save() → Hibernate rastrea cambios (Dirty Checking)
└──────┬──────┘
       │ commit()
       │ detach() / clear() / close()
       ▼
┌─────────────┐
│  Detached   │  Objeto con ID pero desconectado de la sesión
└──────┬──────┘
       │ merge()
       ▼
┌─────────────┐
│ Persistent  │
└─────────────┘
       │ remove()
       ▼
┌─────────────┐
│   Removed   │  Marcado para DELETE
└─────────────┘
```

**1. Transient (Transitorio)**:
```java
User user = new User();  // Transient: Hibernate no lo conoce
user.setNombre("Juan");
// No está en BD, no tiene ID
```

**2. Persistent (Persistente)**:
```java
entityManager.persist(user);  // Ahora es Persistent
// Hibernate rastrea cambios automáticamente
user.setEmail("nuevo@mail.com");  // Cambio detectado
// Al commit, se genera UPDATE automático (Dirty Checking)
```

**3. Detached (Separado)**:
```java
entityManager.detach(user);  // Ya no está managed
// O al cerrar la sesión/transacción
user.setNombre("Pedro");  // Cambio NO se sincroniza con BD
```

**4. Removed (Eliminado)**:
```java
entityManager.remove(user);  // Marcado para DELETE
// Se ejecuta DELETE al commit
```

**Operaciones principales:**

```java
// persist: Transient → Persistent
User user = new User("Juan");
entityManager.persist(user);  // INSERT en commit

// find: BD → Persistent (o devuelve cached)
User found = entityManager.find(User.class, 1L);  // SELECT

// merge: Detached → Persistent (copia estado)
User detached = new User();
detached.setId(1L);
detached.setNombre("Pedro");
User managed = entityManager.merge(detached);  // UPDATE en commit

// remove: Persistent → Removed
entityManager.remove(managed);  // DELETE en commit

// detach: Persistent → Detached
entityManager.detach(managed);  // Deja de rastrear cambios

// refresh: Recarga desde BD (sobrescribe cambios no guardados)
entityManager.refresh(managed);  // SELECT FROM DB
```

**Dirty Checking:**
Hibernate compara automáticamente el estado actual de entidades Persistent con el snapshot original. Si hay cambios, genera UPDATE al commit.

```java
@Transactional
public void actualizarUsuario(Long id) {
    User user = userRepository.findById(id).get();  // Persistent
    user.setNombre("Nuevo Nombre");  // Cambio detectado
    // No hace falta llamar a save() ✅
}  // Al salir del método, @Transactional hace commit → UPDATE automático
```

---

### 🔹 Relaciones y Fetching

#### 3. ¿Cuál es la diferencia entre LAZY y EAGER fetching?

**Respuesta:**

**FetchType** define **cuándo** se cargan las asociaciones (relaciones) de una entidad desde la base de datos.

| Característica | LAZY | EAGER |
|----------------|------|-------|
| **Cuándo carga** | Cuando accedes al getter | Inmediatamente al cargar la entidad |
| **Proxy** | Sí (objeto vacío hasta acceso) | No |
| **Performance** | Mejor (carga bajo demanda) | Peor (puede traer datos innecesarios) |
| **Default en** | `@OneToMany`, `@ManyToMany` | `@ManyToOne`, `@OneToOne` |
| **Excepción** | `LazyInitializationException` | N/A |

**LAZY (Recomendado):**
```java
@Entity
public class User {
    @Id
    private Long id;
    
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)  // Default
    private List<Order> orders;
}

// Uso
User user = userRepository.findById(1L).get();
// SELECT * FROM users WHERE id = 1  (solo user, sin orders)

List<Order> orders = user.getOrders();  // Acceso al getter
// SELECT * FROM orders WHERE user_id = 1  (ahora carga orders)
```

**EAGER:**
```java
@Entity
public class User {
    @OneToMany(mappedBy = "user", fetch = FetchType.EAGER)  // ❌ No recomendado
    private List<Order> orders;
}

// Uso
User user = userRepository.findById(1L).get();
// SELECT u.*, o.* FROM users u LEFT JOIN orders o ON u.id = o.user_id WHERE u.id = 1
// Trae TODO siempre, aunque no lo necesites
```

**Problema: LazyInitializationException**

Ocurre al acceder a una relación LAZY **fuera** de una transacción/sesión:

```java
// ❌ MAL
@Service
public class UserService {
    public User getUser(Long id) {
        return userRepository.findById(id).get();
    }  // Transacción termina aquí, sesión se cierra
}

// En el Controller:
User user = userService.getUser(1L);
List<Order> orders = user.getOrders();  // ❌ LazyInitializationException
```

**Soluciones:**

**1. JOIN FETCH (mejor opción):**
```java
@Query("SELECT u FROM User u JOIN FETCH u.orders WHERE u.id = :id")
Optional<User> findByIdWithOrders(@Param("id") Long id);
// Una sola query con JOIN
```

**2. @Transactional en el método que usa la relación:**
```java
@Transactional(readOnly = true)
public UserDTO getUserWithOrders(Long id) {
    User user = userRepository.findById(id).get();
    user.getOrders().size();  // Fuerza la carga dentro de la transacción
    return toDTO(user);
}
```

**3. Entity Graph:**
```java
@EntityGraph(attributePaths = {"orders"})
@Query("SELECT u FROM User u WHERE u.id = :id")
Optional<User> findByIdWithOrders(@Param("id") Long id);
```

**Best Practice:**
- ✅ Usa LAZY por defecto en todas las relaciones.
- ✅ Usa JOIN FETCH en queries específicas cuando necesites la relación.
- ❌ Evita EAGER (salvo casos muy específicos).

---

#### 4. ¿Qué es el problema N+1 y cómo solucionarlo?

**Respuesta:**

El **problema N+1** es uno de los problemas de rendimiento más comunes en aplicaciones que usan ORM. Ocurre cuando cargas N entidades y cada una ejecuta 1 query adicional para traer su relación.

**Escenario:**

Tienes 10 usuarios y quieres listarlos con sus direcciones.

```java
@Entity
public class User {
    @ManyToOne(fetch = FetchType.LAZY)
    private Address address;
}

// Controller
List<User> users = userRepository.findAll();  // 1 query: SELECT * FROM users

for (User user : users) {
    System.out.println(user.getAddress().getCity());  // 10 queries: SELECT * FROM address WHERE id = ?
}

// TOTAL: 1 + 10 = 11 queries ❌
// Con 1000 usuarios: 1 + 1000 = 1001 queries 😱
```

**Hibernate log:**
```sql
SELECT * FROM users;              -- Query 1
SELECT * FROM address WHERE id = 1;  -- Query 2
SELECT * FROM address WHERE id = 2;  -- Query 3
SELECT * FROM address WHERE id = 3;  -- Query 4
...
SELECT * FROM address WHERE id = 10; -- Query 11
```

**Cómo detectarlo:**

Activa el logging de Hibernate:
```properties
# application.properties
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

**Soluciones:**

**1. JOIN FETCH (mejor solución):**
```java
@Query("SELECT u FROM User u JOIN FETCH u.address")
List<User> findAllWithAddress();

// SQL generado:
// SELECT u.*, a.* FROM users u INNER JOIN address a ON u.address_id = a.id
// ✅ UNA SOLA QUERY
```

**2. Entity Graph:**
```java
@EntityGraph(attributePaths = {"address"})
List<User> findAll();
```

**3. Batch Fetching (reduce N+1 a 1+ceil(N/batchSize)):**
```java
@Entity
public class User {
    @ManyToOne(fetch = FetchType.LAZY)
    @BatchSize(size = 10)  // Carga en lotes de 10
    private Address address;
}

// En lugar de 1 query por usuario, hace 1 query por lote:
// SELECT * FROM address WHERE id IN (1, 2, 3, ..., 10)
// SELECT * FROM address WHERE id IN (11, 12, 13, ..., 20)
```

**4. DTO Projection (no carga la entidad):**
```java
@Query("SELECT new com.example.UserDTO(u.id, u.nombre, a.city) " +
       "FROM User u JOIN u.address a")
List<UserDTO> findAllWithCity();
// ✅ Una query, devuelve DTOs directamente
```

**5. Subselect (para @OneToMany):**
```java
@Entity
public class User {
    @OneToMany(mappedBy = "user")
    @Fetch(FetchMode.SUBSELECT)  // Hibernate-specific
    private List<Order> orders;
}

// Genera:
// SELECT * FROM users
// SELECT * FROM orders WHERE user_id IN (SELECT id FROM users)
// 2 queries en total ✅
```

**Comparación de soluciones:**

| Solución | Queries | Complejidad | Uso |
|----------|---------|-------------|-----|
| JOIN FETCH | 1 | Baja | ✅ Caso general |
| Entity Graph | 1 | Media | Para múltiples relaciones |
| Batch Fetching | 1 + ceil(N/batch) | Baja | Cuando JOIN no es posible |
| DTO Projection | 1 | Media | Solo lectura, no entities |
| Subselect | 2 | Baja | @OneToMany |

**Best Practice:**
- ✅ Usa JOIN FETCH para la mayoría de los casos.

> 🔗 **Ver también:** [Cómo detectar queries lentas con EXPLAIN en PostgreSQL](./04-database-postgresql.md#6-qué-es-explain-y-explain-analyze)
- ✅ Monitorea queries en dev con logs.
- ✅ Usa herramientas APM en producción.
- ❌ Nunca uses EAGER como solución al N+1.

---

### 🔹 Transacciones

#### 5. ¿Qué hace @Transactional y cómo funciona?

**Respuesta:**

`@Transactional` es una anotación de Spring que envuelve el método en una **transacción de base de datos**, garantizando atomicidad (todo o nada).

**Ciclo de vida de una transacción:**

```
1. @Transactional detectado
   ↓
2. Spring crea un Proxy alrededor del bean
   ↓
3. Antes del método: BEGIN TRANSACTION
   ↓
4. Ejecuta el método
   ↓
5a. Si todo OK → COMMIT
5b. Si RuntimeException → ROLLBACK
```

**Ejemplo básico:**

```java
@Service
public class UserService {
    
    @Transactional
    public void transferMoney(Long fromId, Long toId, BigDecimal amount) {
        Account from = accountRepository.findById(fromId).get();
        Account to = accountRepository.findById(toId).get();
        
        from.setBalance(from.getBalance().subtract(amount));
        to.setBalance(to.getBalance().add(amount));
        
        accountRepository.save(from);
        accountRepository.save(to);
        
        if (amount.compareTo(BigDecimal.valueOf(10000)) > 0) {
            throw new RuntimeException("Monto sospechoso");  // ❌ ROLLBACK
        }
    }  // ✅ COMMIT si no hay excepción
}
```

**Atributos importantes:**

```java
@Transactional(
    propagation = Propagation.REQUIRED,     // Comportamiento de propagación
    isolation = Isolation.DEFAULT,          // Nivel de aislamiento
    timeout = 30,                           // Timeout en segundos
    readOnly = false,                       // Solo lectura (optimización)
    rollbackFor = Exception.class,          // Rollback para estas excepciones
    noRollbackFor = IllegalArgumentException.class  // NO rollback para estas
)
```

**Propagation (Propagación):**

| Tipo | Descripción | Uso |
|------|-------------|-----|
| **REQUIRED** (default) | Usa transacción existente o crea una nueva | Caso general |
| **REQUIRES_NEW** | Siempre crea nueva transacción (suspende la actual) | Logging, auditoría independiente |
| **NESTED** | Transacción anidada (savepoint) | Rollback parcial |
| **MANDATORY** | Debe existir transacción, sino lanza excepción | Métodos que siempre deben ser transaccionales |
| **SUPPORTS** | Usa transacción si existe, sino corre sin ella | Métodos flexibles |
| **NOT_SUPPORTED** | Suspende transacción existente | Operaciones que no deben ser transaccionales |
| **NEVER** | Lanza excepción si hay transacción activa | Garantizar no-transaccionalidad |

**Isolation (Aislamiento):**

| Nivel | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|-------|------------|---------------------|--------------|-------------|
| **READ_UNCOMMITTED** | ✅ Permite | ✅ Permite | ✅ Permite | Rápido |
| **READ_COMMITTED** | ❌ Previene | ✅ Permite | ✅ Permite | Normal |
| **REPEATABLE_READ** | ❌ Previene | ❌ Previene | ✅ Permite | Lento |
| **SERIALIZABLE** | ❌ Previene | ❌ Previene | ❌ Previene | Muy lento |

**Excepciones y Rollback:**

Por defecto:
- **RuntimeException** y **Error** → ROLLBACK
- **Checked Exceptions** → NO rollback

```java
@Transactional(rollbackFor = Exception.class)  // Rollback para todas
public void metodo() throws Exception {
    // ...
    throw new IOException();  // Ahora sí hace rollback
}

@Transactional(noRollbackFor = BusinessException.class)
public void metodo2() {
    // ...
    throw new BusinessException();  // NO hace rollback
}
```

**readOnly = true (importante para optimización):**

```java
@Transactional(readOnly = true)
public List<UserDTO> getAllUsers() {
    // Optimizaciones:
    // - No hace flush
    // - Hint para BD (puede usar replicas de solo lectura)
    // - Hibernate no rastrea cambios (Dirty Checking desactivado)
    return userRepository.findAll().stream()
        .map(this::toDTO)
        .collect(Collectors.toList());
}
```

**Problemas comunes:**

**1. @Transactional en métodos privados (NO funciona):**
```java
@Service
public class UserService {
    @Transactional  // ❌ NO funciona, Spring usa proxies y no intercepta métodos privados
    private void metodoPrivado() {
        // ...
    }
}
```

**2. Llamadas internas (self-invocation):**
```java
@Service
public class UserService {
    public void metodoA() {
        metodoB();  // ❌ @Transactional en metodoB no se aplica (llamada directa, no por proxy)
    }
    
    @Transactional
    public void metodoB() {
        // ...
    }
}

// Solución: Inyectar el mismo servicio
@Service
public class UserService {
    @Autowired
    private UserService self;
    
    public void metodoA() {
        self.metodoB();  // ✅ Ahora sí funciona (llamada por proxy)
    }
    
    @Transactional
    public void metodoB() {
        // ...
    }
}
```

**Best Practices:**
- ✅ Coloca @Transactional en la capa de Service, no en Controller ni Repository.
- ✅ Usa `readOnly = true` para métodos de solo lectura.
- ✅ Mantén las transacciones cortas (no hagas operaciones lentas dentro).
- ✅ Especifica `rollbackFor` si usas checked exceptions.
- ❌ No uses @Transactional en métodos privados.

---

#### 6. ¿Cómo manejas grandes volúmenes de datos (Paginación)?

**Respuesta:**

Nunca debes usar `findAll()` en tablas grandes porque podrías **saturar la memoria** (OutOfMemoryError). Spring Data JPA proporciona la interfaz `Pageable` para manejar paginación y ordenamiento eficientemente.

**Uso en Repository:**

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Retorna una "página" de datos y metadata (total de páginas, elementos, etc.)
    Page<User> findAll(Pageable pageable);
    
    // Queries custom con paginación
    Page<User> findByActiveTrue(Pageable pageable);
    
    @Query("SELECT u FROM User u WHERE u.email LIKE %:domain%")
    Page<User> findByEmailDomain(@Param("domain") String domain, Pageable pageable);
    
    // Slice: más rápido si NO necesitas el total de elementos
    // (no ejecuta COUNT(*), útil para "scroll infinito")
    Slice<User> findByAgeGreaterThan(int age, Pageable pageable);
}
```

**Diferencia Page vs Slice:**

| Feature | Page | Slice |
|---------|------|-------|
| **Contiene** | Lista de elementos + metadata completa | Lista de elementos + hasNext() |
| **Queries ejecutadas** | SELECT + COUNT(*) | Solo SELECT |
| **Performance** | Más lento (2 queries) | Más rápido (1 query) |
| **Metadata** | Total pages, total elements, hasNext, hasPrevious | Solo hasNext |
| **Uso** | Paginación con número de páginas | Infinite scroll |

**Uso en Controller/Service:**

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserRepository userRepository;
    
    @GetMapping
    public ResponseEntity<Page<UserDTO>> getUsers(
        @RequestParam(defaultValue = "0") int page,           // Página 0-indexed
        @RequestParam(defaultValue = "10") int size,          // Tamaño de página
        @RequestParam(defaultValue = "id") String sortBy,     // Campo de ordenamiento
        @RequestParam(defaultValue = "asc") String direction  // asc o desc
    ) {
        // Crear Pageable
        Sort sort = direction.equalsIgnoreCase("desc") 
            ? Sort.by(sortBy).descending() 
            : Sort.by(sortBy).ascending();
        
        Pageable pageable = PageRequest.of(page, size, sort);
        
        // Obtener página
        Page<User> userPage = userRepository.findAll(pageable);
        
        // Convertir a DTO
        Page<UserDTO> dtoPage = userPage.map(this::toDTO);
        
        return ResponseEntity.ok(dtoPage);
    }
    
    private UserDTO toDTO(User user) {
        return new UserDTO(user.getId(), user.getName(), user.getEmail());
    }
}
```

**Request de ejemplo:**
```
GET /api/users?page=0&size=20&sortBy=name&direction=desc
```

**Respuesta JSON:**
```json
{
  "content": [
    {"id": 1, "name": "Zoe", "email": "zoe@example.com"},
    {"id": 2, "name": "Xavier", "email": "xavier@example.com"}
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 20,
    "sort": {"sorted": true, "unsorted": false}
  },
  "totalPages": 5,
  "totalElements": 97,
  "last": false,
  "first": true,
  "numberOfElements": 20,
  "size": 20,
  "number": 0
}
```

**Ordenamiento múltiple:**

```java
// Ordenar por múltiples campos
Sort sort = Sort.by(
    Sort.Order.desc("createdAt"),
    Sort.Order.asc("name")
);
Pageable pageable = PageRequest.of(page, size, sort);
```

**Uso con Specifications (filtros dinámicos):**

```java
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    public Page<User> searchUsers(String name, Boolean active, int page, int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("name").ascending());
        
        Specification<User> spec = Specification.where(null);
        
        if (name != null) {
            spec = spec.and((root, query, cb) -> 
                cb.like(cb.lower(root.get("name")), "%" + name.toLowerCase() + "%"));
        }
        
        if (active != null) {
            spec = spec.and((root, query, cb) -> 
                cb.equal(root.get("active"), active));
        }
        
        return userRepository.findAll(spec, pageable);
    }
}
```

**Paginación con DTO usando Projections:**

```java
// Interface Projection (más eficiente)
public interface UserSummary {
    Long getId();
    String getName();
    String getEmail();
}

public interface UserRepository extends JpaRepository<User, Long> {
    Page<UserSummary> findByActiveTrue(Pageable pageable);
}

// Uso
@GetMapping("/active")
public Page<UserSummary> getActiveUsers(Pageable pageable) {
    return userRepository.findByActiveTrue(pageable);
}
```

**Best Practices:**

- ✅ **Siempre usa paginación** para endpoints que retornan colecciones.
- ✅ Configura límites máximos:
  ```java
  @PageableDefault(size = 20, sort = "id")
  @Parameter(description = "Page number", example = "0")
  public Page<UserDTO> getUsers(Pageable pageable) {
      // Spring limita automáticamente el tamaño
  }
  ```
- ✅ Usa `Slice` en lugar de `Page` si no necesitas el total de elementos (más rápido).
- ✅ Crea índices en columnas usadas para ordenamiento:
  ```sql
  CREATE INDEX idx_users_name ON users(name);
  ```
- ✅ Para grandes volúmenes (millones), considera **Keyset Pagination** (más eficiente que offset):
  ```java
  // En lugar de page=0, page=1, etc.
  // Usa: lastId=1000 (busca los siguientes 20 después de ID 1000)
  @Query("SELECT u FROM User u WHERE u.id > :lastId ORDER BY u.id")
  List<User> findNext(@Param("lastId") Long lastId, Pageable pageable);
  ```

---

#### 7. ¿Qué son las Proyecciones en Spring Data?

**Respuesta:**

Las **Proyecciones** permiten recuperar **solo campos específicos** de una entidad, en lugar de traer el objeto completo. Esto mejora el **performance** al reducir:
- Cantidad de datos transferidos desde la BD.
- Memoria usada.
- Tiempo de serialización JSON.

**Problema sin proyecciones:**

```java
@Entity
public class User {
    @Id
    private Long id;
    private String name;
    private String email;
    private String password;  // No queremos exponer esto
    private byte[] profileImage;  // Pesado, no siempre necesario
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    
    @OneToMany(mappedBy = "user")
    private List<Order> orders;  // Relación que no necesitamos
}

// ❌ Trae TODOS los campos (incluyendo password, imagen, relaciones)
List<User> users = userRepository.findAll();
```

**Soluciones con Proyecciones:**

**1. Interface-Based Projections (Recomendado)**

Define una **interfaz** con solo los getters que necesitas. Spring Data JPA genera la implementación automáticamente.

```java
// Proyección: solo id, name y email
public interface UserSummary {
    Long getId();
    String getName();
    String getEmail();
}

public interface UserRepository extends JpaRepository<User, Long> {
    List<UserSummary> findBy();  // Retorna solo id, name, email
    
    Optional<UserSummary> findById(Long id);
    
    List<UserSummary> findByActiveTrue();
}

// Uso en Controller
@GetMapping("/users/summary")
public List<UserSummary> getUsersSummary() {
    return userRepository.findBy();  // Solo 3 campos, no 10
}
```

**SQL generado:**
```sql
SELECT u.id, u.name, u.email FROM users u;
-- En lugar de: SELECT * FROM users;
```

**2. Class-Based Projections (DTOs)**

Define una clase (puede ser un Record) con los campos necesarios.

```java
// DTO con Record (Java 16+)
public record UserDTO(Long id, String name, String email) {}

public interface UserRepository extends JpaRepository<User, Long> {
    
    @Query("SELECT new com.example.dto.UserDTO(u.id, u.name, u.email) FROM User u")
    List<UserDTO> findAllUserDTOs();
    
    @Query("SELECT new com.example.dto.UserDTO(u.id, u.name, u.email) " +
           "FROM User u WHERE u.active = true")
    List<UserDTO> findActiveUserDTOs();
}
```

**3. Dynamic Projections (Flexible)**

Permite elegir la proyección en tiempo de ejecución.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    <T> List<T> findByActiveTrue(Class<T> type);  // Genérico
}

// Uso
List<UserSummary> summaries = userRepository.findByActiveTrue(UserSummary.class);
List<UserDetailView> details = userRepository.findByActiveTrue(UserDetailView.class);
```

**4. Proyecciones Anidadas (con relaciones)**

```java
public interface OrderSummary {
    Long getId();
    BigDecimal getTotal();
    
    UserInfo getUser();  // Proyección anidada
    
    interface UserInfo {
        String getName();
        String getEmail();
    }
}

@Query("SELECT o FROM Order o JOIN FETCH o.user WHERE o.status = :status")
List<OrderSummary> findByStatus(@Param("status") OrderStatus status);
```

**5. Proyecciones con SpEL (expresiones)**

```java
public interface UserFullName {
    
    @Value("#{target.firstName + ' ' + target.lastName}")
    String getFullName();  // Combina firstName y lastName
    
    @Value("#{target.orders.size()}")
    int getOrderCount();  // Cuenta relaciones
}
```

**6. Open Projections (computed properties)**

```java
public interface UserView {
    String getName();
    String getEmail();
    
    // Propiedad computada (ejecuta código Java, no SQL)
    default String getDisplayName() {
        return getName().toUpperCase();
    }
}
```

**Comparación:**

| Tipo | Performance | Flexibilidad | Uso |
|------|-------------|--------------|-----|
| **Interface Projection** | ⚡⚡⚡ Rápido | Media | Casos simples, fields específicos |
| **DTO (Class)** | ⚡⚡ Medio | Alta | Control total, validaciones custom |
| **Dynamic Projection** | ⚡⚡ Medio | ⚡⚡⚡ Alta | Múltiples vistas del mismo entity |
| **Nested Projection** | ⚡ Depende | Media | Relaciones, evitar N+1 |

**Best Practices:**

- ✅ Usa **Interface Projections** para APIs públicas (evita exponer entidades JPA).
- ✅ Usa **DTOs con Records** para respuestas inmutables.
- ✅ Combina con **Paginación**:
  ```java
  Page<UserSummary> findByActiveTrue(Pageable pageable);
  ```
- ✅ Usa proyecciones para **listados** (tablas, dropdowns).
- ✅ Usa entidades completas para **operaciones CRUD** (edición).
- ❌ No uses open projections con `@Value` en queries pesadas (procesamiento en Java, no en BD).

**Ejemplo completo:**

```java
// Entity
@Entity
public class Product {
    @Id
    private Long id;
    private String name;
    private String description;
    private BigDecimal price;
    private Integer stock;
    private byte[] image;  // Pesado
    private LocalDateTime createdAt;
}

// Proyección para listado (tabla)
public interface ProductListView {
    Long getId();
    String getName();
    BigDecimal getPrice();
    Integer getStock();
}

// Proyección para detalle (modal)
public interface ProductDetailView {
    Long getId();
    String getName();
    String getDescription();
    BigDecimal getPrice();
    Integer getStock();
    // NO incluye image ni createdAt
}

// Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    Page<ProductListView> findAll(Pageable pageable);
    
    Optional<ProductDetailView> findById(Long id);
}

// Controller
@GetMapping("/products")
public Page<ProductListView> listProducts(Pageable pageable) {
    return productRepository.findAll(pageable);  // Solo 4 campos
}

@GetMapping("/products/{id}")
public ProductDetailView getProduct(@PathVariable Long id) {
    return productRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
}
```

**Resultado:**
- Endpoint `/products` retorna solo: `id`, `name`, `price`, `stock`.
- Endpoint `/products/1` añade: `description`.
- Nunca exponemos: `image`, `createdAt` (a menos que sea necesario).
- **50-70% menos datos transferidos** en listados grandes.

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

[⬅️ Anterior: Spring Boot](./02-spring-boot.md) | [🏠 Volver al Inicio](./README.md) | [Siguiente: Base de Datos ➡️](./04-database-postgresql.md)