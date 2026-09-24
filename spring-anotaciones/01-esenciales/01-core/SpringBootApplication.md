# @SpringBootApplication

> Nivel: esencial · Categoría: core / arranque

## 1. Nombre
`@SpringBootApplication`

## 2. Paquete
`org.springframework.boot.autoconfigure`

## 3. Framework / librería
Spring Boot (módulo `spring-boot-autoconfigure`).

## 4. Propósito
Marcar la clase principal de la aplicación. Es una **anotación compuesta** (meta‑anotación) que equivale a poner juntas tres anotaciones:

- `@SpringBootConfiguration` → la clase es una fuente de configuración (`@Configuration`).
- `@EnableAutoConfiguration` → activa la autoconfiguración según lo que haya en el classpath.
- `@ComponentScan` → escanea el paquete de esta clase y todos sus subpaquetes buscando componentes.

Con una sola anotación se obtiene una aplicación lista para arrancar.

## 5. Target
`ElementType.TYPE` (clases).

## 6. Retention
`RetentionPolicy.RUNTIME`. Además está marcada con `@Documented` e `@Inherited` (las subclases la heredan).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `exclude` | `Class<?>[]` | Clases de autoconfiguración que NO se deben aplicar. Alias de `@EnableAutoConfiguration.exclude`. |
| `excludeName` | `String[]` | Igual que `exclude` pero por nombre de clase completo (útil si la clase no está en el classpath). |
| `scanBasePackages` | `String[]` | Paquetes base a escanear. Alias de `@ComponentScan.basePackages`. |
| `scanBasePackageClasses` | `Class<?>[]` | Forma *type‑safe* de indicar paquetes: se escanea el paquete de cada clase indicada. |
| `nameGenerator` | `Class<? extends BeanNameGenerator>` | Estrategia para generar nombres de beans detectados por escaneo. |
| `proxyBeanMethods` | `boolean` | Si los métodos `@Bean` de esta clase se interceptan con un proxy CGLIB (ver `@Configuration`). |

## 8. Valores por defecto
| Atributo | Default |
|---|---|
| `exclude` | `{}` |
| `excludeName` | `{}` |
| `scanBasePackages` | `{}` → se usa el paquete de la clase anotada |
| `scanBasePackageClasses` | `{}` |
| `nameGenerator` | `BeanNameGenerator.class` (sentinela: usa el generador por defecto `AnnotationBeanNameGenerator`) |
| `proxyBeanMethods` | `true` |

## 9. Quién la procesa
- `SpringApplication.run(...)` crea el `ApplicationContext` y registra la clase principal como fuente.
- `ConfigurationClassPostProcessor` (un `BeanDefinitionRegistryPostProcessor`) lee la clase y procesa sus meta‑anotaciones:
  - `@ComponentScan` → `ComponentScanAnnotationParser` + `ClassPathBeanDefinitionScanner`.
  - `@EnableAutoConfiguration` → `@Import(AutoConfigurationImportSelector.class)` y `@AutoConfigurationPackage`.

## 10. Cuándo se procesa
En el arranque, durante `refresh()` del contexto, en la fase de **post‑procesado de la BeanFactory** (`invokeBeanFactoryPostProcessors`), antes de instanciar cualquier bean singleton.

## 11. Efecto observable
- Aparece el banner de Spring Boot y el log `Started XxxApplication in N seconds`.
- Los `@Component`/`@Service`/`@RestController` de tus subpaquetes se registran como beans.
- Si hay `spring-boot-starter-web`, arranca Tomcat en el puerto 8080; si hay un driver JDBC y URL, se crea un `DataSource`, etc.

## 12. Qué ocurre internamente
1. `SpringApplication` detecta el tipo de aplicación (servlet, reactiva o ninguna) según el classpath.
2. Prepara el `Environment` (properties, YAML, variables de entorno, perfiles).
3. Crea el contexto (`AnnotationConfigServletWebServerApplicationContext` en apps web servlet) y registra la clase principal.
4. `ConfigurationClassPostProcessor` parsea la clase:
   - Ejecuta el escaneo de componentes desde el paquete de la clase.
   - `AutoConfigurationImportSelector` (es un `DeferredImportSelector`, se ejecuta **después** de procesar tu configuración) lee los candidatos de `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` de todos los JARs, quita los excluidos, y evalúa sus condiciones (`@ConditionalOnClass`, `@ConditionalOnMissingBean`…).
   - `@AutoConfigurationPackage` registra el paquete de la clase principal para que otros (JPA entity scan, Spring Data repositorios) sepan dónde buscar.
5. Se instancian los singletons y se inicia el servidor embebido.

## 13. Relación con otras anotaciones
- Compuesta por `@SpringBootConfiguration`, `@EnableAutoConfiguration`, `@ComponentScan`.
- Puedes combinarla con `@EnableScheduling`, `@EnableAsync`, `@EnableCaching`, `@ConfigurationPropertiesScan`, etc.
- Los *test slices* (`@SpringBootTest`, `@WebMvcTest`) buscan hacia arriba en los paquetes una clase con `@SpringBootConfiguration` (la encuentran gracias a esta anotación).

## 14. Dependencias necesarias
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
</dependency>
```
(Cualquier starter la trae, por ejemplo `spring-boot-starter-web`.)

## 15. Alternativas
Usar las tres anotaciones por separado cuando necesitas controlar cada una:
```java
@SpringBootConfiguration
@EnableAutoConfiguration(exclude = DataSourceAutoConfiguration.class)
@ComponentScan(basePackages = {"com.banco.core", "com.banco.pagos"})
public class App { ... }
```

## 16. Limitaciones
- Solo debe haber **una** por aplicación (o al menos una por jerarquía de paquetes); varias confunden a los tests (`Found multiple @SpringBootConfiguration`).
- El escaneo empieza en su paquete: clases en paquetes "hermanos" o superiores no se detectan.
- No sirve para excluir componentes propios; `exclude` solo aplica a clases de autoconfiguración.

## 17. Errores comunes
- **Ponerla en el paquete por defecto** (sin `package`): Spring escanea TODO el classpath, arranque lentísimo o errores.
- **Clase principal en un subpaquete** (`com.banco.app`) y servicios en `com.banco.servicios` → `NoSuchBeanDefinitionException`.
- Usar `exclude` con una clase propia (`@Service`) → error: `The following classes could not be excluded because they are not auto-configuration classes`.
- Poner `@ComponentScan` adicional en la misma clase: reemplaza al escaneo por defecto y pierde los filtros `TypeExcludeFilter`/`AutoConfigurationExcludeFilter`.

## 18. Ejemplo básico
```java
package com.ejemplo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

## 19. Ejemplo real
**FinTech — núcleo bancario modular** en el que el módulo de reporting vive en otro paquete raíz y la base de datos de auditoría se configura a mano:

```java
package com.neobank.core;

@SpringBootApplication(
    scanBasePackages = {"com.neobank.core", "com.neobank.reporting"},
    exclude = { DataSourceAutoConfiguration.class } // tenemos dos DataSources propios (ledger + auditoría)
)
@ConfigurationPropertiesScan
@EnableScheduling // cierre contable nocturno
public class CoreBankingApplication {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(CoreBankingApplication.class);
        app.setBannerMode(Banner.Mode.OFF);
        app.run(args);
    }
}
```

**Otro dominio — e‑commerce**: una tienda que no usa base de datos en un microservicio de catálogo en caché:
```java
@SpringBootApplication(excludeName = "org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration")
public class CatalogoApplication { ... }
```

## 20. Qué ocurre si la elimino
- `SpringApplication.run` sigue funcionando, pero la clase ya no es configuración: **no se escanea ningún componente** y **no hay autoconfiguración**.
- En una app web: no arranca Tomcat (no hay `ServletWebServerFactory`) → `ApplicationContextException: Unable to start ServletWebServerApplicationContext due to missing ServletWebServerFactory bean`.
- Los tests con `@SpringBootTest` fallan con `Unable to find a @SpringBootConfiguration`.
