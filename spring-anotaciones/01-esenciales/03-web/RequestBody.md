# @RequestBody

> Nivel: esencial · Categoría: web / parámetros

## 1. Nombre
`@RequestBody`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`).

## 4. Propósito
Leer el **cuerpo de la petición HTTP** y convertirlo (deserializarlo) a un objeto Java usando un `HttpMessageConverter` — típicamente JSON con Jackson.

## 5. Target
`ElementType.PARAMETER`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `required` | `boolean` | Si el cuerpo es obligatorio. |

## 8. Valores por defecto
`required = true`.

## 9. Quién la procesa
`RequestResponseBodyMethodProcessor` (MVC), apoyado en la lista de `HttpMessageConverter`s configurada (Jackson, String, ByteArray, XML si hay Jackson XML…). En WebFlux, `RequestBodyMethodArgumentResolver` con `HttpMessageReader`s.

## 10. Cuándo se procesa
En cada petición, al resolver argumentos, antes de invocar el método.

## 11. Efecto observable
Un `POST` con `{"importe": 100.50, "moneda": "ARS"}` llega como un objeto `PagoRequest` con esos campos.

## 12. Qué ocurre internamente
1. Lee el `Content-Type`. Elige el primer conversor que `canRead(tipoParametro, contentType)`.
2. Aplica `RequestBodyAdvice` (hooks antes/después de leer; p. ej. para descifrar).
3. El conversor deserializa (Jackson `ObjectMapper.readValue`).
4. Si hay `@Valid`/`@Validated`, valida; errores → `MethodArgumentNotValidException` → **400**.
5. JSON mal formado → `HttpMessageNotReadableException` → **400**. Tipo no soportado → `HttpMediaTypeNotSupportedException` → **415**. Cuerpo vacío con `required = true` → 400.

## 13. Relación con otras anotaciones
- `@Valid` / `@Validated` para validar el DTO.
- `@PostMapping`, `@PutMapping`, `@PatchMapping`.
- Anotaciones Jackson en el DTO (`@JsonProperty`, `@JsonFormat`, `@JsonIgnoreProperties`).
- `@ResponseBody` es su contraparte para la salida.

## 14. Dependencias necesarias
`spring-boot-starter-web` (incluye Jackson vía `spring-boot-starter-json`; en Boot 4, Jackson 3).

## 15. Alternativas
- `HttpEntity<T>` / `RequestEntity<T>` (cuerpo + cabeceras).
- `@ModelAttribute` para formularios.
- `@RequestPart` para multipart.

## 16. Limitaciones
- Solo **un** `@RequestBody` por método (el stream se lee una vez).
- El cuerpo se carga en memoria: para ficheros grandes usa streaming o multipart.
- Por defecto Jackson ignora propiedades desconocidas en Spring Boot (`FAIL_ON_UNKNOWN_PROPERTIES = false`): errores tipográficos del cliente pasan en silencio.

## 17. Errores comunes
- Olvidarla → objeto con todo a `null`.
- Deserializar directamente a la **entidad JPA** → *mass assignment* (el cliente puede enviar `"saldo": 1000000`).
- `double`/`float` para importes → errores de redondeo. Usa `BigDecimal`.
- Olvidar `@Valid` → no se valida nada.
- Records sin constructor accesible o DTO sin constructor por defecto ni `@JsonCreator` (Jackson soporta records de forma nativa desde 2.12).

## 18. Ejemplo básico
```java
public record NuevoComentario(String autor, String texto) {}

@PostMapping("/comentarios")
public Comentario crear(@RequestBody NuevoComentario req) {
    return servicio.crear(req);
}
```

## 19. Ejemplo real
**FinTech — alta de un pago con tarjeta**, DTO específico (sin campos internos), importes con `BigDecimal` y validación:
```java
public record PagoTarjetaRequest(
        @NotNull @Positive @Digits(integer = 12, fraction = 2) BigDecimal importe,
        @NotNull Currency moneda,
        @NotBlank @Size(max = 64) String comercioId,
        @NotBlank String tokenTarjeta,          // nunca el PAN en claro: token del vault PCI
        @Size(max = 140) String concepto) {}

@PostMapping("/api/v1/pagos")
public ResponseEntity<PagoDto> pagar(@Valid @RequestBody PagoTarjetaRequest req) {
    PagoDto pago = pagos.autorizar(req);
    return ResponseEntity.status(HttpStatus.CREATED).body(pago);
}
```
**Otro dominio — IoT**: `POST /lecturas` con un array de mediciones de sensores: `@RequestBody List<@Valid Lectura> lecturas`.

## 20. Qué ocurre si la elimino
Spring trata el parámetro como `@ModelAttribute`: intenta rellenarlo con parámetros de query/formulario, no con el JSON. Resultado: un objeto con todos los campos `null` (o error si es un record con parámetros primitivos).
