# @Qualifier

> Nivel: esencial · Categoría: core / inyección de dependencias

## 1. Nombre
`@Qualifier`

## 2. Paquete
`org.springframework.beans.factory.annotation`

## 3. Framework / librería
Spring Framework (`spring-beans`).

## 4. Propósito
**Desambiguar** qué bean inyectar cuando hay varios del mismo tipo. También sirve para "etiquetar" beans y para crear anotaciones de calificador propias.

## 5. Target
`FIELD`, `METHOD`, `PARAMETER`, `TYPE`, `ANNOTATION_TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Inherited`, `@Documented`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` | `String` | Calificador a buscar (o a asignar, si se usa sobre la clase/método `@Bean`). |

## 8. Valores por defecto
`value = ""`. Si en el punto de inyección se usa sin valor, se compara con el nombre del bean.

## 9. Quién la procesa
`QualifierAnnotationAutowireCandidateResolver` (en Spring Boot, su subclase `ContextAnnotationAutowireCandidateResolver`), consultado por `DefaultListableBeanFactory` al resolver dependencias.

## 10. Cuándo se procesa
Al resolver cada punto de inyección durante la creación de beans.

## 11. Efecto observable
Se inyecta exactamente el bean cuyo nombre o calificador coincide con `value`.

## 12. Qué ocurre internamente
1. Se buscan los candidatos por tipo.
2. Para cada candidato, `isAutowireCandidate()` compara las anotaciones calificadoras del punto de inyección con:
   - Los `@Qualifier` declarados en la clase o en el método `@Bean`.
   - El **nombre del bean** (y sus alias) como fallback.
3. Anotaciones propias meta‑anotadas con `@Qualifier` se comparan por tipo de anotación y atributos.

## 13. Relación con otras anotaciones
- Complementa a `@Autowired`, `@Bean`, `@Component`.
- Tiene prioridad sobre `@Primary`: si pides un calificador concreto, `@Primary` no interviene.
- Equivalente JSR‑330: `@jakarta.inject.Named` y `@jakarta.inject.Qualifier`.

## 14. Dependencias necesarias
`spring-beans`.

## 15. Alternativas
- `@Primary` para marcar el candidato por defecto.
- Nombrar el parámetro igual que el bean (funciona como último recurso de desempate).
- Inyectar `Map<String, T>` y elegir en tiempo de ejecución.
- Crear una anotación propia (más legible y sin strings mágicos).

## 16. Limitaciones
- Basada en `String`: un error tipográfico solo se detecta al arrancar.
- Con Lombok `@RequiredArgsConstructor`, el `@Qualifier` del campo no se copia al parámetro del constructor salvo que configures `lombok.copyableAnnotations += org.springframework.beans.factory.annotation.Qualifier` en `lombok.config`.

## 17. Errores comunes
- Olvidar el calificador en uno de los puntos de inyección → `NoUniqueBeanDefinitionException: expected single matching bean but found 2`.
- Confundir el nombre del bean (nombre del método `@Bean`) con el nombre de la clase.
- El problema de Lombok descrito arriba: el calificador "desaparece" y se inyecta el `@Primary`.

## 18. Ejemplo básico
```java
@Bean @Qualifier("rapido") Cache cacheRapida() { ... }
@Bean @Qualifier("persistente") Cache cacheRedis() { ... }

@Service
public class CatalogoService {
    public CatalogoService(@Qualifier("rapido") Cache cache) { ... }
}
```

## 19. Ejemplo real
**FinTech — dos proveedores de pago con anotaciones calificadoras propias** (sin strings mágicos):
```java
@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Adquirente {
    Red value();
    enum Red { VISA, MASTERCARD }
}

@Component @Adquirente(Adquirente.Red.VISA)
class VisaGateway implements PasarelaTarjeta { ... }

@Component @Adquirente(Adquirente.Red.MASTERCARD)
class MastercardGateway implements PasarelaTarjeta { ... }

@Service
class AutorizacionService {
    private final PasarelaTarjeta visa;
    private final PasarelaTarjeta master;

    AutorizacionService(@Adquirente(Adquirente.Red.VISA) PasarelaTarjeta visa,
                        @Adquirente(Adquirente.Red.MASTERCARD) PasarelaTarjeta master) {
        this.visa = visa;
        this.master = master;
    }

    Autorizacion autorizar(Tarjeta t, BigDecimal importe) {
        return (t.bin().startsWith("4") ? visa : master).autorizar(t, importe);
    }
}
```
**Otro dominio — multi‑tenant SaaS**: `@Qualifier("tenantA") DataSource` y `@Qualifier("tenantB") DataSource`.

## 20. Qué ocurre si la elimino
Si hay varios beans del tipo y ninguno es `@Primary` ni coincide por nombre con el parámetro, el arranque falla con `NoUniqueBeanDefinitionException`. Si hay un `@Primary`, arranca pero inyecta **ese** (posible bug silencioso: pagos Mastercard enviados por la pasarela Visa).
