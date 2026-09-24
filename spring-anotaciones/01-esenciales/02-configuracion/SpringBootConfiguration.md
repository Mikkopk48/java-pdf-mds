# @SpringBootConfiguration

> Nivel: esencial · Categoría: configuración / arranque

## 1. Nombre
`@SpringBootConfiguration`

## 2. Paquete
`org.springframework.boot`

## 3. Framework / librería
Spring Boot (`spring-boot`), desde 1.4.

## 4. Propósito
Marcar la **clase de configuración principal** de una aplicación Spring Boot. Funcionalmente es un `@Configuration`, pero además sirve como "ancla" que los tests usan para encontrar automáticamente la configuración de la app.

## 5. Target
`ElementType.TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`). Meta‑anotada con `@Configuration` e `@Indexed`.

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `proxyBeanMethods` | `boolean` | Alias de `@Configuration.proxyBeanMethods`. |

## 8. Valores por defecto
`proxyBeanMethods = true`.

## 9. Quién la procesa
- Como configuración: `ConfigurationClassPostProcessor`.
- En tests: `SpringBootTestContextBootstrapper` usa `AnnotatedClassFinder` para buscarla subiendo por los paquetes desde la clase de test.

## 10. Cuándo se procesa
Arranque de la app y arranque del contexto de test.

## 11. Efecto observable
`@SpringBootTest`, `@WebMvcTest`, `@DataJpaTest`… encuentran la configuración sin indicar `classes = ...`.

## 12. Qué ocurre internamente
En tests, el bootstrapper empieza en el paquete de la clase de test y sube (`com.banco.pagos.api` → `com.banco.pagos` → `com.banco`…) hasta encontrar una clase con `@SpringBootConfiguration`. Si encuentra más de una en el mismo paquete → error.

## 13. Relación con otras anotaciones
- Incluida en `@SpringBootApplication`.
- Especialización de `@Configuration`.
- Usada por `@SpringBootTest` y los test slices.

## 14. Dependencias necesarias
`spring-boot`.

## 15. Alternativas
`@Configuration` (pierdes la detección automática en tests: tendrás que indicar `@SpringBootTest(classes = MiConfig.class)`).

## 16. Limitaciones
Solo una por aplicación (en la misma jerarquía de paquetes).

## 17. Errores comunes
- Crear en `src/test` otra clase con `@SpringBootApplication` "para tests" → `Found multiple @SpringBootConfiguration annotated classes`. Usa `@TestConfiguration`.
- Tests en un paquete que no cuelga del paquete de la app → `Unable to find a @SpringBootConfiguration`.

## 18. Ejemplo básico
```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
public class App { }
```

## 19. Ejemplo real
**FinTech — test de un módulo de pagos** que encuentra solo la configuración de la app:
```java
package com.neobank.pagos.api;   // subpaquete de com.neobank

@WebMvcTest(PagoController.class)       // busca hacia arriba y encuentra com.neobank.CoreBankingApplication
class PagoControllerTest {
    @Autowired MockMvc mvc;
    @MockitoBean PagoService pagos;

    @Test
    void rechazaImporteNegativo() throws Exception {
        mvc.perform(post("/pagos").contentType(APPLICATION_JSON)
                .content("{\"importe\": -10, \"moneda\": \"ARS\"}"))
           .andExpect(status().isBadRequest());
    }
}
```
**Otro dominio — librería multi‑módulo**: un módulo de pruebas de integración con su propia `@SpringBootConfiguration` mínima en `src/test/java` para testear una librería que no tiene aplicación principal.

## 20. Qué ocurre si la elimino
Si la clase no tiene otra anotación de configuración, deja de ser configuración (ver `@SpringBootApplication`). Si la sustituyes por `@Configuration`, la app arranca igual, pero los tests fallan con `Unable to find a @SpringBootConfiguration, you need to use @ContextConfiguration or @SpringBootTest(classes=...)`.
