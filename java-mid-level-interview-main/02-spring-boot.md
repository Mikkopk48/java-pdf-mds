## 🌱 Spring Boot y APIs REST

[⬆️ Volver al índice](./README.md)

---

## 📑 Contenidos de esta sección

### 🔹 Spring Core
1. [IoC e Inyección de Dependencias](#1-qué-es-inversión-de-control-ioc-e-inyección-de-dependencias-di)
2. [Estereotipos de Beans](#2-qué-son-los-estereotipos-de-beans-en-spring)
3. [Ciclo de vida de un Bean](#3-cuál-es-el-ciclo-de-vida-de-un-bean-en-spring)
4. [Auto-configuración](#4-qué-es-la-auto-configuración-en-spring-boot)
5. [Starters](#5-qué-son-los-starters-en-spring-boot)
6. [@Value vs @ConfigurationProperties](#6-gestión-de-configuración-value-vs-configurationproperties)
7. [Spring Profiles](#7-qué-son-los-spring-profiles)

### 🔹 APIs REST
8. [Métodos HTTP](#8-cuáles-son-los-métodos-http-y-cuándo-usar-cada-uno)
9. [Códigos de estado HTTP](#9-cuáles-son-los-códigos-de-estado-http-más-importantes)
10. [Versionado de APIs](#10-qué-estrategias-existen-para-versionar-una-api-rest)
11. [DTO Pattern](#11-qué-es-un-dto-y-por-qué-usarlo)
12. [Manejo de errores global](#12-cómo-manejar-errores-globalmente-en-spring-boot)

### 🔹 Testing
13. [Testing Slices](#13-testing-slices-springboottest-vs-webmvctest-vs-datajpatest)

### 🔹 Conceptos Avanzados
14. [Spring AOP y Aspectos](#14-qué-es-spring-aop-y-cómo-crear-aspectos)
15. [Events en Spring](#15-qué-son-los-events-en-spring)
16. [Transacciones avanzadas](#16-transacciones-avanzadas-propagation-isolation)
17. [Caching en Spring](#17-caching-en-spring-boot)
18. [Actuator y Métricas](#18-spring-boot-actuator-y-métricas)
19. [Validaciones personalizadas](#19-validaciones-personalizadas-con-bean-validation)
20. [Spring Security básico](#20-spring-security-configuración-básica)
21. [WebClient vs RestTemplate](#21-webclient-vs-resttemplate)
22. [Async y Scheduling](#22-programación-asíncrona-y-scheduling)
23. [Multi-tenancy](#23-estrategias-de-multi-tenancy)
24. [Spring Boot 3 novedades](#24-novedades-de-spring-boot-3)

---

### 🔹 Spring Core

#### 1. ¿Qué es Inversión de Control (IoC) e Inyección de Dependencias (DI)?

**Respuesta:**

**Inversión de Control (IoC)**:
Es un principio de diseño donde el **control del flujo del programa** pasa del desarrollador al framework. En lugar de que tu código instancie objetos (`new Service()`), el framework (Spring) lo hace por ti.

- **Tradicional**: Tu código controla la creación de objetos.
- **IoC**: El framework crea y gestiona el ciclo de vida de los objetos.

**Inyección de Dependencias (DI)**:
Es el **patrón principal** para implementar IoC. Los objetos reciben sus dependencias desde el exterior (usualmente por constructor) en lugar de crearlas ellos mismos.

**Sin DI:**
```java
// ❌ Alto acoplamiento, difícil de testear
public class UserController {
    private UserService userService = new UserService();  // Dependencia hardcodeada
}
```

**Con DI:**
```java
// ✅ Bajo acoplamiento, fácil de testear
@RestController
public class UserController {
    private final UserService userService;
    
    // Constructor injection (recomendado)
    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```

**Beneficios:**
1. **Desacoplamiento**: Las clases no conocen las implementaciones concretas.
2. **Testabilidad**: Puedes inyectar mocks/stubs en tests unitarios.
3. **Mantenibilidad**: Cambiar una implementación no requiere modificar dependientes.
4. **Reutilización**: Los componentes son más independientes y reutilizables.

**Tipos de Inyección:**

```java
// 1. Constructor injection ✅ RECOMENDADO
@Service
public class UserService {
    private final UserRepository repository;
    
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}

// 2. Field injection ❌ No recomendado (dificulta testing)
@Service
public class UserService {
    @Autowired
    private UserRepository repository;
}

// 3. Setter injection (solo para dependencias opcionales)
@Service
public class UserService {
    private UserRepository repository;
    
    @Autowired
    public void setRepository(UserRepository repository) {
        this.repository = repository;
    }
}
```

**Best Practice:** Usa **constructor injection** con campos `final`. Spring lo detecta automáticamente desde Spring 4.3+.

---

#### 2. ¿Qué son los estereotipos de Beans en Spring?

**Respuesta:**

Los **estereotipos** son anotaciones que marcan clases para que Spring las registre automáticamente como **Beans** en el `ApplicationContext` mediante component scanning.

| Anotación | Capa | Propósito |
|-----------|------|-----------|
| **@Component** | Genérico | Bean genérico sin semántica específica |
| **@Service** | Negocio | Contiene lógica de negocio |
| **@Repository** | Datos | Acceso a datos (traduce excepciones SQL) |
| **@Controller** | Web | Controlador MVC (devuelve vistas) |
| **@RestController** | Web | API REST (`@Controller` + `@ResponseBody`) |
| **@Configuration** | Config | Declara Beans con `@Bean` |

**Características:**

**@Component** - Estereotipo genérico:
```java
@Component
public class EmailValidator {
    public boolean isValid(String email) {
        return email.contains("@");
    }
}
```

**@Service** - Lógica de negocio:
```java
@Service
public class UserService {
    private final UserRepository repository;
    
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
    
    @Transactional
    public User createUser(UserDTO dto) {
        // Lógica de negocio
        return repository.save(mapToEntity(dto));
    }
}
```

**@Repository** - Acceso a datos:
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}

// Traduce SQLExceptions a DataAccessException
```

**@RestController** - API REST:
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService service;
    
    public UserController(UserService service) {
        this.service = service;
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(service.findById(id));
    }
}
```

**¿Por qué usar específicos y no solo @Component?**
1. **Semántica clara**: El código es más legible.
2. **AOP automático**: `@Repository` traduce excepciones, `@Transactional` funciona mejor en `@Service`.
3. **Tooling**: IDEs y frameworks entienden mejor las capas.

---

#### 3. ¿Cuál es el ciclo de vida de un Bean en Spring?

**Respuesta:**

El **ciclo de vida de un Bean** son las fases que atraviesa desde su creación hasta su destrucción:

**Fases del ciclo de vida:**

1. **Instanciación**: Spring llama al constructor de la clase.
2. **Población de propiedades**: Spring inyecta las dependencias (DI).
3. **BeanNameAware / BeanFactoryAware**: Si implementa estas interfaces, Spring les pasa el nombre/factory del bean.
4. **@PostConstruct** o `InitializingBean`: Método de inicialización custom.
5. **Bean listo**: El bean está completamente configurado y listo para usar.
6. **Uso de la aplicación**: El bean es usado por otros componentes.
7. **@PreDestroy** o `DisposableBean`: Método de destrucción antes de cerrar el contexto.

**Ejemplo práctico:**

```java
@Component
public class UserService {
    private final UserRepository repository;
    private EmailValidator validator;
    
    // 1. Constructor (instanciación)
    public UserService(UserRepository repository) {
        System.out.println("1. Constructor llamado");
        this.repository = repository;
    }
    
    // 2. Setter injection (población de propiedades)
    @Autowired
    public void setValidator(EmailValidator validator) {
        System.out.println("2. Dependencias inyectadas");
        this.validator = validator;
    }
    
    // 4. Inicialización (post-construcción)
    @PostConstruct
    public void init() {
        System.out.println("3. @PostConstruct: Bean inicializado");
        // Aquí ya tienes todas las dependencias inyectadas
        // Ideal para: validaciones, conexiones, caché
    }
    
    // 7. Destrucción
    @PreDestroy
    public void cleanup() {
        System.out.println("4. @PreDestroy: Bean destruido");
        // Liberar recursos: cerrar conexiones, archivos, etc.
    }
}
```

**¿Cuándo usar @PostConstruct?**
- Inicializar cachés
- Validar configuración
- Abrir conexiones a recursos externos
- **NO** crear beans (usa @Bean para eso)

**¿Cuándo usar @PreDestroy?**
- Cerrar conexiones a BD/servicios externos
- Liberar recursos (archivos, sockets)
- Guardar estado antes del cierre

**Scopes de Bean:**
```java
@Scope("singleton")    // Default: una instancia por contexto
@Scope("prototype")    // Nueva instancia cada vez que se solicita
@Scope("request")      // Una por request HTTP (solo web)
@Scope("session")      // Una por sesión HTTP (solo web)
```

---

#### 4. ¿Qué es la auto-configuración en Spring Boot?

**Respuesta:**

La **auto-configuración** es el mecanismo "mágico" de Spring Boot que configura automáticamente tu aplicación basándose en:
1. Las **dependencias** presentes en el classpath.
2. Los **beans** que ya has definido.
3. Las **propiedades** de configuración.

**¿Cómo funciona?**

Spring Boot escanea el classpath y aplica configuraciones condicionales:
- Si encuentra `spring-boot-starter-data-jpa` + un driver de BD → Configura `DataSource`, `EntityManagerFactory`, etc.
- Si encuentra `spring-boot-starter-web` → Configura `DispatcherServlet`, Tomcat embebido, Jackson, etc.
- Si encuentra H2 en memoria → Configura una BD en memoria automáticamente.

**Ejemplo:**

```java
// Solo con estas dependencias en pom.xml:
// - spring-boot-starter-web
// - spring-boot-starter-data-jpa
// - h2

@SpringBootApplication
public class MiApp {
    public static void main(String[] args) {
        SpringApplication.run(MiApp.class, args);
    }
}

// Spring Boot automáticamente configura:
// ✅ Tomcat embebido en puerto 8080
// ✅ DataSource con H2 en memoria
// ✅ EntityManagerFactory
// ✅ TransactionManager
// ✅ Jackson para JSON
// ✅ Y mucho más...
```

**Anotación mágica: @SpringBootApplication**
```java
@SpringBootApplication  // Es la combinación de:
// @Configuration        → Clase de configuración
// @EnableAutoConfiguration  → Activa auto-configuración
// @ComponentScan        → Escanea componentes en este paquete y sub-paquetes
```

**Ver qué se auto-configura:**
```bash
# Añade en application.properties:
debug=true

# O ejecuta:
java -jar miapp.jar --debug
```

**Personalizar/Deshabilitar auto-configuración:**

```java
// Excluir configuraciones específicas
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})

// O en application.properties:
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

**Crear tu propia auto-configuración:**
```java
@Configuration
@ConditionalOnClass(MyService.class)  // Solo si MyService está en el classpath
@EnableConfigurationProperties(MyProperties.class)
public class MyAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean  // Solo si el usuario no definió uno
    public MyService myService() {
        return new MyService();
    }
}
```

---

#### 5. ¿Qué son los Starters en Spring Boot?

**Respuesta:**

Los **Starters** son dependencias "paraguas" (umbrella dependencies) que agrupan todas las librerías necesarias para una funcionalidad específica, con versiones compatibles entre sí.

**Beneficios:**
- **Simplificación**: No necesitas buscar qué librerías se necesitan ni sus versiones.
- **Compatibilidad**: Spring Boot garantiza que todas las versiones funcionen juntas.
- **Convención sobre configuración**: Configuración por defecto lista para usar.

**Starters más comunes:**

| Starter | Incluye | Uso |
|---------|---------|-----|
| **spring-boot-starter-web** | Spring MVC, Tomcat, Jackson, Validation | APIs REST, Web MVC |
| **spring-boot-starter-data-jpa** | Hibernate, Spring Data JPA, JDBC | Persistencia con JPA |
| **spring-boot-starter-security** | Spring Security | Autenticación/Autorización |
| **spring-boot-starter-test** | JUnit, Mockito, AssertJ, Hamcrest | Testing |
| **spring-boot-starter-validation** | Hibernate Validator | Validación de beans |
| **spring-boot-starter-actuator** | Actuator (endpoints de métricas) | Monitoreo, health checks |
| **spring-boot-starter-data-mongodb** | Spring Data MongoDB | BD NoSQL MongoDB |
| **spring-boot-starter-data-redis** | Jedis/Lettuce, Spring Data Redis | Caché con Redis |
| **spring-boot-starter-amqp** | RabbitMQ | Mensajería con RabbitMQ |
| **spring-boot-starter-mail** | JavaMail | Envío de emails |
| **spring-boot-starter-thymeleaf** | Thymeleaf | Motor de plantillas |

**Ejemplo: spring-boot-starter-web incluye:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Internamente trae: -->
<!-- - spring-web, spring-webmvc -->
<!-- - tomcat-embed-core, tomcat-embed-el, tomcat-embed-websocket -->
<!-- - jackson-databind (JSON) -->
<!-- - spring-boot-starter (core, logging, etc.) -->
<!-- - hibernate-validator -->
```

**Crear tu propio Starter:**

1. Crear módulo con sufijo `-spring-boot-starter`
2. Incluir `spring-boot-autoconfigure`
3. Crear clase `@Configuration` con `@ConditionalOn...`
4. Crear `META-INF/spring.factories`

---

#### 6. Gestión de configuración: @Value vs @ConfigurationProperties

**Respuesta:**

Ambos permiten inyectar valores de configuración desde archivos `.properties` o `.yml`, pero con diferentes enfoques.

**@Value - Para valores individuales:**

```java
@Component
public class AppConfig {
    @Value("${app.name}")
    private String appName;
    
    @Value("${app.timeout:5000}")  // Con valor por defecto
    private int timeout;
    
    @Value("${app.features}")
    private List<String> features;  // Soporta listas (separadas por coma)
}
```

**application.yml:**
```yaml
app:
  name: MyApp
  timeout: 3000
  features: feature1,feature2,feature3
```

**Desventajas de @Value:**
- ❌ No hay validación de tipos en compile-time (solo runtime).
- ❌ Difícil de testear y mockear.
- ❌ No soporta validación con Bean Validation (`@NotNull`, `@Min`, etc.).
- ❌ Verboso si tienes muchas propiedades relacionadas.

---

**@ConfigurationProperties - Para grupos de configuración:**

```java
@Component
@ConfigurationProperties(prefix = "app")
@Validated  // Habilita validación con Bean Validation
public class AppProperties {
    
    @NotBlank
    private String name;
    
    @Min(1000)
    @Max(30000)
    private int timeout = 5000;  // Valor por defecto
    
    private List<String> features = new ArrayList<>();
    
    private Database database = new Database();
    
    // Getters y Setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public int getTimeout() { return timeout; }
    public void setTimeout(int timeout) { this.timeout = timeout; }
    
    public List<String> getFeatures() { return features; }
    public void setFeatures(List<String> features) { this.features = features; }
    
    public Database getDatabase() { return database; }
    public void setDatabase(Database database) { this.database = database; }
    
    // Clase anidada para propiedades agrupadas
    public static class Database {
        private String url;
        private String username;
        private String password;
        
        // Getters y Setters
        public String getUrl() { return url; }
        public void setUrl(String url) { this.url = url; }
        
        public String getUsername() { return username; }
        public void setUsername(String username) { this.username = username; }
        
        public String getPassword() { return password; }
        public void setPassword(String password) { this.password = password; }
    }
}
```

**application.yml:**
```yaml
app:
  name: MyApp
  timeout: 3000
  features:
    - feature1
    - feature2
    - feature3
  database:
    url: jdbc:postgresql://localhost:5432/mydb
    username: user
    password: pass
```

**Uso en un servicio:**
```java
@Service
public class MyService {
    private final AppProperties appProperties;
    
    public MyService(AppProperties appProperties) {
        this.appProperties = appProperties;
    }
    
    public void doSomething() {
        String appName = appProperties.getName();
        int timeout = appProperties.getTimeout();
        String dbUrl = appProperties.getDatabase().getUrl();
    }
}
```

**Ventajas de @ConfigurationProperties:**
- ✅ **Type-safe**: Errores de tipo en compile-time.
- ✅ **Validación**: Soporta Bean Validation (`@NotNull`, `@Min`, `@Pattern`, etc.).
- ✅ **Organización**: Agrupa propiedades relacionadas.
- ✅ **Testeable**: Fácil de mockear o instanciar con valores de prueba.
- ✅ **IDE support**: Autocompletado en `application.yml` (con annotation processor).
- ✅ **Documentación**: Las propiedades quedan documentadas en la clase.

**Habilitar metadata para autocompletado en IDE:**
```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

**¿Cuándo usar cada uno?**

| Caso | Usar |
|------|------|
| Valor único y simple | `@Value` |
| Múltiples propiedades relacionadas | `@ConfigurationProperties` |
| Necesitas validación | `@ConfigurationProperties` + `@Validated` |
| Configuración de terceros (ej: DB, cache) | `@ConfigurationProperties` |
| Valores dinámicos o SpEL | `@Value` |

**Ejemplo con Records (Java 16+):**
```java
@ConfigurationProperties(prefix = "app")
public record AppProperties(
    @NotBlank String name,
    @Min(1000) int timeout,
    List<String> features
) {}
```

---

#### 7. ¿Qué son los Spring Profiles?

**Respuesta:**

Los **Profiles** permiten separar la configuración de la aplicación por **entornos** (desarrollo, testing, producción). Puedes activar/desactivar beans y propiedades según el perfil activo.

**Uso básico:**

**1. Crear archivos de propiedades por perfil:**
```
src/main/resources/
├── application.yml               # Configuración común
├── application-dev.yml           # Desarrollo
├── application-test.yml          # Testing
└── application-prod.yml          # Producción
```

**application.yml (común):**
```yaml
spring:
  application:
    name: my-app

app:
  name: MyApp
```

**application-dev.yml:**
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb_dev
    username: dev_user
    password: dev_pass
  jpa:
    show-sql: true  # Mostrar SQLs en desarrollo

logging:
  level:
    root: DEBUG
```

**application-prod.yml:**
```yaml
spring:
  datasource:
    url: jdbc:postgresql://prod-server:5432/mydb_prod
    username: prod_user
    password: ${DB_PASSWORD}  # Desde variable de entorno
  jpa:
    show-sql: false

logging:
  level:
    root: WARN
```

**2. Activar un perfil:**

**Opción A: En `application.yml`:**
```yaml
spring:
  profiles:
    active: dev  # Perfil por defecto
```

**Opción B: Variable de entorno:**
```bash
export SPRING_PROFILES_ACTIVE=prod
java -jar myapp.jar
```

**Opción C: Argumento de línea de comandos:**
```bash
java -jar myapp.jar --spring.profiles.active=prod
```

**Opción D: En IDE (IntelliJ):**
```
Run -> Edit Configurations -> Environment Variables:
SPRING_PROFILES_ACTIVE=dev
```

**3. Beans condicionales por perfil:**

```java
@Configuration
public class DataSourceConfig {
    
    @Bean
    @Profile("dev")  // Solo se crea en perfil "dev"
    public DataSource devDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
    
    @Bean
    @Profile("prod")  // Solo se crea en perfil "prod"
    public DataSource prodDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:postgresql://prod-server/mydb");
        return dataSource;
    }
    
    @Bean
    @Profile("!prod")  // En TODOS los perfiles EXCEPTO "prod"
    public DebugLogger debugLogger() {
        return new DebugLogger();
    }
}
```

**4. Múltiples perfiles activos:**

Puedes activar varios perfiles a la vez:
```bash
java -jar myapp.jar --spring.profiles.active=prod,monitoring,cache
```

**5. Perfiles en tests:**

```java
@SpringBootTest
@ActiveProfiles("test")  // Activa perfil "test" solo para este test
class UserServiceTest {
    // Test usa application-test.yml
}
```

**6. Profile groups (Spring Boot 2.4+):**

```yaml
spring:
  profiles:
    group:
      local: dev,debug,h2
      production: prod,monitoring,postgres

# Activar grupo:
# --spring.profiles.active=local
# Activa automáticamente: dev, debug, h2
```

**Casos de uso comunes:**

| Perfil | Configuración típica |
|--------|---------------------|
| **dev** | Base de datos local (H2, PostgreSQL local), logs DEBUG, mock services |
| **test** | Base de datos en memoria (H2), Testcontainers, sin emails reales |
| **staging** | Base de datos staging, logs INFO, servicios staging |
| **prod** | Base de datos producción, logs WARN, servicios reales, caché habilitado |

**Best Practices:**
- ✅ Nunca commitees contraseñas en archivos de propiedades. Usa variables de entorno: `${DB_PASSWORD}`.
- ✅ Usa `application.yml` para configuración común y `application-{profile}.yml` para específica.
- ✅ En producción, inyecta propiedades sensibles desde **secretos** (AWS Secrets Manager, Azure Key Vault, Kubernetes Secrets).
- ✅ Documenta qué perfiles existen y cuándo usarlos en el README.

---

### 🔹 APIs REST

#### 8. ¿Cuáles son los métodos HTTP y cuándo usar cada uno?

**Respuesta:**

| Método | Propósito | Idempotente | Body Request | Body Response | Código Éxito |
|--------|-----------|-------------|--------------|---------------|--------------|
| **GET** | Leer recursos | ✅ Sí | ❌ No | ✅ Sí | 200 OK |
| **POST** | Crear recurso | ❌ No | ✅ Sí | ✅ Sí | 201 Created |
| **PUT** | Reemplazar recurso completo | ✅ Sí | ✅ Sí | ✅ Opcional | 200 OK / 204 |
| **PATCH** | Actualización parcial | ❌ Depende | ✅ Sí | ✅ Opcional | 200 OK / 204 |
| **DELETE** | Eliminar recurso | ✅ Sí | ❌ Opcional | ❌ Opcional | 204 No Content |
| **HEAD** | Como GET sin body | ✅ Sí | ❌ No | ❌ No | 200 OK |
| **OPTIONS** | Opciones permitidas | ✅ Sí | ❌ No | ✅ Sí | 200 OK |

**Idempotente**: Llamar N veces produce el mismo resultado que llamar 1 vez.

**Ejemplos prácticos:**

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    // GET: Leer usuario
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }
    
    // GET: Listar usuarios
    @GetMapping
    public ResponseEntity<List<UserDTO>> getUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size
    ) {
        return ResponseEntity.ok(userService.findAll(page, size));
    }
    
    // POST: Crear usuario
    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody UserDTO dto) {
        UserDTO created = userService.create(dto);
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();
        return ResponseEntity.created(location).body(created);
    }
    
    // PUT: Reemplazar usuario completo
    @PutMapping("/{id}")
    public ResponseEntity<UserDTO> updateUser(
        @PathVariable Long id,
        @Valid @RequestBody UserDTO dto
    ) {
        return ResponseEntity.ok(userService.update(id, dto));
    }
    
    // PATCH: Actualización parcial
    @PatchMapping("/{id}")
    public ResponseEntity<UserDTO> patchUser(
        @PathVariable Long id,
        @RequestBody Map<String, Object> updates
    ) {
        return ResponseEntity.ok(userService.partialUpdate(id, updates));
    }
    
    // DELETE: Eliminar usuario
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

**PUT vs PATCH:**
- **PUT**: Envías el objeto completo, reemplaza todo.
- **PATCH**: Envías solo los campos a modificar.

```json
// PUT /api/users/1 (reemplaza todo, incluye TODOS los campos)
{
  "id": 1,
  "nombre": "Juan",
  "email": "juan@mail.com",
  "edad": 30,
  "ciudad": "Madrid"
}

// PATCH /api/users/1 (solo lo que cambió)
{
  "email": "nuevo@mail.com"
}
```

---

#### 9. ¿Cuáles son los códigos de estado HTTP más importantes?

**Respuesta:**

Los códigos de estado HTTP se dividen en 5 categorías:

**1xx - Informational** (raramente usados):
- `100 Continue`: El servidor recibió los headers, continúa enviando el body.

**2xx - Success:**
| Código | Significado | Cuándo usar |
|--------|-------------|-------------|
| **200 OK** | Éxito general | GET, PUT, PATCH exitosos |
| **201 Created** | Recurso creado | POST exitoso |
| **204 No Content** | Éxito sin body | DELETE, PUT sin respuesta |
| **206 Partial Content** | Contenido parcial | Streaming, descarga resumable |

**3xx - Redirection:**
| Código | Significado | Cuándo usar |
|--------|-------------|-------------|
| **301 Moved Permanently** | Recurso movido permanentemente | Cambio de URL definitivo |
| **302 Found** | Redirección temporal | Login redirect |
| **304 Not Modified** | Usa caché | Caché válida (If-Modified-Since) |

**4xx - Client Error:**
| Código | Significado | Cuándo usar |
|--------|-------------|-------------|
| **400 Bad Request** | Error de sintaxis/validación | JSON inválido, campos faltantes |
| **401 Unauthorized** | No autenticado | Sin token o token inválido |
| **403 Forbidden** | Autenticado pero sin permisos | Usuario sin rol necesario |
| **404 Not Found** | Recurso no existe | ID no encontrado |
| **405 Method Not Allowed** | Método HTTP no soportado | POST en endpoint solo GET |
| **409 Conflict** | Conflicto con estado actual | Email duplicado |
| **422 Unprocessable Entity** | Validación de negocio falla | "Saldo insuficiente" |
| **429 Too Many Requests** | Rate limiting | Demasiadas peticiones |

**5xx - Server Error:**
| Código | Significado | Cuándo usar |
|--------|-------------|-------------|
| **500 Internal Server Error** | Error genérico del servidor | Excepción no controlada |
| **502 Bad Gateway** | Gateway recibió respuesta inválida | Servicio downstream caído |
| **503 Service Unavailable** | Servicio temporalmente no disponible | Mantenimiento |
| **504 Gateway Timeout** | Gateway timeout | Servicio downstream lento |

**Ejemplo en Spring:**

```java
@RestController
public class UserController {
    
    @GetMapping("/users/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        return userService.findById(id)
            .map(user -> ResponseEntity.ok(user))          // 200 OK
            .orElse(ResponseEntity.notFound().build());     // 404 Not Found
    }
    
    @PostMapping("/users")
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody UserDTO dto) {
        if (userService.emailExists(dto.getEmail())) {
            return ResponseEntity.status(HttpStatus.CONFLICT).build();  // 409 Conflict
        }
        UserDTO created = userService.create(dto);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);  // 201 Created
    }
    
    @DeleteMapping("/users/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();  // 204 No Content
    }
}
```

---

#### 10. ¿Qué estrategias existen para versionar una API REST?

**Respuesta:**

El **versionado de APIs** es crucial para **evolucionar tu API sin romper clientes existentes**. Permite introducir cambios mientras mantienes retrocompatibilidad.

**Principales estrategias:**

**1. URI Versioning (path-based) ✅ Más común**

Versión en la URL como parte del path.

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 {
    
    @GetMapping("/{id}")
    public UserDTOV1 getUser(@PathVariable Long id) {
        return userService.findByIdV1(id);
    }
}

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 {
    
    @GetMapping("/{id}")
    public UserDTOV2 getUser(@PathVariable Long id) {
        return userService.findByIdV2(id);
    }
}
```

**Ejemplo de uso:**
```bash
# Versión 1 (antigua)
GET https://api.ejemplo.com/api/v1/users/123

# Versión 2 (nueva)
GET https://api.ejemplo.com/api/v2/users/123
```

**Pros:**
- ✅ Muy claro y explícito
- ✅ Fácil de probar en navegador/Postman
- ✅ Simple de documentar
- ✅ Compatible con caché HTTP
- ✅ Usado por grandes empresas: Twitter, Stripe, GitHub

**Contras:**
- ❌ Duplicación de código (controladores separados)
- ❌ Aumenta el tamaño de la URL
- ❌ No cumple estrictamente REST (un recurso tiene múltiples URIs)

---

**2. Header Versioning (custom header)**

Versión en un header HTTP personalizado.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping(value = "/{id}", headers = "X-API-VERSION=1")
    public UserDTOV1 getUserV1(@PathVariable Long id) {
        return userService.findByIdV1(id);
    }
    
    @GetMapping(value = "/{id}", headers = "X-API-VERSION=2")
    public UserDTOV2 getUserV2(@PathVariable Long id) {
        return userService.findByIdV2(id);
    }
}
```

**Ejemplo de uso:**
```bash
# Versión 1
curl -H "X-API-VERSION: 1" https://api.ejemplo.com/api/users/123

# Versión 2
curl -H "X-API-VERSION: 2" https://api.ejemplo.com/api/users/123
```

**Pros:**
- ✅ URL limpia (mismo recurso, misma URI)
- ✅ RESTful puro (un recurso = una URI)
- ✅ No contamina la URL

**Contras:**
- ❌ Más difícil de probar (necesitas configurar headers)
- ❌ No visible en navegador
- ❌ Requiere documentación explícita
- ❌ Puede no funcionar bien con algunos proxies/CDNs

---

**3. Media Type Versioning (content negotiation)**

Versión en el header `Accept` usando tipos MIME personalizados.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping(value = "/{id}", produces = "application/vnd.company.v1+json")
    public UserDTOV1 getUserV1(@PathVariable Long id) {
        return userService.findByIdV1(id);
    }
    
    @GetMapping(value = "/{id}", produces = "application/vnd.company.v2+json")
    public UserDTOV2 getUserV2(@PathVariable Long id) {
        return userService.findByIdV2(id);
    }
}
```

**Ejemplo de uso:**
```bash
# Versión 1
curl -H "Accept: application/vnd.company.v1+json" https://api.ejemplo.com/api/users/123

# Versión 2
curl -H "Accept: application/vnd.company.v2+json" https://api.ejemplo.com/api/users/123
```

**Pros:**
- ✅ RESTful puro (negociación de contenido estándar HTTP)
- ✅ URL limpia
- ✅ Usado por GitHub API

**Contras:**
- ❌ Más complejo de implementar y entender
- ❌ Difícil de probar manualmente
- ❌ No soportado nativamente por todos los frameworks

---

**4. Query Parameter Versioning**

Versión como parámetro de consulta en la URL.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping("/{id}")
    public ResponseEntity<?> getUser(
        @PathVariable Long id,
        @RequestParam(defaultValue = "1") int version
    ) {
        if (version == 1) {
            return ResponseEntity.ok(userService.findByIdV1(id));
        } else if (version == 2) {
            return ResponseEntity.ok(userService.findByIdV2(id));
        }
        return ResponseEntity.badRequest().build();
    }
}
```

**Ejemplo de uso:**
```bash
# Versión 1
GET https://api.ejemplo.com/api/users/123?version=1

# Versión 2 (o por defecto)
GET https://api.ejemplo.com/api/users/123?version=2
```

**Pros:**
- ✅ Fácil de implementar
- ✅ Fácil de probar
- ✅ Compatible con caché (si se configura bien)

**Contras:**
- ❌ No es estándar
- ❌ Puede contaminar la URL con parámetros innecesarios
- ❌ Menos profesional

---

**Comparación y Recomendaciones:**

| Estrategia | Claridad | RESTful | Facilidad Testing | Usado por |
|------------|----------|---------|-------------------|-----------|
| **URI Versioning** | ⭐⭐⭐ | ⭐ | ⭐⭐⭐ | Twitter, Stripe, AWS |
| **Header Versioning** | ⭐⭐ | ⭐⭐⭐ | ⭐ | Microsoft Graph |
| **Media Type** | ⭐ | ⭐⭐⭐ | ⭐ | GitHub API |
| **Query Parameter** | ⭐⭐ | ⭐ | ⭐⭐ | Netflix (internamente) |

**Recomendación general:**
- **Para APIs públicas y startups:** **URI Versioning** (`/api/v1/`) - Es el más pragmático y claro
- **Para APIs internas:** **Header Versioning** - Mantiene URLs limpias
- **Para APIs muy RESTful:** **Media Type Versioning** - Más ortodoxo

**Buenas prácticas:**

```java
// ✅ Versión en URL (recomendado para la mayoría de casos)
@RestController
@RequestMapping("/api/v{version}/users")
public class UserController {
    
    @GetMapping("/{id}")
    public ResponseEntity<?> getUser(
        @PathVariable String version,
        @PathVariable Long id
    ) {
        return switch (version) {
            case "1" -> ResponseEntity.ok(userService.findByIdV1(id));
            case "2" -> ResponseEntity.ok(userService.findByIdV2(id));
            default -> ResponseEntity.badRequest().build();
        };
    }
}
```

**Estrategia de deprecación:**

```java
@RestController
@RequestMapping("/api/v1/users")
@Deprecated  // Marcar versión antigua
public class UserControllerV1 {
    
    @GetMapping("/{id}")
    public ResponseEntity<UserDTOV1> getUser(@PathVariable Long id) {
        // Añadir header informando de deprecación
        return ResponseEntity.ok()
            .header("X-API-Deprecation", "This version will be removed on 2025-12-31")
            .header("X-API-Deprecation-Info", "https://api.ejemplo.com/docs/migration-v2")
            .body(userService.findByIdV1(id));
    }
}
```

---

#### 11. ¿Qué es un DTO y por qué usarlo?

**Respuesta:**

Un **DTO (Data Transfer Object)** es un objeto plano (POJO) usado **exclusivamente** para transportar datos entre capas (especialmente entre el cliente y el servidor en APIs REST).

**¿Por qué NO exponer Entidades directamente?**

**Problemas de exponer Entities:**
1. **Seguridad**: Expones campos sensibles (`password`, `roles`, `version`).
2. **Lazy Loading**: Puede lanzar `LazyInitializationException`.
3. **Recursión infinita**: Relaciones bidireccionales causan JSON infinito.
4. **Acoplamiento**: El cliente depende de tu modelo de BD.
5. **Sobrecarga**: Envías más datos de los necesarios.
6. **Evolución**: Cambiar la BD rompe la API.

**Comparación:**

```java
// ❌ MAL: Exponer Entity directamente
@Entity
public class User {
    @Id
    private Long id;
    private String nombre;
    private String email;
    private String password;     // ❌ No debe ser visible
    private String passwordSalt; // ❌ Campo interno
    
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Order> orders;  // ❌ Puede causar LazyInitializationException
    
    @ManyToOne
    @JoinColumn(name = "company_id")
    private Company company;     // ❌ Recursión infinita si Company tiene List<User>
}

// ✅ BIEN: Usar DTO
public class UserDTO {
    private Long id;
    private String nombre;
    private String email;
    // Sin password, sin relaciones problemáticas
}

public class UserDetailDTO {
    private Long id;
    private String nombre;
    private String email;
    private String companyName;  // Solo el nombre, no el objeto completo
    private int orderCount;      // Solo el conteo, no la lista
}
```

**Mapeo Entity ↔ DTO:**

```java
// Opción 1: Manual (control total)
@Service
public class UserService {
    public UserDTO toDTO(User entity) {
        UserDTO dto = new UserDTO();
        dto.setId(entity.getId());
        dto.setNombre(entity.getNombre());
        dto.setEmail(entity.getEmail());
        return dto;
    }
    
    public User toEntity(UserDTO dto) {
        User entity = new User();
        entity.setNombre(dto.getNombre());
        entity.setEmail(dto.getEmail());
        return entity;
    }
}

// Opción 2: MapStruct (generación automática en compile-time)
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserDTO toDTO(User entity);
    User toEntity(UserDTO dto);
    
    @Mapping(target = "companyName", source = "company.name")
    UserDetailDTO toDetailDTO(User entity);
}

// Opción 3: ModelMapper (reflexión en runtime, más lento)
@Service
public class UserService {
    private final ModelMapper mapper = new ModelMapper();
    
    public UserDTO toDTO(User entity) {
        return mapper.map(entity, UserDTO.class);
    }
}
```

**Best Practices:**
- ✅ Usa DTOs en la capa de Controller.
- ✅ Usa Entities en la capa de Service y Repository.
- ✅ Mapea en el Service o usa un Mapper dedicado.
- ✅ Valida DTOs con `@Valid` y Bean Validation.

---

#### 12. ¿Cómo manejar errores globalmente en Spring Boot?

**Respuesta:**

El **manejo global de errores** permite capturar excepciones en cualquier controlador y devolver respuestas estandarizadas en lugar del HTML de error por defecto de Tomcat.

**Opción 1: @ControllerAdvice / @RestControllerAdvice**

```java
@RestControllerAdvice  // @ControllerAdvice + @ResponseBody
public class GlobalExceptionHandler {
    
    // Recurso no encontrado
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(ResourceNotFoundException ex) {
        return new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
    }
    
    // Validación fallida
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error -> {
            errors.put(error.getField(), error.getDefaultMessage());
        });
        
        return new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Errores de validación",
            LocalDateTime.now(),
            errors
        );
    }
    
    // Conflicto (ej: email duplicado)
    @ExceptionHandler(DuplicateResourceException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ErrorResponse handleConflict(DuplicateResourceException ex) {
        return new ErrorResponse(
            HttpStatus.CONFLICT.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
    }
    
    // Error genérico (catch-all)
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneric(Exception ex) {
        // Log del error real
        log.error("Error inesperado", ex);
        
        // No expongas el stacktrace al cliente
        return new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "Error interno del servidor",
            LocalDateTime.now()
        );
    }
}

// Clase de respuesta de error
@Data
@AllArgsConstructor
public class ErrorResponse {
    private int status;
    private String message;
    private LocalDateTime timestamp;
    private Map<String, String> errors;  // Opcional para validaciones
    
    public ErrorResponse(int status, String message, LocalDateTime timestamp) {
        this.status = status;
        this.message = message;
        this.timestamp = timestamp;
    }
}
```

**Excepciones custom:**

```java
@ResponseStatus(HttpStatus.NOT_FOUND)  // Opcional
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

@ResponseStatus(HttpStatus.CONFLICT)
public class DuplicateResourceException extends RuntimeException {
    public DuplicateResourceException(String message) {
        super(message);
    }
}
```

**Uso en Service:**

```java
@Service
public class UserService {
    public UserDTO findById(Long id) {
        return userRepository.findById(id)
            .map(this::toDTO)
            .orElseThrow(() -> new ResourceNotFoundException("Usuario no encontrado con ID: " + id));
    }
    
    public UserDTO create(UserDTO dto) {
        if (userRepository.existsByEmail(dto.getEmail())) {
            throw new DuplicateResourceException("El email ya está en uso");
        }
        User user = userRepository.save(toEntity(dto));
        return toDTO(user);
    }
}
```

**Respuesta JSON de error:**

```json
// GET /api/users/999 (no existe)
{
  "status": 404,
  "message": "Usuario no encontrado con ID: 999",
  "timestamp": "2025-12-22T10:30:00"
}

// POST /api/users (validación falla)
{
  "status": 400,
  "message": "Errores de validación",
  "timestamp": "2025-12-22T10:30:00",
  "errors": {
    "email": "Debe ser un email válido",
    "nombre": "No puede estar vacío"
  }
}
```

**Opción 2: ResponseEntityExceptionHandler (más control)**

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(
        MethodArgumentNotValidException ex,
        HttpHeaders headers,
        HttpStatus status,
        WebRequest request
    ) {
        // Tu lógica custom
        ErrorResponse error = new ErrorResponse(/*...*/);
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
}
```

[➡️ Siguiente: JPA y Hibernate](./03-jpa-hibernate.md)

---

### 🔹 Testing en Spring Boot

#### 13. Testing Slices: @SpringBootTest vs @WebMvcTest vs @DataJpaTest

**Respuesta:**

Spring Boot ofrece diferentes **anotaciones de test** que cargan solo partes específicas del contexto de la aplicación, haciendo los tests más **rápidos y enfocados**.

**1. @SpringBootTest - Tests de Integración (Full Context)**

Carga **TODO el contexto** de la aplicación (todos los beans, configuraciones, BD real o embebida).

```java
@SpringBootTest
@AutoConfigureMockMvc  // Si necesitas MockMvc
class UserIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void testCompleteFlow() throws Exception {
        // Test end-to-end: Controller -> Service -> Repository -> DB
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"name\":\"Juan\",\"email\":\"juan@test.com\"}"))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.name").value("Juan"));
        
        // Verificar que se guardó en BD
        User saved = userRepository.findByEmail("juan@test.com").orElseThrow();
        assertThat(saved.getName()).isEqualTo("Juan");
    }
}
```

**Características:**
- ✅ Carga todos los beans (Controller, Service, Repository, DataSource, etc.).
- ✅ Útil para tests end-to-end.
- ❌ **Lento** (puede tardar varios segundos en iniciar).
- ❌ Pesado (consume mucha memoria).

**Cuándo usar:**
- Tests de integración completos.
- Validar flujos end-to-end.
- Tests de smoke (¿arranca la app?).

---

**2. @WebMvcTest - Tests de Controladores (Web Layer)**

Carga **solo la capa web** (Controllers, `@ControllerAdvice`, filtros, pero NO services ni repositories).

```java
@WebMvcTest(UserController.class)  // Solo carga este controller
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean  // Mock del servicio (no se carga el real)
    private UserService userService;
    
    @Test
    void getUserById_Found_ReturnsUser() throws Exception {
        // Given: Mock del servicio
        User user = new User(1L, "Juan", "juan@test.com");
        when(userService.findById(1L)).thenReturn(user);
        
        // When & Then
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("Juan"))
            .andExpect(jsonPath("$.email").value("juan@test.com"));
    }
    
    @Test
    void getUserById_NotFound_Returns404() throws Exception {
        // Given
        when(userService.findById(99L))
            .thenThrow(new ResourceNotFoundException("Usuario no encontrado"));
        
        // When & Then
        mockMvc.perform(get("/api/users/99"))
            .andExpect(status().isNotFound());
    }
}
```

**Características:**
- ✅ **Rápido** (solo carga capa web).
- ✅ Enfocado en lógica HTTP (validaciones, serialización JSON, status codes).
- ✅ Auto-configura `MockMvc`.
- ❌ Requiere `@MockBean` para dependencias.

**Qué se carga:**
- `@RestController`, `@Controller`
- `@RestControllerAdvice`, `@ControllerAdvice`
- Filtros, interceptors
- Serialización/deserialización JSON

**Qué NO se carga:**
- `@Service`, `@Repository`, `@Component`
- DataSource, JPA

**Cuándo usar:**
- Tests unitarios de controllers.
- Validar request/response (JSON, validaciones, status codes).
- Probar `@ControllerAdvice` (manejo de errores).

---

**3. @DataJpaTest - Tests de Repositorios (Persistence Layer)**

Carga **solo la capa de persistencia** (JPA, Hibernate, base de datos embebida).

```java
@DataJpaTest  // Usa BD embebida (H2 por defecto)
class UserRepositoryTest {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private TestEntityManager entityManager;  // Helper para tests
    
    @Test
    void findByEmail_ExistingUser_ReturnsUser() {
        // Given: Insertar datos de prueba
        User user = new User();
        user.setName("Juan");
        user.setEmail("juan@test.com");
        entityManager.persist(user);
        entityManager.flush();
        
        // When
        Optional<User> found = userRepository.findByEmail("juan@test.com");
        
        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getName()).isEqualTo("Juan");
    }
    
    @Test
    void customQuery_FindsActiveUsers() {
        // Given
        User active = new User();
        active.setName("Juan");
        active.setActive(true);
        entityManager.persist(active);
        
        User inactive = new User();
        inactive.setName("Pedro");
        inactive.setActive(false);
        entityManager.persist(inactive);
        entityManager.flush();
        
        // When
        List<User> activeUsers = userRepository.findByActiveTrue();
        
        // Then
        assertThat(activeUsers).hasSize(1);
        assertThat(activeUsers.get(0).getName()).isEqualTo("Juan");
    }
}
```

**Características:**
- ✅ **Rápido** (solo capa de datos).
- ✅ Usa BD embebida (H2) por defecto.
- ✅ Transaccional (rollback automático después de cada test).
- ✅ Auto-configura `TestEntityManager`.
- ❌ No carga `@Service`, `@Controller`.

**Qué se carga:**
- `@Repository`, JPA repositories
- `@Entity`
- DataSource (embebido)
- Hibernate, Spring Data JPA

**Qué NO se carga:**
- `@Service`, `@RestController`
- Capa de negocio

**Cuándo usar:**
- Tests de repositorios JPA.
- Validar queries custom (`@Query`).
- Probar relaciones entre entidades.

---

**Comparación:**

| Anotación | Capa | Beans cargados | Velocidad | BD | MockMvc | Uso |
|-----------|------|---------------|-----------|-----|---------|------|
| **@SpringBootTest** | Full | Todos | ❌ Lento | Real/Embebida | Opcional | Tests de integración end-to-end |
| **@WebMvcTest** | Web | Controllers, Advice | ⚡ Rápido | ❌ No | ✅ Sí | Tests unitarios de controllers |
| **@DataJpaTest** | Persistence | Repositories, Entities | ⚡ Rápido | Embebida | ❌ No | Tests de repositorios JPA |

**Otras anotaciones de slice útiles:**

```java
@JsonTest              // Solo serialización/deserialización JSON
@RestClientTest        // Tests de RestTemplate/WebClient
@JdbcTest              // Tests de JDBC (sin JPA)
@DataMongoTest         // Tests de MongoDB
@DataRedisTest         // Tests de Redis
```

**Best Practices:**

1. **Pirámide de tests:**
```
     /\         @SpringBootTest (pocos)
    /  \        Tests de integración
   /    \       
  /------\      @WebMvcTest, @DataJpaTest (moderados)
 /        \     Tests de slice
/__________\    Tests unitarios puros (muchos)
                Mockito, sin Spring
```

2. **Test unitario puro (sin Spring) cuando sea posible:**
```java
// ✅ Más rápido: sin Spring
class UserServiceTest {
    private UserRepository repository = mock(UserRepository.class);
    private UserService service = new UserService(repository);
    
    @Test
    void createUser_ValidData_SavesUser() {
        // Test puro con Mockito, no Spring Context
    }
}
```

3. **Usa @SpringBootTest solo cuando necesites:**
- Probar interacción completa entre capas.
- Validar auto-configuración.
- Tests de smoke.

4. **Combina slices con Testcontainers para tests realistas:**
```java
@DataJpaTest
@Testcontainers
class UserRepositoryIntegrationTest {
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    // Tests contra PostgreSQL real en contenedor
}
```

**Resumen:**
- **@SpringBootTest**: Full context, lento, tests de integración.
- **@WebMvcTest**: Solo web layer, rápido, tests de controllers.
- **@DataJpaTest**: Solo persistence layer, rápido, tests de repositorios.

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

### 🔹 Conceptos Avanzados

#### 14. ¿Qué es Spring AOP y cómo crear aspectos?

**Respuesta:**

**AOP (Aspect-Oriented Programming)** permite separar **cross-cutting concerns** (logging, seguridad, transacciones, caching) de la lógica de negocio mediante **aspectos**.

**Conceptos clave:**

- **Aspecto**: Módulo que encapsula un cross-cutting concern
- **Join Point**: Punto de ejecución (llamada a método, excepción)
- **Pointcut**: Expresión que selecciona join points
- **Advice**: Acción que toma el aspecto (before, after, around)
- **Weaving**: Proceso de aplicar aspectos al código

**Configuración:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

**Ejemplo 1 - Logging de métodos:**
```java
@Aspect
@Component
public class LoggingAspect {
    
    private static final Logger log = LoggerFactory.getLogger(LoggingAspect.class);
    
    // Pointcut: todos los métodos de @Service
    @Pointcut("within(@org.springframework.stereotype.Service *)")
    public void serviceMethods() {}
    
    // Before advice: ejecuta ANTES del método
    @Before("serviceMethods()")
    public void logBefore(JoinPoint joinPoint) {
        log.info("Ejecutando: {}", joinPoint.getSignature().getName());
    }
    
    // AfterReturning: ejecuta DESPUÉS si termina OK
    @AfterReturning(
        pointcut = "serviceMethods()",
        returning = "result"
    )
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        log.info("Método {} completado. Resultado: {}", 
            joinPoint.getSignature().getName(), result);
    }
    
    // AfterThrowing: ejecuta si lanza excepción
    @AfterThrowing(
        pointcut = "serviceMethods()",
        throwing = "exception"
    )
    public void logAfterThrowing(JoinPoint joinPoint, Exception exception) {
        log.error("Método {} lanzó excepción: {}", 
            joinPoint.getSignature().getName(), exception.getMessage());
    }
    
    // After (finally): ejecuta siempre
    @After("serviceMethods()")
    public void logAfter(JoinPoint joinPoint) {
        log.debug("Método {} finalizado", joinPoint.getSignature().getName());
    }
}
```

**Ejemplo 2 - Around advice (medición de performance):**
```java
@Aspect
@Component
public class PerformanceAspect {
    
    @Around("@annotation(medirTiempo)")
    public Object medirTiempo(ProceedingJoinPoint joinPoint, MedirTiempo medirTiempo) 
            throws Throwable {
        long inicio = System.currentTimeMillis();
        
        Object resultado = joinPoint.proceed();  // Ejecutar método original
        
        long fin = System.currentTimeMillis();
        long duracion = fin - inicio;
        
        if (duracion > medirTiempo.umbralMs()) {
            log.warn("Método {} tardó {}ms (umbral: {}ms)",
                joinPoint.getSignature().getName(), duracion, medirTiempo.umbralMs());
        }
        
        return resultado;
    }
}

// Anotación personalizada
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MedirTiempo {
    long umbralMs() default 1000;
}

// Uso
@Service
public class UserService {
    @MedirTiempo(umbralMs = 500)
    public User findById(Long id) {
        // lógica
    }
}
```

**Pointcut expressions comunes:**
```java
// Todos los métodos de una clase
@Pointcut("execution(* com.example.UserService.*(..))")

// Métodos públicos de un paquete
@Pointcut("execution(public * com.example.service..*.*(..))")

// Métodos con anotación específica
@Pointcut("@annotation(org.springframework.transaction.annotation.Transactional)")

// Todos los métodos de clases con anotación @RestController
@Pointcut("within(@org.springframework.web.bind.annotation.RestController *)")

// Combinar pointcuts
@Pointcut("execution(* com.example..*.*(..)) && @annotation(Secured)")
```

**Ejemplo 3 - Validación de seguridad:**
```java
@Aspect
@Component
public class SecurityAspect {
    
    @Autowired
    private SecurityService securityService;
    
    @Around("@annotation(requiereRol)")
    public Object verificarRol(ProceedingJoinPoint joinPoint, RequiereRol requiereRol) 
            throws Throwable {
        String[] rolesRequeridos = requiereRol.value();
        
        if (!securityService.tieneAlgunRol(rolesRequeridos)) {
            throw new AccessDeniedException("No tienes permisos");
        }
        
        return joinPoint.proceed();
    }
}

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequiereRol {
    String[] value();
}

// Uso
@RestController
public class AdminController {
    @RequiereRol({"ADMIN", "SUPERUSER"})
    @DeleteMapping("/usuarios/{id}")
    public void eliminar(@PathVariable Long id) {
        // Solo admins
    }
}
```

**Best Practices:**
- Usa AOP para cross-cutting concerns, no para lógica de negocio
- Define pointcuts reutilizables
- Ten cuidado con el orden de aspectos (`@Order`)
- Documenta bien los aspectos (son "mágicos" para otros devs)
- Considera el impacto en performance

---

#### 15. ¿Qué son los Events en Spring?

**Respuesta:**

El **sistema de eventos** de Spring permite **comunicación desacoplada** entre componentes mediante el patrón **Publisher-Subscriber**.

**Componentes:**
1. **Event**: Objeto que contiene datos del evento
2. **Publisher**: Publica el evento (`ApplicationEventPublisher`)
3. **Listener**: Escucha y reacciona al evento (`@EventListener`)

**Ejemplo básico:**

```java
// 1. Definir evento
public class UsuarioRegistradoEvent {
    private final String email;
    private final LocalDateTime fecha;
    
    public UsuarioRegistradoEvent(String email) {
        this.email = email;
        this.fecha = LocalDateTime.now();
    }
    
    // getters
}

// 2. Publicar evento
@Service
public class UsuarioService {
    
    @Autowired
    private ApplicationEventPublisher eventPublisher;
    
    public Usuario registrar(UsuarioDTO dto) {
        Usuario usuario = // crear usuario
        usuarioRepository.save(usuario);
        
        // Publicar evento
        eventPublisher.publishEvent(new UsuarioRegistradoEvent(usuario.getEmail()));
        
        return usuario;
    }
}

// 3. Escuchar evento
@Component
public class NotificacionListener {
    
    @EventListener
    public void manejarUsuarioRegistrado(UsuarioRegistradoEvent event) {
        enviarEmailBienvenida(event.getEmail());
        log.info("Email de bienvenida enviado a: {}", event.getEmail());
    }
}

@Component
public class AuditoriaListener {
    
    @EventListener
    public void auditar(UsuarioRegistradoEvent event) {
        guardarAuditoria("Nuevo usuario: " + event.getEmail());
    }
}
```

**Event asíncrono:**
```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Component
public class NotificacionListener {
    
    @Async
    @EventListener
    public void manejarUsuarioRegistrado(UsuarioRegistradoEvent event) {
        // Se ejecuta en thread separado
        enviarEmail(event.getEmail());
    }
}
```

**Event transaccional:**
```java
@Component
public class NotificacionListener {
    
    // Se ejecuta DESPUÉS de que la transacción hace commit
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void manejarDespuesDeCommit(UsuarioRegistradoEvent event) {
        enviarEmail(event.getEmail());
    }
    
    // Si la transacción falla (rollback)
    @TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK)
    public void manejarDespuesDeRollback(UsuarioRegistradoEvent event) {
        log.error("Error al registrar usuario: {}", event.getEmail());
    }
}
```

**Eventos genéricos:**
```java
// Evento genérico
public class EntidadCreadaEvent<T> {
    private final T entidad;
    
    public EntidadCreadaEvent(T entidad) {
        this.entidad = entidad;
    }
    
    public T getEntidad() { return entidad; }
}

// Publisher
public void crear(Usuario usuario) {
    usuarioRepository.save(usuario);
    eventPublisher.publishEvent(new EntidadCreadaEvent<>(usuario));
}

// Listener con tipo específico
@EventListener
public void manejarUsuarioCreado(EntidadCreadaEvent<Usuario> event) {
    Usuario usuario = event.getEntidad();
    // procesar
}
```

**Orden de listeners:**
```java
@Component
public class PrioridadListener {
    
    @Order(1)
    @EventListener
    public void primero(UsuarioRegistradoEvent event) {
        // Se ejecuta primero
    }
    
    @Order(2)
    @EventListener
    public void segundo(UsuarioRegistradoEvent event) {
        // Se ejecuta después
    }
}
```

**Best Practices:**
- Usa eventos para desacoplar componentes
- No abuses (puede hacer el código difícil de seguir)
- Usa `@Async` para operaciones lentas (emails, logs externos)
- Usa `@TransactionalEventListener` para garantizar consistencia
- Documenta qué eventos existen y quién los escucha

---

#### 16. Transacciones avanzadas: Propagation & Isolation

**Respuesta:**

**@Transactional** gestiona transacciones de BD. Los niveles de **propagación** e **isolation** controlan su comportamiento.

**Propagation (Propagación):**

```java
public enum Propagation {
    REQUIRED,      // Default: usa transacción existente o crea nueva
    REQUIRES_NEW,  // Siempre crea nueva transacción (suspende existente)
    SUPPORTS,      // Usa transacción si existe, sino ejecuta sin transacción
    NOT_SUPPORTED, // Ejecuta sin transacción (suspende existente)
    MANDATORY,     // Requiere transacción existente, sino lanza excepción
    NEVER,         // No permite transacción, lanza excepción si existe una
    NESTED         // Crea nested transaction (savepoint)
}
```

**Ejemplos:**

```java
@Service
public class PedidoService {
    
    @Autowired
    private AuditoriaService auditoriaService;
    
    @Transactional  // REQUIRED (default)
    public void crearPedido(PedidoDTO dto) {
        Pedido pedido = pedidoRepository.save(dto);
        
        // Auditoria usa la MISMA transacción
        auditoriaService.registrar("Pedido creado");
        
        // Si auditoria falla, el pedido hace rollback también
    }
}

@Service
public class AuditoriaService {
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void registrar(String mensaje) {
        Auditoria log = new Auditoria(mensaje);
        auditoriaRepository.save(log);
        
        // Se ejecuta en transacción NUEVA
        // Si esta falla, NO afecta la transacción del pedido
    }
}
```

**Caso real - Log siempre, incluso si falla:**
```java
@Service
public class TransferenciaService {
    
    @Autowired
    private LogService logService;
    
    @Transactional
    public void transferir(Long desde, Long hacia, BigDecimal monto) {
        try {
            cuentaRepository.debitar(desde, monto);
            cuentaRepository.acreditar(hacia, monto);
            
            // Log en transacción separada
            logService.registrarExito(desde, hacia, monto);
        } catch (Exception e) {
            // Log en transacción separada (se guarda aunque falle la transferencia)
            logService.registrarError(desde, hacia, monto, e.getMessage());
            throw e;
        }
    }
}

@Service
public class LogService {
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void registrarExito(Long desde, Long hacia, BigDecimal monto) {
        // Siempre se guarda, independiente de la transacción padre
        logRepository.save(new Log("Transferencia exitosa"));
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void registrarError(Long desde, Long hacia, BigDecimal monto, String error) {
        // Se guarda aunque la transacción padre falle
        logRepository.save(new Log("Error: " + error));
    }
}
```

**Isolation (Aislamiento):**

Controla qué puede ver una transacción de los cambios de otras transacciones concurrentes.

```java
public enum Isolation {
    DEFAULT,           // Usa el default de la BD
    READ_UNCOMMITTED,  // Puede leer cambios no commiteados (dirty reads)
    READ_COMMITTED,    // Solo lee cambios commiteados (default PostgreSQL)
    REPEATABLE_READ,   // Lecturas consistentes (default MySQL)
    SERIALIZABLE       // Máximo aislamiento, serializa transacciones
}
```

**Problemas de concurrencia:**

```java
// Dirty Read (lectura sucia)
@Transactional(isolation = Isolation.READ_UNCOMMITTED)
public void leerSucio() {
    // Puede leer datos de transacciones no commiteadas
    // Si esa transacción hace rollback, leíste datos "fantasma"
}

// Non-Repeatable Read
@Transactional(isolation = Isolation.READ_COMMITTED)
public void lecturaNoRepetible() {
    Usuario user1 = usuarioRepository.findById(1L);  // edad = 25
    
    // Otra transacción actualiza y commitea: edad = 30
    
    Usuario user2 = usuarioRepository.findById(1L);  // edad = 30
    
    // Misma transacción, diferentes resultados ❌
}

// Phantom Read
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void lecturaFantasma() {
    List<Usuario> users1 = usuarioRepository.findAll();  // 10 usuarios
    
    // Otra transacción inserta usuario y commitea
    
    List<Usuario> users2 = usuarioRepository.findAll();  // 11 usuarios
    
    // Apareció un usuario "fantasma" ❌
}

// Solución: SERIALIZABLE
@Transactional(isolation = Isolation.SERIALIZABLE)
public void serializable() {
    // Máximo aislamiento, como si las transacciones se ejecutaran secuencialmente
    // ⚠️ Menor concurrencia, mayor latencia
}
```

**Ejemplo práctico - Transferencia bancaria:**
```java
@Service
public class CuentaService {
    
    @Transactional(
        isolation = Isolation.SERIALIZABLE,  // Evitar race conditions
        timeout = 5  // Timeout de 5 segundos
    )
    public void transferir(Long cuentaOrigen, Long cuentaDestino, BigDecimal monto) {
        Cuenta origen = cuentaRepository.findById(cuentaOrigen)
            .orElseThrow(() -> new NotFoundException("Cuenta origen no encontrada"));
        
        Cuenta destino = cuentaRepository.findById(cuentaDestino)
            .orElseThrow(() -> new NotFoundException("Cuenta destino no encontrada"));
        
        if (origen.getSaldo().compareTo(monto) < 0) {
            throw new SaldoInsuficienteException();
        }
        
        origen.setSaldo(origen.getSaldo().subtract(monto));
        destino.setSaldo(destino.getSaldo().add(monto));
        
        cuentaRepository.save(origen);
        cuentaRepository.save(destino);
        
        // Si cualquier línea falla, TODO hace rollback
    }
}
```

**Comparación Isolation Levels:**

| Nivel | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|-------|------------|---------------------|--------------|-------------|
| **READ_UNCOMMITTED** | ✗ Sí | ✗ Sí | ✗ Sí | ⚡⚡⚡ Rápido |
| **READ_COMMITTED** | ✓ No | ✗ Sí | ✗ Sí | ⚡⚡ Medio |
| **REPEATABLE_READ** | ✓ No | ✓ No | ✗ Sí | ⚡ Lento |
| **SERIALIZABLE** | ✓ No | ✓ No | ✓ No | ❌ Muy lento |

**Best Practices:**
- Usa `REQUIRED` para operaciones normales
- Usa `REQUIRES_NEW` para logs/auditoría independiente
- Usa `READ_COMMITTED` para la mayoría de casos
- Usa `SERIALIZABLE` solo para operaciones críticas (transferencias)
- Evita transacciones largas (aumenta contention)
- Considera usar locks optimistas (`@Version`)

---

#### 17. Caching en Spring Boot

**Respuesta:**

**Spring Cache Abstraction** proporciona una API unificada para gestionar cachés con diferentes providers (Caffeine, Redis, Ehcache).

**Configuración básica:**
```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        // Caffeine (en memoria, rápido)
        return new CaffeineCacheManager("usuarios", "productos");
    }
}

// Dependencia
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

**Anotaciones principales:**

```java
@Service
public class UsuarioService {
    
    // @Cacheable: Cachea el resultado
    @Cacheable(value = "usuarios", key = "#id")
    public Usuario findById(Long id) {
        // Solo se ejecuta si no está en caché
        return usuarioRepository.findById(id).orElse(null);
    }
    
    // @CachePut: Actualiza el caché
    @CachePut(value = "usuarios", key = "#usuario.id")
    public Usuario actualizar(Usuario usuario) {
        return usuarioRepository.save(usuario);
    }
    
    // @CacheEvict: Elimina del caché
    @CacheEvict(value = "usuarios", key = "#id")
    public void eliminar(Long id) {
        usuarioRepository.deleteById(id);
    }
    
    // @CacheEvict con allEntries: Limpia todo el caché
    @CacheEvict(value = "usuarios", allEntries = true)
    public void limpiarCacheTodos() {
        // Limpia toda la caché "usuarios"
    }
    
    // @Caching: Múltiples operaciones
    @Caching(
        evict = {
            @CacheEvict(value = "usuarios", key = "#usuario.id"),
            @CacheEvict(value = "usuariosPorEmail", key = "#usuario.email")
        }
    )
    public void actualizarEmail(Usuario usuario) {
        usuarioRepository.save(usuario);
    }
}
```

**Keys personalizadas:**
```java
// Key simple
@Cacheable(value = "usuarios", key = "#id")

// Key compuesta
@Cacheable(value = "usuarios", key = "#nombre + '_' + #edad")

// Key condicional
@Cacheable(value = "usuarios", key = "#id", condition = "#id > 0")

// Unless (no cachear si...)
@Cacheable(value = "usuarios", key = "#id", unless = "#result == null")

// SpEL complejo
@Cacheable(value = "usuarios", key = "T(java.lang.String).valueOf(#usuario.id).concat('-').concat(#usuario.email)")
```

**Configuración de Redis:**
```java
@Configuration
@EnableCaching
public class RedisCacheConfig {
    
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))  // TTL de 10 minutos
            .serializeKeysWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new StringRedisSerializer()
                )
            )
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new GenericJackson2JsonRedisSerializer()
                )
            );
        
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .build();
    }
}

// application.yml
spring:
  redis:
    host: localhost
    port: 6379
  cache:
    type: redis
```

**Cache personalizado con Caffeine:**
```java
@Configuration
@EnableCaching
public class CaffeineConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(caffeineCacheBuilder());
        return cacheManager;
    }
    
    Caffeine<Object, Object> caffeineCacheBuilder() {
        return Caffeine.newBuilder()
            .maximumSize(1000)              // Máximo 1000 entradas
            .expireAfterWrite(10, TimeUnit.MINUTES)  // Expira después de 10 min
            .recordStats();                 // Habilitar estadísticas
    }
}
```

**Múltiples Cache Managers:**
```java
@Configuration
@EnableCaching
public class MultiCacheConfig {
    
    @Bean
    @Primary
    public CacheManager caffeineCacheManager() {
        return new CaffeineCacheManager("usuarios", "productos");
    }
    
    @Bean
    public CacheManager redisCacheManager(RedisConnectionFactory factory) {
        return RedisCacheManager.builder(factory).build();
    }
}

// Uso
@Cacheable(value = "usuarios", cacheManager = "caffeineCacheManager")
public Usuario findById(Long id) { }

@Cacheable(value = "sesiones", cacheManager = "redisCacheManager")
public Sesion findSesion(String token) { }
```

**Ejemplo completo:**
```java
@Service
public class ProductoService {
    
    @Autowired
    private ProductoRepository repository;
    
    @Cacheable(
        value = "productos",
        key = "#id",
        unless = "#result == null",
        condition = "#id > 0"
    )
    public Producto findById(Long id) {
        log.info("Buscando producto en BD: {}", id);  // Solo si no está en caché
        return repository.findById(id).orElse(null);
    }
    
    @CachePut(value = "productos", key = "#producto.id")
    public Producto actualizar(Producto producto) {
        log.info("Actualizando producto en BD y caché: {}", producto.getId());
        return repository.save(producto);
    }
    
    @CacheEvict(value = "productos", key = "#id")
    public void eliminar(Long id) {
        log.info("Eliminando producto de BD y caché: {}", id);
        repository.deleteById(id);
    }
    
    @CacheEvict(value = "productos", allEntries = true)
    @Scheduled(fixedRate = 3600000)  // Cada hora
    public void limpiarCachePeriodicamente() {
        log.info("Limpiando caché de productos");
    }
}
```

**Best Practices:**
- Cachea operaciones costosas (BD, APIs externas)
- Define TTL apropiados
- Usa keys descriptivas
- Monitorea hit/miss ratio
- Considera caché distribuido (Redis) para múltiples instancias
- No caches datos sensibles sin cifrar
- Invalida caché cuando actualizas datos

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

[⬅️ Anterior: Java Core](./01-java-core.md) | [🏠 Volver al Inicio](./README.md) | [Siguiente: JPA y Hibernate ➡️](./03-jpa-hibernate.md)