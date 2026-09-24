# @Value

> Nivel: esencial · Categoría: core / configuración externa

## 1. Nombre
`@Value`

## 2. Paquete
`org.springframework.beans.factory.annotation`

## 3. Framework / librería
Spring Framework (`spring-beans`).

## 4. Propósito
Inyectar **valores** (no beans) en campos, parámetros o métodos: propiedades de configuración con `${...}` o expresiones SpEL con `#{...}`.

## 5. Target
`FIELD`, `METHOD`, `PARAMETER`, `ANNOTATION_TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` | `String` | Expresión: `${clave}`, `${clave:valorPorDefecto}`, `#{expresionSpEL}` o combinación. **Obligatorio**. |

## 8. Valores por defecto
No tiene default (es obligatorio). El valor por defecto de la **propiedad** se indica dentro de la expresión: `${app.timeout:30}`.

## 9. Quién la procesa
- `AutowiredAnnotationBeanPostProcessor` detecta el punto de inyección.
- `QualifierAnnotationAutowireCandidateResolver.getSuggestedValue()` extrae el texto.
- Los placeholders `${}` los resuelve el *embedded value resolver* del `BeanFactory` (en Boot, `PropertySourcesPlaceholderConfigurer` sobre el `Environment`).
- Las expresiones `#{}` las evalúa `StandardBeanExpressionResolver` (SpEL).
- La conversión de tipo la hace el `TypeConverter`/`ConversionService`.

## 10. Cuándo se procesa
Durante la creación del bean (constructor o fase de *populate*). Una sola vez: no se actualiza si la propiedad cambia después.

## 11. Efecto observable
El campo contiene el valor de `application.yml`, variable de entorno o argumento de línea de comandos correspondiente, convertido al tipo del campo.

## 12. Qué ocurre internamente
1. Resuelve placeholders recorriendo los `PropertySource`s del `Environment` por prioridad: argumentos de línea de comandos > variables de entorno > `application-{perfil}.yml` > `application.yml` > …
2. Si el resultado contiene `#{}`, evalúa SpEL (puede referenciar beans: `#{@miBean.valor}`).
3. Convierte el `String` al tipo destino (`int`, `Duration`, `List<String>` separados por coma, etc.).
4. Si no encuentra la clave y no hay default → `IllegalArgumentException: Could not resolve placeholder 'x' in value "${x}"`.

## 13. Relación con otras anotaciones
- Alternativa ligera a `@ConfigurationProperties`.
- Se procesa en el mismo post‑procesador que `@Autowired`.
- Las propiedades pueden venir de `@PropertySource`, `@TestPropertySource`, `@DynamicPropertySource`.
- `@RefreshScope` (Spring Cloud) permite re‑leer valores.

## 14. Dependencias necesarias
`spring-beans` / `spring-context`.

## 15. Alternativas
- `@ConfigurationProperties` (tipado, validable, agrupado) — **recomendada** para más de 2‑3 valores.
- Inyectar `Environment` y llamar a `env.getProperty("x")`.

## 16. Limitaciones
- No soporta *relaxed binding* completo (`${app.max-retries}` debe coincidir con el formato de la clave; las variables de entorno sí se mapean).
- No valida con Bean Validation.
- No funciona en campos `static` ni en objetos que no sean beans.
- Sin autocompletado en el IDE (no genera metadatos).

## 17. Errores comunes
- Usar `#{}` en lugar de `${}` (o al revés).
- Propiedades sensibles con default hardcodeado en código (`${db.password:admin}`).
- Olvidar el default en propiedades opcionales y romper el arranque en otros entornos.
- `@Value` en un campo `static` → queda `null`.
- Leer el campo en el constructor cuando se inyecta por campo → aún es `null`.

## 18. Ejemplo básico
```java
@Component
public class ClienteClima {
    private final String url;
    private final Duration timeout;

    public ClienteClima(@Value("${clima.url}") String url,
                        @Value("${clima.timeout:5s}") Duration timeout) {
        this.url = url;
        this.timeout = timeout;
    }
}
```

## 19. Ejemplo real
**FinTech — límite diario de transferencias y lista de países bloqueados**:
```yaml
transferencias:
  limite-diario: 10000.00
  paises-bloqueados: KP,IR,SY
```
```java
@Service
public class LimitesService {

    private final BigDecimal limiteDiario;
    private final Set<String> paisesBloqueados;

    public LimitesService(
            @Value("${transferencias.limite-diario}") BigDecimal limiteDiario,
            @Value("#{'${transferencias.paises-bloqueados}'.split(',')}") Set<String> paisesBloqueados) {
        this.limiteDiario = limiteDiario;
        this.paisesBloqueados = paisesBloqueados;
    }

    public void validar(Transferencia t, BigDecimal acumuladoHoy) {
        if (paisesBloqueados.contains(t.paisDestino()))
            throw new DestinoBloqueadoException(t.paisDestino());
        if (acumuladoHoy.add(t.importe()).compareTo(limiteDiario) > 0)
            throw new LimiteExcedidoException(limiteDiario);
    }
}
```
> En producción, un límite así suele ir por cliente en base de datos; la propiedad actúa como tope global.

**Otro dominio — salud**: `@Value("${citas.duracion-minutos:20}") int duracionCita`.

## 20. Qué ocurre si la elimino
- En un parámetro de constructor de tipo `String`/`BigDecimal`: Spring intenta inyectar un **bean** de ese tipo y falla (`No qualifying bean of type 'java.math.BigDecimal'`).
- En un campo: queda con su valor por defecto de Java (`null`, `0`, `false`).
