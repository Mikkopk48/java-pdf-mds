# @Autowired

> Nivel: esencial · Categoría: core / inyección de dependencias

## 1. Nombre
`@Autowired`

## 2. Paquete
`org.springframework.beans.factory.annotation`

## 3. Framework / librería
Spring Framework (`spring-beans`).

## 4. Propósito
Pedir a Spring que **inyecte una dependencia** automáticamente, resolviéndola por tipo (y, en caso de empate, por calificador, `@Primary` o nombre).

## 5. Target
`CONSTRUCTOR`, `METHOD`, `PARAMETER`, `FIELD`, `ANNOTATION_TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `required` | `boolean` | Si la dependencia es obligatoria. Con `false`, si no existe el bean se deja el campo sin inyectar (o no se llama al método). |

## 8. Valores por defecto
`required = true`.

## 9. Quién la procesa
`AutowiredAnnotationBeanPostProcessor` (registrado automáticamente por `AnnotationConfigUtils`). Para constructores, `determineCandidateConstructors()`; para campos/métodos, `postProcessProperties()`. La resolución la hace `DefaultListableBeanFactory.resolveDependency()`.

## 10. Cuándo se procesa
Durante la creación de cada bean: constructor al instanciar; campos y métodos justo después, en la fase de *populate*, antes de `@PostConstruct`.

## 11. Efecto observable
Las dependencias llegan con valor. Si no hay candidato: `UnsatisfiedDependencyException` / `NoSuchBeanDefinitionException` al arrancar.

## 12. Qué ocurre internamente
1. Al procesar la clase, el post‑procesador construye un `InjectionMetadata` con todos los puntos de inyección (usa reflexión y lo cachea).
2. Para cada punto llama a `resolveDependency(DependencyDescriptor)`:
   - Busca todos los beans del tipo (incluidos genéricos: `Repositorio<Cuenta>`).
   - Filtra por `@Qualifier` y `autowireCandidate`.
   - Si hay varios: elige `@Primary`; si no, el de mayor prioridad (`@Priority`); si no, coincide nombre del parámetro/campo con el nombre del bean; si no, error.
3. Soporta `Optional<T>`, `ObjectProvider<T>`, `List<T>`/`Map<String,T>` (todos los beans del tipo, ordenados por `@Order`).
4. Desde Spring 4.3, si la clase tiene **un único constructor**, se usa sin necesidad de `@Autowired`.

## 13. Relación con otras anotaciones
- `@Qualifier`, `@Primary`, `@Fallback`: desempate.
- `@Value`: también procesada por el mismo post‑procesador.
- `@Lazy` en el punto de inyección: inyecta un proxy perezoso.
- Equivalentes: `@Inject` (JSR‑330), `@Resource` (JSR‑250, por nombre).

## 14. Dependencias necesarias
`spring-beans` (incluido en todos los starters).

## 15. Alternativas
- **Inyección por constructor sin anotación** (recomendada): campos `final`, fácil de testear.
- `@Inject`, `@Resource`.
- `ObjectProvider<T>` para dependencias opcionales o perezosas.
- Lombok `@RequiredArgsConstructor` para generar el constructor.

## 16. Limitaciones
- No funciona en objetos creados con `new`.
- No funciona en campos `static`.
- En inyección por campo no puedes usar `final` y el objeto no es testeable sin reflexión o contexto.
- Dependencias circulares por constructor fallan (Spring Boot 2.6+ prohíbe también las circulares por campo por defecto: `spring.main.allow-circular-references=false`).

## 17. Errores comunes
- Inyección por campo en todo: dificulta tests y oculta clases con demasiadas dependencias.
- `@Autowired` en un campo `static` → se ignora (warning en log) y queda `null`.
- Usar la dependencia en el constructor cuando se inyecta por campo → `NullPointerException` (aún no se ha inyectado).
- Ciclo A→B→A → `BeanCurrentlyInCreationException` / `The dependencies of some of the beans in the application context form a cycle`.

## 18. Ejemplo básico
```java
@Service
public class NotificacionService {

    private final EmailSender email;

    @Autowired // opcional: hay un solo constructor
    public NotificacionService(EmailSender email) {
        this.email = email;
    }
}
```

## 19. Ejemplo real
**FinTech — motor de reglas antifraude** que recibe *todas* las reglas registradas como beans, en orden:
```java
public interface ReglaFraude {
    Optional<Alerta> evaluar(Operacion op);
}

@Component @Order(1) class ReglaImporteAlto implements ReglaFraude { ... }
@Component @Order(2) class ReglaPaisDeRiesgo implements ReglaFraude { ... }
@Component @Order(3) class ReglaVelocidad    implements ReglaFraude { ... } // >5 ops/min

@Service
public class MotorAntifraude {

    private final List<ReglaFraude> reglas;
    private final ObjectProvider<ScoringMlClient> scoringMl; // opcional: solo en algunos países

    @Autowired
    public MotorAntifraude(List<ReglaFraude> reglas, ObjectProvider<ScoringMlClient> scoringMl) {
        this.reglas = reglas;
        this.scoringMl = scoringMl;
    }

    public List<Alerta> evaluar(Operacion op) {
        List<Alerta> alertas = reglas.stream()
            .map(r -> r.evaluar(op)).flatMap(Optional::stream).toList();
        scoringMl.ifAvailable(ml -> ml.puntuar(op));
        return alertas;
    }
}
```
Añadir una regla nueva = crear una clase `@Component`; el motor no cambia (principio abierto/cerrado).

**Otro dominio — e‑commerce**: `Map<String, CalculadoraEnvio>` inyectado para elegir la calculadora por nombre de transportista.

## 20. Qué ocurre si la elimino
- En un constructor único: **nada**, Spring lo usa igualmente.
- Con varios constructores: Spring usa el constructor sin argumentos; si no existe → `No default constructor found`.
- En campos o setters: la dependencia queda `null` y aparece un `NullPointerException` en tiempo de ejecución.
