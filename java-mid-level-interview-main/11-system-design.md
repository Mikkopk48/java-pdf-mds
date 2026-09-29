## 🏛️ System Design y Arquitectura

[⬆️ Volver al índice](./README.md)

---

## 📑 Contenidos de esta sección

### 🔹 Patrones de Arquitectura
1. [Arquitectura Monolítica vs Microservicios](#1-monolito-vs-microservicios)
2. [Event-Driven Architecture](#2-event-driven-architecture)
3. [CQRS](#3-cqrs-command-query-responsibility-segregation)
4. [Saga Pattern](#4-saga-pattern)
5. [API Gateway](#5-api-gateway-pattern)

### 🔹 Escalabilidad
6. [Escalabilidad Horizontal vs Vertical](#6-escalabilidad-horizontal-vs-vertical)
7. [Load Balancing](#7-load-balancing)
8. [Caching strategies](#8-estrategias-de-caching)
9. [Database Sharding](#9-database-sharding)
10. [CDN](#10-content-delivery-network-cdn)

### 🔹 Confiabilidad
11. [Circuit Breaker](#11-circuit-breaker-pattern)
12. [Retry y Backoff](#12-retry-strategies-y-exponential-backoff)
13. [Rate Limiting](#13-rate-limiting)
14. [Idempotencia](#14-idempotencia)

### 🔹 Casos Prácticos
15. [Diseño: Sistema de e-commerce](#15-diseño-sistema-de-e-commerce)
16. [Diseño: URL Shortener](#16-diseño-url-shortener)
17. [Diseño: Sistema de Notificaciones](#17-diseño-sistema-de-notificaciones)
18. [Diseño: Rate Limiter](#18-diseño-rate-limiter)

---

### 🔹 Patrones de Arquitectura

#### 1. Monolito vs Microservicios

**Respuesta:**

**Arquitectura Monolítica:**
```
┌─────────────────────────────────┐
│        Aplicación               │
│  ┌──────┐  ┌───────┐  ┌──────┐│
│  │ UI   │  │Business│  │ Data ││
│  │Layer │→ │ Logic  │→ │Layer ││
│  └──────┘  └───────┘  └──────┘│
│                                 │
│  Todo en un solo proceso/JAR    │
└─────────────────────────────────┘
         ↓
    ┌────────┐
    │   DB   │
    └────────┘
```

**Microservicios:**
```
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Service  │  │ Service  │  │ Service  │
│  Users   │  │ Products │  │  Orders  │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │              │
   ┌─┴──┐       ┌──┴─┐        ┌──┴─┐
   │DB  │       │DB  │        │DB  │
   └────┘       └────┘        └────┘
```

**Comparación:**

| Aspecto | Monolito | Microservicios |
|---------|----------|----------------|
| **Deployment** | Todo junto | Independiente por servicio |
| **Escalabilidad** | Vertical (más recursos) | Horizontal (más instancias) |
| **Desarrollo** | Simple inicialmente | Complejo (distribuido) |
| **Testing** | Fácil | Complejo (integración) |
| **Performance** | Rápido (in-process) | Latencia de red |
| **Fallas** | Todo cae | Aisladas por servicio |
| **Stack tecnológico** | Único | Heterogéneo |
| **Base de datos** | Compartida | Por servicio |

**¿Cuándo usar Monolito?**
✅ Startups/MVP
✅ Equipo pequeño (<10 devs)
✅ Dominio simple
✅ Bajo tráfico

**¿Cuándo usar Microservicios?**
✅ Equipos grandes distribuidos
✅ Dominio complejo (bounded contexts claros)
✅ Necesidad de escalar partes independientemente
✅ Despliegues frecuentes e independientes

**Ejemplo - E-commerce:**

**Monolito:**
```java
@Service
public class PedidoService {
    @Autowired
    private UsuarioRepository usuarioRepo;
    @Autowired
    private ProductoRepository productoRepo;
    @Autowired
    private PedidoRepository pedidoRepo;
    @Autowired
    private PagoService pagoService;
    
    @Transactional
    public Pedido crear(Long usuarioId, List<Long> productoIds) {
        Usuario usuario = usuarioRepo.findById(usuarioId).orElseThrow();
        List<Producto> productos = productoRepo.findAllById(productoIds);
        
        Pedido pedido = new Pedido(usuario, productos);
        pedidoRepo.save(pedido);
        
        pagoService.procesar(pedido);
        
        return pedido;
    }
}
```

**Microservicios:**
```java
// Servicio de Pedidos
@RestController
public class PedidoController {
    @Autowired
    private UsuarioClient usuarioClient;  // Feign Client
    @Autowired
    private ProductoClient productoClient;
    @Autowired
    private PagoClient pagoClient;
    
    @PostMapping("/pedidos")
    public Pedido crear(@RequestBody CrearPedidoRequest request) {
        // Llamadas HTTP a otros servicios
        Usuario usuario = usuarioClient.obtener(request.getUsuarioId());
        List<Producto> productos = productoClient.obtenerVarios(request.getProductoIds());
        
        Pedido pedido = new Pedido(usuario.getId(), productos);
        pedidoRepository.save(pedido);
        
        // Comunicación asíncrona con servicio de pagos
        eventPublisher.publishEvent(new PedidoCreadoEvent(pedido));
        
        return pedido;
    }
}
```

**Desafíos de Microservicios:**
- Complejidad operacional (monitoreo, logs distribuidos)
- Transacciones distribuidas
- Data consistency (eventual consistency)
- Latencia de red
- Testing complejo

---

#### 2. Event-Driven Architecture

**Respuesta:**

**Event-Driven Architecture (EDA)** es un patrón donde los servicios se comunican mediante **eventos asíncronos** en lugar de llamadas síncronas.

**Componentes:**

```
┌──────────┐        ┌───────────┐       ┌──────────┐
│Producer  │─evento→│Event Bus  │→evento│Consumer  │
│(Servicio)│        │(Kafka/    │       │(Servicio)│
└──────────┘        │RabbitMQ)  │       └──────────┘
                    └───────────┘
```

**Tipos de eventos:**

**1. Event Notification:** Notifica que algo ocurrió
```java
// Producer
@Service
public class PedidoService {
    @Autowired
    private ApplicationEventPublisher eventPublisher;
    
    public Pedido crear(PedidoDTO dto) {
        Pedido pedido = pedidoRepository.save(dto);
        
        // Publicar evento
        eventPublisher.publishEvent(new PedidoCreadoEvent(
            pedido.getId(),
            pedido.getUsuarioId(),
            pedido.getTotal()
        ));
        
        return pedido;
    }
}

// Consumer
@Component
public class EmailListener {
    @EventListener
    @Async
    public void enviarEmailConfirmacion(PedidoCreadoEvent event) {
        emailService.enviar(
            event.getUsuarioEmail(),
            "Pedido #" + event.getPedidoId() + " creado"
        );
    }
}

@Component
public class InventarioListener {
    @EventListener
    @Async
    public void actualizarStock(PedidoCreadoEvent event) {
        for (Producto p : event.getProductos()) {
            inventarioService.decrementar(p.getId(), p.getCantidad());
        }
    }
}
```

**2. Event-Carried State Transfer:** Evento contiene todos los datos
```java
public class UsuarioActualizadoEvent {
    private Long usuarioId;
    private String nombre;
    private String email;
    private String telefono;
    // Todos los datos necesarios
}

// Consumers no necesitan llamar al servicio de usuarios
```

**3. Event Sourcing:** Estado se reconstruye desde eventos
```java
// En lugar de guardar estado actual, guardas eventos
public class CuentaBancaria {
    private List<Event> eventos = new ArrayList<>();
    
    public void depositar(BigDecimal monto) {
        eventos.add(new DepositoRealizadoEvent(monto, LocalDateTime.now()));
    }
    
    public void retirar(BigDecimal monto) {
        eventos.add(new RetiroRealizadoEvent(monto, LocalDateTime.now()));
    }
    
    public BigDecimal getSaldo() {
        // Reconstruir saldo desde eventos
        return eventos.stream()
            .map(e -> e instanceof DepositoRealizadoEvent 
                ? ((DepositoRealizadoEvent) e).getMonto() 
                : ((RetiroRealizadoEvent) e).getMonto().negate())
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

**Con Kafka:**
```java
// Producer
@Service
public class PedidoService {
    @Autowired
    private KafkaTemplate<String, PedidoCreadoEvent> kafkaTemplate;
    
    public Pedido crear(PedidoDTO dto) {
        Pedido pedido = pedidoRepository.save(dto);
        
        PedidoCreadoEvent event = new PedidoCreadoEvent(pedido);
        kafkaTemplate.send("pedidos.creados", event.getPedidoId().toString(), event);
        
        return pedido;
    }
}

// Consumer
@Component
public class EmailConsumer {
    @KafkaListener(topics = "pedidos.creados", groupId = "email-service")
    public void consumir(PedidoCreadoEvent event) {
        emailService.enviarConfirmacion(event);
    }
}

@Component
public class InventarioConsumer {
    @KafkaListener(topics = "pedidos.creados", groupId = "inventario-service")
    public void consumir(PedidoCreadoEvent event) {
        inventarioService.actualizarStock(event.getProductos());
    }
}
```

**Ventajas:**
✅ Desacoplamiento (producer no conoce consumers)
✅ Escalabilidad (consumers independientes)
✅ Resiliencia (si consumer cae, eventos quedan en cola)
✅ Auditabilidad (historial de eventos)

**Desventajas:**
❌ Eventual consistency
❌ Debugging complejo
❌ Orden de eventos
❌ Duplicados (idempotencia necesaria)

---

#### 3. CQRS (Command Query Responsibility Segregation)

**Respuesta:**

**CQRS** separa las operaciones de **escritura (Commands)** de las de **lectura (Queries)** usando modelos diferentes.

**Sin CQRS (modelo único):**
```java
@Service
public class ProductoService {
    // Mismo modelo para lectura y escritura
    public Producto crear(ProductoDTO dto) { }
    public Producto actualizar(Long id, ProductoDTO dto) { }
    public void eliminar(Long id) { }
    public Producto obtener(Long id) { }
    public List<Producto> buscar(String filtro) { }
}
```

**Con CQRS:**
```
┌─────────────┐          ┌──────────────┐
│  Commands   │          │   Queries    │
│  (Write)    │          │   (Read)     │
│             │          │              │
│ - Crear     │          │ - Obtener    │
│ - Actualizar│          │ - Buscar     │
│ - Eliminar  │          │ - Listar     │
└──────┬──────┘          └──────┬───────┘
       │                        │
       ▼                        ▼
┌──────────────┐          ┌──────────────┐
│  Write DB    │─eventos→│   Read DB    │
│(PostgreSQL)  │          │(Elasticsearch│
│Normalizada   │          │Desnormalizada│
└──────────────┘          └──────────────┘
```

**Implementación:**

```java
// Command Side (Write Model)
@Service
public class ProductoCommandService {
    @Autowired
    private ProductoRepository writeRepository;
    @Autowired
    private ApplicationEventPublisher eventPublisher;
    
    @Transactional
    public void crear(CrearProductoCommand command) {
        Producto producto = new Producto(
            command.getNombre(),
            command.getPrecio(),
            command.getStock()
        );
        
        writeRepository.save(producto);
        
        // Publicar evento
        eventPublisher.publishEvent(new ProductoCreadoEvent(
            producto.getId(),
            producto.getNombre(),
            producto.getPrecio(),
            producto.getStock()
        ));
    }
    
    @Transactional
    public void actualizar(ActualizarProductoCommand command) {
        Producto producto = writeRepository.findById(command.getId()).orElseThrow();
        producto.actualizar(command.getNombre(), command.getPrecio());
        
        writeRepository.save(producto);
        
        eventPublisher.publishEvent(new ProductoActualizadoEvent(producto));
    }
}

// Query Side (Read Model)
@Service
public class ProductoQueryService {
    @Autowired
    private ProductoReadRepository readRepository;  // Puede ser MongoDB, Elasticsearch
    
    public ProductoDTO obtener(Long id) {
        return readRepository.findById(id)
            .map(this::toDTO)
            .orElseThrow();
    }
    
    public List<ProductoDTO> buscar(String termino) {
        // Búsqueda optimizada en modelo desnormalizado
        return readRepository.buscarPorTexto(termino);
    }
}

// Event Handler: Sincroniza Read Model
@Component
public class ProductoEventHandler {
    @Autowired
    private ProductoReadRepository readRepository;
    
    @EventListener
    public void cuando(ProductoCreadoEvent event) {
        ProductoReadModel modelo = new ProductoReadModel(
            event.getId(),
            event.getNombre(),
            event.getPrecio(),
            event.getStock(),
            calcularDescripcionCompleta(event)
        );
        
        readRepository.save(modelo);
    }
    
    @EventListener
    public void cuando(ProductoActualizadoEvent event) {
        ProductoReadModel modelo = readRepository.findById(event.getId()).orElseThrow();
        modelo.actualizar(event);
        readRepository.save(modelo);
    }
}
```

**¿Cuándo usar CQRS?**

✅ **Usar cuando:**
- Diferentes requisitos de performance para lectura/escritura
- Lectura >> Escritura (ej: redes sociales, e-commerce)
- Necesitas múltiples representaciones de datos
- Domain complejo

❌ **NO usar cuando:**
- CRUD simple
- Lectura ≈ Escritura
- Equipo pequeño
- Overhead no justificado

**Ejemplo real - E-commerce:**

**Write Model (PostgreSQL):**
```sql
-- Normalizado para integridad
CREATE TABLE productos (
    id BIGSERIAL PRIMARY KEY,
    nombre VARCHAR(255),
    precio DECIMAL(10, 2),
    categoria_id BIGINT
);

CREATE TABLE categorias (
    id BIGSERIAL PRIMARY KEY,
    nombre VARCHAR(100)
);
```

**Read Model (Elasticsearch):**
```json
{
  "id": 123,
  "nombre": "Laptop HP",
  "precio": 1299.99,
  "categoria": "Electrónica",
  "subcategoria": "Computadoras",
  "marca": "HP",
  "rating": 4.5,
  "totalReviews": 256,
  "stock": 15,
  "imagenUrl": "...",
  "descripcion": "...",
  "especificaciones": [...],
  "productosRelacionados": [...]
}
```

Ventaja: Búsqueda instantánea, datos desnormalizados listos para mostrar

---

#### 11. Circuit Breaker Pattern

**Respuesta:**

**Circuit Breaker** previene que una aplicación intente ejecutar operaciones que probablemente fallarán, mejorando resiliencia.

**Estados:**

```
┌─────────┐ Fallas    ┌─────────┐ Timeout  ┌─────────┐
│ CLOSED  │──superan──→│  OPEN   │─────────→│HALF_OPEN│
│         │  threshold │         │          │         │
└────▲────┘            └────┬────┘          └────┬────┘
     │                      │                    │
     │    Éxitos            │ Falla inmediata    │Éxito
     └──────────────────────┘                    │
                                          ┌──────┘
                                          │
                                     ┌────▼────┐
                                     │ CLOSED  │
                                     └─────────┘
```

**Estados:**

1. **CLOSED:** Funciona normal, permite requests
2. **OPEN:** Muchas fallas, rechaza requests inmediatamente
3. **HALF_OPEN:** Permite algunos requests de prueba

**Con Resilience4j:**

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
</dependency>
```

```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      pagoService:
        registerHealthIndicator: true
        slidingWindowSize: 10  # Ventana de 10 requests
        failureRateThreshold: 50  # 50% de fallas abre circuito
        waitDurationInOpenState: 10s  # Espera 10s antes de HALF_OPEN
        permittedNumberOfCallsInHalfOpenState: 3
        minimumNumberOfCalls: 5
```

```java
@Service
public class PagoClient {
    
    @CircuitBreaker(name = "pagoService", fallbackMethod = "pagoFallback")
    public PagoResponse procesarPago(PagoRequest request) {
        // Llamada a servicio externo que puede fallar
        return restTemplate.postForObject(
            "https://api-pagos.com/procesar",
            request,
            PagoResponse.class
        );
    }
    
    // Fallback ejecutado cuando circuito está OPEN
    private PagoResponse pagoFallback(PagoRequest request, Exception ex) {
        log.warn("Circuit breaker abierto, usando fallback: {}", ex.getMessage());
        
        return PagoResponse.builder()
            .estado("PENDIENTE")
            .mensaje("Servicio de pagos temporalmente no disponible")
            .build();
    }
}
```

**Ejemplo completo con múltiples servicios:**

```java
@Service
public class ProductoService {
    @Autowired
    private RecomendacionClient recomendacionClient;
    
    public ProductoDetalleDTO obtener(Long id) {
        Producto producto = productoRepository.findById(id).orElseThrow();
        
        // Llamada con circuit breaker
        List<Producto> recomendados = recomendacionClient.obtenerRecomendaciones(id);
        
        return ProductoDetalleDTO.builder()
            .producto(producto)
            .recomendados(recomendados)
            .build();
    }
}

@Component
public class RecomendacionClient {
    @Autowired
    private RestTemplate restTemplate;
    
    @CircuitBreaker(
        name = "recomendaciones",
        fallbackMethod = "recomendacionesFallback"
    )
    public List<Producto> obtenerRecomendaciones(Long productoId) {
        return restTemplate.getForObject(
            "https://api-recomendaciones.com/productos/" + productoId,
            List.class
        );
    }
    
    private List<Producto> recomendacionesFallback(Long productoId, Exception ex) {
        log.warn("No se pudieron obtener recomendaciones: {}", ex.getMessage());
        
        // Devolver lista vacía o recomendaciones default
        return productoRepository.findMasVendidos(5);
    }
}
```

**Monitoreo:**

```java
@Component
public class CircuitBreakerMetrics {
    
    @Autowired
    private CircuitBreakerRegistry circuitBreakerRegistry;
    
    @Scheduled(fixedRate = 60000)
    public void monitorear() {
        circuitBreakerRegistry.getAllCircuitBreakers().forEach(cb -> {
            CircuitBreaker.Metrics metrics = cb.getMetrics();
            
            log.info("Circuit Breaker: {}", cb.getName());
            log.info("  Estado: {}", cb.getState());
            log.info("  Tasa de fallas: {}%", metrics.getFailureRate());
            log.info("  Llamadas: {}", metrics.getNumberOfSuccessfulCalls());
            log.info("  Fallas: {}", metrics.getNumberOfFailedCalls());
        });
    }
}
```

**Combinado con Retry:**

```yaml
resilience4j:
  retry:
    instances:
      pagoService:
        maxAttempts: 3
        waitDuration: 1s
```

```java
@Retry(name = "pagoService")
@CircuitBreaker(name = "pagoService", fallbackMethod = "fallback")
public PagoResponse procesar(PagoRequest request) {
    // Primero intenta 3 veces (Retry)
    // Si sigue fallando, Circuit Breaker se abre
}
```

---

#### 15. Diseño: Sistema de E-commerce

**Respuesta:**

**Requisitos funcionales:**
- Catálogo de productos
- Carrito de compras
- Procesamiento de pedidos
- Pagos
- Gestión de inventario
- Notificaciones

**Requisitos no funcionales:**
- 10,000 usuarios concurrentes
- 99.9% disponibilidad
- Latencia < 500ms
- Escalabilidad horizontal

**Arquitectura:**

```
                  ┌──────────┐
                  │  CDN     │ (imágenes estáticas)
                  └──────────┘
                       │
┌──────────┐    ┌──────▼───────┐    ┌────────────┐
│ Usuarios │───→│ Load Balancer│───→│API Gateway │
└──────────┘    └──────────────┘    └─────┬──────┘
                                           │
        ┌──────────────────────────────────┼──────────────────┐
        │                                  │                  │
  ┌─────▼──────┐  ┌──────────────┐  ┌─────▼──────┐  ┌──────▼────────┐
  │  Catálogo  │  │   Carrito    │  │  Pedidos   │  │    Pagos      │
  │  Service   │  │   Service    │  │  Service   │  │   Service     │
  └─────┬──────┘  └──────┬───────┘  └─────┬──────┘  └──────┬────────┘
        │                │                 │                │
  ┌─────▼────┐    ┌──────▼──────┐   ┌─────▼─────┐   ┌─────▼─────┐
  │PostgreSQL│    │    Redis    │   │PostgreSQL │   │Stripe API │
  │Read      │    │             │   │           │   └───────────┘
  │Replicas  │    │             │   │           │
  └──────────┘    └─────────────┘   └─────┬─────┘
                                           │
                                    ┌──────▼──────┐
                                    │   Kafka     │
                                    │ Event Bus   │
                                    └──────┬──────┘
                                           │
        ┌──────────────────────────────────┼──────────────────┐
        │                                  │                  │
  ┌─────▼────────┐  ┌──────────────┐  ┌──▼───────────┐ ┌───▼──────────┐
  │ Inventario   │  │ Notificaciones│  │  Analytics   │ │ Recomendac.  │
  │   Service    │  │    Service    │  │   Service    │ │   Service    │
  └──────────────┘  └───────────────┘  └──────────────┘ └──────────────┘
```

**Flujo de pedido:**

```java
// 1. Usuario agrega producto al carrito
POST /api/carrito/agregar
{
  "productoId": 123,
  "cantidad": 2
}

// CarritoService (Redis para velocidad)
@Service
public class CarritoService {
    @Autowired
    private RedisTemplate<String, Carrito> redisTemplate;
    
    public void agregar(Long usuarioId, Long productoId, int cantidad) {
        String key = "carrito:" + usuarioId;
        Carrito carrito = redisTemplate.opsForValue().get(key);
        
        if (carrito == null) {
            carrito = new Carrito(usuarioId);
        }
        
        carrito.agregar(productoId, cantidad);
        redisTemplate.opsForValue().set(key, carrito, 7, TimeUnit.DAYS);
    }
}

// 2. Usuario confirma pedido
POST /api/pedidos
{
  "items": [...],
  "direccionId": 456,
  "metodoPagoId": 789
}

// PedidoService
@Service
public class PedidoService {
    @Autowired
    private InventarioClient inventarioClient;
    @Autowired
    private PagoClient pagoClient;
    @Autowired
    private KafkaTemplate<String, PedidoCreadoEvent> kafka;
    
    @Transactional
    public Pedido crear(CrearPedidoRequest request) {
        // 1. Verificar stock
        boolean stockDisponible = inventarioClient.verificarStock(request.getItems());
        if (!stockDisponible) {
            throw new StockInsuficienteException();
        }
        
        // 2. Crear pedido (estado: PENDIENTE_PAGO)
        Pedido pedido = new Pedido();
        pedido.setUsuarioId(request.getUsuarioId());
        pedido.setItems(request.getItems());
        pedido.setEstado(EstadoPedido.PENDIENTE_PAGO);
        pedidoRepository.save(pedido);
        
        // 3. Procesar pago (síncrono)
        PagoResponse pago = pagoClient.procesar(
            new PagoRequest(pedido.getId(), pedido.getTotal())
        );
        
        if (pago.isExitoso()) {
            pedido.setEstado(EstadoPedido.PAGADO);
            pedidoRepository.save(pedido);
            
            // 4. Publicar evento (asíncrono)
            kafka.send("pedidos.creados", 
                new PedidoCreadoEvent(pedido.getId(), pedido.getItems())
            );
        } else {
            throw new PagoFallidoException();
        }
        
        return pedido;
    }
}

// 3. Inventario escucha evento y actualiza stock
@Component
public class InventarioEventHandler {
    @KafkaListener(topics = "pedidos.creados")
    public void procesarPedidoCreado(PedidoCreadoEvent event) {
        for (ItemPedido item : event.getItems()) {
            inventarioService.decrementarStock(
                item.getProductoId(),
                item.getCantidad()
            );
        }
    }
}

// 4. Notificaciones envía email
@Component
public class NotificacionEventHandler {
    @KafkaListener(topics = "pedidos.creados")
    public void enviarNotificacion(PedidoCreadoEvent event) {
        Pedido pedido = pedidoRepository.findById(event.getPedidoId()).orElseThrow();
        Usuario usuario = usuarioRepository.findById(pedido.getUsuarioId()).orElseThrow();
        
        emailService.enviar(
            usuario.getEmail(),
            "Pedido confirmado",
            generarTemplateEmail(pedido)
        );
    }
}
```

**Optimizaciones:**

**1. Caching:**
```java
// Catálogo con cache agresivo
@Cacheable(value = "productos", key = "#id")
public Producto obtenerProducto(Long id) {
    return productoRepository.findById(id).orElseThrow();
}

// Cache de búsquedas populares
@Cacheable(value = "busquedas", key = "#termino")
public List<Producto> buscar(String termino) {
    return productoRepository.buscar(termino);
}
```

**2. Read replicas para consultas:**
```java
// Master para escrituras
@Transactional
public void actualizarStock(Long productoId, int cantidad) {
    masterRepository.actualizarStock(productoId, cantidad);
}

// Replica para lecturas
@Transactional(readOnly = true)
public List<Producto> buscar(String filtro) {
    return replicaRepository.buscar(filtro);
}
```

**3. CQRS para catálogo:**
```
Write: PostgreSQL (normalizado)
Read: Elasticsearch (búsqueda full-text, filtros, facets)
```

**Métricas a monitorear:**
- Latencia P95 de APIs
- Tasa de conversión (carrito → pedido)
- Disponibilidad de servicios
- Tasa de error de pagos
- Stock disponible

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

[⬅️ Anterior: Testing](./10-testing.md) | [🏠 Volver al Inicio](./README.md)
