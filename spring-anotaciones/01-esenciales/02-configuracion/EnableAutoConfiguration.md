# @EnableAutoConfiguration

> Nivel: esencial · Categoría: configuración / autoconfiguración

## 1. Nombre
`@EnableAutoConfiguration`

## 2. Paquete
`org.springframework.boot.autoconfigure`

## 3. Framework / librería
Spring Boot (`spring-boot-autoconfigure`).

## 4. Propósito
Activar la **autoconfiguración**: Spring Boot mira qué hay en el classpath, qué beans has declarado y qué propiedades hay, y crea automáticamente los beans que faltan (un `DataSource`, un `ObjectMapper`, Tomcat, el `DispatcherServlet`, etc.).

## 5. Target
`ElementType.TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`, `@Inherited`). Meta‑anotada con `@AutoConfigurationPackage` e `@Import(AutoConfigurationImportSelector.class)`.

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `exclude` | `Class<?>[]` | Autoconfiguraciones a excluir. |
| `excludeName` | `String[]` | Igual, por nombre de clase. |

Constante: `ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration"` (si vale `false`, se desactiva toda la autoconfiguración).

## 8. Valores por defecto
`exclude = {}`, `excludeName = {}`.

## 9. Quién la procesa
- `AutoConfigurationImportSelector` (un `DeferredImportSelector`).
- `AutoConfigurationPackages.Registrar` (por `@AutoConfigurationPackage`).
- Las condiciones de cada autoconfiguración las evalúa `ConditionEvaluator` con las clases `OnClassCondition`, `OnBeanCondition`, `OnPropertyCondition`, etc.

## 10. Cuándo se procesa
Arranque. Al ser *deferred*, se procesa **después** de todas tus clases `@Configuration`, para que `@ConditionalOnMissingBean` sepa qué has definido tú.

## 11. Efecto observable
- Beans "mágicos" disponibles sin configurarlos.
- Con `--debug` o `debug=true` se imprime el **CONDITIONS EVALUATION REPORT** con las autoconfiguraciones aplicadas (*Positive matches*) y descartadas (*Negative matches*) y el motivo.
- `/actuator/conditions` muestra lo mismo.

## 12. Qué ocurre internamente
1. Lee los candidatos de todos los ficheros `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` del classpath (en Boot 2.x era `spring.factories`).
2. Elimina duplicados y exclusiones (atributos + propiedad `spring.autoconfigure.exclude`).
3. Aplica un filtro rápido (`AutoConfigurationImportFilter`) con metadatos precompilados (`spring-autoconfigure-metadata.properties`) para descartar sin cargar clases las que tienen `@ConditionalOnClass` insatisfecho.
4. Ordena por `@AutoConfigureOrder`, `@AutoConfigureBefore/After`.
5. Importa las supervivientes; cada una evalúa sus condiciones a nivel de clase y de método `@Bean`.

## 13. Relación con otras anotaciones
- Parte de `@SpringBootApplication`.
- Trabaja con `@AutoConfiguration`, `@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`…
- `@ImportAutoConfiguration` importa autoconfiguraciones concretas (usado por los test slices).

## 14. Dependencias necesarias
`spring-boot-autoconfigure` (incluido en `spring-boot-starter`).

## 15. Alternativas
- `@ImportAutoConfiguration({...})` para elegir una lista explícita.
- Configurar todo a mano con `@Configuration` (sin Boot).
- Excluir por propiedad: `spring.autoconfigure.exclude=org.springframework.boot...`.

## 16. Limitaciones
- Solo excluye clases de autoconfiguración, no componentes propios.
- La "magia" dificulta entender de dónde sale un bean si no se consulta el informe de condiciones.

## 17. Errores comunes
- Añadir un starter y no entender por qué la app ahora pide una URL de base de datos (`Failed to configure a DataSource: 'url' attribute is not specified`).
- Excluir `DataSourceAutoConfiguration` y olvidar excluir también `HibernateJpaAutoConfiguration` → error al no encontrar `DataSource`.
- Ponerla en más de una clase.
- En Boot 4 los módulos de autoconfiguración están más separados; algunos paquetes de clases cambiaron: revisa los nombres completos al usar `excludeName`.

## 18. Ejemplo básico
```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
public class App { ... }
```

## 19. Ejemplo real
**FinTech — servicio de conciliación bancaria** que solo lee ficheros y publica en Kafka, sin base de datos ni servidor web:
```java
@SpringBootApplication
@EnableAutoConfiguration(excludeName = {
    "org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration",
    "org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration"
})
public class ConciliacionApplication {
    public static void main(String[] args) {
        new SpringApplicationBuilder(ConciliacionApplication.class)
            .web(WebApplicationType.NONE)
            .run(args);
    }
}
```
> Nota: la forma más habitual es usar `@SpringBootApplication(exclude = ...)`; aquí se muestra la anotación directa. Si tu versión de Boot movió esas clases de paquete, revisa el informe `--debug`.

**Otro dominio — CLI**: herramienta de migración de datos que desactiva toda la autoconfiguración web.

## 20. Qué ocurre si la elimino
No se crea ningún bean de infraestructura: sin `DataSource`, sin `EntityManagerFactory`, sin servidor web, sin `ObjectMapper`… La app arranca vacía o falla por beans que faltan.
