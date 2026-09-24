# @Service

> Nivel: esencial · Categoría: core / estereotipos

## 1. Nombre
`@Service`

## 2. Paquete
`org.springframework.stereotype`

## 3. Framework / librería
Spring Framework (`spring-context`).

## 4. Propósito
Marcar una clase como **servicio de la capa de negocio**. Técnicamente es un `@Component` (se detecta igual), pero comunica intención: aquí vive la lógica de negocio, las reglas y la orquestación de repositorios y clientes externos.

## 5. Target
`ElementType.TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` | `String` | Nombre del bean. Es `@AliasFor(annotation = Component.class)`. |

## 8. Valores por defecto
`value = ""` → nombre generado (`TransferenciaService` → `transferenciaService`).

## 9. Quién la procesa
El mismo mecanismo que `@Component`: `ClassPathBeanDefinitionScanner`. No hay ningún procesador específico de `@Service`.

## 10. Cuándo se procesa
Arranque, durante el escaneo de componentes. Instanciación en `finishBeanFactoryInitialization`.

## 11. Efecto observable
El servicio se registra como bean singleton y puede inyectarse en controladores u otros servicios. Si tiene métodos `@Transactional`, `@Async` o `@Cacheable`, lo que se inyecta es un **proxy**.

## 12. Qué ocurre internamente
Como `@Service` está meta‑anotada con `@Component`, el `AnnotationTypeFilter` la reconoce por la jerarquía de anotaciones (`MergedAnnotations`). A partir de ahí el ciclo es idéntico al de `@Component`. Lo único "especial" es que herramientas y AOP pueden usar el estereotipo como punto de corte (`@within(org.springframework.stereotype.Service)`).

## 13. Relación con otras anotaciones
- Es un `@Component` especializado.
- Suele combinarse con `@Transactional`, `@Validated`, `@Cacheable`, `@Async`, `@PreAuthorize`.
- Inyecta `@Repository` y es inyectado por `@RestController`.

## 14. Dependencias necesarias
`spring-context` (cualquier starter).

## 15. Alternativas
`@Component` (funciona igual), `@Bean` en una `@Configuration`, `@Named` de JSR‑330.

## 16. Limitaciones
- No añade comportamiento propio: no abre transacciones ni valida nada por sí misma.
- Las llamadas internas (`this.otroMetodo()`) no pasan por el proxy, así que las anotaciones AOP de ese método no se aplican.

## 17. Errores comunes
- Creer que `@Service` hace transaccional la clase. Falta `@Transactional`.
- Poner la anotación en la **interfaz** en lugar de la implementación.
- Varias implementaciones de la misma interfaz con `@Service` sin `@Primary`/`@Qualifier` → `NoUniqueBeanDefinitionException`.
- Servicios "dios" de 3000 líneas: la anotación no sustituye a un buen diseño por casos de uso.

## 18. Ejemplo básico
```java
@Service
public class SaludoService {
    public String saludar(String nombre) {
        return "Hola, " + nombre;
    }
}
```

## 19. Ejemplo real
**FinTech — transferencia entre cuentas** con reglas de negocio y transacción:
```java
@Service
public class TransferenciaService {

    private final CuentaRepository cuentas;
    private final MovimientoRepository movimientos;
    private final AntifraudeClient antifraude;

    public TransferenciaService(CuentaRepository cuentas,
                                MovimientoRepository movimientos,
                                AntifraudeClient antifraude) {
        this.cuentas = cuentas;
        this.movimientos = movimientos;
        this.antifraude = antifraude;
    }

    @Transactional
    public Movimiento transferir(String origenIban, String destinoIban, BigDecimal importe) {
        if (importe.signum() <= 0) throw new ImporteInvalidoException(importe);

        Cuenta origen  = cuentas.findByIbanForUpdate(origenIban).orElseThrow();
        Cuenta destino = cuentas.findByIbanForUpdate(destinoIban).orElseThrow();

        antifraude.evaluar(origen, destino, importe); // lanza excepción si es sospechosa

        origen.debitar(importe);
        destino.acreditar(importe);
        return movimientos.save(Movimiento.transferencia(origen, destino, importe));
    }
}
```
**Otro dominio — salud**: `CitaService` que comprueba disponibilidad del médico y reserva la franja.

## 20. Qué ocurre si la elimino
Igual que con `@Component`: el servicio deja de ser bean y el controlador que lo inyecta falla al arrancar con `required a bean of type ... that could not be found`. Si lo sustituyes por `@Component`, todo sigue funcionando exactamente igual.
