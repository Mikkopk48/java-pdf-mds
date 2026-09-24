# 01 · Esenciales

Las 53 anotaciones que necesitas para construir una API REST con Spring Boot: arranque, inyección de dependencias, configuración, endpoints, persistencia con JPA, transacciones y validación de entrada.

**Orden de estudio sugerido:** core → configuración → web → datos → validación. Al terminar esta carpeta deberías poder escribir, sin mirar, un microservicio de cuentas con alta, consulta, transferencia transaccional y errores bien formateados.

## 01-core — contenedor e inyección (11)
| Anotación | Para qué, en una línea |
|---|---|
| [@SpringBootApplication](01-core/SpringBootApplication.md) | Clase principal: configuración + autoconfiguración + escaneo |
| [@Component](01-core/Component.md) | Estereotipo genérico: "esto es un bean" |
| [@Service](01-core/Service.md) | Bean de lógica de negocio |
| [@Repository](01-core/Repository.md) | Bean de acceso a datos + traducción de excepciones |
| [@Controller](01-core/Controller.md) | Controlador web con vistas |
| [@Configuration](01-core/Configuration.md) | Clase que declara beans con `@Bean` |
| [@Bean](01-core/Bean.md) | Método que produce un bean |
| [@Autowired](01-core/Autowired.md) | Inyección de dependencias |
| [@Qualifier](01-core/Qualifier.md) | Elegir entre varios beans del mismo tipo |
| [@Primary](01-core/Primary.md) | Bean preferido por defecto |
| [@Value](01-core/Value.md) | Inyectar una propiedad o expresión SpEL |

## 02-configuracion — propiedades, perfiles y escaneo (7)
| Anotación | Para qué, en una línea |
|---|---|
| [@ConfigurationProperties](02-configuracion/ConfigurationProperties.md) | Enlazar un grupo de propiedades a un objeto tipado |
| [@EnableConfigurationProperties](02-configuracion/EnableConfigurationProperties.md) | Registrar clases de propiedades concretas |
| [@ConfigurationPropertiesScan](02-configuracion/ConfigurationPropertiesScan.md) | Registrar clases de propiedades por escaneo |
| [@Profile](02-configuracion/Profile.md) | Bean solo en ciertos entornos |
| [@ComponentScan](02-configuracion/ComponentScan.md) | Qué paquetes escanear |
| [@EnableAutoConfiguration](02-configuracion/EnableAutoConfiguration.md) | Activar la autoconfiguración de Boot |
| [@SpringBootConfiguration](02-configuracion/SpringBootConfiguration.md) | Configuración principal, ancla de los tests |

## 03-web — API REST (16)
| Anotación | Para qué, en una línea |
|---|---|
| [@RestController](03-web/RestController.md) | Controlador que devuelve JSON |
| [@RequestMapping](03-web/RequestMapping.md) | Mapeo general de rutas y condiciones |
| [@GetMapping](03-web/GetMapping.md) | Leer |
| [@PostMapping](03-web/PostMapping.md) | Crear / ejecutar acción |
| [@PutMapping](03-web/PutMapping.md) | Reemplazar (idempotente) |
| [@DeleteMapping](03-web/DeleteMapping.md) | Eliminar |
| [@PatchMapping](03-web/PatchMapping.md) | Modificar parcialmente |
| [@PathVariable](03-web/PathVariable.md) | Valor de la ruta `/{id}` |
| [@RequestParam](03-web/RequestParam.md) | Valor de la query `?x=` |
| [@RequestBody](03-web/RequestBody.md) | Cuerpo JSON → objeto |
| [@ResponseBody](03-web/ResponseBody.md) | Objeto → cuerpo JSON |
| [@ResponseStatus](03-web/ResponseStatus.md) | Código HTTP fijo |
| [@RequestHeader](03-web/RequestHeader.md) | Valor de una cabecera |
| [@ExceptionHandler](03-web/ExceptionHandler.md) | Método que convierte excepciones en respuestas |
| [@ControllerAdvice](03-web/ControllerAdvice.md) | Lógica común a controladores (vistas) |
| [@RestControllerAdvice](03-web/RestControllerAdvice.md) | Manejo global de errores de la API |

## 04-datos — JPA y transacciones (6)
| Anotación | Para qué, en una línea |
|---|---|
| [@Entity](04-datos/Entity.md) | Clase persistente |
| [@Table](04-datos/Table.md) | Tabla, esquema, índices |
| [@Id](04-datos/Id.md) | Clave primaria |
| [@GeneratedValue](04-datos/GeneratedValue.md) | Generación de la clave |
| [@Column](04-datos/Column.md) | Mapeo de columna (nombre, precisión, nulabilidad) |
| [@Transactional](04-datos/Transactional.md) | Todo o nada |

## 05-validacion — Bean Validation (13)
| Anotación | Para qué, en una línea |
|---|---|
| [@Valid](05-validacion/Valid.md) | Disparar validación y cascada |
| [@Validated](05-validacion/Validated.md) | Validación con grupos y en métodos de servicios |
| [@NotNull](05-validacion/NotNull.md) | No nulo |
| [@NotBlank](05-validacion/NotBlank.md) | Texto con contenido real |
| [@NotEmpty](05-validacion/NotEmpty.md) | Texto/colección no vacía |
| [@Size](05-validacion/Size.md) | Longitud o tamaño |
| [@Min](05-validacion/Min.md) | Mínimo entero |
| [@Max](05-validacion/Max.md) | Máximo entero |
| [@Email](05-validacion/Email.md) | Formato de correo |
| [@Pattern](05-validacion/Pattern.md) | Expresión regular |
| [@Positive](05-validacion/Positive.md) | Mayor que cero |
| [@Digits](05-validacion/Digits.md) | Dígitos enteros y decimales (dinero) |
| [@DecimalMin](05-validacion/DecimalMin.md) | Mínimo decimal exacto |

## Ejercicio de cierre
Construye `mini-banco` usando solo estas 53 anotaciones:
1. Entidad `Cuenta` (IBAN único, saldo `NUMERIC(19,4)`, `@Version`… — `@Version` la verás en nivel medio, puedes adelantarte).
2. `POST /cuentas`, `GET /cuentas/{iban}`, `POST /transferencias` con `Idempotency-Key`.
3. `TransferenciaService` con `@Transactional` y reglas de saldo.
4. Límite diario en `@ConfigurationProperties` validado al arrancar.
5. `@RestControllerAdvice` que devuelva `ProblemDetail` para saldo insuficiente (422), cuenta inexistente (404) y validación (400).
6. Perfil `dev` con H2 y perfil `prod` con PostgreSQL.

Luego borra una anotación cada vez y comprueba que ocurre lo que dice la sección 20 de su archivo.
