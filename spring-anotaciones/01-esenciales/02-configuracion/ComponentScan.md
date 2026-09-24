# @ComponentScan

> Nivel: esencial · Categoría: configuración / escaneo

## 1. Nombre
`@ComponentScan`

## 2. Paquete
`org.springframework.context.annotation`

## 3. Framework / librería
Spring Framework (`spring-context`).

## 4. Propósito
Indicar a Spring **qué paquetes recorrer** para detectar clases anotadas con `@Component` (y sus derivados) y registrarlas como beans.

## 5. Target
`ElementType.TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`). Es `@Repeatable(ComponentScans.class)`.

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` / `basePackages` | `String[]` | Paquetes base. |
| `basePackageClasses` | `Class<?>[]` | Clases marcadoras cuyos paquetes se escanean. |
| `nameGenerator` | `Class<? extends BeanNameGenerator>` | Generador de nombres de beans. |
| `scopeResolver` | `Class<? extends ScopeMetadataResolver>` | Cómo resolver el scope de los componentes. |
| `scopedProxy` | `ScopedProxyMode` | Proxies para beans con scope no singleton. |
| `resourcePattern` | `String` | Patrón de ficheros a considerar. |
| `useDefaultFilters` | `boolean` | Si detecta `@Component`, `@Repository`, `@Service`, `@Controller`, `@Named`… |
| `includeFilters` | `Filter[]` | Filtros adicionales de inclusión. |
| `excludeFilters` | `Filter[]` | Filtros de exclusión. |
| `lazyInit` | `boolean` | Registrar los beans escaneados como *lazy*. |

`@ComponentScan.Filter` tiene: `type` (`FilterType.ANNOTATION`, `ASSIGNABLE_TYPE`, `ASPECTJ`, `REGEX`, `CUSTOM`), `value`/`classes`, `pattern`.

## 8. Valores por defecto
| Atributo | Default |
|---|---|
| `basePackages`/`basePackageClasses` | `{}` → paquete de la clase anotada |
| `nameGenerator` | `BeanNameGenerator.class` (usa `AnnotationBeanNameGenerator`) |
| `scopeResolver` | `AnnotationScopeMetadataResolver.class` |
| `scopedProxy` | `ScopedProxyMode.DEFAULT` |
| `resourcePattern` | `"**/*.class"` |
| `useDefaultFilters` | `true` |
| `includeFilters`/`excludeFilters` | `{}` |
| `lazyInit` | `false` |

## 9. Quién la procesa
`ConfigurationClassPostProcessor` → `ComponentScanAnnotationParser` → `ClassPathBeanDefinitionScanner.doScan()`.

## 10. Cuándo se procesa
Arranque, al parsear la clase de configuración que la contiene, antes de instanciar beans.

## 11. Efecto observable
Los componentes de los paquetes indicados se registran; los excluidos no.

## 12. Qué ocurre internamente
1. Convierte cada paquete en un patrón `classpath*:com/banco/**/*.class`.
2. Lee cada clase con ASM (`MetadataReader`) sin cargarla.
3. Aplica `excludeFilters` y luego `includeFilters`.
4. Evalúa `@Conditional`/`@Profile`.
5. Registra las definiciones y, si alguna es a su vez `@Configuration`, la procesa recursivamente (sus `@Bean`, `@Import`, otros `@ComponentScan`).

## 13. Relación con otras anotaciones
- Incluida en `@SpringBootApplication` con dos filtros de exclusión: `TypeExcludeFilter` (lo usan los test slices) y `AutoConfigurationExcludeFilter` (evita escanear autoconfiguraciones).
- Detecta `@Component` y derivados.
- `@ComponentScans` agrupa varias.

## 14. Dependencias necesarias
`spring-context`.

## 15. Alternativas
- `scanBasePackages` en `@SpringBootApplication` (conserva los filtros de Boot).
- `@Import(Clase.class)` para registrar clases concretas sin escanear.
- Organizar paquetes bajo el paquete raíz y no usarla explícitamente.

## 16. Limitaciones
- Escanear paquetes amplios (`com`, `org`) ralentiza el arranque y puede registrar clases de librerías.
- No funciona con clases que no tengan anotación de estereotipo (salvo filtros `include`).

## 17. Errores comunes
- Añadir `@ComponentScan` junto a `@SpringBootApplication` → se sustituye el escaneo por defecto y se pierden los filtros de Boot; en tests `@WebMvcTest` empiezan a cargarse servicios y repositorios.
- Filtros `REGEX` que no coinciden por no escapar puntos.
- Escanear dos veces el mismo paquete desde configuraciones distintas.

## 18. Ejemplo básico
```java
@Configuration
@ComponentScan(basePackages = "com.ejemplo.utilidades")
public class UtilidadesConfig { }
```

## 19. Ejemplo real
**FinTech — monolito modular** que excluye los adaptadores de un proveedor de tarjetas que está en desuso, y registra implementaciones de una interfaz aunque no tengan anotación:
```java
@Configuration
@ComponentScan(
    basePackages = "com.neobank.tarjetas",
    excludeFilters = @ComponentScan.Filter(type = FilterType.REGEX, pattern = "com\\.neobank\\.tarjetas\\.legacy\\..*"),
    includeFilters = @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = ReglaFraude.class)
)
public class TarjetasModuleConfig { }
```
**Otro dominio — plugins**: una plataforma educativa que escanea `com.edu.plugins` para cargar extensiones de evaluación.

## 20. Qué ocurre si la elimino
Si no hay otro escaneo que cubra esos paquetes, sus componentes dejan de registrarse y la app falla al inyectarlos. Si la quitas de una clase que también tiene `@SpringBootApplication`, vuelve el escaneo por defecto (a veces es justo lo que arregla los tests).
