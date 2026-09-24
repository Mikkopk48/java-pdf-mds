# @Component

> Nivel: esencial · Categoría: core / estereotipos

## 1. Nombre
`@Component`

## 2. Paquete
`org.springframework.stereotype`

## 3. Framework / librería
Spring Framework (módulo `spring-context`).

## 4. Propósito
Indicar que una clase es un **componente gestionado por Spring**: el escaneo de classpath la detecta, crea una instancia (bean) y la inyecta donde se necesite. Es el estereotipo genérico del que derivan `@Service`, `@Repository`, `@Controller` y `@Configuration`.

## 5. Target
`ElementType.TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME`. Está marcada con `@Documented` e `@Indexed` (esta última permite generar el índice `META-INF/spring.components` en compilación, hoy deprecado en favor de AOT).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` | `String` | Nombre del bean. Si se omite, se genera a partir del nombre de la clase. |

## 8. Valores por defecto
`value = ""` → nombre generado por `AnnotationBeanNameGenerator`: nombre simple de la clase con la primera letra en minúscula (`TasaCambioClient` → `tasaCambioClient`). Si las dos primeras letras son mayúsculas se deja igual (`IBANValidator` → `IBANValidator`), según `Introspector.decapitalize`.

## 9. Quién la procesa
`ClassPathBeanDefinitionScanner` (invocado por `ConfigurationClassPostProcessor` al procesar `@ComponentScan`). Usa `ClassPathScanningCandidateComponentProvider` con un `AnnotationTypeFilter(Component.class)` que también reconoce meta‑anotaciones.

## 10. Cuándo se procesa
Arranque del contexto: fase de registro de definiciones de beans (`invokeBeanFactoryPostProcessors`). La **instancia** se crea después, en `finishBeanFactoryInitialization` (salvo que sea `@Lazy` o de otro scope).

## 11. Efecto observable
- La clase aparece en `context.getBeanDefinitionNames()` y en `/actuator/beans`.
- Puedes inyectarla con `@Autowired` o por constructor en cualquier otro bean.
- Es singleton por defecto: una única instancia compartida.

## 12. Qué ocurre internamente
1. El escáner recorre los `.class` de los paquetes base leyendo su bytecode con ASM (**sin cargar las clases**), vía `MetadataReader`.
2. Si la clase (o una de sus anotaciones) tiene `@Component`, crea un `ScannedGenericBeanDefinition`.
3. Resuelve el nombre, scope (`@Scope`), `@Lazy`, `@Primary`, `@DependsOn` y registra la definición en el `BeanDefinitionRegistry`.
4. Más tarde, `AbstractAutowireCapableBeanFactory.createBean()` elige constructor, lo invoca, aplica `BeanPostProcessor`s (inyección `@Autowired`, `@PostConstruct`, proxies AOP) y guarda el resultado en la caché de singletons.

## 13. Relación con otras anotaciones
- Meta‑anotación de `@Service`, `@Repository`, `@Controller`, `@RestController`, `@Configuration`, `@ControllerAdvice`.
- Se combina con `@Scope`, `@Lazy`, `@Primary`, `@Qualifier`, `@Profile`, `@Conditional…`, `@Order`, `@DependsOn`.
- Requiere un `@ComponentScan` activo (lo aporta `@SpringBootApplication`).

## 14. Dependencias necesarias
`spring-context` (incluido en cualquier `spring-boot-starter`).

## 15. Alternativas
- Declarar el bean con un método `@Bean` en una clase `@Configuration` (útil para clases de terceros que no puedes anotar).
- Estereotipos específicos (`@Service`, `@Repository`) para expresar intención.
- Registro programático: `GenericApplicationContext.registerBean(...)` o `BeanRegistrar` (Spring Framework 7).
- JSR‑330: `@Named` (si `jakarta.inject` está en el classpath).

## 16. Limitaciones
- Solo funciona sobre clases que Spring escanea y que puede instanciar (clase concreta, con constructor accesible).
- No sirve para clases de librerías externas (no puedes modificarlas).
- Anotar interfaces con `@Component` no hace nada útil (no se pueden instanciar); el escáner las descarta salvo casos especiales como repositorios Spring Data.

## 17. Errores comunes
- Clase fuera del paquete escaneado → `NoSuchBeanDefinitionException`.
- Crearla con `new` a mano: esa instancia **no** es un bean; sus `@Autowired` quedan a `null`.
- Guardar estado mutable por petición en un singleton (p. ej. un campo `usuarioActual`) → condiciones de carrera entre hilos.
- Dos clases con el mismo nombre simple en paquetes distintos → `ConflictingBeanDefinitionException`. Solución: darles `value` distinto.

## 18. Ejemplo básico
```java
@Component
public class GeneradorReferencia {
    public String nueva() {
        return UUID.randomUUID().toString();
    }
}

@Service
public class PedidoService {
    private final GeneradorReferencia generador;
    public PedidoService(GeneradorReferencia generador) { this.generador = generador; }
}
```

## 19. Ejemplo real
**FinTech — cálculo de IBAN**: utilidad sin estado inyectada en varios servicios (alta de cuentas, transferencias SEPA).
```java
@Component("ibanCalculator")
public class IbanCalculator {

    public boolean esValido(String iban) {
        String limpio = iban.replace(" ", "").toUpperCase();
        String reordenado = limpio.substring(4) + limpio.substring(0, 4);
        StringBuilder numerico = new StringBuilder();
        for (char c : reordenado.toCharArray()) {
            numerico.append(Character.isLetter(c) ? c - 'A' + 10 : c);
        }
        return new BigInteger(numerico.toString()).mod(BigInteger.valueOf(97)).intValue() == 1;
    }
}
```
**Otro dominio — logística**: un `@Component` que calcula el coste de envío por peso y zona, inyectado en el checkout.

## 20. Qué ocurre si la elimino
La clase deja de ser un bean. Cualquier otro bean que la pida falla al arrancar:
```
Parameter 0 of constructor in com.neobank.TransferService required a bean of type
'com.neobank.IbanCalculator' that could not be found.
```
