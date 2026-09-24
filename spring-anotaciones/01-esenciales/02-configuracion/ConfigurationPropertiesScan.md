# @ConfigurationPropertiesScan

> Nivel: esencial · Categoría: configuración externa

## 1. Nombre
`@ConfigurationPropertiesScan`

## 2. Paquete
`org.springframework.boot.context.properties`

## 3. Framework / librería
Spring Boot (`spring-boot`), desde la versión 2.2.

## 4. Propósito
**Escanear paquetes** buscando clases con `@ConfigurationProperties` y registrarlas como beans automáticamente, sin listarlas una a una.

## 5. Target
`ElementType.TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`). Meta‑anotada con `@Import(ConfigurationPropertiesScanRegistrar.class)` y `@EnableConfigurationProperties`.

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` / `basePackages` | `String[]` | Paquetes a escanear. |
| `basePackageClasses` | `Class<?>[]` | Clases cuyos paquetes se escanean (type‑safe). |

## 8. Valores por defecto
Todos `{}` → se escanea el paquete de la clase anotada y sus subpaquetes.

## 9. Quién la procesa
`ConfigurationPropertiesScanRegistrar`, que usa un `ClassPathScanningCandidateComponentProvider` con filtro por `@ConfigurationProperties`.

## 10. Cuándo se procesa
Arranque, al procesar las clases de configuración.

## 11. Efecto observable
Todas las clases/records `@ConfigurationProperties` del paquete quedan disponibles para inyección.

## 12. Qué ocurre internamente
1. Determina los paquetes base.
2. Escanea buscando `@ConfigurationProperties`.
3. **Excluye** las clases que ya son `@Component` (se registrarán por el escaneo normal).
4. Registra cada una con `ConfigurationPropertiesBeanRegistrar` (mismo mecanismo que `@EnableConfigurationProperties`).

## 13. Relación con otras anotaciones
- Incluye `@EnableConfigurationProperties`.
- Se suele colocar junto a `@SpringBootApplication`.
- No depende de `@ComponentScan` (usa su propio escaneo).

## 14. Dependencias necesarias
`spring-boot`.

## 15. Alternativas
`@EnableConfigurationProperties(Clase.class)`; `@Component` en clases JavaBean.

## 16. Limitaciones
- Escaneo de classpath: coste mínimo en arranque; en imágenes nativas se resuelve en AOT.
- No recoge clases fuera de los paquetes indicados.

## 17. Errores comunes
- Ponerlo en una clase de un subpaquete y esperar que encuentre propiedades en paquetes hermanos.
- Añadir `@Component` a un `record` de propiedades además del scan.

## 18. Ejemplo básico
```java
@SpringBootApplication
@ConfigurationPropertiesScan
public class App { ... }
```

## 19. Ejemplo real
**FinTech — microservicio de préstamos** con varias clases de propiedades en un paquete dedicado:
```java
package com.neobank.prestamos;

@SpringBootApplication
@ConfigurationPropertiesScan("com.neobank.prestamos.config")
public class PrestamosApplication { ... }

// com.neobank.prestamos.config
@ConfigurationProperties("prestamos.scoring")
public record ScoringProperties(int puntuacionMinima, BigDecimal ratioEndeudamientoMax) {}

@ConfigurationProperties("prestamos.tasas")
public record TasasProperties(BigDecimal tnaBase, BigDecimal spreadRiesgoAlto) {}

@ConfigurationProperties("prestamos.buro")
public record BuroCreditoProperties(URI url, Duration timeout) {}
```
**Otro dominio — videojuegos**: `MatchmakingProperties`, `RankingProperties` en un servidor de partidas.

## 20. Qué ocurre si la elimino
Las clases de propiedades dejan de registrarse: cualquier bean que las inyecte falla con `NoSuchBeanDefinitionException`.
