# Anotaciones de Spring Boot — Enciclopedia práctica

Referencia en español de las anotaciones que se usan en aplicaciones Spring Boot: las del propio Spring y las de las librerías que casi siempre viajan con él (JPA, Bean Validation, Jackson, Lombok, JUnit, Mockito…).

Cada anotación tiene su propio archivo `.md` con **20 secciones fijas** (ver [PLANTILLA.md](PLANTILLA.md)). Los ejemplos reales están orientados sobre todo a **backend Java para finanzas** (cuentas, transferencias, pagos, ledger, antifraude), con algunos de otros dominios (e‑commerce, logística, salud, educación) para ver la anotación en contextos distintos.

## Versiones de referencia

| Tecnología | Versión base | Notas |
|---|---|---|
| Java | 17+ (ejemplos compatibles con 21) | Records, `var`, text blocks |
| Spring Boot | 3.x y 4.x | Donde algo cambia en Boot 4 se indica |
| Spring Framework | 6.x y 7.x | |
| Jakarta EE | 10 / 11 | Paquetes `jakarta.*` (ya no `javax.*`) |
| Hibernate ORM | 6.x / 7.x | Implementación JPA por defecto |

> Si trabajas con Spring Boot 2.x, la mayoría de las anotaciones son iguales, pero las de JPA y validación están en `javax.*` en lugar de `jakarta.*`.

## Cómo está organizado

Las carpetas van de lo que usas el primer día a lo que usas cuando ya diseñas arquitectura. Dentro de cada nivel, las subcarpetas agrupan por tema.

| Carpeta | Para qué sirve | Estado |
|---|---|---|
| [01-esenciales](01-esenciales/README.md) | Lo mínimo para levantar una API REST con base de datos y validación | ✅ Lista (53) |
| `02-nivel-medio` | Ciclo de vida de beans, condicionales, web avanzada, relaciones JPA, Spring Data, caché, async, scheduling, testing de slices | ⏳ Pendiente |
| `03-avanzadas` | Seguridad, AOP, autoconfiguración propia, Actuator/observabilidad, testing avanzado, resiliencia | ⏳ Pendiente |
| `04-ecosistema-spring` | Kafka, RabbitMQ, JMS, WebSocket, Batch, Cloud (Feign, Config, Discovery), GraphQL, WebFlux, Modulith | ⏳ Pendiente |
| `05-librerias-complementarias` | Lombok, Jackson, MapStruct, JUnit 5, Mockito, OpenAPI/Swagger, extensiones de Hibernate | ⏳ Pendiente |
| `06-extras-y-legacy` | JSR‑250/330, nulabilidad, AOT/GraalVM, anotaciones deprecadas o eliminadas y su reemplazo | ⏳ Pendiente |

## Plan de contenido (provisional)

El total real ronda las 500 anotaciones. No se inventan anotaciones para inflar el número: todo lo listado existe en las librerías indicadas.

### 01-esenciales (53)
- **01-core**: `@SpringBootApplication`, `@Component`, `@Service`, `@Repository`, `@Controller`, `@Configuration`, `@Bean`, `@Autowired`, `@Qualifier`, `@Primary`, `@Value`
- **02-configuracion**: `@ConfigurationProperties`, `@EnableConfigurationProperties`, `@ConfigurationPropertiesScan`, `@Profile`, `@ComponentScan`, `@EnableAutoConfiguration`, `@SpringBootConfiguration`
- **03-web**: `@RestController`, `@RequestMapping`, `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`, `@PathVariable`, `@RequestParam`, `@RequestBody`, `@ResponseBody`, `@ResponseStatus`, `@RequestHeader`, `@ExceptionHandler`, `@ControllerAdvice`, `@RestControllerAdvice`
- **04-datos**: `@Entity`, `@Table`, `@Id`, `@GeneratedValue`, `@Column`, `@Transactional`
- **05-validacion**: `@Valid`, `@Validated`, `@NotNull`, `@NotBlank`, `@NotEmpty`, `@Size`, `@Min`, `@Max`, `@Email`, `@Pattern`, `@Positive`, `@Digits`, `@DecimalMin`

### 02-nivel-medio (~120)
- **Contexto y ciclo de vida**: `@Scope`, `@Lazy`, `@DependsOn`, `@Order`, `@PostConstruct`, `@PreDestroy`, `@Import`, `@ImportResource`, `@PropertySource`, `@Description`, `@Role`, `@Fallback`, `@Lookup`, `@EventListener`, `@TransactionalEventListener`…
- **Condicionales**: `@Conditional`, `@ConditionalOnClass`, `@ConditionalOnMissingClass`, `@ConditionalOnBean`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`, `@ConditionalOnBooleanProperty`, `@ConditionalOnResource`, `@ConditionalOnWebApplication`, `@ConditionalOnExpression`, `@ConditionalOnJava`, `@ConditionalOnSingleCandidate`, `@ConditionalOnCloudPlatform`, `@ConditionalOnThreading`…
- **Web avanzada**: `@RequestPart`, `@CookieValue`, `@MatrixVariable`, `@ModelAttribute`, `@SessionAttribute(s)`, `@RequestAttribute`, `@InitBinder`, `@CrossOrigin`, `@BindParam`, `@HttpExchange` y variantes…
- **JPA mapeo y relaciones**: `@OneToOne`, `@OneToMany`, `@ManyToOne`, `@ManyToMany`, `@JoinColumn`, `@JoinTable`, `@Enumerated`, `@Embedded`, `@Embeddable`, `@EmbeddedId`, `@Version`, `@MappedSuperclass`, `@Inheritance`, callbacks (`@PrePersist`…), `@Convert`…
- **Spring Data**: `@Query`, `@Modifying`, `@Param`, `@EnableJpaRepositories`, `@EntityScan`, `@NoRepositoryBean`, auditoría (`@CreatedDate`…), `@Lock`, `@EntityGraph`…
- **Transacciones, caché, async, scheduling**: `@EnableTransactionManagement`, `@EnableCaching`, `@Cacheable`, `@CachePut`, `@CacheEvict`, `@EnableAsync`, `@Async`, `@EnableScheduling`, `@Scheduled`…
- **Validación extendida**: resto de constraints de Jakarta y de Hibernate Validator, `@Constraint`, grupos…
- **Testing básico**: `@SpringBootTest`, `@WebMvcTest`, `@DataJpaTest`, `@MockitoBean`, `@AutoConfigureMockMvc`…

### 03-avanzadas (~110)
Spring Security, AOP/AspectJ, autoconfiguración y binding avanzado, Actuator y Micrometer, testing avanzado (Testcontainers, `@Sql`, `@DynamicPropertySource`, `@ServiceConnection`…), resiliencia (`@Retryable`, `@ConcurrencyLimit`…), formato y conversión.

### 04-ecosistema-spring (~120)
Kafka, AMQP/RabbitMQ, JMS, WebSocket/STOMP, Spring Batch, Spring Cloud, Resilience4j, GraphQL, Modulith, WebFlux.

### 05-librerias-complementarias (~150)
Lombok, Jackson, MapStruct, JUnit 5, Mockito, springdoc‑openapi/Swagger, anotaciones propias de Hibernate.

### 06-extras-y-legacy (~40)
`@Inject`, `@Named`, `@Resource`, nulabilidad (`@Nullable`, JSpecify), AOT (`@Reflective`, `@RegisterReflectionForBinding`, `@ImportRuntimeHints`), y anotaciones retiradas (`@MockBean`, `@Required`…) con su sustituto.

## Cómo estudiar con este proyecto

1. Lee primero la sección **1‑4** para saber qué es.
2. Salta a **18 (Ejemplo básico)** y escríbelo tú en un proyecto de prueba.
3. Lee **9‑12** (quién la procesa, cuándo y qué ocurre dentro): es lo que separa a quien usa Spring de quien lo entiende.
4. Cierra con **17 (Errores comunes)** y **20 (Qué ocurre si la elimino)**, y compruébalo borrándola en tu proyecto.
