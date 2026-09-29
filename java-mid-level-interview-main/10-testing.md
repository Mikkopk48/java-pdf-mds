## 🧪 Testing Avanzado

[⬆️ Volver al índice](./README.md)

---

## 📑 Contenidos de esta sección

### 🔹 Fundamentos
1. [Pirámide de Testing](#1-pirámide-de-testing)
2. [Test Unitario vs Integración vs E2E](#2-diferencias-entre-tipos-de-tests)
3. [TDD vs BDD](#3-tdd-vs-bdd)
4. [Test Doubles: Mock vs Stub vs Spy](#4-test-doubles-mock-vs-stub-vs-spy-vs-fake)

### 🔹 Testing en Java/Spring
5. [JUnit 5 features](#5-junit-5-características-principales)
6. [Mockito avanzado](#6-mockito-técnicas-avanzadas)
7. [Testing de Controllers](#7-testing-de-controllers-en-spring)
8. [Testing de Repositorios](#8-testing-de-repositorios-jpa)
9. [Testing de Services](#9-testing-de-services)

### 🔹 Testing Avanzado
10. [Testcontainers](#10-testcontainers)
11. [Contract Testing](#11-contract-testing)
12. [Performance Testing](#12-performance-testing)
13. [Mutation Testing](#13-mutation-testing)
14. [Test Coverage](#14-test-coverage-y-métricas)

---

### 🔹 Fundamentos

#### 1. Pirámide de Testing

**Respuesta:**

La **pirámide de testing** representa la proporción ideal de tipos de tests en un proyecto.

```
           /\
          /  \      E2E Tests (5%)
         /    \     - Selenium, Cypress
        /      \    - Lentos, costosos
       /--------\   - Frágiles
      /          \  
     /            \ Integration Tests (15%)
    /              \- @SpringBootTest
   /                \- Base de datos real
  /                  \- Testcontainers
 /--------------------\
/                      \ Unit Tests (80%)
/                        \- JUnit + Mockito
/                          \- Rápidos, aislados
──────────────────────────── - Sin dependencias externas
```

**Principios:**

1. **Base ancha (Unit Tests):**
   - Más cantidad
   - Más rápidos
   - Más baratos
   - Más estables

2. **Medio (Integration Tests):**
   - Menos cantidad
   - Lentos
   - Prueban integración entre componentes

3. **Cima (E2E Tests):**
   - Pocos
   - Muy lentos
   - Costosos de mantener
   - Prueban flujos completos

**Ejemplo de distribución:**
```
Proyecto con 1000 tests:
- 800 tests unitarios (80%)
- 150 tests de integración (15%)
- 50 tests E2E (5%)
```

**Anti-patrón: Ice Cream Cone** ❌
```
         ┌─────┐
         │ E2E │  ← Muchos tests E2E
       ┌─┴─────┴─┐
       │  Integ  │  ← Algunos tests integración
     ┌─┴─────────┴─┐
     │    Unit     │  ← Pocos tests unitarios
     └─────────────┘
```
Problema: Tests lentos, frágiles, costosos

---

#### 2. Diferencias entre tipos de tests

**Respuesta:**

**Test Unitario:**
```java
// Prueba UNA unidad de código (clase/método) de forma aislada
// No usa Spring Context, no toca BD, no hace HTTP

@Test
void calcularTotal_ConDescuento_DevuelveMontoCorrect() {
    // Given
    CarritoService service = new CarritoService();
    List<Producto> productos = Arrays.asList(
        new Producto("A", 100.0),
        new Producto("B", 50.0)
    );
    
    // When
    double total = service.calcularTotal(productos, 0.10);
    
    // Then
    assertEquals(135.0, total);  // 150 - 10% = 135
}

// ✅ Rápido (<1ms)
// ✅ No dependencias externas
// ✅ Fácil de mantener
```

**Test de Integración:**
```java
// Prueba integración entre componentes
// Usa Spring Context, BD real/embebida

@SpringBootTest
@AutoConfigureTestDatabase(replace = Replace.NONE)  // BD real
class UsuarioServiceIntegrationTest {
    
    @Autowired
    private UsuarioService usuarioService;
    
    @Autowired
    private UsuarioRepository usuarioRepository;
    
    @Test
    @Transactional
    void crearUsuario_GuardaEnBD() {
        // Given
        UsuarioDTO dto = new UsuarioDTO("Juan", "juan@test.com");
        
        // When
        Usuario usuario = usuarioService.crear(dto);
        
        // Then
        Optional<Usuario> encontrado = usuarioRepository.findById(usuario.getId());
        assertTrue(encontrado.isPresent());
        assertEquals("Juan", encontrado.get().getNombre());
    }
}

// ⏱️ Lento (~100ms-1s por test)
// 🔌 Requiere BD, Spring Context
// 🔄 Más complejo de mantener
```

**Test End-to-End (E2E):**
```java
// Prueba flujo completo desde UI hasta BD
// Usuario real interactuando con la aplicación

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class RegistroUsuarioE2ETest {
    
    @LocalServerPort
    private int port;
    
    private WebDriver driver;
    
    @BeforeEach
    void setUp() {
        driver = new ChromeDriver();
    }
    
    @Test
    void registrarUsuario_FlujCompleto() {
        // Abrir navegador
        driver.get("http://localhost:" + port + "/registro");
        
        // Llenar formulario
        driver.findElement(By.id("nombre")).sendKeys("Juan");
        driver.findElement(By.id("email")).sendKeys("juan@test.com");
        driver.findElement(By.id("password")).sendKeys("secret123");
        driver.findElement(By.id("btnRegistrar")).click();
        
        // Verificar
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        WebElement mensaje = wait.until(
            ExpectedConditions.presenceOfElementLocated(By.id("mensajeExito"))
        );
        assertEquals("Registro exitoso", mensaje.getText());
    }
}

// ⏱️ Muy lento (~5-10s por test)
// 🌐 Requiere servidor, BD, navegador
// 💔 Frágil (cambios en UI rompen tests)
```

**Comparación:**

| Aspecto | Unit | Integration | E2E |
|---------|------|-------------|-----|
| **Velocidad** | ⚡ <1ms | ⏱️ 100ms-1s | 🐌 5-10s |
| **Scope** | 1 clase | Múltiples componentes | Sistema completo |
| **Dependencias** | Ninguna | BD, Context | Servidor, navegador |
| **Confianza** | Baja | Media | Alta |
| **Mantenimiento** | Fácil | Medio | Difícil |
| **Cantidad** | 80% | 15% | 5% |

---

#### 4. Test Doubles: Mock vs Stub vs Spy vs Fake

**Respuesta:**

**Test Doubles** son objetos que reemplazan dependencias reales en tests.

**1. Mock:** Verifica interacciones (¿se llamó el método?)
```java
@Test
void enviarNotificacion_LlamaEmailService() {
    // Mock: objeto "inteligente" que verifica llamadas
    EmailService emailMock = mock(EmailService.class);
    NotificacionService service = new NotificacionService(emailMock);
    
    // When
    service.notificarUsuario("user@test.com", "Hola");
    
    // Then: Verificar que se llamó el método
    verify(emailMock).enviar("user@test.com", "Hola");
    verify(emailMock, times(1)).enviar(anyString(), anyString());
}
```

**2. Stub:** Proporciona respuestas predefinidas
```java
@Test
void obtenerPrecio_ConDescuento() {
    // Stub: devuelve valores hardcodeados
    ProductoRepository repoStub = mock(ProductoRepository.class);
    when(repoStub.findById(1L)).thenReturn(Optional.of(new Producto(1L, "Laptop", 1000.0)));
    
    PrecioService service = new PrecioService(repoStub);
    
    // When
    double precio = service.calcularConDescuento(1L, 0.10);
    
    // Then
    assertEquals(900.0, precio);
}
```

**3. Spy:** Objeto real con métodos "espiados"
```java
@Test
void procesarPago_LlamaMetodoPrivado() {
    // Spy: objeto real, pero puedes verificar llamadas
    PagoService service = new PagoService();
    PagoService spy = spy(service);
    
    // When
    spy.procesar(100.0);
    
    // Then: Verifica que se llamó método interno
    verify(spy).validarMonto(100.0);
}
```

**4. Fake:** Implementación simplificada real
```java
// Fake Repository (in-memory, sin BD real)
public class FakeUsuarioRepository implements UsuarioRepository {
    private Map<Long, Usuario> db = new HashMap<>();
    private Long nextId = 1L;
    
    @Override
    public Usuario save(Usuario usuario) {
        usuario.setId(nextId++);
        db.put(usuario.getId(), usuario);
        return usuario;
    }
    
    @Override
    public Optional<Usuario> findById(Long id) {
        return Optional.ofNullable(db.get(id));
    }
}

@Test
void crearUsuario_GuardaEnRepositorio() {
    // Fake: implementación real simplificada
    UsuarioRepository fakeRepo = new FakeUsuarioRepository();
    UsuarioService service = new UsuarioService(fakeRepo);
    
    Usuario usuario = service.crear("Juan");
    
    assertEquals(1L, usuario.getId());
    assertTrue(fakeRepo.findById(1L).isPresent());
}
```

**Dummy:** Objeto que se pasa pero nunca se usa
```java
@Test
void metodoQueNoUsaDependencia() {
    Logger dummyLogger = null;  // Dummy: nunca se usa
    CalculadoraService service = new CalculadoraService(dummyLogger);
    
    int resultado = service.sumar(2, 3);
    
    assertEquals(5, resultado);
}
```

**Resumen:**

| Tipo | Propósito | Cuándo usar |
|------|-----------|-------------|
| **Mock** | Verificar comportamiento | Cuando importa QUÉ se llamó |
| **Stub** | Proporcionar respuestas | Cuando necesitas datos específicos |
| **Spy** | Espiar objeto real | Cuando quieres usar implementación real + verificar |
| **Fake** | Implementación simplificada | Tests de integración rápidos |
| **Dummy** | Llenar parámetros | Cuando el objeto no se usa |

---

### 🔹 Testing en Java/Spring

#### 5. JUnit 5 características principales

**Respuesta:**

**JUnit 5 (Jupiter)** introduce arquitectura modular y nuevas features.

**Anotaciones principales:**
```java
@Test  // Marca método como test
void miTest() { }

@BeforeEach  // Antes de CADA test
void setUp() { }

@AfterEach  // Después de CADA test
void tearDown() { }

@BeforeAll  // Una vez ANTES de todos
static void setUpClass() { }

@AfterAll  // Una vez DESPUÉS de todos
static void tearDownClass() { }

@Disabled("Temporalmente deshabilitado")  // Ignorar test
@Test
void testDeshabilitado() { }

@DisplayName("Crear usuario con email válido")  // Nombre legible
@Test
void test1() { }

@Tag("integration")  // Etiquetar tests
@Test
void testIntegracion() { }
```

**Assertions:**
```java
// Assertions básicas
assertEquals(expected, actual);
assertNotEquals(unexpected, actual);
assertTrue(condition);
assertFalse(condition);
assertNull(object);
assertNotNull(object);
assertSame(expected, actual);  // Misma referencia

// AssertAll: ejecuta todas aunque falle una
assertAll("Usuario válido",
    () -> assertEquals("Juan", usuario.getNombre()),
    () -> assertEquals("juan@test.com", usuario.getEmail()),
    () -> assertTrue(usuario.isActivo())
);

// AssertThrows: verifica excepciones
Exception ex = assertThrows(IllegalArgumentException.class, () -> {
    service.crear(null);
});
assertEquals("Email no puede ser null", ex.getMessage());

// AssertTimeout
assertTimeout(Duration.ofSeconds(1), () -> {
    // Código que debe completarse en <1s
    service.operacionLenta();
});
```

**Tests parametrizados:**
```java
@ParameterizedTest
@ValueSource(strings = {"", "  ", "   "})
void emailVacio_LanzaExcepcion(String email) {
    assertThrows(IllegalArgumentException.class, () -> {
        validador.validarEmail(email);
    });
}

@ParameterizedTest
@CsvSource({
    "juan@test.com, true",
    "invalid-email, false",
    "@test.com, false",
    "test@, false"
})
void validarEmail(String email, boolean esperado) {
    assertEquals(esperado, validador.esValido(email));
}

@ParameterizedTest
@MethodSource("proveedorDeUsuarios")
void crearUsuario(Usuario usuario) {
    // test
}

static Stream<Usuario> proveedorDeUsuarios() {
    return Stream.of(
        new Usuario("Juan", "juan@test.com"),
        new Usuario("Ana", "ana@test.com")
    );
}
```

**Nested tests:**
```java
@DisplayName("UsuarioService")
class UsuarioServiceTest {
    
    @Nested
    @DisplayName("Crear usuario")
    class CrearUsuarioTests {
        
        @Test
        @DisplayName("con email válido")
        void emailValido() { }
        
        @Test
        @DisplayName("con email inválido lanza excepción")
        void emailInvalido() { }
    }
    
    @Nested
    @DisplayName("Actualizar usuario")
    class ActualizarUsuarioTests {
        
        @Test
        @DisplayName("usuario existente")
        void usuarioExistente() { }
        
        @Test
        @DisplayName("usuario no existente lanza excepción")
        void usuarioNoExistente() { }
    }
}
```

**Test condicionales:**
```java
@Test
@EnabledOnOs(OS.LINUX)
void soloEnLinux() { }

@Test
@EnabledOnJre(JRE.JAVA_17)
void soloJava17() { }

@Test
@EnabledIfEnvironmentVariable(named = "ENV", matches = "prod")
void soloEnProduccion() { }

@Test
@EnabledIf("customCondition")
void testCondicional() { }

boolean customCondition() {
    return System.getProperty("run.tests").equals("true");
}
```

---

#### 6. Mockito técnicas avanzadas

**Respuesta:**

**ArgumentCaptor:** Captura argumentos pasados a mocks
```java
@Test
void enviarEmail_CapturaMensaje() {
    EmailService emailMock = mock(EmailService.class);
    NotificacionService service = new NotificacionService(emailMock);
    
    service.notificarCompra("user@test.com", 150.0);
    
    // Capturar argumento
    ArgumentCaptor<String> mensajeCaptor = ArgumentCaptor.forClass(String.class);
    verify(emailMock).enviar(eq("user@test.com"), mensajeCaptor.capture());
    
    String mensaje = mensajeCaptor.getValue();
    assertTrue(mensaje.contains("150.0"));
}
```

**Answer:** Lógica custom para respuestas
```java
@Test
void calcular_ConLogicaCompleja() {
    CalculadoraService mock = mock(CalculadoraService.class);
    
    when(mock.calcular(anyInt(), anyInt())).thenAnswer(invocation -> {
        int a = invocation.getArgument(0);
        int b = invocation.getArgument(1);
        return a * b + 10;  // Lógica custom
    });
    
    assertEquals(16, mock.calcular(2, 3));  // 2*3 + 10 = 16
}
```

**Spy:** Partial mocking
```java
@Test
void spy_PartialMocking() {
    List<String> lista = new ArrayList<>();
    List<String> spy = spy(lista);
    
    // Métodos reales
    spy.add("uno");
    assertEquals(1, spy.size());  // Comportamiento real
    
    // Mock de método específico
    when(spy.size()).thenReturn(100);
    assertEquals(100, spy.size());  // Mock
}
```

**InOrder:** Verificar orden de llamadas
```java
@Test
void procesarPago_OrdenCorrecto() {
    PagoService pagoMock = mock(PagoService.class);
    EmailService emailMock = mock(EmailService.class);
    
    CheckoutService service = new CheckoutService(pagoMock, emailMock);
    service.completarCompra();
    
    InOrder inOrder = inOrder(pagoMock, emailMock);
    inOrder.verify(pagoMock).procesar();     // Primero pago
    inOrder.verify(emailMock).enviarConfirmacion();  // Luego email
}
```

**Mockear métodos estáticos (Mockito 3.4+):**
```java
@Test
void mockMetodoEstatico() {
    try (MockedStatic<Utils> utilsMock = mockStatic(Utils.class)) {
        utilsMock.when(Utils::obtenerFechaActual).thenReturn(LocalDate.of(2024, 1, 1));
        
        // Ahora Utils.obtenerFechaActual() devuelve 2024-01-01
        assertEquals(LocalDate.of(2024, 1, 1), Utils.obtenerFechaActual());
    }
    // Fuera del try, vuelve a comportamiento normal
}
```

**Mockear constructores:**
```java
@Test
void mockConstructor() {
    try (MockedConstruction<Usuario> mocked = mockConstruction(Usuario.class)) {
        Usuario usuario = new Usuario("Juan");  // Mock automático
        
        when(usuario.getNombre()).thenReturn("Mock");
        assertEquals("Mock", usuario.getNombre());
    }
}
```

**BDDMockito (Behavior-Driven):**
```java
import static org.mockito.BDDMockito.*;

@Test
void testConBDD() {
    // Given
    UsuarioRepository repo = mock(UsuarioRepository.class);
    given(repo.findById(1L)).willReturn(Optional.of(new Usuario("Juan")));
    
    UsuarioService service = new UsuarioService(repo);
    
    // When
    Usuario usuario = service.obtener(1L);
    
    // Then
    then(repo).should().findById(1L);
    assertEquals("Juan", usuario.getNombre());
}
```

---

#### 10. Testcontainers

**Respuesta:**

**Testcontainers** permite ejecutar contenedores Docker en tests para usar bases de datos, servicios reales.

**Dependencia:**
```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
```

**Ejemplo básico:**
```java
@SpringBootTest
@Testcontainers
class UsuarioRepositoryTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    @Autowired
    private UsuarioRepository repository;
    
    @Test
    void guardarUsuario_EnPostgresReal() {
        Usuario usuario = new Usuario("Juan", "juan@test.com");
        Usuario guardado = repository.save(usuario);
        
        assertNotNull(guardado.getId());
        assertEquals("Juan", guardado.getNombre());
    }
}
```

**Múltiples contenedores:**
```java
@SpringBootTest
@Testcontainers
class IntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");
    
    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7-alpine")
        .withExposedPorts(6379);
    
    @Container
    static KafkaContainer kafka = new KafkaContainer(
        DockerImageName.parse("confluentinc/cp-kafka:7.5.0")
    );
    
    @DynamicPropertySource
    static void properties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.redis.host", redis::getHost);
        registry.add("spring.redis.port", redis::getFirstMappedPort);
        registry.add("spring.kafka.bootstrap-servers", kafka::getBootstrapServers);
    }
}
```

**Reutilizar contenedores (singleton):**
```java
public abstract class AbstractIntegrationTest {
    
    static final PostgreSQLContainer<?> postgres;
    
    static {
        postgres = new PostgreSQLContainer<>("postgres:15")
            .withReuse(true);  // Reutilizar entre tests
        postgres.start();
    }
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
}

// Extender en tests
@SpringBootTest
class UsuarioServiceTest extends AbstractIntegrationTest {
    // Tests
}
```

**Beneficios:**
- Tests contra BD real (no H2 incompatible)
- Elimina diferencias entre test y producción
- Tests aislados (contenedor se destruye después)
- CI/CD friendly

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

[⬅️ Anterior: Kubernetes](./08-kubernetes.md) | [🏠 Volver al Inicio](./README.md) | [Siguiente: AWS ➡️](./09-aws.md)
