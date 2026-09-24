# @Transactional

> Nivel: esencial · Categoría: datos / transacciones

## 1. Nombre
`@Transactional` (Spring)

## 2. Paquete
`org.springframework.transaction.annotation`

> Existe también `jakarta.transaction.Transactional` (JTA), que Spring también soporta con menos opciones.

## 3. Framework / librería
Spring Framework (`spring-tx`).

## 4. Propósito
Ejecutar un método (o todos los de una clase) **dentro de una transacción**: si termina bien se hace *commit*; si lanza una excepción de las configuradas, *rollback*. Garantiza atomicidad: o se aplican todos los cambios, o ninguno.

## 5. Target
`ElementType.TYPE`, `ElementType.METHOD`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Inherited`, `@Documented`, `@Reflective`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` / `transactionManager` | `String` | Bean `TransactionManager` a usar (si hay varios). |
| `label` | `String[]` | Etiquetas descriptivas (las pueden interpretar gestores concretos). |
| `propagation` | `Propagation` | Qué hacer si ya hay una transacción: `REQUIRED`, `REQUIRES_NEW`, `NESTED`, `SUPPORTS`, `MANDATORY`, `NOT_SUPPORTED`, `NEVER`. |
| `isolation` | `Isolation` | Nivel de aislamiento: `DEFAULT`, `READ_UNCOMMITTED`, `READ_COMMITTED`, `REPEATABLE_READ`, `SERIALIZABLE`. |
| `timeout` / `timeoutString` | `int` / `String` | Tiempo máximo en segundos. |
| `readOnly` | `boolean` | Pista de solo lectura (optimizaciones). |
| `rollbackFor` / `rollbackForClassName` | `Class[]` / `String[]` | Excepciones que **sí** provocan rollback (además de las por defecto). |
| `noRollbackFor` / `noRollbackForClassName` | `Class[]` / `String[]` | Excepciones que **no** provocan rollback. |

## 8. Valores por defecto
| Atributo | Default |
|---|---|
| `transactionManager` | `""` → el `TransactionManager` principal |
| `propagation` | `Propagation.REQUIRED` |
| `isolation` | `Isolation.DEFAULT` (el de la base de datos; PostgreSQL/Oracle: READ COMMITTED; MySQL InnoDB: REPEATABLE READ) |
| `timeout` | `-1` (sin límite, o el del gestor) |
| `readOnly` | `false` |
| rollback | Solo `RuntimeException` y `Error`. **Las excepciones checked hacen commit.** |

## 9. Quién la procesa
- `@EnableTransactionManagement` (Spring Boot la activa automáticamente con `TransactionAutoConfiguration`).
- `AnnotationTransactionAttributeSource` lee la anotación.
- `BeanFactoryTransactionAttributeSourceAdvisor` + `TransactionInterceptor` envuelven el bean en un **proxy AOP**.
- El `PlatformTransactionManager` (`JpaTransactionManager`, `DataSourceTransactionManager`, `JtaTransactionManager`…) abre, confirma o revierte la transacción.

## 10. Cuándo se procesa
- Proxy: al crear el bean (`postProcessAfterInitialization`).
- Transacción: en cada llamada al método **a través del proxy**.

## 11. Efecto observable
- Si el método lanza `RuntimeException`, ningún cambio en BD persiste.
- Con JPA, las entidades cargadas dentro del método se guardan automáticamente al terminar (*dirty checking*) sin llamar a `save()`.
- Con `logging.level.org.springframework.transaction=debug` verás `Creating new transaction with name [...]`, `Committing` / `Rolling back`.

## 12. Qué ocurre internamente
1. Llamada entra en el proxy → `TransactionInterceptor.invoke()`.
2. `TransactionAspectSupport.invokeWithinTransaction()` obtiene los atributos y llama a `transactionManager.getTransaction(def)`:
   - Según la propagación, se une a la transacción existente, la suspende o crea una nueva.
   - Con JPA: se obtiene una conexión, `setAutoCommit(false)`, se asocia el `EntityManager` al hilo (`TransactionSynchronizationManager`, basado en `ThreadLocal`).
3. Se ejecuta el método real.
4. Si hay excepción y `rollbackOn(ex)` es true → `rollback`. Si no → `commit` (Hibernate hace `flush` antes).
5. Se limpian recursos del hilo.

## 13. Relación con otras anotaciones
- `@Service` (donde suele colocarse), `@Repository`.
- `@EnableTransactionManagement` (implícita en Boot).
- `@TransactionalEventListener` para reaccionar tras el commit.
- `@Lock` y `@Version` para concurrencia.
- En tests: `@Transactional` en la clase de test hace rollback al final de cada test; `@Commit`/`@Rollback` lo controlan.

## 14. Dependencias necesarias
`spring-tx`, incluido en `spring-boot-starter-data-jpa`, `spring-boot-starter-jdbc`, etc.

## 15. Alternativas
- `TransactionTemplate` (programática, útil para bloques pequeños o dentro de un mismo bean).
- `jakarta.transaction.Transactional`.
- Transacciones reactivas (`TransactionalOperator`) en WebFlux/R2DBC.

## 16. Limitaciones
- **Auto‑invocación**: `this.metodoTransaccional()` desde otro método del mismo bean **no pasa por el proxy** → no hay transacción.
- Solo métodos **públicos** en proxies JDK; con CGLIB, desde Spring 6.0 también `protected` y package‑private, nunca `private` ni `final`.
- Se basa en `ThreadLocal`: código en otro hilo (`@Async`, `CompletableFuture`, streams paralelos) no comparte la transacción.
- No abarca llamadas HTTP o mensajes a otros sistemas (no hay rollback de un pago enviado a un tercero): para eso se usan patrones como *outbox* o sagas.

## 17. Errores comunes
- Esperar rollback con una excepción checked (`throws IOException`) → hace **commit**. Usa `rollbackFor = Exception.class`.
- Capturar la excepción dentro del método (`try/catch` y log) → la transacción hace commit de un estado parcial.
- `@Transactional` en un método `private` o llamado internamente.
- Transacciones largas que incluyen llamadas HTTP lentas (conexiones de BD retenidas → pool agotado).
- `REQUIRES_NEW` anidado dentro de un bucle → agotamiento del pool de conexiones.
- `UnexpectedRollbackException`: un método interno con `REQUIRED` marcó rollback‑only y el externo capturó la excepción e intentó hacer commit.

## 18. Ejemplo básico
```java
@Service
public class InventarioService {
    @Transactional
    public void mover(long origenId, long destinoId, int cantidad) {
        almacenes.findById(origenId).orElseThrow().retirar(cantidad);
        almacenes.findById(destinoId).orElseThrow().agregar(cantidad);
    }   // commit automático; si algo falla, rollback de ambos
}
```

## 19. Ejemplo real
**FinTech — transferencia atómica + auditoría que sobrevive al rollback**:
```java
@Service
public class TransferenciaService {

    private final CuentaRepository cuentas;
    private final MovimientoRepository movimientos;
    private final OutboxRepository outbox;
    private final AuditoriaService auditoria;

    // constructor...

    @Transactional(isolation = Isolation.READ_COMMITTED, timeout = 5,
                   rollbackFor = Exception.class)      // también ante excepciones checked
    public TransferenciaDto transferir(TransferenciaRequest req, String usuario) throws LimiteException {
        auditoria.registrarIntento(usuario, req);      // REQUIRES_NEW: queda aunque esto falle

        // Bloqueo pesimista en orden de id para evitar deadlocks entre transferencias cruzadas
        List<Cuenta> bloqueadas = cuentas.findAllByIbanInForUpdateOrderById(List.of(req.origen(), req.destino()));
        Cuenta origen  = buscar(bloqueadas, req.origen());
        Cuenta destino = buscar(bloqueadas, req.destino());

        origen.debitar(req.importe());                 // lanza SaldoInsuficienteException (runtime) → rollback
        destino.acreditar(req.importe());

        Movimiento mov = movimientos.save(Movimiento.transferencia(origen, destino, req.importe()));

        // Patrón outbox: el evento se guarda en la MISMA transacción; otro proceso lo publica a Kafka
        outbox.save(EventoOutbox.de("TransferenciaRealizada", mov));
        return TransferenciaDto.de(mov);
    }   // commit → cuentas, movimiento y evento, todo o nada
}

@Service
class AuditoriaService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void registrarIntento(String usuario, TransferenciaRequest req) { ... }
}

@Service
class ConsultaSaldoService {
    @Transactional(readOnly = true)                // Hibernate no hace dirty checking; puede usar réplica
    public SaldoDto saldo(String iban) { ... }
}
```
**Otro dominio — reservas**: reservar asiento de avión y cobrar la tarifa en la misma transacción local, con compensación si el cobro externo falla.

## 20. Qué ocurre si la elimino
- Cada operación del repositorio se ejecuta en **su propia transacción corta** (los métodos de `SimpleJpaRepository` son transaccionales): si falla el `acreditar` después del `debitar`, **el dinero desaparece** de la cuenta origen sin llegar al destino.
- Las modificaciones de entidades sin `save()` explícito dejan de persistirse.
- Accesos a relaciones lazy fuera de transacción → `LazyInitializationException` (salvo con Open Session In View activo, que Boot habilita por defecto en apps web con un warning).
