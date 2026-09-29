# 🔥 Apache Kafka

## Índice
1. [¿Qué es Apache Kafka?](#1-qué-es-apache-kafka)
2. [Arquitectura de Kafka](#2-arquitectura-de-kafka)
3. [Topics y Particiones](#3-topics-y-particiones)
4. [Producers: Envío de Mensajes](#4-producers-envío-de-mensajes)
5. [Consumers y Consumer Groups](#5-consumers-y-consumer-groups)
6. [Kafka con Spring Boot](#6-kafka-con-spring-boot)
7. [Configuración de Producers en Java](#7-configuración-de-producers-en-java)
8. [Configuración de Consumers en Java](#8-configuración-de-consumers-en-java)
9. [Serialización y Deserialización](#9-serialización-y-deserialización)
10. [Manejo de Errores y Retry](#10-manejo-de-errores-y-retry)
11. [Kafka Streams](#11-kafka-streams)
12. [Transacciones en Kafka](#12-transacciones-en-kafka)
13. [Monitoreo y Troubleshooting](#13-monitoreo-y-troubleshooting)
14. [Patrones de Diseño con Kafka](#14-patrones-de-diseño-con-kafka)

---

## 1. ¿Qué es Apache Kafka?

### 📌 Definición

**Apache Kafka** es un **distributed commit log** diseñado como plataforma de streaming con tres capacidades core:
- **Pub/Sub**: Publish-subscribe pattern con persistencia durable
- **Storage**: Distributed, replicated, fault-tolerant log storage
- **Processing**: Stream processing con exactly-once semantics

**Modelo fundamental**: Append-only log con offsets inmutables, optimizado para high-throughput sequential writes.

### 🎯 Casos de Uso Enterprise

1. **Event-Driven Architecture**: Event bus central para microservicios
2. **Real-Time Data Pipelines**: Ingestión y transformación de datos en streaming
3. **Event Sourcing/CQRS**: Source of truth para eventos de dominio
4. **Log Aggregation**: Centralización de logs distribuidos (alta escala)
5. **Change Data Capture (CDC)**: Replicación de cambios desde bases de datos
6. **Metrics/Monitoring**: Pipeline de métricas para observabilidad

### ✅ Ventajas Técnicas

```
✓ Throughput excepcional: >1M msgs/sec/broker (batching + zero-copy)
✓ Horizontal scaling: Linear scalability agregando partitions/brokers
✓ Durabilidad configurable: Replication factor + min.insync.replicas
✓ Latencia predecible: p99 < 50ms con configuración optimizada
✓ Fault tolerance: Leader election automática (< 1s downtime)
✓ Ordering guarantee: Per-partition FIFO con keys determinísticas
✓ Pull-based consumers: Backpressure natural, replay capability
```

### ⚠️ Trade-offs y Limitaciones

```
✗ No message prioritization: FIFO estricto por partición
✗ No TTL per-message: Retención global por topic (time/size based)
✗ Rebalancing overhead: Consumer group coordination latency
✗ Complejidad operacional: Cluster management, monitoring, tuning
✗ No built-in schema validation: Requiere Schema Registry externo
✗ Partition count immutable: Reducir particiones no soportado
```

---

## 2. Arquitectura de Kafka

### 📊 Componentes Principales

```
┌──────────────────────────────────────────────────────────┐
│                    KAFKA CLUSTER                          │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐         │
│  │  Broker 1  │  │  Broker 2  │  │  Broker 3  │         │
│  │ (Leader P0)│  │(Replica P0)│  │(Replica P0)│         │
│  └────────────┘  └────────────┘  └────────────┘         │
│         ▲                ▲                ▲               │
└─────────┼────────────────┼────────────────┼───────────────┘
          │                │                │
    ┌─────┴────┐     ┌─────┴────┐    ┌─────┴────┐
    │ Producer │     │ Producer │    │ Consumer │
    │    App   │     │    App   │    │   Group  │
    └──────────┘     └──────────┘    └──────────┘
          │                                 │
          └──── Topic: orders ──────────────┘
                   [P0][P1][P2]
```

### 🔑 Elementos Clave

#### **Broker**
- Servidor de Kafka que almacena y sirve mensajes
- Cada broker maneja miles de particiones
- Se identifican por un ID único

#### **Coordination: ZooKeeper vs KRaft**

**ZooKeeper (legacy, < 3.3)**:
- External dependency para metadata y coordination
- Bottleneck en clusters grandes (> 100K partitions)
- Split-brain risk durante network partitions

**KRaft Mode (GA en Kafka 3.3+)** ✅:
- Raft consensus protocol interno (no external deps)
- Metadata como event log (mejor scalability)
- Faster controller failover (< 1s vs ~30s ZK)
- Simplified operations y deployment

```bash
# Migración recomendada: ZK → KRaft para nuevos clusters
```

#### **Controller & Quorum**
- **Active Controller**: Broker con rol de metadata leader (KRaft) o ZK-elected
- Responsabilidades: Partition assignment, leader election, metadata replication
- **Quorum**: Majority-based (2N+1 para tolerar N fallos)

### 🔄 Replicación y Garantías ISR

```bash
# Topic con replicación y guarantees
kafka-topics.sh --create \
  --topic orders \
  --partitions 3 \
  --replication-factor 3 \
  --config min.insync.replicas=2 \
  --bootstrap-server localhost:9092

# Resultado:
# Topic: orders
# Partition 0: Leader=1, Replicas=[1,2,3], ISR=[1,2,3]
# Partition 1: Leader=2, Replicas=[2,3,1], ISR=[2,3,1]  
# Partition 2: Leader=3, Replicas=[3,1,2], ISR=[3,1,2]
```

**ISR (In-Sync Replicas)** - Réplicas que cumplen:
1. Heartbeat activo con leader (< `replica.lag.time.max.ms`)
2. Fetching data del leader continuamente
3. Fully caught-up (offset dentro de `replica.lag.time.max.ms`)

**Garantía de durabilidad**: `acks=all` + `min.insync.replicas=2`
- Producer solo recibe ACK si al menos 2 réplicas (leader + 1) confirman
- Trade-off: Mayor durabilidad vs mayor latencia

---

## 3. Topics y Particiones

### 📁 Topics

Un **Topic** es una categoría o feed de mensajes. Similar a una tabla en DB.

```bash
# Crear topic
kafka-topics.sh --create \
  --topic user-events \
  --partitions 5 \
  --replication-factor 2 \
  --bootstrap-server localhost:9092

# Listar topics
kafka-topics.sh --list --bootstrap-server localhost:9092

# Describir topic
kafka-topics.sh --describe \
  --topic user-events \
  --bootstrap-server localhost:9092
```

### 🗂️ Particiones

Cada topic se divide en **particiones**:

```
Topic: user-events (3 particiones)

Partition 0: [msg0][msg3][msg6][msg9]  ← Leader: Broker 1
Partition 1: [msg1][msg4][msg7]        ← Leader: Broker 2
Partition 2: [msg2][msg5][msg8]        ← Leader: Broker 3
```

### 🎯 Ventajas de Particiones

1. **Paralelismo**: Múltiples consumers pueden leer simultáneamente
2. **Escalabilidad**: Distribuir carga entre brokers
3. **Orden garantizado**: Dentro de cada partición
4. **Alta disponibilidad**: Réplicas en diferentes brokers

### 🔑 Partition Keys & Hashing Strategy

```java
// Sin key: Round-robin load balancing (no ordering)
producer.send(new ProducerRecord<>("orders", null, orderJson));

// Con key: Deterministic partitioning (ordering per key)
String key = order.getUserId(); 
producer.send(new ProducerRecord<>("orders", key, orderJson));

// Hashing interno de Kafka:
// partition = murmur2(keyBytes) % numPartitions
// Todos los mensajes con mismo key → misma partition (orden garantizado)
```

**👥 Caso de uso**: UserID como key para ordenar eventos por usuario.

**⚠️ Hot partitions**: Keys con alta cardinalidad desbalancean carga.

```java
// Solución: Composite keys para mejor distribución
String key = userId + "-" + (counter % 10); // Sub-partitioning
```

**🔄 Partition reassignment**: Añadir particiones cambia hashing.

```
Antes (3 partitions): hash("user123") % 3 = 2
Después (5 partitions): hash("user123") % 5 = 3  ❌ Ordering se rompe
```

**Best practice**: Overprovisionar particiones desde el inicio.

---

## 4. Producers: Envío de Mensajes

### 📤 Flujo de Envío

```
Producer → [Serializer] → [Partitioner] → [Buffer] → Batch → Broker
                                                              ↓
                                                         [Ack/Error]
```

### ⚙️ Configuración Básica

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

// Configuraciones importantes
props.put("acks", "all");              // 0, 1, all/-1
props.put("retries", 3);               // Reintentos automáticos
props.put("batch.size", 16384);        // 16 KB
props.put("linger.ms", 10);            // Esperar 10ms antes de enviar
props.put("compression.type", "snappy"); // none, gzip, snappy, lz4, zstd

KafkaProducer<String, String> producer = new KafkaProducer<>(props);
```

### 🎛️ Modos de Acknowledgment (acks)

| acks | Descripción | Durabilidad | Performance |
|------|-------------|-------------|-------------|
| `0` | No espera confirmación | ❌ Baja | ⚡ Máxima |
| `1` | Espera ack del líder | ⚠️ Media | 🔶 Alta |
| `all/-1` | Espera ack de todos los ISR | ✅ Alta | 🔻 Media |

### 📨 Envío de Mensajes

```java
// 1. Fire-and-forget (no verificar resultado)
producer.send(new ProducerRecord<>("orders", "order-key", orderJson));

// 2. Síncrono (bloquea hasta recibir ack)
try {
    RecordMetadata metadata = producer.send(
        new ProducerRecord<>("orders", "order-key", orderJson)
    ).get(); // Bloquea
    
    System.out.printf("Enviado a partition %d, offset %d%n",
        metadata.partition(), metadata.offset());
} catch (Exception e) {
    e.printStackTrace();
}

// 3. Asíncrono (callback)
producer.send(
    new ProducerRecord<>("orders", "order-key", orderJson),
    (metadata, exception) -> {
        if (exception != null) {
            System.err.println("Error enviando: " + exception.getMessage());
        } else {
            System.out.printf("Enviado a %s-%d offset %d%n",
                metadata.topic(), metadata.partition(), metadata.offset());
        }
    }
);

// Importante: Flush y cerrar
producer.flush(); // Forzar envío de mensajes en buffer
producer.close(); // Liberar recursos
```

### 🎲 Partitioner Personalizado

```java
public class CountryPartitioner implements Partitioner {
    @Override
    public int partition(String topic, Object key, byte[] keyBytes,
                        Object value, byte[] valueBytes, Cluster cluster) {
        int numPartitions = cluster.partitionCountForTopic(topic);
        
        if (key == null) {
            return ThreadLocalRandom.current().nextInt(numPartitions);
        }
        
        String country = (String) key;
        
        // Europa → Partition 0
        if (Arrays.asList("ES", "FR", "DE", "IT").contains(country)) {
            return 0;
        }
        // América → Partition 1
        else if (Arrays.asList("US", "MX", "BR", "AR").contains(country)) {
            return 1;
        }
        // Asia → Partition 2
        else {
            return 2;
        }
    }
    
    @Override
    public void close() {}
    
    @Override
    public void configure(Map<String, ?> configs) {}
}

// Usar partitioner personalizado
props.put("partitioner.class", "com.example.CountryPartitioner");
```

---

## 5. Consumers y Consumer Groups

### 📥 Consumer Básico

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("group.id", "my-consumer-group");
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

// Configuraciones importantes
props.put("enable.auto.commit", "true");    // true/false
props.put("auto.commit.interval.ms", "5000"); // Cada 5 segundos
props.put("auto.offset.reset", "earliest");  // earliest, latest, none

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Collections.singletonList("orders"));

try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
        
        for (ConsumerRecord<String, String> record : records) {
            System.out.printf("partition=%d, offset=%d, key=%s, value=%s%n",
                record.partition(), record.offset(), record.key(), record.value());
            
            // Procesar mensaje...
        }
    }
} finally {
    consumer.close();
}
```

### 👥 Consumer Groups

**Concepto**: Grupo de consumers que consumen en paralelo de un topic.

```
Topic: orders (3 particiones)

Consumer Group: "order-processors"
┌─────────────────────────────────────────────────┐
│ Consumer 1 → Partition 0                        │
│ Consumer 2 → Partition 1                        │
│ Consumer 3 → Partition 2                        │
└─────────────────────────────────────────────────┘

Si Consumer 2 falla:
┌─────────────────────────────────────────────────┐
│ Consumer 1 → Partition 0, 1  (rebalance)        │
│ Consumer 3 → Partition 2                        │
└─────────────────────────────────────────────────┘
```

### ⚖️ Rebalancing

Ocurre cuando:
- Un consumer se une o sale del grupo
- Se crean o eliminan particiones
- Un consumer deja de enviar heartbeats

```java
// Configurar heartbeat y session
props.put("session.timeout.ms", "10000");     // 10 segundos
props.put("heartbeat.interval.ms", "3000");   // 3 segundos
props.put("max.poll.interval.ms", "300000");  // 5 minutos
```

### 🎯 Commit de Offsets

#### **Auto-commit**
```java
props.put("enable.auto.commit", "true");
props.put("auto.commit.interval.ms", "5000");

// Kafka commitea automáticamente cada 5 segundos
// Riesgo: Si falla antes del commit, se reprocesarán mensajes
```

#### **Manual commit síncrono**
```java
props.put("enable.auto.commit", "false");

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    
    for (ConsumerRecord<String, String> record : records) {
        processRecord(record);
    }
    
    // Commitear después de procesar todo el batch
    consumer.commitSync();
}
```

#### **Manual commit asíncrono**
```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    
    for (ConsumerRecord<String, String> record : records) {
        processRecord(record);
    }
    
    consumer.commitAsync((offsets, exception) -> {
        if (exception != null) {
            log.error("Error commiting offsets", exception);
        }
    });
}
```

#### **Commit por partición**
```java
for (ConsumerRecord<String, String> record : records) {
    processRecord(record);
    
    // Commit específico por partición y offset
    Map<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();
    offsets.put(
        new TopicPartition(record.topic(), record.partition()),
        new OffsetAndMetadata(record.offset() + 1)
    );
    
    consumer.commitSync(offsets);
}
```

---

## 6. Kafka con Spring Boot

### 📦 Dependencias Maven

```xml
<dependencies>
    <!-- Spring Kafka -->
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
    
    <!-- Para testing -->
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### ⚙️ Configuración application.yml

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      acks: all
      retries: 3
      properties:
        linger.ms: 10
        compression.type: snappy
    
    consumer:
      group-id: my-app-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      auto-offset-reset: earliest
      enable-auto-commit: false
      properties:
        spring.json.trusted.packages: com.example.dto
    
    listener:
      ack-mode: manual
      concurrency: 3
```

### 📤 Producer con Spring Boot (Enterprise Pattern)

```java
@Service
@Slf4j
@RequiredArgsConstructor
public class OrderProducerService {
    
    private final KafkaTemplate<String, OrderDTO> kafkaTemplate;
    private static final String TOPIC = "orders";
    
    /**
     * Send con CompletableFuture (non-blocking, async)
     * Best practice para high-throughput scenarios
     */
    public CompletableFuture<SendResult<String, OrderDTO>> sendOrderAsync(OrderDTO order) {
        ProducerRecord<String, OrderDTO> record = buildRecord(order);
        
        return kafkaTemplate.send(record)
            .whenComplete((result, ex) -> {
                if (ex != null) {
                    log.error("Failed to send order {} to {}: {}", 
                        order.getId(), TOPIC, ex.getMessage());
                    // Metric: increment failure counter
                    metricsService.incrementFailure(TOPIC);
                } else {
                    RecordMetadata metadata = result.getRecordMetadata();
                    log.debug("Order {} sent to partition {} offset {}",
                        order.getId(), metadata.partition(), metadata.offset());
                    // Metric: record latency
                    metricsService.recordLatency(metadata);
                }
            });
    }
    
    /**
     * Send síncrono con retry manual (backpressure control)
     * Use case: Critical operations que requieren confirmación
     */
    @Retryable(
        retryFor = {KafkaException.class},
        maxAttempts = 3,
        backoff = @Backoff(delay = 1000, multiplier = 2)
    )
    public void sendOrderSync(OrderDTO order) {
        try {
            SendResult<String, OrderDTO> result = kafkaTemplate
                .send(buildRecord(order))
                .get(5, TimeUnit.SECONDS);
                
            log.info("Order {} committed at offset {}", 
                order.getId(), result.getRecordMetadata().offset());
                
        } catch (TimeoutException e) {
            throw new KafkaTimeoutException("Producer timeout", e);
        } catch (InterruptedException | ExecutionException e) {
            Thread.currentThread().interrupt();
            throw new KafkaException("Send failed", e);
        }
    }
    
    /**
     * Batch sending (optimizado para throughput)
     */
    public List<CompletableFuture<SendResult<String, OrderDTO>>> sendBatch(
            List<OrderDTO> orders) {
        
        return orders.stream()
            .map(this::sendOrderAsync)
            .collect(Collectors.toList());
    }
    
    private ProducerRecord<String, OrderDTO> buildRecord(OrderDTO order) {
        ProducerRecord<String, OrderDTO> record = new ProducerRecord<>(
            TOPIC,
            null, // partition (null = usar partitioner)
            System.currentTimeMillis(), // timestamp
            order.getUserId(), // key
            order // value
        );
        
        // Headers para tracing/debugging
        record.headers().add("correlation-id", 
            MDC.get("correlationId").getBytes(StandardCharsets.UTF_8));
        record.headers().add("source", "order-service".getBytes());
        record.headers().add("version", "v1".getBytes());
        
        return record;
    }
}
```

### 📥 Consumer con Spring Boot (Production-Ready)

```java
@Service
@Slf4j
public class OrderConsumerService {
    
    private final OrderProcessingService processingService;
    private final MeterRegistry meterRegistry;
    
    /**
     * Consumer con manual acknowledgment y error handling
     * Best practice: Control granular de commits
     */
    @KafkaListener(
        topics = "orders",
        groupId = "order-processor-v1",
        concurrency = "3", // 3 threads = 3 consumers en el group
        properties = {
            "max.poll.records=100",
            "max.poll.interval.ms=300000" // 5 min
        }
    )
    public void consumeOrder(
            @Payload OrderDTO order,
            @Header(KafkaHeaders.RECEIVED_PARTITION) int partition,
            @Header(KafkaHeaders.OFFSET) long offset,
            @Header(KafkaHeaders.RECEIVED_KEY) String key,
            @Header(KafkaHeaders.RECEIVED_TIMESTAMP) long timestamp,
            Acknowledgment ack) {
        
        MDC.put("orderId", order.getId());
        MDC.put("partition", String.valueOf(partition));
        
        try {
            log.info("Processing order {} from partition {} offset {}", 
                order.getId(), partition, offset);
            
            // Business logic
            processingService.process(order);
            
            // Commit solo si procesamiento exitoso
            ack.acknowledge();
            
            // Metrics
            meterRegistry.counter("orders.processed", 
                "status", "success").increment();
                
        } catch (RecoverableException e) {
            log.warn("Recoverable error processing order {}: {}", 
                order.getId(), e.getMessage());
            // No ack → mensaje se reprocesará
            throw e; // Trigger retry via error handler
            
        } catch (NonRecoverableException e) {
            log.error("Non-recoverable error processing order {}", 
                order.getId(), e);
            
            // Commit para evitar bloqueo (irá a DLQ)
            ack.acknowledge();
            
            meterRegistry.counter("orders.processed", 
                "status", "failed").increment();
        } finally {
            MDC.clear();
        }
    }
    
    /**
     * Batch consumer (throughput optimization)
     * Use case: Processing que se beneficia de batching (DB inserts, etc.)
     */
    @KafkaListener(
        topics = "orders",
        groupId = "order-batch-processor",
        containerFactory = "batchListenerContainerFactory"
    )
    public void consumeBatch(
            List<OrderDTO> orders,
            List<Long> offsets,
            List<Integer> partitions,
            Acknowledgment ack) {
        
        log.info("Processing batch of {} orders", orders.size());
        
        try {
            // Bulk processing
            processingService.processBatch(orders);
            
            // Single commit para todo el batch (efficiency)
            ack.acknowledge();
            
        } catch (Exception e) {
            log.error("Batch processing failed, will retry", e);
            throw e; // Reprocess entire batch
        }
    }
    
    /**
     * Filtered consumer (pre-filtering no deseados)
     */
    @KafkaListener(
        topics = "orders",
        groupId = "premium-order-processor",
        containerFactory = "filteringListenerContainerFactory"
    )
    public void consumePremiumOrders(OrderDTO order) {
        // Solo llega si order.isPremium() == true
        log.info("Processing premium order: {}", order.getId());
        processingService.processPremiumOrder(order);
    }
}
```

### 🔧 Configuración Avanzada

```java
@Configuration
@EnableKafka
public class KafkaConfig {
    
    // Producer con configuración personalizada
    @Bean
    public ProducerFactory<String, OrderDTO> producerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        config.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        config.put(ProducerConfig.ACKS_CONFIG, "all");
        config.put(ProducerConfig.RETRIES_CONFIG, 3);
        config.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy");
        
        return new DefaultKafkaProducerFactory<>(config);
    }
    
    @Bean
    public KafkaTemplate<String, OrderDTO> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
    
    // Consumer con configuración personalizada
    @Bean
    public ConsumerFactory<String, OrderDTO> consumerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ConsumerConfig.GROUP_ID_CONFIG, "order-processors");
        config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
        config.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        config.put(JsonDeserializer.TRUSTED_PACKAGES, "com.example.dto");
        
        return new DefaultKafkaConsumerFactory<>(config);
    }
    
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, OrderDTO> 
            kafkaListenerContainerFactory() {
        
        ConcurrentKafkaListenerContainerFactory<String, OrderDTO> factory =
            new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(3); // 3 threads por listener
        factory.getContainerProperties().setAckMode(AckMode.MANUAL);
        
        return factory;
    }
    
    // Container factory con filtro
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, OrderDTO>
            filterKafkaListenerContainerFactory() {
        
        ConcurrentKafkaListenerContainerFactory<String, OrderDTO> factory =
            new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        
        // Filtrar solo orders premium
        factory.setRecordFilterStrategy(record -> {
            OrderDTO order = (OrderDTO) record.value();
            return !order.isPremium(); // true = filtrar (no procesar)
        });
        
        return factory;
    }
}
```

---

## 7. Configuración de Producers en Java

### 🎛️ Parámetros Importantes

```java
Properties props = new Properties();

// ========== CONEXIÓN ==========
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
props.put(ProducerConfig.CLIENT_ID_CONFIG, "my-producer-1");

// ========== SERIALIZACIÓN ==========
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, 
    "org.apache.kafka.common.serialization.StringSerializer");
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
    "org.apache.kafka.common.serialization.StringSerializer");

// ========== DURABILIDAD ==========
props.put(ProducerConfig.ACKS_CONFIG, "all");  // 0, 1, all
props.put(ProducerConfig.RETRIES_CONFIG, 3);
props.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, 5);
props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true); // Evitar duplicados

// ========== PERFORMANCE ==========
props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);        // 16 KB
props.put(ProducerConfig.LINGER_MS_CONFIG, 10);            // Esperar 10ms
props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);  // 32 MB buffer
props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy"); // gzip, snappy, lz4, zstd

// ========== TIMEOUTS ==========
props.put(ProducerConfig.REQUEST_TIMEOUT_MS_CONFIG, 30000);   // 30s
props.put(ProducerConfig.DELIVERY_TIMEOUT_MS_CONFIG, 120000); // 2min
```

### 🔒 Idempotent Producer (Exactly-Once Delivery)

**Problema**: Network failures + retries → duplicados en broker.

**Solución**: Producer idempotence con sequence numbers.

```java
// Idempotencia (default en Kafka 3.0+)
props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);

// Requisitos automáticos:
// - acks = all (forced)
// - retries = Integer.MAX_VALUE (forced)  
// - max.in.flight.requests.per.connection ≤ 5 (forced)
```

**Mecanismo interno**:
1. Producer obtiene **Producer ID (PID)** único del broker
2. Cada mensaje tiene **sequence number** incremental
3. Broker detecta duplicados: mismo PID + sequence number

```
Producer (PID=123):
  Msg1: PID=123, Seq=0 ✓
  Msg2: PID=123, Seq=1 ✓
  Msg2: PID=123, Seq=1 ❌ Duplicado, ignorado
```

**Scope**: Per partition, per session (PID reset si producer se reinicia).

**⚠️ Limitación**: No protege contra duplicados cross-session.

**Upgrade path**: Usar **transactional producer** para exactly-once cross-session.

### ⚡ Performance Tuning: Throughput vs Latency

```java
// 🚀 HIGH THROUGHPUT (batch-oriented, +latency)
props.put(ProducerConfig.BATCH_SIZE_CONFIG, 131072);      // 128 KB batches
props.put(ProducerConfig.LINGER_MS_CONFIG, 100);          // Wait 100ms for batch
props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "lz4"); // Best compression/CPU
props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 67108864); // 64 MB buffer
props.put(ProducerConfig.ACKS_CONFIG, "1");               // Leader only

// 🔥 LOW LATENCY (individual sends, -throughput)
props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);       // Small batches
props.put(ProducerConfig.LINGER_MS_CONFIG, 0);            // Send immediately
props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "none");// No CPU overhead
props.put(ProducerConfig.ACKS_CONFIG, "all");             // Durability first

// ⚖️ BALANCED (production default)
props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);       // 16 KB
props.put(ProducerConfig.LINGER_MS_CONFIG, 10);           // Small latency hit
props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy");// Fast compression
props.put(ProducerConfig.ACKS_CONFIG, "all");             // Data safety
props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);// Exactly-once
```

**Compression benchmarks** (approx):
- `lz4`: 4x ratio, fastest CPU
- `snappy`: 3x ratio, balanced
- `gzip`: 5x ratio, slower CPU
- `zstd`: Best ratio, moderate CPU (Kafka 2.1+)

---

## 8. Configuración de Consumers en Java

### 🎛️ Parámetros Importantes

```java
Properties props = new Properties();

// ========== CONEXIÓN ==========
props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
props.put(ConsumerConfig.GROUP_ID_CONFIG, "my-consumer-group");
props.put(ConsumerConfig.CLIENT_ID_CONFIG, "consumer-1");

// ========== DESERIALIZACIÓN ==========
props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, 
    "org.apache.kafka.common.serialization.StringDeserializer");
props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, 
    "org.apache.kafka.common.serialization.StringDeserializer");

// ========== OFFSET MANAGEMENT ==========
props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest"); // earliest, latest, none
props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "5000");

// ========== FETCHING ==========
props.put(ConsumerConfig.FETCH_MIN_BYTES_CONFIG, 1);           // Min bytes por fetch
props.put(ConsumerConfig.FETCH_MAX_WAIT_MS_CONFIG, 500);       // Max espera
props.put(ConsumerConfig.MAX_PARTITION_FETCH_BYTES_CONFIG, 1048576); // 1 MB por partición
props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 500);        // Max registros por poll

// ========== HEARTBEAT & SESSION ==========
props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 10000);    // 10s
props.put(ConsumerConfig.HEARTBEAT_INTERVAL_MS_CONFIG, 3000);  // 3s
props.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, 300000); // 5min
```

### 🎯 Offset Reset Strategies

```java
// EARLIEST: Replay desde el primer mensaje disponible (full history)
props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
// Use case: Data reprocessing, backfill, new consumer group

// LATEST: Ignorar mensajes pasados, consumir solo nuevos (default)
props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "latest"); 
// Use case: Real-time processing, no interesa historial

// NONE: Fail-fast si no hay offset commiteado (strict)
props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "none");
// Use case: Production safety, evitar pérdida de mensajes accidental
```

**Trigger condition**: Solo aplica cuando `no existe offset commiteado` para el consumer group.

**Best practice**: `earliest` en development, `none` en production.

### 🔄 Rebalance Listeners & Graceful Handling

```java
consumer.subscribe(
    Collections.singletonList("orders"),
    new ConsumerRebalanceListener() {
        
        @Override
        public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
            // CRITICAL: Commit offsets antes de perder ownership
            log.info("Rebalancing... revoking partitions: {}", partitions);
            
            try {
                // Flush pending work
                processingService.flushPendingRecords();
                
                // Synchronous commit (blocking, ensure success)
                consumer.commitSync();
                
            } catch (CommitFailedException e) {
                log.error("Failed to commit on rebalance", e);
                // Mensajes se reprocesarán (at-least-once)
            }
        }
        
        @Override
        public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
            log.info("Assigned partitions: {}", partitions);
            
            // Optional: Inicializar estado per-partition
            for (TopicPartition partition : partitions) {
                processingService.initializePartition(partition);
                
                // Advanced: Seek a offset específico (recovery scenarios)
                // consumer.seek(partition, recoveryOffset);
            }
        }
        
        @Override
        public void onPartitionsLost(Collection<TopicPartition> partitions) {
            // Kafka 2.4+: Partition reassigned sin graceful revoke
            // (e.g., consumer timeout)
            log.warn("Partitions LOST (forced rebalance): {}", partitions);
            
            // No commit aquí, partitions ya no owned
            processingService.cleanupLostPartitions(partitions);
        }
    }
);
```

**Rebalancing triggers**:
1. Consumer join/leave group
2. Consumer heartbeat timeout (`session.timeout.ms`)
3. Consumer no llama `poll()` en `max.poll.interval.ms`
4. Partition count change
5. Topic subscription change

### 📌 Seek: Control Manual de Offsets

```java
// Leer desde offset específico
TopicPartition partition = new TopicPartition("orders", 0);
consumer.assign(Collections.singletonList(partition));
consumer.seek(partition, 100); // Leer desde offset 100

// Leer desde el inicio
consumer.seekToBeginning(Collections.singletonList(partition));

// Leer desde el final
consumer.seekToEnd(Collections.singletonList(partition));

// Leer desde timestamp
long timestamp = System.currentTimeMillis() - (24 * 60 * 60 * 1000); // Hace 24h
Map<TopicPartition, Long> timestampsToSearch = new HashMap<>();
timestampsToSearch.put(partition, timestamp);

Map<TopicPartition, OffsetAndTimestamp> offsets = 
    consumer.offsetsForTimes(timestampsToSearch);

consumer.seek(partition, offsets.get(partition).offset());
```

---

## 9. Serialización y Deserialización

### 🔧 Serializadores Built-in

```java
// String
StringSerializer / StringDeserializer

// Integer
IntegerSerializer / IntegerDeserializer

// Long
LongSerializer / LongDeserializer

// ByteArray
ByteArraySerializer / ByteArrayDeserializer
```

### 📦 JSON Serialization (Spring Kafka)

```java
// DTO
@Data
@NoArgsConstructor
@AllArgsConstructor
public class OrderDTO {
    private String orderId;
    private String userId;
    private BigDecimal amount;
    private LocalDateTime createdAt;
    private boolean premium;
}

// Producer config
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);

// Consumer config
props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
props.put(JsonDeserializer.TRUSTED_PACKAGES, "com.example.dto");
props.put(JsonDeserializer.VALUE_DEFAULT_TYPE, OrderDTO.class);
```

### 🎨 Serializador Personalizado

```java
public class OrderSerializer implements Serializer<Order> {
    
    private final ObjectMapper objectMapper = new ObjectMapper();
    
    @Override
    public void configure(Map<String, ?> configs, boolean isKey) {
        objectMapper.registerModule(new JavaTimeModule());
    }
    
    @Override
    public byte[] serialize(String topic, Order order) {
        if (order == null) return null;
        
        try {
            return objectMapper.writeValueAsBytes(order);
        } catch (JsonProcessingException e) {
            throw new SerializationException("Error serializing Order", e);
        }
    }
    
    @Override
    public void close() {}
}

public class OrderDeserializer implements Deserializer<Order> {
    
    private final ObjectMapper objectMapper = new ObjectMapper();
    
    @Override
    public void configure(Map<String, ?> configs, boolean isKey) {
        objectMapper.registerModule(new JavaTimeModule());
    }
    
    @Override
    public Order deserialize(String topic, byte[] data) {
        if (data == null) return null;
        
        try {
            return objectMapper.readValue(data, Order.class);
        } catch (IOException e) {
            throw new SerializationException("Error deserializing Order", e);
        }
    }
    
    @Override
    public void close() {}
}
```

### 🔍 Avro Serialization (Schema Registry)

```xml
<dependency>
    <groupId>io.confluent</groupId>
    <artifactId>kafka-avro-serializer</artifactId>
    <version>7.5.0</version>
</dependency>
```

```java
// Schema: order.avsc
{
  "type": "record",
  "name": "Order",
  "namespace": "com.example.avro",
  "fields": [
    {"name": "orderId", "type": "string"},
    {"name": "userId", "type": "string"},
    {"name": "amount", "type": "double"}
  ]
}

// Producer config
props.put("schema.registry.url", "http://localhost:8081");
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
    KafkaAvroSerializer.class);

// Consumer config
props.put("schema.registry.url", "http://localhost:8081");
props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, 
    KafkaAvroDeserializer.class);
props.put("specific.avro.reader", "true");
```

---

## 10. Manejo de Errores y Retry

### ❌ Error Classification

**Retriable** (transient, reintentar):
- `TimeoutException`: Network/broker latency
- `NotLeaderOrFollowerException`: Leader election en curso
- `NotEnoughReplicasException`: ISR temporalmente bajo
- Database connection failures
- Rate limiting (429 errors)

**Non-Retriable** (permanent, no reintentar):
- `RecordTooLargeException`: Mensaje excede `max.message.bytes`
- `SerializationException`: Formato inválido
- `InvalidTopicException`: Topic no existe
- Business validation errors
- Authorization errors (403)

### 🔄 Retry Strategy (Spring Kafka 2.7+)

```java
@Configuration
public class KafkaErrorHandlingConfig {
    
    @Bean
    public DefaultErrorHandler errorHandler(
            KafkaTemplate<String, Object> kafkaTemplate) {
        
        // DLQ configuration
        DeadLetterPublishingRecoverer recoverer = 
            new DeadLetterPublishingRecoverer(kafkaTemplate,
                (record, ex) -> {
                    // Route to topic-specific DLQ
                    String dlqTopic = record.topic() + ".DLT";
                    
                    // Preserve partition for ordering (optional)
                    // return new TopicPartition(dlqTopic, record.partition());
                    
                    // Single DLQ partition (simpler ops)
                    return new TopicPartition(dlqTopic, 0);
                }
            );
        
        // Exponential backoff: 1s, 2s, 4s, 8s, 16s
        ExponentialBackOffWithMaxRetries backOff = 
            new ExponentialBackOffWithMaxRetries(5);
        backOff.setInitialInterval(1000L);
        backOff.setMultiplier(2.0);
        backOff.setMaxInterval(30000L); // Cap at 30s
        
        DefaultErrorHandler handler = new DefaultErrorHandler(
            recoverer, backOff);
        
        // Non-retriable exceptions → inmediato a DLQ
        handler.addNotRetryableExceptions(
            SerializationException.class,
            DeserializationException.class,
            ValidationException.class,
            MessageConversionException.class
        );
        
        // Listener para auditoría
        handler.setRetryListeners((record, ex, deliveryAttempt) -> {
            log.warn("Retry attempt {} for record from {}-{} offset {}: {}",
                deliveryAttempt,
                record.topic(),
                record.partition(),
                record.offset(),
                ex.getMessage());
            
            metricsRegistry.counter("kafka.consumer.retries",
                "topic", record.topic(),
                "attempt", String.valueOf(deliveryAttempt)
            ).increment();
        });
        
        return handler;
    }
    
    /**
     * Alternative: Custom error handler con business logic
     */
    @Bean
    public ConsumerAwareListenerErrorHandler customErrorHandler() {
        return (message, exception, consumer) -> {
            log.error("Error in consumer: {}", exception.getMessage());
            
            if (exception.getCause() instanceof RecoverableException) {
                // Seek back para retry (manual reprocess)
                ConsumerRecord<?, ?> record = 
                    (ConsumerRecord<?, ?>) message.getPayload();
                TopicPartition partition = new TopicPartition(
                    record.topic(), record.partition());
                consumer.seek(partition, record.offset());
            }
            
            return null; // Don't propagate
        };
    }
}
```

### 📮 Dead Letter Queue (DLQ)

```java
@Configuration
public class KafkaConfig {
    
    @Bean
    public DeadLetterPublishingRecoverer deadLetterPublishingRecoverer(
            KafkaTemplate<String, Object> kafkaTemplate) {
        
        return new DeadLetterPublishingRecoverer(kafkaTemplate,
            (record, ex) -> {
                // Enviar a DLQ con sufijo .DLT
                String dlqTopic = record.topic() + ".DLT";
                return new TopicPartition(dlqTopic, record.partition());
            }
        );
    }
    
    @Bean
    public DefaultErrorHandler errorHandler(
            DeadLetterPublishingRecoverer recoverer) {
        
        return new DefaultErrorHandler(recoverer, 
            new FixedBackOff(1000L, 3L));
    }
}

// Consumer para DLQ
@KafkaListener(topics = "orders.DLT", groupId = "dlq-processor")
public void processDLQ(
        @Payload OrderDTO order,
        @Header(KafkaHeaders.EXCEPTION_MESSAGE) String exceptionMessage,
        @Header(KafkaHeaders.EXCEPTION_STACKTRACE) String stacktrace) {
    
    log.error("Processing DLQ message. Order: {}, Error: {}", 
        order, exceptionMessage);
    
    // Notificar, guardar en DB, enviar alerta, etc.
}
```

### 🛡️ Circuit Breaker Pattern

```java
@Service
public class ResilientOrderConsumerService {
    
    private final CircuitBreaker circuitBreaker;
    
    public ResilientOrderConsumerService() {
        CircuitBreakerConfig config = CircuitBreakerConfig.custom()
            .failureRateThreshold(50)                  // 50% errores
            .waitDurationInOpenState(Duration.ofSeconds(30)) // Esperar 30s
            .slidingWindowSize(10)                     // Ventana de 10 llamadas
            .build();
        
        CircuitBreakerRegistry registry = CircuitBreakerRegistry.of(config);
        this.circuitBreaker = registry.circuitBreaker("orderProcessing");
    }
    
    @KafkaListener(topics = "orders", groupId = "order-processors")
    public void consumeOrder(OrderDTO order) {
        Try.of(() -> circuitBreaker.executeSupplier(() -> {
            processOrder(order);
            return null;
        }))
        .onFailure(ex -> {
            if (ex instanceof CallNotPermittedException) {
                log.warn("Circuit breaker OPEN. Skipping order {}", order.getId());
                // Enviar a DLQ o reintentar más tarde
            } else {
                log.error("Error processing order", ex);
            }
        });
    }
    
    private void processOrder(OrderDTO order) {
        // Lógica que puede fallar
        externalService.call(order);
    }
}
```

---

## 11. Kafka Streams

### 🌊 Kafka Streams vs Alternatives

**Kafka Streams** es una librería Java para stream processing que corre **embedded** en tu aplicación.

| Feature | Kafka Streams | Flink | Spark Streaming |
|---------|--------------|-------|----------------|
| **Deployment** | Embedded lib | Separate cluster | Separate cluster |
| **Latency** | Very low (ms) | Low (ms) | Micro-batch (seconds) |
| **Scalability** | Horizontal (instances) | High | Very high |
| **State mgmt** | RocksDB local | Distributed | RDD/DataFrame |
| **Exactly-once** | ✅ Native | ✅ | ✅ |
| **Complexity** | Low | Medium | High |
| **Use case** | Kafka-centric | Complex CEP | Batch + streaming |

**Cuándo usar Kafka Streams**:
- ✅ Tu data pipeline está 100% en Kafka
- ✅ Necesitas low-latency processing (< 100ms)
- ✅ Prefieres simplicidad operacional (no cluster separado)
- ❌ Necesitas joins con external systems (usar Flink)
- ❌ Processing muy complejo (usar Flink)

### 📚 Conceptos Core

**KStream**: Unbounded stream (append-only log)
```
KStream<K,V>: [evt1, evt2, evt3, ...]
```

**KTable**: Changelog stream (latest value per key)
```
KTable<K,V>: {key1=valA, key2=valB}  // Mutable state
```

**GlobalKTable**: Replicated KTable (todos los datos en cada instancia)
```
GlobalKTable<K,V>: Broadcast para joins
```

### � Stream Processing Patterns

#### **Pattern 1: Stateless Transformation (Map/Filter)**

```java
@Configuration
@EnableKafkaStreams
public class StreamProcessingConfig {
    
    @Bean
    public KafkaStreamsConfiguration streamsConfig() {
        Map<String, Object> props = new HashMap<>();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "order-processor-v1");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, 
            Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, 
            Serdes.String().getClass());
        
        // Production configs
        props.put(StreamsConfig.PROCESSING_GUARANTEE_CONFIG, 
            "exactly_once_v2"); // Exactly-once semantics
        props.put(StreamsConfig.NUM_STREAM_THREADS_CONFIG, 4); // Parallelism
        props.put(StreamsConfig.COMMIT_INTERVAL_MS_CONFIG, 100); // 100ms commits
        props.put(StreamsConfig.CACHE_MAX_BYTES_BUFFERING_CONFIG, 10 * 1024 * 1024); // 10MB
        
        return new KafkaStreamsConfiguration(props);
    }
    
    @Bean
    public KStream<String, OrderEvent> processOrders(
            StreamsBuilder builder) {
        
        // Input stream
        KStream<String, OrderEvent> orders = builder
            .stream("raw-orders", 
                Consumed.with(Serdes.String(), orderEventSerde()));
        
        // Transformaciones stateless
        KStream<String, OrderEvent> processedOrders = orders
            .filter((key, order) -> order.getAmount().compareTo(BigDecimal.ZERO) > 0)
            .filterNot((key, order) -> order.getStatus().equals("CANCELLED"))
            .mapValues(order -> {
                // Enrich: Add timestamp, normalize
                order.setProcessedAt(Instant.now());
                order.setUserId(order.getUserId().toLowerCase());
                return order;
            })
            .peek((key, order) -> 
                log.debug("Processing order: {}", order.getId()));
        
        // Branch por tipo
        Map<String, KStream<String, OrderEvent>> branches = processedOrders
            .split(Named.as("order-type-"))
            .branch((key, order) -> order.isPremium(), 
                Branched.as("premium"))
            .branch((key, order) -> order.getAmount().compareTo(new BigDecimal("1000")) > 0,
                Branched.as("high-value"))
            .defaultBranch(Branched.as("standard"));
        
        // Output streams
        branches.get("order-type-premium")
            .to("premium-orders", Produced.with(Serdes.String(), orderEventSerde()));
        
        branches.get("order-type-high-value")
            .to("high-value-orders", Produced.with(Serdes.String(), orderEventSerde()));
        
        branches.get("order-type-standard")
            .to("standard-orders", Produced.with(Serdes.String(), orderEventSerde()));
        
        return processedOrders;
    }
}
```

#### **Pattern 2: Aggregation con Windowing**

```java
/**
 * Real-time metrics: Orders per user en ventanas de 1 minuto
 */
@Bean
public KTable<Windowed<String>, OrderMetrics> orderMetricsStream(
        StreamsBuilder builder) {
    
    KStream<String, OrderEvent> orders = builder
        .stream("orders", Consumed.with(Serdes.String(), orderEventSerde()));
    
    // Tumbling window: Non-overlapping, fixed-size
    TimeWindows tumblingWindow = TimeWindows
        .ofSizeWithNoGrace(Duration.ofMinutes(1));
    
    KTable<Windowed<String>, OrderMetrics> metrics = orders
        .groupByKey(Grouped.with(Serdes.String(), orderEventSerde()))
        .windowedBy(tumblingWindow)
        .aggregate(
            OrderMetrics::new, // Initializer
            (key, order, metrics) -> { // Aggregator
                metrics.incrementCount();
                metrics.addAmount(order.getAmount());
                metrics.updateAverage();
                return metrics;
            },
            Materialized.<String, OrderMetrics, WindowStore<Bytes, byte[]>>as(
                "order-metrics-store")
                .withKeySerde(Serdes.String())
                .withValueSerde(orderMetricsSerde())
                .withRetention(Duration.ofHours(24)) // Keep 24h of windows
        );
    
    // Materialize to output topic
    metrics.toStream()
        .map((windowedKey, metrics) -> {
            String key = String.format("%s@%d",
                windowedKey.key(),
                windowedKey.window().start());
            return KeyValue.pair(key, metrics);
        })
        .to("order-metrics", Produced.with(Serdes.String(), orderMetricsSerde()));
    
    return metrics;
}
```

#### **Pattern 3: Stream-Table Join (Enrichment)**

```java
/**
 * Enrich orders con user profile data
 */
@Bean
public KStream<String, EnrichedOrder> enrichOrders(
        StreamsBuilder builder) {
    
    // Stream: Incoming orders
    KStream<String, OrderEvent> orders = builder
        .stream("orders",
            Consumed.with(Serdes.String(), orderEventSerde()))
        .selectKey((key, order) -> order.getUserId()); // Re-key por userId
    
    // Table: User profiles (changelog)
    KTable<String, UserProfile> users = builder
        .table("user-profiles",
            Consumed.with(Serdes.String(), userProfileSerde()),
            Materialized.as("user-profiles-store"));
    
    // Inner join: Solo orders con user existente
    KStream<String, EnrichedOrder> enrichedOrders = orders
        .join(users,
            (order, user) -> EnrichedOrder.builder()
                .orderId(order.getId())
                .userId(user.getId())
                .userName(user.getName())
                .userTier(user.getTier())
                .userEmail(user.getEmail())
                .orderAmount(order.getAmount())
                .orderItems(order.getItems())
                .build(),
            Joined.with(
                Serdes.String(),
                orderEventSerde(),
                userProfileSerde()
            )
        );
    
    enrichedOrders.to("enriched-orders",
        Produced.with(Serdes.String(), enrichedOrderSerde()));
    
    return enrichedOrders;
}
```

---

## 12. Transacciones en Kafka (Exactly-Once Semantics)

### 🔒 Garantías de Entrega

| Semantic | Guarantee | Implementation |
|----------|-----------|----------------|
| **At-most-once** | 0 o 1 vez | `acks=0`, no retries |
| **At-least-once** | ≥1 vez (duplicados OK) | `acks=all` + retries |
| **Exactly-once** | Exactamente 1 vez | Idempotence + Transactions |

**Exactly-Once Semantics (EOS)** garantiza:
1. Producer: No duplicados (idempotence)
2. Consumer-Producer: Atomic read-process-write
3. Streams: End-to-end exactly-once

### ⚙️ Configuración Transactional Producer

```java
// PRODUCER (transactional)
Properties producerProps = new Properties();
producerProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
producerProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, 
    StringSerializer.class);
producerProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
    StringSerializer.class);

// Transactional settings
producerProps.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, 
    "transaction-producer-1"); // Unique per instance
producerProps.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true); // Required
producerProps.put(ProducerConfig.ACKS_CONFIG, "all"); // Required

KafkaProducer<String, String> producer = new KafkaProducer<>(producerProps);
producer.initTransactions(); // Initialize before first transaction

// CONSUMER (read_committed isolation)
Properties consumerProps = new Properties();
consumerProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
consumerProps.put(ConsumerConfig.GROUP_ID_CONFIG, "transaction-consumer");
consumerProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, 
    StringDeserializer.class);
consumerProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, 
    StringDeserializer.class);

// Critical: Isolation level
consumerProps.put(ConsumerConfig.ISOLATION_LEVEL_CONFIG, "read_committed");
consumerProps.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProps);
consumer.subscribe(Collections.singletonList("input-topic"));
```

**Isolation Levels**:
- `read_uncommitted` (default): Lee todo, incluso transacciones en curso
- `read_committed`: Solo lee mensajes de transacciones commiteadas

### 🔄 Pattern: Consume-Transform-Produce (Exactly-Once)

```java
/**
 * Atomic pipeline: Read from Kafka → Process → Write to Kafka
 * Guarantees: No duplicates, no data loss, no partial commits
 */
public class ExactlyOnceProcessor {
    
    public void processMessages() {
        while (true) {
            ConsumerRecords<String, String> records = 
                consumer.poll(Duration.ofMillis(100));
            
            if (!records.isEmpty()) {
                try {
                    // BEGIN TRANSACTION
                    producer.beginTransaction();
                    
                    // Process and send
                    for (ConsumerRecord<String, String> record : records) {
                        String transformed = transform(record.value());
                        
                        ProducerRecord<String, String> outputRecord = 
                            new ProducerRecord<>(
                                "output-topic",
                                record.key(),
                                transformed
                            );
                        
                        producer.send(outputRecord);
                    }
                    
                    // COMMIT OFFSETS inside transaction
                    Map<TopicPartition, OffsetAndMetadata> offsets = 
                        buildOffsetMap(records);
                    
                    producer.sendOffsetsToTransaction(
                        offsets,
                        consumer.groupMetadata()
                    );
                    
                    // COMMIT TRANSACTION (atomic)
                    producer.commitTransaction();
                    
                    log.debug("Transaction committed for {} records", 
                        records.count());
                    
                } catch (ProducerFencedException | OutOfOrderSequenceException e) {
                    // Fatal: Producer zombie or out-of-order
                    log.error("Fatal producer error, closing", e);
                    producer.close();
                    break;
                    
                } catch (KafkaException e) {
                    // Retriable error
                    log.error("Transient error, aborting transaction", e);
                    producer.abortTransaction();
                    
                    // Seek to last committed offset (reprocess)
                    consumer.seek(/* last committed */);
                }
            }
        }
    }
    
    private Map<TopicPartition, OffsetAndMetadata> buildOffsetMap(
            ConsumerRecords<String, String> records) {
        
        Map<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();
        
        for (TopicPartition partition : records.partitions()) {
            List<ConsumerRecord<String, String>> partitionRecords = 
                records.records(partition);
            
            long lastOffset = partitionRecords
                .get(partitionRecords.size() - 1)
                .offset();
            
            offsets.put(partition, new OffsetAndMetadata(lastOffset + 1));
        }
        
        return offsets;
    }
    
    private String transform(String input) {
        // Business logic
        return input.toUpperCase();
    }
}
```

### ⚠️ Transactional Caveats

**Performance impact**:
- ~20-30% latency increase (extra RTTs para commit protocol)
- Higher CPU usage (state management)

**Operational complexity**:
- `transactional.id` must be unique per producer instance
- Zombie fencing: Old producers fenced out automáticamente
- Transaction timeout default: 60s (`transaction.timeout.ms`)

**When NOT to use**:
- High-throughput, latency-sensitive (use at-least-once + idempotent consumer)
- Simple pub/sub (overkill)

**When to use**:
- Financial transactions
- Exactly-once ETL pipelines
- Stateful stream processing con Kafka Streams

### 🌱 Transacciones en Spring Kafka

```java
@Service
public class TransactionalService {
    
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;
    
    @Transactional("kafkaTransactionManager")
    public void processMessage(String input) {
        // Leer, procesar y escribir en una transacción
        String transformed = transform(input);
        
        kafkaTemplate.send("output-topic", transformed);
        
        // Si lanza excepción, la transacción se hace rollback
        if (transformed.contains("error")) {
            throw new RuntimeException("Processing failed");
        }
    }
}

@Configuration
public class KafkaTransactionConfig {
    
    @Bean
    public ProducerFactory<String, String> producerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        config.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        config.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "tx-");
        config.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
        
        DefaultKafkaProducerFactory<String, String> factory = 
            new DefaultKafkaProducerFactory<>(config);
        factory.setTransactionIdPrefix("tx-");
        
        return factory;
    }
    
    @Bean
    public KafkaTransactionManager<String, String> kafkaTransactionManager() {
        return new KafkaTransactionManager<>(producerFactory());
    }
}
```

---

## 13. Monitoreo y Troubleshooting

### 📊 SLIs & Key Metrics

#### **Producer SLIs**
```
🟢 record-send-rate          Target: > 1000/sec
🔴 record-error-rate         Alert: > 1%  
🟠 request-latency-avg       Target: < 50ms p99
🟡 buffer-available-bytes   Alert: < 10% (backpressure)
🔵 compression-rate-avg     Monitor: baseline changes
```

**Critical alerts**:
```java
// Alert 1: High error rate
record-error-rate > 0.01 (1%) for 5 minutes
→ Action: Check broker health, network, auth

// Alert 2: Buffer exhaustion
buffer-available-bytes < (buffer.memory * 0.1) for 2 minutes
→ Action: Increase buffer.memory or reduce send rate

// Alert 3: Latency spike
request-latency-p99 > 200ms for 5 minutes
→ Action: Check broker load, increase batch.size, tune linger.ms
```

#### **Consumer SLIs**
```
🔴 records-lag-max           Alert: > 10000 (backlog)
🟢 fetch-rate                Target: Consistent baseline
🟢 records-consumed-rate     Target: Match produce rate
🟠 commit-latency-avg       Target: < 100ms
🔵 rebalance-rate-per-hour  Alert: > 5/hour (unstable)
```

**Critical alerts**:
```java
// Alert 1: Consumer lag
records-lag-max > 10000 AND increasing for 10 minutes
→ Action: Scale consumers, optimize processing, check rebalancing

// Alert 2: Frequent rebalancing
rebalance-total > 5 per hour
→ Action: Increase session.timeout.ms, max.poll.interval.ms
         Check for slow poll loops

// Alert 3: Consumer stuck
records-consumed-rate == 0 for 5 minutes
→ Action: Check consumer health, poll loop, exceptions
```

### 🔍 JMX Monitoring

```java
// Habilitar JMX en producer/consumer
props.put(CommonClientConfigs.METRICS_RECORDING_LEVEL_CONFIG, "INFO");
props.put(CommonClientConfigs.METRIC_REPORTER_CLASSES_CONFIG, 
    "org.apache.kafka.common.metrics.JmxReporter");
```

Acceder con JConsole:
```bash
jconsole
# Buscar MBeans: kafka.producer / kafka.consumer
```

### 📈 Actuator Metrics (Spring Boot)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

Endpoints:
- `/actuator/metrics/kafka.producer.record-send-total`
- `/actuator/metrics/kafka.consumer.records-consumed-total`
- `/actuator/metrics/kafka.consumer.fetch-manager.records-lag-max`

### 🛠️ Comandos de Troubleshooting

```bash
# Ver consumer lag
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --describe --group my-consumer-group

# Resultado:
# GROUP           TOPIC     PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
# my-group        orders    0          150             200             50
# my-group        orders    1          180             180             0

# Resetear offsets (ojo: consume todos los mensajes de nuevo)
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --group my-consumer-group \
  --reset-offsets --to-earliest \
  --topic orders \
  --execute

# Ver configuración de topic
kafka-configs.sh --bootstrap-server localhost:9092 \
  --entity-type topics \
  --entity-name orders \
  --describe

# Aumentar particiones
kafka-topics.sh --bootstrap-server localhost:9092 \
  --alter --topic orders \
  --partitions 5

# Ver mensajes en un topic (desarrollo)
kafka-console-consumer.sh --bootstrap-server localhost:9092 \
  --topic orders \
  --from-beginning \
  --max-messages 10
```

### 🚨 Problemas Comunes

#### **1. Consumer Lag Alto**
```
Síntomas: records-lag-max > 10000
Causas:
- Consumer lento (procesamiento pesado)
- Pocas particiones / pocos consumers
- Red lenta

Soluciones:
1. Aumentar paralelismo (más consumers o particiones)
2. Optimizar procesamiento (batch, async)
3. Aumentar max.poll.records
4. Revisar red/latencia
```

#### **2. Rebalancing Frecuente**
```
Síntomas: Logs constantes de "Rebalancing"
Causas:
- max.poll.interval.ms muy bajo
- Procesamiento lento en poll loop
- Heartbeat perdidos

Soluciones:
1. Aumentar max.poll.interval.ms
2. Reducir max.poll.records
3. Procesar en background (no en poll loop)
```

#### **3. Mensajes Duplicados**
```
Causas:
- Consumer crash antes de commit
- enable.auto.commit=true con crash

Soluciones:
1. Habilitar idempotencia en producer
2. Commit manual después de procesar
3. Implementar deduplicación por ID
```

---

## 14. Patrones de Diseño con Kafka

### 🎯 Event Sourcing

```java
// Event Store en Kafka
@Service
public class OrderEventStore {
    
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;
    
    public void saveEvent(OrderEvent event) {
        // Guardar evento en topic (event store)
        kafkaTemplate.send("order-events", event.getOrderId(), event);
    }
    
    // Reconstruir estado desde eventos
    public Order rebuildOrder(String orderId) {
        // Leer todos los eventos del order
        List<OrderEvent> events = readEventsForOrder(orderId);
        
        Order order = new Order(orderId);
        for (OrderEvent event : events) {
            order.apply(event); // Aplicar evento
        }
        
        return order;
    }
}

// Events
@Data
class OrderCreatedEvent extends OrderEvent {
    private String orderId;
    private String userId;
    private BigDecimal amount;
}

@Data
class OrderPaidEvent extends OrderEvent {
    private String orderId;
    private String paymentId;
}

@Data
class OrderShippedEvent extends OrderEvent {
    private String orderId;
    private String trackingNumber;
}
```

### 🔔 CQRS (Command Query Responsibility Segregation)

```java
// Write Model: Commands → Events → Kafka
@Service
public class OrderCommandService {
    
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;
    
    public void createOrder(CreateOrderCommand command) {
        // Validar comando
        validate(command);
        
        // Crear evento
        OrderCreatedEvent event = new OrderCreatedEvent(
            command.getOrderId(),
            command.getUserId(),
            command.getAmount()
        );
        
        // Publicar en Kafka
        kafkaTemplate.send("order-events", event.getOrderId(), event);
    }
}

// Read Model: Materializar vista desde eventos
@Service
public class OrderQueryService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @KafkaListener(topics = "order-events", groupId = "order-query-view")
    public void updateReadModel(OrderEvent event) {
        if (event instanceof OrderCreatedEvent) {
            OrderCreatedEvent created = (OrderCreatedEvent) event;
            
            OrderView view = new OrderView();
            view.setOrderId(created.getOrderId());
            view.setUserId(created.getUserId());
            view.setAmount(created.getAmount());
            view.setStatus("CREATED");
            
            orderRepository.save(view);
            
        } else if (event instanceof OrderPaidEvent) {
            OrderPaidEvent paid = (OrderPaidEvent) event;
            
            OrderView view = orderRepository.findById(paid.getOrderId())
                .orElseThrow();
            view.setStatus("PAID");
            view.setPaymentId(paid.getPaymentId());
            
            orderRepository.save(view);
        }
    }
    
    // Query methods
    public OrderView getOrderById(String orderId) {
        return orderRepository.findById(orderId).orElseThrow();
    }
    
    public List<OrderView> getOrdersByUser(String userId) {
        return orderRepository.findByUserId(userId);
    }
}
```

### 🔄 Saga Pattern (Orchestration)

```java
// Saga Orchestrator
@Service
public class OrderSagaOrchestrator {
    
    @KafkaListener(topics = "order-created", groupId = "saga-orchestrator")
    public void handleOrderCreated(OrderCreatedEvent event) {
        // Step 1: Reserve inventory
        kafkaTemplate.send("inventory-commands", 
            new ReserveInventoryCommand(event.getOrderId(), event.getItems()));
    }
    
    @KafkaListener(topics = "inventory-reserved", groupId = "saga-orchestrator")
    public void handleInventoryReserved(InventoryReservedEvent event) {
        // Step 2: Process payment
        kafkaTemplate.send("payment-commands",
            new ProcessPaymentCommand(event.getOrderId(), event.getAmount()));
    }
    
    @KafkaListener(topics = "payment-processed", groupId = "saga-orchestrator")
    public void handlePaymentProcessed(PaymentProcessedEvent event) {
        // Step 3: Arrange shipping
        kafkaTemplate.send("shipping-commands",
            new ArrangeShippingCommand(event.getOrderId()));
    }
    
    // Compensación en caso de fallo
    @KafkaListener(topics = "payment-failed", groupId = "saga-orchestrator")
    public void handlePaymentFailed(PaymentFailedEvent event) {
        // Compensar: Liberar inventario
        kafkaTemplate.send("inventory-commands",
            new ReleaseInventoryCommand(event.getOrderId()));
        
        // Actualizar orden
        kafkaTemplate.send("order-commands",
            new CancelOrderCommand(event.getOrderId(), "Payment failed"));
    }
}
```

### 📊 Change Data Capture (CDC)

```java
// Kafka Connect + Debezium: Capturar cambios en PostgreSQL

// application.yml (Sink Connector)
spring:
  kafka:
    streams:
      application-id: order-cdc-processor

// Procesar CDC events
@KafkaListener(topics = "postgres.public.orders", groupId = "cdc-processor")
public void processCDCEvent(
        @Payload String changeEvent,
        @Header("__op") String operation) {
    
    switch (operation) {
        case "c": // CREATE
            OrderCreated created = parseCreate(changeEvent);
            notifyOrderCreated(created);
            break;
            
        case "u": // UPDATE
            OrderUpdated updated = parseUpdate(changeEvent);
            notifyOrderUpdated(updated);
            break;
            
        case "d": // DELETE
            OrderDeleted deleted = parseDelete(changeEvent);
            notifyOrderDeleted(deleted);
            break;
    }
}
```

---

## 🎓 Resumen y Mejores Prácticas

### ✅ Do's

```
✓ Usa partitioning keys para ordenar mensajes relacionados
✓ Habilita idempotencia en producers (exactly-once)
✓ Commit offsets manualmente después de procesar
✓ Implementa Dead Letter Queues para errores
✓ Monitorea consumer lag constantemente
✓ Usa compression (snappy/lz4) para reducir red
✓ Configura replication-factor ≥ 3 en producción
✓ Implementa circuit breaker para servicios externos
✓ Usa Avro/Protobuf con Schema Registry para grandes volúmenes
✓ Testea con Testcontainers/EmbeddedKafka
```

### ❌ Don'ts

```
✗ No uses acks=0 en producción (pérdida de datos)
✗ No proceses lógica pesada en el poll loop (rebalancing)
✗ No ignores consumer lag (performance degradation)
✗ No uses auto-commit sin validar procesamiento
✗ No crees topics con 1 partición (no escala)
✗ No mezcles mensajes de diferentes naturalezas en un topic
✗ No uses Kafka como database (tiene retención limitada)
✗ No dejes producers/consumers abiertos (memory leak)
```

### 🧪 Testing

```java
// Embedded Kafka (Spring Boot Test)
@SpringBootTest
@EmbeddedKafka(partitions = 3, topics = {"orders"})
class OrderServiceTest {
    
    @Autowired
    private KafkaTemplate<String, OrderDTO> kafkaTemplate;
    
    @Test
    void shouldConsumeOrder() throws InterruptedException {
        OrderDTO order = new OrderDTO("order1", "user1", BigDecimal.TEN);
        
        kafkaTemplate.send("orders", order.getUserId(), order);
        
        Thread.sleep(1000); // Esperar procesamiento
        
        // Verificar que el mensaje fue procesado
        verify(orderService).processOrder(any(OrderDTO.class));
    }
}

// Testcontainers
@Testcontainers
class OrderIntegrationTest {
    
    @Container
    static KafkaContainer kafka = new KafkaContainer(
        DockerImageName.parse("confluentinc/cp-kafka:7.5.0")
    );
    
    @DynamicPropertySource
    static void kafkaProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.kafka.bootstrap-servers", kafka::getBootstrapServers);
    }
    
    @Test
    void integrationTest() {
        // Test con Kafka real en container
    }
}
```

---

## 📚 Referencias

- [Kafka Documentation](https://kafka.apache.org/documentation/)
- [Confluent Kafka Tutorials](https://developer.confluent.io/)
- [Spring for Apache Kafka](https://spring.io/projects/spring-kafka)
- [Kafka Streams API](https://kafka.apache.org/documentation/streams/)
- [Schema Registry](https://docs.confluent.io/platform/current/schema-registry/index.html)

---

**[⬅️ Volver al índice principal](./README.md)** | **[➡️ Siguiente: Testing Avanzado](./10-testing.md)**
