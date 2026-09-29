## 🎯 Escenarios y Preguntas de Comportamiento

[⬆️ Volver al índice](./README.md)

---

## 📑 Contenidos de esta sección

### 🔹 Resolución de Problemas (Troubleshooting)
1. [La aplicación va lenta](#1-la-aplicación-va-lenta-qué-haces)
2. [NullPointerException en Producción](#2-nullpointerexception-en-producción)
3. [Endpoint falla aleatoriamente](#3-un-endpoint-falla-aleatoriamente-intermitente)

### 🔹 Preguntas de Comportamiento
4. [Desafío técnico más difícil](#4-cuál-ha-sido-el-desafío-técnico-más-difícil-que-has-resuelto)
5. [Mantenerse actualizado](#5-cómo-te-mantienes-actualizado-con-tecnologías)
6. [Monolito vs Microservicios](#6-prefieres-monolito-o-microservicios)
7. [Preguntas para el entrevistador](#7-tienes-alguna-pregunta-para-nosotros)

---

### 🔹 Resolución de Problemas (Troubleshooting)

#### 1. "La aplicación va lenta". ¿Qué haces?

**Respuesta:**

Este es un problema común en producción. La clave es un enfoque **sistemático** para identificar el cuello de botella.

**Paso 1: Recopila información**

Preguntas clave:
- ¿Cuándo empezó? (¿Después de un deploy?)
- ¿Es constante o intermitente? (¿Solo en horas pico?)
- ¿Qué endpoints/funciones son lentas? (¿Todas o algunas específicas?)
- ¿Cuántos usuarios afectados?
- ¿Errores en logs?

**Paso 2: Identifica el cuello de botella**

```
       ┌─────────────────────────────────┐
       │  1. Cliente/Red                 │  Latencia de red, DNS, CDN
       └────────────┬────────────────────┘
                    ▼
       ┌─────────────────────────────────┐
       │  2. Load Balancer / Reverse Proxy│  Nginx, HAProxy
       └────────────┬────────────────────┘
                    ▼
       ┌─────────────────────────────────┐
       │  3. Aplicación (Spring Boot)    │  ← Aquí enfocamos primero
       └────────────┬────────────────────┘
                    ▼
       ┌─────────────────────────────────┐
       │  4. Base de Datos               │  Queries lentas, locks
       └────────────┬────────────────────┘
                    ▼
       ┌─────────────────────────────────┐
       │  5. Servicios Externos          │  APIs de terceros, timeouts
       └─────────────────────────────────┘
```

**Herramientas para diagnóstico:**

**A) Métricas del Sistema (Infraestructura):**
```bash
# CPU
top
htop

# Memoria
free -h
top

# Disco I/O
iostat -x 1
iotop

# Red
netstat -an
iftop
```

**B) Application Performance Monitoring (APM):**
- **New Relic**: Trazas distribuidas, métricas de transacciones.
- **Datadog**: Dashboards, alertas.
- **Prometheus + Grafana**: Open source.
- **Spring Boot Actuator**: Métricas internas de la app.

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

**C) Logs de Aplicación:**
```bash
# Ver logs en tiempo real
tail -f /var/log/myapp/application.log

# Buscar errores
grep -i "error\|exception" application.log

# Buscar queries lentas
grep "Hibernate:" application.log | grep -E "[0-9]{4,}ms"
```

**D) Base de Datos:**

```sql
-- PostgreSQL: Queries lentas actuales
SELECT pid, now() - pg_stat_activity.query_start AS duration, query
FROM pg_stat_activity
WHERE state = 'active' AND now() - pg_stat_activity.query_start > interval '5 seconds';

-- Ver locks
SELECT * FROM pg_locks WHERE NOT granted;

-- Ver conexiones
SELECT count(*) FROM pg_stat_activity;
```

**Paso 3: Soluciones según el problema**

**Problema 1: CPU Alto**
```
Síntoma: CPU > 80% constante

Causas:
- Algoritmo ineficiente (loops anidados, recursión)
- Garbage Collection excesivo
- Threads bloqueados (deadlocks)

Soluciones:
✅ Profiler (VisualVM, YourKit) para identificar hot spots
✅ Optimizar algoritmos
✅ Ajustar GC: -XX:+UseG1GC
✅ Escalar horizontalmente (más instancias)
```

**Problema 2: Memoria Alta (Memory Leak)**
```
Síntoma: Memoria crece constante, OutOfMemoryError

Causas:
- Objetos no liberados (referencias estáticas)
- Cachés sin límite
- Colecciones que crecen infinitamente

Soluciones:
✅ Heap dump: jmap -dump:format=b,file=heap.bin <pid>
✅ Analizar con Eclipse MAT o VisualVM
✅ Implementar límites en cachés (Caffeine, Guava)
✅ Revisar listeners/callbacks no desregistrados
```

**Problema 3: Base de Datos Lenta**
```
Síntoma: Queries > 1 segundo

Causas más comunes:
1. Problema N+1 (ver 03-jpa-hibernate.md)
2. Falta de índices
3. Query mal optimizada
4. Locks/Deadlocks
5. Connection pool agotado

Soluciones:
✅ Activar logs de queries lentas:
   spring.jpa.properties.hibernate.session.events.log.LOG_QUERIES_SLOWER_THAN_MS=100

✅ Usar EXPLAIN ANALYZE

✅ Crear índices:
   CREATE INDEX idx_users_email ON users(email);

✅ JOIN FETCH para evitar N+1

✅ Ajustar connection pool:
   spring.datasource.hikari.maximum-pool-size=20
   spring.datasource.hikari.minimum-idle=10

> 🔗 **Ver también:** [Índices en PostgreSQL](./04-database-postgresql.md#4-qué-son-los-índices-y-cuándo-usarlos) | [Problema N+1 en JPA](./03-jpa-hibernate.md#4-qué-es-el-problema-n1-y-cómo-solucionarlo)
```

**Problema 4: Servicios Externos Lentos**
```
Síntoma: Timeouts en llamadas HTTP

Causas:
- API de terceros lenta
- Red inestable
- Sin timeouts configurados

Soluciones:
✅ Configurar timeouts:
   RestTemplate o WebClient con timeout explícito

✅ Circuit Breaker (Resilience4j):
   Si falla N veces, abre circuito (no sigue llamando)

✅ Caché de respuestas:
   @Cacheable("weather")
   public Weather getWeather(String city) { ... }

✅ Asincronía:
   @Async
   public CompletableFuture<Weather> getWeatherAsync(String city) { ... }
```

**Ejemplo de implementación de timeout:**

```java
@Configuration
public class RestClientConfig {
    
    @Bean
    public RestTemplate restTemplate() {
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        factory.setConnectTimeout(5000);  // 5 segundos para conectar
        factory.setReadTimeout(10000);    // 10 segundos para leer respuesta
        return new RestTemplate(factory);
    }
}

// Circuit Breaker con Resilience4j
@Service
public class WeatherService {
    
    @CircuitBreaker(name = "weatherAPI", fallbackMethod = "getWeatherFallback")
    public Weather getWeather(String city) {
        // Llamada a API externa
        return restTemplate.getForObject("https://api.weather.com/v1/" + city, Weather.class);
    }
    
    // Fallback si el circuito está abierto
    private Weather getWeatherFallback(String city, Exception e) {
        return new Weather(city, "N/A", "Servicio temporalmente no disponible");
    }
}
```

**Paso 4: Monitoreo continuo**

```java
// Actuator + Micrometer para métricas custom
@Service
public class UserService {
    private final MeterRegistry meterRegistry;
    
    @Timed("user.service.create")  // Métrica de tiempo
    public User createUser(UserDTO dto) {
        meterRegistry.counter("user.created").increment();  // Contador
        // ...
    }
}
```

**Checklist de optimización:**
- [ ] ¿Logs activados? (ERROR, WARN level en prod)
- [ ] ¿APM configurado? (New Relic, Datadog)
- [ ] ¿Queries optimizadas? (EXPLAIN ANALYZE)
- [ ] ¿Índices creados en columnas frecuentes?
- [ ] ¿Connection pool ajustado?
- [ ] ¿Timeouts en llamadas externas?
- [ ] ¿Circuit breakers implementados?
- [ ] ¿Caché configurado? (Redis, Caffeine)
- [ ] ¿GC optimizado? (-XX:+UseG1GC)
- [ ] ¿Health checks activos?

---

#### 2. NullPointerException en Producción

**Respuesta:**

Un **NPE** es uno de los errores más comunes. La clave es **reproducirlo**, **arreglarlo**, y **prevenir futuros**.

**Paso 1: Recopila información del stacktrace**

```
java.lang.NullPointerException: Cannot invoke "String.length()" because "user.email" is null
	at com.example.UserService.validateEmail(UserService.java:45)
	at com.example.UserService.createUser(UserService.java:30)
	at com.example.UserController.createUser(UserController.java:25)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	...
```

**Lo que nos dice:**
- **Línea exacta**: `UserService.java:45`
- **Qué es null**: `user.email`
- **Qué intentamos hacer**: Llamar al método `length()`
- **Stack trace**: Flujo de llamadas

**Paso 2: Revisa los logs**

```bash
# Busca el contexto alrededor del error
grep -B 10 -A 10 "NullPointerException" application.log

# Busca el request ID para ver el flujo completo
grep "request-id-12345" application.log
```

**Paso 3: Reproduce el error**

```java
@Test
void createUser_EmailNull_ThrowsException() {
    // Given
    UserDTO dto = new UserDTO();
    dto.setName("Juan");
    dto.setEmail(null);  // ← Reproduce el problema
    
    // When & Then
    assertThatThrownBy(() -> userService.createUser(dto))
        .isInstanceOf(NullPointerException.class);
}
```

**Paso 4: Arregla el código**

**Opción 1: Validación defensiva (quick fix)**
```java
// ❌ Código original (vulnerable)
public void validateEmail(User user) {
    if (user.getEmail().length() < 5) {  // NPE si email es null
        throw new ValidationException("Email demasiado corto");
    }
}

// ✅ Fix: Validación null
public void validateEmail(User user) {
    if (user.getEmail() == null || user.getEmail().length() < 5) {
        throw new ValidationException("Email inválido");
    }
}
```

**Opción 2: Optional (mejor práctica Java 8+)**
```java
public class User {
    private String name;
    private String email;  // Puede ser null ❌
    
    public Optional<String> getEmail() {  // ✅ Explicit nullability
        return Optional.ofNullable(email);
    }
}

// Uso
user.getEmail()
    .filter(email -> email.length() >= 5)
    .orElseThrow(() -> new ValidationException("Email inválido"));
```

**Opción 3: Bean Validation (validación automática)**
```java
public class UserDTO {
    @NotBlank(message = "Nombre requerido")
    private String name;
    
    @NotNull(message = "Email requerido")
    @Email(message = "Email inválido")
    private String email;
}

@RestController
public class UserController {
    @PostMapping("/users")
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody UserDTO dto) {
        // Si dto es inválido, lanza MethodArgumentNotValidException automáticamente
        return ResponseEntity.ok(userService.createUser(dto));
    }
}
```

**Opción 4: Objects.requireNonNull (precondiciones)**
```java
public class UserService {
    public void createUser(UserDTO dto) {
        Objects.requireNonNull(dto, "DTO no puede ser null");
        Objects.requireNonNull(dto.getEmail(), "Email no puede ser null");
        
        // Resto del código
    }
}
```

**Paso 5: Prevención (mejores prácticas)**

**A) Usa anotaciones de nullability:**
```java
// Con Spring / JSR-305
@NonNull
public User findUser(@NonNull Long id) {
    return userRepository.findById(id)
        .orElseThrow(() -> new NotFoundException("Usuario no encontrado"));
}

// IntelliJ / Eclipse te advertirán en compile-time si pasas null
```

**B) Evita retornar null:**
```java
// ❌ MAL
public User findUser(Long id) {
    return userRepository.findById(id).orElse(null);  // Devuelve null
}

// ✅ BIEN: Lanza excepción
public User findUser(Long id) {
    return userRepository.findById(id)
        .orElseThrow(() -> new NotFoundException("Usuario no encontrado"));
}

// ✅ BIEN: Devuelve Optional
public Optional<User> findUser(Long id) {
    return userRepository.findById(id);
}

// ✅ BIEN: Devuelve colección vacía (nunca null)
public List<User> findAll() {
    List<User> users = userRepository.findAll();
    return users != null ? users : Collections.emptyList();
}
```

**C) Inicializa colecciones:**
```java
// ❌ MAL
public class User {
    private List<Order> orders;  // null por defecto
}

// ✅ BIEN
public class User {
    private List<Order> orders = new ArrayList<>();  // Nunca null
}
```

**D) Usa Lombok @NonNull:**
```java
@AllArgsConstructor
public class User {
    @NonNull  // Genera null-check automático en constructor
    private String name;
    
    @NonNull
    private String email;
}
```

**Paso 6: Logging mejorado**

```java
// ❌ MAL: Log sin contexto
catch (Exception e) {
    log.error("Error", e);
}

// ✅ BIEN: Log con contexto
catch (Exception e) {
    log.error("Error al crear usuario. DTO: {}, UserId: {}", dto, userId, e);
}

// ✅ MEJOR: Structured logging
catch (Exception e) {
    log.error("Error al crear usuario", 
        kv("dto", dto),
        kv("userId", userId),
        kv("action", "createUser"),
        e);
}
```

---

#### 3. Un endpoint falla aleatoriamente (Intermitente)

**Respuesta:**

Los **errores intermitentes** son los más difíciles de debuggear porque no son reproducibles consistentemente. Requieren análisis de patrones y correlación.

**Causas comunes:**

**1. Problemas de concurrencia (Race Conditions)**

```java
// ❌ MAL: Variable compartida sin sincronización
@Service
public class CounterService {
    private int count = 0;  // Shared mutable state
    
    public void increment() {
        count++;  // NO es thread-safe (read-modify-write)
    }
    
    public int getCount() {
        return count;
    }
}

// ✅ BIEN: AtomicInteger
@Service
public class CounterService {
    private AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();  // Thread-safe
    }
    
    public int getCount() {
        return count.get();
    }
}

// ✅ MEJOR: Stateless (sin estado compartido)
@Service
public class CounterService {
    @Autowired
    private CounterRepository repository;
    
    @Transactional
    public void increment(Long userId) {
        Counter counter = repository.findByUserId(userId);
        counter.setValue(counter.getValue() + 1);
        repository.save(counter);  // BD maneja concurrencia
    }
}
```

**2. Connection Pool agotado**

```
Síntoma: Funciona bien con pocos usuarios, falla con carga alta
Error: "Unable to acquire JDBC Connection"
```

**Solución:**

```yaml
# application.yml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20  # Aumentar (default: 10)
      minimum-idle: 10
      connection-timeout: 30000  # 30 segundos
      idle-timeout: 600000  # 10 minutos
      max-lifetime: 1800000  # 30 minutos

# Monitorear métricas
management:
  metrics:
    enable:
      hikaricp: true
```

**Ver conexiones activas:**
```sql
SELECT count(*) FROM pg_stat_activity WHERE state = 'active';
```

**3. Timeouts en servicios externos**

```java
// ❌ MAL: Sin timeout
@Service
public class WeatherService {
    public Weather getWeather(String city) {
        return restTemplate.getForObject("https://api.weather.com/" + city, Weather.class);
        // Si la API tarda mucho, tu app se bloquea
    }
}

// ✅ BIEN: Con timeout y retry
@Service
public class WeatherService {
    private final RestTemplate restTemplate;
    
    @Retry(name = "weatherAPI", fallbackMethod = "getWeatherFallback")
    @TimeLimiter(name = "weatherAPI")
    public CompletableFuture<Weather> getWeather(String city) {
        return CompletableFuture.supplyAsync(() -> 
            restTemplate.getForObject("https://api.weather.com/" + city, Weather.class)
        );
    }
    
    private CompletableFuture<Weather> getWeatherFallback(String city, Exception e) {
        return CompletableFuture.completedFuture(new Weather(city, "N/A"));
    }
}
```

**resilience4j-spring-boot configuration:**
```yaml
resilience4j:
  timelimiter:
    instances:
      weatherAPI:
        timeout-duration: 5s
  retry:
    instances:
      weatherAPI:
        max-attempts: 3
        wait-duration: 1s
```

**4. Garbage Collection Pauses**

```
Síntoma: App se congela 1-2 segundos aleatoriamente

Causa: GC "Stop-the-World" pauses

Solución:
1. Activar GC logging:
   -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:gc.log

2. Analizar con GCViewer

3. Optimizar GC:
   -XX:+UseG1GC  # G1 para baja latencia
   -XX:MaxGCPauseMillis=200  # Objetivo: pauses < 200ms
```

**5. Problemas de red/infraestructura**

```
Síntoma: Fallas en horarios específicos o regiones geográficas

Diagnóstico:
- Revisar latencia: ping, traceroute
- Balanceador de carga: revisar health checks
- CDN: caché inválida
- DNS: resolución lenta
```

**Técnicas de diagnóstico:**

**A) Correlación de logs (Request ID)**

```java
// Añadir request ID a cada log
@Component
public class RequestIdFilter extends OncePerRequestFilter {
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                    HttpServletResponse response, 
                                    FilterChain filterChain) {
        String requestId = UUID.randomUUID().toString();
        MDC.put("requestId", requestId);
        response.addHeader("X-Request-ID", requestId);
        
        try {
            filterChain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}

// Logs automáticamente incluyen requestId
// 2025-12-22 10:30:00 [requestId=abc-123] ERROR - Error al procesar...
```

**B) Monitoreo de métricas en tiempo real**

```java
@RestController
public class UserController {
    private final MeterRegistry meterRegistry;
    
    @GetMapping("/users/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        Timer.Sample sample = Timer.start(meterRegistry);
        
        try {
            UserDTO user = userService.findById(id);
            meterRegistry.counter("user.get.success").increment();
            return ResponseEntity.ok(user);
        } catch (Exception e) {
            meterRegistry.counter("user.get.failure").increment();
            throw e;
        } finally {
            sample.stop(meterRegistry.timer("user.get.duration"));
        }
    }
}
```

**C) Distributed Tracing (Sleuth + Zipkin)**

```yaml
# pom.xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>

# application.yml
spring:
  sleuth:
    sampler:
      probability: 1.0  # 100% de requests (reduce en prod)
  zipkin:
    base-url: http://localhost:9411
```

Ahora puedes ver el flujo completo de un request a través de múltiples servicios.

---

### 🔹 Preguntas de Comportamiento

#### 4. "¿Cuál ha sido el desafío técnico más difícil que has resuelto?"

**Respuesta:**

Usa el método **STAR** para estructurar tu respuesta:

**S - Situation (Situación)**: Contexto
**T - Task (Tarea)**: Qué debías lograr
**A - Action (Acción)**: Qué hiciste específicamente
**R - Result (Resultado)**: Qué lograste (con métricas si es posible)

**Ejemplo de respuesta:**

**Situación:**
"En mi proyecto anterior, teníamos una aplicación de e-commerce con Spring Boot y PostgreSQL. Durante el Black Friday, la aplicación empezó a fallar aleatoriamente con timeouts. Los usuarios no podían completar compras y estábamos perdiendo ventas."

**Tarea:**
"Mi tarea era identificar la causa raíz y solucionarlo lo antes posible, idealmente en menos de 2 horas, ya que cada minuto de downtime significaba miles de dólares en pérdidas."

**Acción:**
"Primero, revisé los logs y encontré múltiples `LazyInitializationException`. Activé el logging de Hibernate y descubrí un problema N+1 masivo: al listar productos, por cada producto hacíamos 3 queries adicionales para traer sus imágenes, reviews y categorías. Con 100 productos en la página, eran 301 queries.

Implementé JOIN FETCH en las queries principales:
```java
@Query(\"SELECT p FROM Product p \" +
       \"JOIN FETCH p.images \" +
       \"JOIN FETCH p.category \" +
       \"LEFT JOIN FETCH p.reviews \" +
       \"WHERE p.active = true\")
List<Product> findAllActiveWithDetails();
```

También añadí un índice en `products(active, created_at)` que faltaba, y configuré un cache con Redis para la página principal usando `@Cacheable`.

Finalmente, aumenté el connection pool de 10 a 30 porque con la carga alta se agotaban las conexiones."

**Resultado:**
"Después de desplegar los cambios, la latencia promedio bajó de 8 segundos a 400ms (95% de mejora). El problema de timeouts desapareció completamente. Durante el resto del Black Friday, procesamos 10,000 pedidos sin incidentes. También documenté el problema y creamos alertas en Datadog para detectar queries lentas automáticamente."

**Aprendizajes:**
"Aprendí la importancia de testing de carga antes de eventos grandes, y ahora siempre reviso el EXPLAIN de queries críticas. También implementamos Testcontainers en CI para detectar problemas de N+1 en tests de integración."

---

**Tips para tu respuesta:**
- ✅ Sé específico (menciona tecnologías: Spring Boot, PostgreSQL, Redis).
- ✅ Incluye métricas (latencia reducida X%, error rate bajó Y%).
- ✅ Muestra tu proceso de pensamiento (cómo llegaste a la solución).
- ✅ Menciona aprendizajes (qué harías diferente).
- ❌ No culpes a otros.
- ❌ No hables solo en general, da detalles técnicos.

---

#### 5. "¿Cómo te mantienes actualizado con tecnologías?"

**Respuesta:**

**Estructura tu respuesta en categorías:**

**1. Lectura diaria/semanal:**
- **Blogs técnicos**:
  - Baeldung (Java/Spring)
  - Vlad Mihalcea (Hibernate/JPA)
  - DZone, Medium
  - Martin Fowler (Architecture)
  
- **Newsletters**:
  - Java Weekly (Baeldung)
  - Postgres Weekly
  - Spring Blog

**2. Videos / Cursos:**
- **YouTube channels**:
  - Amigoscode (Spring Boot)
  - Java Brains
  - Tech Primers
  
- **Plataformas de cursos**:
  - Udemy (cursos específicos)
  - Pluralsight
  - Coursera

**3. Práctica activa:**
- **Proyectos personales en GitHub**:
  "Actualmente estoy construyendo una aplicación de gestión de tareas con Spring Boot 3, Spring Security 6 con JWT, y PostgreSQL. La uso para experimentar con nuevas features como Virtual Threads (Project Loom)."
  
- **Code Katas / LeetCode**:
  "Resuelvo problemas en LeetCode 2-3 veces por semana para mantener fresco el pensamiento algorítmico."

**4. Comunidad:**
- **Stack Overflow**: Respondo preguntas de Spring/JPA para reforzar conocimientos.
- **Meetups locales**: Asisto a meetups de Java User Group.
- **Conferencias**: Vi charlas de Spring One, Devoxx en YouTube.

**5. Experimentación:**
"Cuando Spring Boot 3.2 salió, creé un proyecto sandbox para probar las nuevas features de observability y native compilation con GraalVM. Escribí un post en mi blog comparando tiempos de startup."

**Ejemplo de respuesta completa:**

"Me mantengo actualizado de varias formas. Diariamente reviso Baeldung y el blog oficial de Spring para estar al tanto de nuevas releases y best practices. Tengo suscritas newsletters como Java Weekly que me llegan cada lunes.

En cuanto a práctica, mantengo un repositorio en GitHub con proyectos personales donde experimento con tecnologías nuevas. Por ejemplo, recientemente migré un proyecto de Spring Boot 2 a 3 para familiarizarme con los cambios, especialmente con Spring Security 6.

También participo en la comunidad: respondo preguntas en Stack Overflow con el tag 'spring-boot', y asisto a meetups locales del Java User Group cuando hay charlas interesantes.

Además, hago code katas en LeetCode un par de veces por semana para mantener fresco el pensamiento algorítmico, y leo libros técnicos. Actualmente estoy leyendo 'Designing Data-Intensive Applications' de Martin Kleppmann."

---

#### 6. "¿Prefieres Monolito o Microservicios?"

**Respuesta:**

**La respuesta correcta es: "Depende del contexto."**

Un desarrollador senior entiende que no hay una solución única. Demuestra tu pensamiento crítico analizando tradeoffs.

**Respuesta estructurada:**

"Depende del contexto del proyecto, equipo y requisitos. Ambos tienen sus ventajas y desventajas.

**Monolito:**

**Ventajas:**
- ✅ Simplicidad: Un solo deployment, un solo repositorio.
- ✅ Fácil de desarrollar: No hay complejidad de red.
- ✅ Fácil de debuggear: Un stacktrace completo.
- ✅ Transacciones ACID: Operaciones atómicas entre módulos.
- ✅ Performance: Llamadas de método (microsegundos) vs HTTP (milisegundos).
- ✅ Testing más simple: Tests de integración end-to-end sin complicaciones.

**Desventajas:**
- ❌ Escalabilidad: Debes escalar toda la app, no solo el módulo lento.
- ❌ Deployment: Un bug en un módulo requiere redesplegar todo.
- ❌ Tiempo de build: Puede ser lento con codebase grande.
- ❌ Equipos: Difícil que múltiples equipos trabajen en paralelo sin conflictos.

**Microservicios:**

**Ventajas:**
- ✅ Escalabilidad independiente: Escala solo el servicio con alta carga.
- ✅ Deployment independiente: Actualiza un servicio sin tocar otros.
- ✅ Tecnologías heterogéneas: Cada servicio puede usar un stack diferente.
- ✅ Equipos autónomos: Cada equipo posee un servicio completo.
- ✅ Resiliencia: Un servicio caído no tumba todo el sistema.

**Desventajas:**
- ❌ Complejidad de red: Latencia, timeouts, retries, circuit breakers.
- ❌ Transacciones distribuidas: No hay ACID, solo eventual consistency (Saga pattern).
- ❌ Debugging difícil: Stacktraces distribuidos, necesitas tracing (Sleuth, Jaeger).
- ❌ Testing complejo: Necesitas contract testing, mocks de servicios.
- ❌ Overhead operacional: Service discovery, load balancing, API gateway, monitoring distribuido.
- ❌ Data consistency: Cada servicio tiene su BD (duplicación de datos).

**Cuándo usar cada uno:**

**Monolito:**
- Equipo pequeño (< 10 personas)
- Producto nuevo (MVP, validación rápida)
- Dominio bien entendido
- No hay requisitos de escalabilidad extrema
- Startup en fase inicial

**Microservicios:**
- Equipos grandes (> 20 personas)
- Diferentes partes del sistema tienen patrones de carga muy distintos
- Necesitas deploy independiente (CI/CD por equipo)
- Dominio complejo con bounded contexts claros (DDD)
- Ya tienes el expertise operacional (DevOps maduro)

**Mi recomendación:**

'Empezar con un monolito bien estructurado (modular monolith) y extraer microservicios solo cuando haya una razón clara de negocio o técnica. Amazon, Netflix y Uber empezaron con monolitos. La complejidad de microservicios debe estar justificada.'

Si estuviera empezando un nuevo proyecto hoy, usaría un **Modular Monolith**:
- Módulos claramente separados por dominio (packages)
- Cada módulo con su propia BD schema (preparado para split futuro)
- APIs internas bien definidas
- Testing por módulo

Esto te da la simplicidad del monolito con la flexibilidad de migrar a microservicios gradualmente si es necesario."

---

**Otros aspectos a mencionar:**

**Patrón Strangler Fig:**
"Si tengo un monolito legacy, no lo reescribiría todo a microservicios. Usaría el Strangler Fig pattern: gradualmente extraigo funcionalidades a servicios nuevos, manteniendo el monolito funcionando hasta que eventualmente se reemplaza pieza por pieza."

**Modular Monolith:**
"Una opción intermedia es el Modular Monolith: un solo deployment, pero con módulos claramente separados que comunican por interfaces bien definidas. Combina simplicidad de deployment con estructura limpia."

---

### 🔹 Cierre

#### 7. ¿Tienes alguna pregunta para nosotros?

**Respuesta:**

**Siempre haz preguntas.** No hacer preguntas puede verse como falta de interés. Haz 2-3 preguntas que demuestren que pensaste en el rol.

**Buenas preguntas:**

**Sobre el equipo:**
- "¿Cómo está estructurado el equipo de desarrollo? ¿Cuántas personas y qué roles?"
- "¿Cómo es un día típico para alguien en este rol?"
- "¿Qué herramientas de colaboración usan? (Jira, Slack, etc.)"

**Sobre el código/tecnología:**
- "¿Cuál es el stack técnico actual? ¿Hay planes de actualizar/migrar tecnologías?"
- "¿Cómo es el proceso de code review?"
- "¿Qué coverage de tests tienen? ¿Usan TDD?"
- "¿Cómo manejan el deployment? ¿CI/CD?"

**Sobre el producto:**
- "¿Cuáles son los principales desafíos técnicos que enfrenta el equipo actualmente?"
- "¿Qué nuevas features están en el roadmap?"
- "¿Cómo miden el éxito del producto?"

**Sobre crecimiento:**
- "¿Qué oportunidades hay para aprender y crecer en este rol?"
- "¿Ofrecen presupuesto para cursos/conferencias?"
- "¿Cómo es el proceso de feedback y evaluación de desempeño?"

**Sobre cultura:**
- "¿Cómo describirían la cultura de ingeniería?"
- "¿Tienen horarios flexibles? ¿Opción de trabajo remoto?"
- "¿Hacen pair programming o mob programming?"

**❌ Evita preguntar en primera entrevista:**
- Salario (espera a que ellos lo mencionen)
- Vacaciones (muy temprano)
- "¿Qué hace la empresa?" (deberías saberlo)

[⬆️ Volver al índice](./README.md)

---

## 📚 Fin de la Guía

**¡Buena suerte en tu entrevista!** 🚀

Recuerda:
- ✅ Práctica con proyectos reales
- ✅ Entiende los conceptos, no solo memorices
- ✅ Repasa tus propios proyectos (te preguntarán sobre ellos)
- ✅ Sé honesto: "No lo sé, pero así lo investigaría..." está bien
- ✅ Piensa en voz alta durante coding challenges
- ✅ Haz preguntas cuando algo no esté claro

**¡Éxitos!** 💪

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

[⬅️ Anterior: Arquitectura y Testing](./05-architecture-ops.md) | [🏠 Volver al Inicio](./README.md)