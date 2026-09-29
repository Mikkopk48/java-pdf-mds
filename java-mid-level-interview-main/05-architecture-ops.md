## 🏗️ Arquitectura, Testing y DevOps

[⬆️ Volver al índice](./README.md)

---

## 📑 Contenidos de esta sección

### 🔹 Principios de Diseño
1. [Principios SOLID](#1-qué-son-los-principios-solid)
2. [Arquitectura en capas](#2-qué-es-la-arquitectura-en-capas-layered-architecture)

### 🔹 Testing
3. [Test Unitario vs Integración](#3-cuál-es-la-diferencia-entre-test-unitario-y-test-de-integración)
4. [TDD](#4-qué-es-tdd-test-driven-development)

### 🔹 Seguridad
5. [Autenticación vs Autorización](#5-cuál-es-la-diferencia-entre-autenticación-y-autorización)
6. [JWT](#6-qué-es-jwt-y-cómo-funciona)
7. [OAuth2 vs OIDC vs JWT](#7-cuál-es-la-diferencia-entre-oauth2-openid-connect-oidc-y-jwt)

### 🔹 DevOps
8. [Docker](#8-qué-es-docker-y-por-qué-usarlo)
9. [CI/CD](#9-qué-es-cicd)

---

### 🔹 Principios de Diseño y Arquitectura

#### 1. ¿Qué son los principios SOLID?

**Respuesta:**

**SOLID** es un acrónimo de cinco principios de diseño orientado a objetos que hacen el código más **mantenible**, **escalable** y **testeable**.

**S - Single Responsibility Principle (Responsabilidad Única)**

Una clase debe tener **una sola razón para cambiar**.

```java
// ❌ MAL: Múltiples responsabilidades
public class User {
    public void save() {
        // Guarda en BD
    }
    
    public void sendEmail() {
        // Envía email
    }
    
    public String generateReport() {
        // Genera reporte
    }
}

// ✅ BIEN: Responsabilidades separadas
public class User {
    private String name;
    private String email;
    // Solo datos
}

@Repository
public class UserRepository {
    public void save(User user) {
        // Solo persistencia
    }
}

@Service
public class EmailService {
    public void sendWelcomeEmail(User user) {
        // Solo envío de emails
    }
}

@Service
public class ReportService {
    public String generateUserReport(User user) {
        // Solo generación de reportes
    }
}
```

**O - Open/Closed Principle (Abierto/Cerrado)**

Abierto para **extensión**, cerrado para **modificación**.

```java
// ❌ MAL: Modificar clase existente para agregar funcionalidad
public class PaymentProcessor {
    public void processPayment(String type, double amount) {
        if (type.equals("credit")) {
            // Procesar tarjeta
        } else if (type.equals("paypal")) {
            // Procesar PayPal
        } else if (type.equals("bitcoin")) {  // ❌ Modificaste la clase
            // Procesar Bitcoin
        }
    }
}

// ✅ BIEN: Extender sin modificar
public interface PaymentMethod {
    void process(double amount);
}

public class CreditCardPayment implements PaymentMethod {
    public void process(double amount) {
        // Procesar tarjeta
    }
}

public class PayPalPayment implements PaymentMethod {
    public void process(double amount) {
        // Procesar PayPal
    }
}

public class BitcoinPayment implements PaymentMethod {  // ✅ Nueva clase, no modificas las existentes
    public void process(double amount) {
        // Procesar Bitcoin
    }
}

@Service
public class PaymentProcessor {
    public void processPayment(PaymentMethod method, double amount) {
        method.process(amount);  // Polimorfismo
    }
}
```

**L - Liskov Substitution Principle (Sustitución de Liskov)**

Las subclases deben poder **sustituir** a sus clases base sin romper el programa.

```java
// ❌ MAL: Viola Liskov
public class Rectangle {
    protected int width;
    protected int height;
    
    public void setWidth(int width) {
        this.width = width;
    }
    
    public void setHeight(int height) {
        this.height = height;
    }
    
    public int getArea() {
        return width * height;
    }
}

public class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width;  // ❌ Comportamiento inesperado
    }
    
    @Override
    public void setHeight(int height) {
        this.width = height;
        this.height = height;
    }
}

// Uso
Rectangle rect = new Square();
rect.setWidth(5);
rect.setHeight(10);
System.out.println(rect.getArea());  // Esperamos 50, obtenemos 100 ❌

// ✅ BIEN: Jerarquía correcta
public interface Shape {
    int getArea();
}

public class Rectangle implements Shape {
    private int width;
    private int height;
    
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    
    public int getArea() {
        return width * height;
    }
}

public class Square implements Shape {
    private int side;
    
    public Square(int side) {
        this.side = side;
    }
    
    public int getArea() {
        return side * side;
    }
}
```

**I - Interface Segregation Principle (Segregación de Interfaces)**

Muchas interfaces **específicas** son mejores que una interface **general**.

```java
// ❌ MAL: Interfaz grande, clientes forzados a implementar métodos que no usan
public interface Worker {
    void work();
    void eat();
    void sleep();
}

public class Robot implements Worker {
    public void work() { /* OK */ }
    public void eat() { throw new UnsupportedOperationException(); }  // ❌ Robot no come
    public void sleep() { throw new UnsupportedOperationException(); }  // ❌ Robot no duerme
}

// ✅ BIEN: Interfaces segregadas
public interface Workable {
    void work();
}

public interface Eatable {
    void eat();
}

public interface Sleepable {
    void sleep();
}

public class Human implements Workable, Eatable, Sleepable {
    public void work() { /* ... */ }
    public void eat() { /* ... */ }
    public void sleep() { /* ... */ }
}

public class Robot implements Workable {
    public void work() { /* ... */ }
    // No implementa Eatable ni Sleepable ✅
}
```

**D - Dependency Inversion Principle (Inversión de Dependencias)**

Depender de **abstracciones**, no de **concreciones**.

```java
// ❌ MAL: Dependencia directa de implementación concreta
public class UserService {
    private MySQLUserRepository repository = new MySQLUserRepository();  // ❌ Acoplado a MySQL
    
    public User findUser(Long id) {
        return repository.findById(id);
    }
}

// ✅ BIEN: Depende de abstracción
public interface UserRepository {
    User findById(Long id);
}

@Repository
public class MySQLUserRepository implements UserRepository {
    public User findById(Long id) {
        // Implementación MySQL
    }
}

@Repository
public class MongoUserRepository implements UserRepository {
    public User findById(Long id) {
        // Implementación MongoDB
    }
}

@Service
public class UserService {
    private final UserRepository repository;  // ✅ Depende de interfaz
    
    public UserService(UserRepository repository) {  // DI
        this.repository = repository;
    }
    
    public User findUser(Long id) {
        return repository.findById(id);
    }
}
```

**Beneficios de SOLID:**
- ✅ Código más mantenible y extensible.
- ✅ Facilita el testing (puedes mockear interfaces).
- ✅ Reduce acoplamiento.
- ✅ Código más legible y comprensible.

---

#### 2. ¿Qué es la arquitectura en capas (Layered Architecture)?

**Respuesta:**

La **arquitectura en capas** separa la aplicación en niveles con responsabilidades específicas. Cada capa solo puede comunicarse con la capa inmediatamente inferior.

**Capas típicas en Spring Boot:**

```
┌─────────────────────────────────┐
│   PRESENTATION LAYER            │  @RestController / @Controller
│   (Controller)                  │  - Recibe HTTP requests
│                                 │  - Valida inputs
│                                 │  - Devuelve HTTP responses
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│   BUSINESS LOGIC LAYER          │  @Service
│   (Service)                     │  - Lógica de negocio
│                                 │  - Transacciones
│                                 │  - Orquesta Repository
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│   PERSISTENCE LAYER             │  @Repository
│   (Repository / DAO)            │  - Acceso a BD
│                                 │  - Queries
│                                 │  - CRUD
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│   DATABASE                      │  PostgreSQL, MySQL, etc.
└─────────────────────────────────┘
```

**Ejemplo práctico:**

```java
// ========== PRESENTATION LAYER ==========
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        // ✅ Solo valida y delega al Service
        return ResponseEntity.ok(userService.findById(id));
    }
    
    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserDTO dto) {
        // ✅ Validación con @Valid
        UserDTO created = userService.createUser(dto);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
}

// ========== BUSINESS LOGIC LAYER ==========
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
    
    public UserDTO findById(Long id) {
        // ✅ Lógica de negocio
        return userRepository.findById(id)
            .map(this::toDTO)
            .orElseThrow(() -> new ResourceNotFoundException("Usuario no encontrado"));
    }
    
    @Transactional
    public UserDTO createUser(CreateUserDTO dto) {
        // ✅ Validación de negocio
        if (userRepository.existsByEmail(dto.getEmail())) {
            throw new DuplicateResourceException("Email ya registrado");
        }
        
        // ✅ Orquestación
        User user = toEntity(dto);
        user.setPassword(passwordEncoder.encode(dto.getPassword()));
        user = userRepository.save(user);
        
        // ✅ Envío de email (operación de negocio)
        emailService.sendWelcomeEmail(user);
        
        return toDTO(user);
    }
}

// ========== PERSISTENCE LAYER ==========
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // ✅ Solo acceso a datos
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
    
    @Query("SELECT u FROM User u JOIN FETCH u.roles WHERE u.id = :id")
    Optional<User> findByIdWithRoles(@Param("id") Long id);
}
```

**Reglas de comunicación:**

```
Controller → Service ✅
Controller → Repository ❌ (salta capa de negocio)
Service → Repository ✅
Service → Service ✅ (mismo nivel)
Repository → Service ❌ (capa inferior no llama a superior)
```

**Beneficios:**
- ✅ **Separación de responsabilidades**: Cada capa tiene una función clara.
- ✅ **Mantenibilidad**: Cambios en una capa no afectan a otras.
- ✅ **Testabilidad**: Puedes testear cada capa por separado.
- ✅ **Escalabilidad**: Puedes reemplazar capas (ej: cambiar BD).

**Desventajas:**
- ❌ Puede agregar complejidad innecesaria en apps pequeñas.
- ❌ Puede llevar a "Anemic Domain Model" (entidades sin lógica).

---

### 🔹 Testing

#### 3. ¿Cuál es la diferencia entre Test Unitario y Test de Integración?

**Respuesta:**

| Característica | Test Unitario | Test de Integración |
|----------------|---------------|---------------------|
| **Alcance** | Una clase/método aislado | Múltiples componentes juntos |
| **Dependencias** | Mockeadas | Reales o parcialmente reales |
| **Velocidad** | Muy rápido (milisegundos) | Más lento (segundos) |
| **Contexto Spring** | No se levanta | Se levanta (o parcialmente) |
| **BD** | Mockeada | BD real (H2, Testcontainers) |
| **Herramientas** | JUnit + Mockito | @SpringBootTest, Testcontainers |
| **Cantidad** | Muchos (70-80%) | Menos (20-30%) |

**Test Unitario:**

```java
// Testea SOLO UserService, mockea Repository
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;  // Mock (no real)
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;  // Clase bajo test
    
    @Test
    void findById_UserExists_ReturnsUser() {
        // Given
        Long userId = 1L;
        User user = new User(userId, "Juan", "juan@mail.com");
        when(userRepository.findById(userId)).thenReturn(Optional.of(user));
        
        // When
        UserDTO result = userService.findById(userId);
        
        // Then
        assertThat(result.getId()).isEqualTo(userId);
        assertThat(result.getNombre()).isEqualTo("Juan");
        verify(userRepository).findById(userId);  // Verificar interacción
    }
    
    @Test
    void findById_UserNotExists_ThrowsException() {
        // Given
        Long userId = 999L;
        when(userRepository.findById(userId)).thenReturn(Optional.empty());
        
        // When & Then
        assertThatThrownBy(() -> userService.findById(userId))
            .isInstanceOf(ResourceNotFoundException.class)
            .hasMessageContaining("Usuario no encontrado");
    }
    
    @Test
    void createUser_EmailDuplicate_ThrowsException() {
        // Given
        CreateUserDTO dto = new CreateUserDTO("test@mail.com", "password");
        when(userRepository.existsByEmail(dto.getEmail())).thenReturn(true);
        
        // When & Then
        assertThatThrownBy(() -> userService.createUser(dto))
            .isInstanceOf(DuplicateResourceException.class);
        
        // Verifica que NO se intentó guardar
        verify(userRepository, never()).save(any());
    }
}
```

**Test de Integración:**

```java
// Testea Controller + Service + Repository + BD real
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers  // BD real en Docker
class UserControllerIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb");
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    @BeforeEach
    void setUp() {
        userRepository.deleteAll();  // Limpia BD antes de cada test
    }
    
    @Test
    void createUser_ValidData_ReturnsCreated() {
        // Given
        CreateUserDTO dto = new CreateUserDTO("test@mail.com", "password123");
        
        // When
        ResponseEntity<UserDTO> response = restTemplate.postForEntity(
            "/api/users",
            dto,
            UserDTO.class
        );
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody().getEmail()).isEqualTo("test@mail.com");
        
        // Verifica que se guardó en BD real
        List<User> users = userRepository.findAll();
        assertThat(users).hasSize(1);
        assertThat(users.get(0).getEmail()).isEqualTo("test@mail.com");
    }
    
    @Test
    void getUser_UserExists_ReturnsUser() {
        // Given - Prepara datos en BD real
        User user = userRepository.save(new User("Juan", "juan@mail.com"));
        
        // When
        ResponseEntity<UserDTO> response = restTemplate.getForEntity(
            "/api/users/" + user.getId(),
            UserDTO.class
        );
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody().getNombre()).isEqualTo("Juan");
    }
    
    @Test
    void getUser_UserNotExists_ReturnsNotFound() {
        // When
        ResponseEntity<UserDTO> response = restTemplate.getForEntity(
            "/api/users/999",
            UserDTO.class
        );
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
    }
}
```

**Test de Repository (más específico):**

```java
@DataJpaTest  // Solo levanta JPA, más rápido que @SpringBootTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class UserRepositoryTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void findByEmail_UserExists_ReturnsUser() {
        // Given
        User user = new User("Juan", "juan@mail.com");
        userRepository.save(user);
        
        // When
        Optional<User> found = userRepository.findByEmail("juan@mail.com");
        
        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getNombre()).isEqualTo("Juan");
    }
}
```

**Pirámide de Testing:**

```
         /\
        /  \       E2E Tests (muy pocos)
       /____\
      /      \
     / Integration \    Integration Tests (algunos)
    /___________\
   /             \
  /   Unit Tests  \     Unit Tests (muchos)
 /_________________\
```

**Best Practices:**
- ✅ Escribe MÁS tests unitarios (rápidos, baratos).
- ✅ Escribe ALGUNOS tests de integración (cubren flujos críticos).
- ✅ Usa Testcontainers para BD real en tests de integración.
- ✅ Nombra tests descriptivamente: `metodo_condicion_resultado()`.
- ✅ AAA pattern: Arrange (Given), Act (When), Assert (Then).

---

#### 4. ¿Qué es TDD (Test Driven Development)?

**Respuesta:**

**TDD** es una metodología de desarrollo donde escribes el **test ANTES que el código de producción**.

**Ciclo Red-Green-Refactor:**

```
1. 🔴 RED: Escribe un test que falla (no existe código aún)
   ↓
2. 🟢 GREEN: Escribe el código MÍNIMO para que pase el test
   ↓
3. 🔵 REFACTOR: Mejora el código manteniendo los tests en verde
   ↓
Repite
```

**Ejemplo práctico:**

**Paso 1: RED (escribir test que falla)**
```java
@Test
void calculate_TwoNumbers_ReturnsSum() {
    // Given
    Calculator calculator = new Calculator();
    
    // When
    int result = calculator.add(2, 3);
    
    // Then
    assertThat(result).isEqualTo(5);
}

// ❌ Test falla: Calculator no existe
```

**Paso 2: GREEN (código mínimo para pasar)**
```java
public class Calculator {
    public int add(int a, int b) {
        return 5;  // ✅ Hardcoded, pero el test pasa
    }
}
```

**Paso 3: Añadir más tests (RED)**
```java
@Test
void calculate_DifferentNumbers_ReturnsCorrectSum() {
    Calculator calculator = new Calculator();
    assertThat(calculator.add(10, 20)).isEqualTo(30);  // ❌ Falla
}
```

**Paso 4: GREEN (implementación real)**
```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;  // ✅ Implementación real
    }
}
```

**Paso 5: REFACTOR (si es necesario)**
```java
// Refactorizar sin cambiar comportamiento
// Eliminar duplicación, mejorar nombres, etc.
```

**Beneficios:**
- ✅ **Cobertura de tests** al 100% (escribes tests para todo).
- ✅ **Diseño mejorado**: Pensar en la API antes de implementar.
- ✅ **Regresiones detectadas**: Tests existentes evitan romper funcionalidad.
- ✅ **Confianza**: Puedes refactorizar sin miedo.

**Desventajas:**
- ❌ Más lento inicialmente.
- ❌ Curva de aprendizaje.
- ❌ Puede llevar a over-engineering en casos simples.

---

### 🔹 Seguridad

#### 5. ¿Cuál es la diferencia entre Autenticación y Autorización?

**Respuesta:**

| Concepto | Definición | Pregunta | Ejemplo |
|----------|------------|----------|---------|
| **Autenticación (AuthN)** | Verificar **identidad** | ¿Quién eres? | Login con usuario/contraseña |
| **Autorización (AuthZ)** | Verificar **permisos** | ¿Qué puedes hacer? | Usuario tiene rol ADMIN |

**Flujo típico:**

```
1. Autenticación: Usuario ingresa credenciales → Sistema verifica → Emite token JWT
2. Autorización: Usuario intenta acceder a /admin → Sistema verifica rol en token → Permitir/Denegar
```

**Ejemplo en Spring Security:**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()  // Sin autenticación
                .requestMatchers("/api/users/**").authenticated()  // Requiere autenticación
                .requestMatchers("/api/admin/**").hasRole("ADMIN")  // Requiere rol ADMIN (autorización)
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults())  // Autenticación
            .logout(Customizer.withDefaults());
        
        return http.build();
    }
}

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping("/me")
    @PreAuthorize("isAuthenticated()")  // Autenticación
    public UserDTO getCurrentUser(Authentication auth) {
        // auth contiene información del usuario autenticado
        return userService.findByUsername(auth.getName());
    }
    
    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")  // Autorización
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

**Códigos HTTP:**
- **401 Unauthorized**: Autenticación falla (no estás identificado).
- **403 Forbidden**: Autorización falla (estás identificado, pero sin permisos).

---

#### 6. ¿Qué es JWT y cómo funciona?

**Respuesta:**

**JWT (JSON Web Token)** es un estándar (RFC 7519) para transmitir información de forma segura entre dos partes como un objeto JSON. Es **stateless** (el servidor no guarda sesión).

**Estructura de un JWT:**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

[HEADER].[PAYLOAD].[SIGNATURE]
```

**1. Header (cabecera):**
```json
{
  "alg": "HS256",     // Algoritmo de firma
  "typ": "JWT"        // Tipo de token
}
```

**2. Payload (datos):**
```json
{
  "sub": "1234567890",           // Subject (user ID)
  "name": "John Doe",             // Claims personalizados
  "email": "john@mail.com",
  "roles": ["USER", "ADMIN"],
  "iat": 1516239022,              // Issued At (timestamp)
  "exp": 1516242622               // Expiration (timestamp)
}
```

**3. Signature (firma):**
```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret_key
)
```

**Flujo de autenticación con JWT:**

```
1. Cliente: POST /login { username, password }
   ↓
2. Servidor: Verifica credenciales
   ↓
3. Servidor: Genera JWT y lo devuelve
   ← { "token": "eyJhbGci..." }
   ↓
4. Cliente: Guarda token (localStorage, cookie)
   ↓
5. Cliente: GET /api/users/me
           Header: Authorization: Bearer eyJhbGci...
   ↓
6. Servidor: Valida firma del token
   ↓
7. Servidor: Lee claims del token (user ID, roles)
   ↓
8. Servidor: Devuelve datos
   ← { "id": 1, "name": "John" }
```

**Implementación en Spring:**

```java
@Service
public class JwtService {
    
    @Value("${jwt.secret}")
    private String secret;
    
    @Value("${jwt.expiration}")
    private long expiration;
    
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", userDetails.getAuthorities());
        
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(SignatureAlgorithm.HS256, secret)
            .compact();
    }
    
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }
    
    public boolean isTokenValid(String token, UserDetails userDetails) {
        String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }
    
    private boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }
    
    private Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }
    
    private <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        Claims claims = Jwts.parser()
            .setSigningKey(secret)
            .parseClaimsJws(token)
            .getBody();
        return claimsResolver.apply(claims);
    }
}

@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;
    
    @Override
    protected void doFilterInternal(
        HttpServletRequest request,
        HttpServletResponse response,
        FilterChain filterChain
    ) throws ServletException, IOException {
        
        String authHeader = request.getHeader("Authorization");
        
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }
        
        String jwt = authHeader.substring(7);
        String username = jwtService.extractUsername(jwt);
        
        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            
            if (jwtService.isTokenValid(jwt, userDetails)) {
                UsernamePasswordAuthenticationToken authToken = 
                    new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                    );
                
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        
        filterChain.doFilter(request, response);
    }
}
```

**Ventajas:**
- ✅ **Stateless**: Servidor no guarda sesión (escalable horizontalmente).
- ✅ **Portabilidad**: Funciona entre dominios (CORS).
- ✅ **Auto-contenido**: Toda la información está en el token.

**Desventajas:**
- ❌ **No se puede revocar**: Una vez emitido, es válido hasta que expire.
- ❌ **Tamaño**: Más grande que un session ID simple.
- ❌ **Seguridad**: Si roban el token, tienen acceso hasta que expire.

**Best Practices:**
- ✅ Usa HTTPS siempre.
- ✅ Tiempo de expiración corto (15-60 minutos).
- ✅ Implementa refresh tokens para renovar.
- ✅ No guardes información sensible en el payload (es visible en base64).
- ✅ Usa algoritmos fuertes (RS256 mejor que HS256 para producción).

> 🔗 **Ver también:** [Implementación de seguridad con Spring Security](./02-spring-boot.md#-spring-core) | [Autenticación vs Autorización](#5-cuál-es-la-diferencia-entre-autenticación-y-autorización)

---

#### 7. ¿Cuál es la diferencia entre OAuth2, OpenID Connect (OIDC) y JWT?

**Respuesta:**

Esta es una de las **confusiones más comunes** en seguridad. Son conceptos relacionados pero con propósitos diferentes:

> 🔗 **Conceptos previos:** [JWT básico](#6-qué-es-jwt-y-cómo-funciona) | [Autenticación vs Autorización](#5-cuál-es-la-diferencia-entre-autenticación-y-autorización)

| Concepto | Tipo | Propósito | Pregunta que responde |
|----------|------|-----------|----------------------|
| **OAuth 2.0** | Protocolo de **Autorización** | Delegar acceso a recursos | ¿Qué permisos tiene este usuario? |
| **OpenID Connect (OIDC)** | Protocolo de **Autenticación** (capa sobre OAuth2) | Verificar identidad de usuarios | ¿Quién es este usuario? |
| **JWT** | Formato de **Token** | Transportar información de forma segura | ¿Cómo empaqueto los datos? |

---

**1. OAuth 2.0 - Protocolo de Autorización**

**Definición:** Permite que una aplicación acceda a recursos de un usuario en otro servicio **sin compartir contraseñas**.

**Caso de uso clásico:** "Permitir que una app de terceros acceda a tu Google Drive"

```
Usuario: "Quiero que FotoApp suba mis fotos a mi Google Drive"

1. FotoApp redirige al usuario a Google
2. Google pregunta: "¿Quieres que FotoApp acceda a tu Drive?" → Usuario acepta
3. Google devuelve un ACCESS TOKEN a FotoApp
4. FotoApp usa ese token para subir fotos a Google Drive (sin conocer tu contraseña)
```

**Flujo OAuth 2.0 (Authorization Code):**

```
┌────────┐                                           ┌───────────┐
│ Client │                                           │   OAuth   │
│  App   │                                           │  Server   │
└───┬────┘                                           └─────┬─────┘
    │                                                      │
    │  1. Usuario hace clic en "Login con Google"         │
    │─────────────────────────────────────────────────────>│
    │                                                      │
    │  2. Redirige a página de autorización               │
    │<─────────────────────────────────────────────────────│
    │                                                      │
    │  3. Usuario acepta permisos                         │
    │─────────────────────────────────────────────────────>│
    │                                                      │
    │  4. Devuelve AUTHORIZATION CODE                     │
    │<─────────────────────────────────────────────────────│
    │                                                      │
    │  5. Intercambia código por ACCESS TOKEN             │
    │─────────────────────────────────────────────────────>│
    │                                                      │
    │  6. Devuelve ACCESS TOKEN                           │
    │<─────────────────────────────────────────────────────│
    │                                                      │
    │  7. Usa token para acceder a API protegida          │
    │─────────────────────────────────────────────────────>│
```

**OAuth 2.0 NO te dice QUIÉN es el usuario**, solo te da permisos para acceder a recursos.

**Ejemplo en Spring Boot:**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .oauth2Login(Customizer.withDefaults())  // OAuth2 login
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(Customizer.withDefaults())  // Valida JWT del access token
            );
        return http.build();
    }
}
```

---

**2. OpenID Connect (OIDC) - Capa de Autenticación sobre OAuth2**

**Definición:** Extensión de OAuth 2.0 que **añade autenticación** (identidad del usuario). Responde "¿Quién eres?"

**Diferencia clave:** Además del ACCESS TOKEN, devuelve un **ID TOKEN** (formato JWT) con información del usuario.

```
OAuth 2.0:  Solo ACCESS TOKEN (permisos)
OIDC:       ACCESS TOKEN + ID TOKEN (permisos + identidad)
```

**Flujo OIDC (OAuth2 + Identity):**

```
1. Usuario hace clic en "Login con Google"
2. Google autentica al usuario
3. Google devuelve:
   - ACCESS TOKEN (para acceder a APIs de Google)
   - ID TOKEN (JWT con info del usuario: email, nombre, foto)
```

**Contenido del ID TOKEN (JWT):**

```json
{
  "iss": "https://accounts.google.com",
  "sub": "110169484474386276334",
  "email": "juan@example.com",
  "name": "Juan Pérez",
  "picture": "https://lh3.googleusercontent.com/a/...",
  "iat": 1516239022,
  "exp": 1516242622
}
```

**Ejemplo en Spring Boot (Login con Google usando OIDC):**

```yaml
# application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: YOUR_CLIENT_ID
            client-secret: YOUR_SECRET
            scope:
              - openid      # ← Esto activa OIDC
              - email
              - profile
```

```java
@RestController
public class UserController {
    
    @GetMapping("/user-info")
    public Map<String, Object> getUserInfo(@AuthenticationPrincipal OidcUser principal) {
        // El ID TOKEN viene automáticamente parseado
        return Map.of(
            "name", principal.getFullName(),
            "email", principal.getEmail(),
            "picture", principal.getPicture(),
            "claims", principal.getClaims()  // Todos los claims del ID TOKEN
        );
    }
}
```

---

**3. JWT (JSON Web Token) - Formato de Token**

**Definición:** Formato estándar para representar **claims** (afirmaciones) de forma segura entre dos partes. **No es un protocolo**, es solo un formato.

**Estructura:** `header.payload.signature` (3 partes en Base64URL)

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**JWT puede ser usado como:**
- **ID TOKEN** en OIDC (para identidad)
- **ACCESS TOKEN** en OAuth 2.0 (para permisos)
- **Tokens personalizados** en tu propia aplicación

**Ejemplo de JWT personalizado en Spring Boot:**

```java
@Service
public class JwtService {
    
    private final String SECRET_KEY = "mi-clave-secreta-super-larga-y-segura-2025";
    
    public String generateToken(String username, List<String> roles) {
        return Jwts.builder()
            .setSubject(username)
            .claim("roles", roles)
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + 3600000)) // 1 hora
            .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
            .compact();
    }
    
    public Claims parseToken(String token) {
        return Jwts.parser()
            .setSigningKey(SECRET_KEY)
            .parseClaimsJws(token)
            .getBody();
    }
}
```

---

**Comparación clara: OAuth2 vs OIDC vs JWT**

**Escenario 1: "Quiero que mi app acceda a Google Drive del usuario"**
- Usa: **OAuth 2.0**
- Obtienes: **Access Token** (puede ser JWT o no)
- Propósito: Autorización (permisos)

**Escenario 2: "Quiero hacer login con Google en mi app"**
- Usa: **OpenID Connect** (OIDC)
- Obtienes: **ID Token** (JWT) + Access Token
- Propósito: Autenticación (identidad)

**Escenario 3: "Quiero mi propio sistema de autenticación con tokens"**
- Usa: **JWT** (tu propio formato)
- Generas: Tokens JWT personalizados
- Propósito: Transportar información de sesión

---

**Ejemplo real: Login con Google (OIDC completo)**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/login**", "/error").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .defaultSuccessUrl("/dashboard", true)
            );
        return http.build();
    }
}

@RestController
public class DashboardController {
    
    @GetMapping("/dashboard")
    public ResponseEntity<Map<String, Object>> dashboard(@AuthenticationPrincipal OidcUser user) {
        
        // ID TOKEN (JWT) - Información del usuario
        String idToken = user.getIdToken().getTokenValue();
        Map<String, Object> idClaims = user.getClaims();  // email, name, picture, etc.
        
        // ACCESS TOKEN - Para llamar a APIs de Google
        String accessToken = user.getOAuth2Token().getTokenValue();
        
        return ResponseEntity.ok(Map.of(
            "message", "Bienvenido " + user.getFullName(),
            "email", user.getEmail(),
            "picture", user.getPicture(),
            "idToken", idToken,  // JWT con tu identidad
            "accessToken", accessToken  // Token para Google APIs
        ));
    }
}
```

---

**Confusión más común: "¿Es OAuth2 lo mismo que JWT?"**

❌ **NO**. OAuth2 es un **protocolo** (conjunto de reglas). JWT es un **formato** (estructura de datos).

```
Analogía:
- OAuth2 = HTTP (protocolo de comunicación)
- JWT = JSON (formato de datos)
```

OAuth2 **puede usar** JWT como formato de Access Token, pero también puede usar tokens opacos (random strings).

---

**Resumen ejecutivo:**

| Pregunta | Respuesta |
|----------|-----------|
| ¿Cómo delego permisos sin compartir mi contraseña? | **OAuth 2.0** |
| ¿Cómo autentico usuarios con Google/GitHub/Microsoft? | **OpenID Connect (OIDC)** |
| ¿Qué formato uso para mis tokens? | **JWT** |

**Stack recomendado para 2025:**
- **OIDC** para login social (Google, GitHub, Microsoft)
- **JWT** como formato de tokens
- **OAuth 2.0** para delegar permisos entre tus propios microservicios

```java
// Spring Boot 3+ con Spring Security 6
@Configuration
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            // OIDC para login de usuarios
            .oauth2Login(Customizer.withDefaults())
            
            // JWT para validar tokens en APIs
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(Customizer.withDefaults())
            )
            
            .build();
    }
}
```

---

### 🔹 DevOps Básico

#### 8. ¿Qué es Docker y por qué usarlo?

**Respuesta:**

**Docker** es una plataforma de **containerización** que empaqueta una aplicación con todas sus dependencias en una unidad portable e inmutable llamada **contenedor**.

**Analogía:** Un contenedor Docker es como un contenedor de barco: puedes mover tu app (el contenido) entre ambientes (puertos) sin problemas, porque todo lo necesario está dentro del contenedor.

**Problema que resuelve:**

```
Desarrollador: "Funciona en mi máquina" 🤷
Ops: "No funciona en producción" 😠

Razones:
- Versión de Java diferente (local: Java 17, prod: Java 11)
- Dependencias del SO (librerías nativas)
- Variables de entorno
- Configuraciones del sistema
```

**Solución con Docker:**

```
Docker: "Funciona en todos lados" ✅

Todo empaquetado:
✅ Aplicación (JAR)
✅ Runtime (JDK 17)
✅ Librerías del SO
✅ Variables de entorno
✅ Configuración
```

**Conceptos clave:**

**1. Imagen (template inmutable):**
Plantilla de solo lectura que contiene todo lo necesario.

**2. Contenedor (instancia en ejecución):**
Instancia ejecutable de una imagen.

```
Imagen : Contenedor
Clase  : Objeto
```

**3. Dockerfile (receta):**
Archivo de texto con instrucciones para construir una imagen.

**Ejemplo: Dockerizar app Spring Boot**

```dockerfile
# Dockerfile
# Etapa 1: Build (Maven)
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Etapa 2: Runtime (solo JRE, más ligero)
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/myapp-0.0.1-SNAPSHOT.jar app.jar

# Usuario no-root (seguridad)
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

# Puerto
EXPOSE 8080

# Comando de inicio
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Comandos básicos:**

```bash
# Construir imagen
docker build -t myapp:1.0 .

# Ver imágenes
docker images

# Ejecutar contenedor
docker run -d -p 8080:8080 --name myapp-container myapp:1.0
# -d: detached (background)
# -p: mapeo de puertos (host:container)
# --name: nombre del contenedor

# Ver contenedores en ejecución
docker ps

# Ver logs
docker logs myapp-container

# Detener contenedor
docker stop myapp-container

# Eliminar contenedor
docker rm myapp-container

# Entrar al contenedor
docker exec -it myapp-container /bin/sh
```

**Docker Compose (múltiples servicios):**

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/mydb
      SPRING_DATASOURCE_USERNAME: user
      SPRING_DATASOURCE_PASSWORD: password
    depends_on:
      - db
    networks:
      - mynetwork

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - mynetwork

volumes:
  postgres-data:

networks:
  mynetwork:
```

```bash
# Iniciar todos los servicios
docker-compose up -d

# Detener todos
docker-compose down

# Ver logs
docker-compose logs -f app
```

**Ventajas:**
- ✅ **Portabilidad**: "Funciona en mi máquina" = "Funciona en todas las máquinas".
- ✅ **Aislamiento**: Cada contenedor es independiente.
- ✅ **Eficiencia**: Más ligero que VMs (comparten kernel del SO).
- ✅ **Escalabilidad**: Fácil replicar contenedores.
- ✅ **Versionado**: Imágenes versionadas (rollback fácil).

---

#### 9. ¿Qué es CI/CD?

**Respuesta:**

**CI/CD** es una práctica de DevOps que automatiza la integración y entrega de código.

**CI - Continuous Integration (Integración Continua):**
- Desarrolladores hacen **push frecuente** de código (varias veces al día).
- Un servidor (Jenkins, GitHub Actions, GitLab CI) automáticamente:
  1. **Compila** el código.
  2. **Ejecuta tests** unitarios e integración.
  3. **Reporta** fallos inmediatamente.

**CD - Continuous Delivery/Deployment:**
- **Continuous Delivery**: El código está siempre listo para producción, pero deploy es manual.
- **Continuous Deployment**: El código se despliega automáticamente a producción si pasa todos los tests.

**Pipeline típico:**

```
1. Developer → git push
   ↓
2. CI Server detecta cambio
   ↓
3. Checkout código
   ↓
4. Build (mvn clean install)
   ↓
5. Run Tests (unit + integration)
   ↓
6. Static Analysis (SonarQube, Checkstyle)
   ↓
7. Build Docker Image
   ↓
8. Push to Docker Registry
   ↓
9. Deploy to Staging
   ↓
10. Smoke Tests
   ↓
11. (Manual approval)
   ↓
12. Deploy to Production
```

**Ejemplo: GitHub Actions**

```.yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
    
    - name: Cache Maven packages
      uses: actions/cache@v3
      with:
        path: ~/.m2
        key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
    
    - name: Build with Maven
      run: mvn clean install
    
    - name: Run tests
      run: mvn test
    
    - name: Run integration tests
      run: mvn verify
    
    - name: Build Docker image
      run: docker build -t myapp:${{ github.sha }} .
    
    - name: Push to Docker Hub
      if: github.ref == 'refs/heads/main'
      run: |
        echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
        docker push myapp:${{ github.sha }}
    
    - name: Deploy to production
      if: github.ref == 'refs/heads/main'
      run: |
        # Deploy script (kubectl, AWS, etc.)
        echo "Deploying to production..."
```

**Beneficios:**
- ✅ **Detección temprana de errores**: Tests automáticos atrapan bugs rápido.
- ✅ **Deploys más frecuentes**: Menos riesgo por deploy (cambios pequeños).
- ✅ **Menos errores humanos**: Automatización elimina pasos manuales.
- ✅ **Feedback rápido**: Desarrolladores saben inmediatamente si algo falló.

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

[⬅️ Anterior: Base de Datos](./04-database-postgresql.md) | [🏠 Volver al Inicio](./README.md) | [Siguiente: Escenarios y Behavioral ➡️](./06-behavioral.md)