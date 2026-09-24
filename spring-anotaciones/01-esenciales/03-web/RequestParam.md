# @RequestParam

> Nivel: esencial · Categoría: web / parámetros

## 1. Nombre
`@RequestParam`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`).

## 4. Propósito
Enlazar un **parámetro de la petición** — query string (`?desde=2026-01-01`), datos de formulario (`application/x-www-form-urlencoded`) o partes de un multipart — a un parámetro del método.

## 5. Target
`ElementType.PARAMETER`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` / `name` | `String` | Nombre del parámetro HTTP. |
| `required` | `boolean` | Si es obligatorio. |
| `defaultValue` | `String` | Valor si falta o viene vacío. **Poner un default implica `required = false`.** |

## 8. Valores por defecto
| Atributo | Default |
|---|---|
| `value`/`name` | `""` → nombre del parámetro Java |
| `required` | `true` |
| `defaultValue` | `ValueConstants.DEFAULT_NONE` (cadena interna que significa "sin default") |

## 9. Quién la procesa
`RequestParamMethodArgumentResolver` (y `RequestParamMapMethodArgumentResolver` para `Map`/`MultiValueMap`).

## 10. Cuándo se procesa
En cada petición, al resolver argumentos.

## 11. Efecto observable
`GET /movimientos?desde=2026-01-01&tipo=DEBITO` → `desde = LocalDate(2026-01-01)`, `tipo = DEBITO`.

## 12. Qué ocurre internamente
1. Lee `request.getParameterValues(nombre)` (o las partes multipart si el tipo es `MultipartFile`).
2. Si no hay valor: aplica `defaultValue`; si no hay default y es `required` → `MissingServletRequestParameterException` → **400**.
3. Convierte con `WebDataBinder`/`ConversionService`: `List<String>` acepta `?x=a&x=b` o `?x=a,b`; `Optional<T>` se trata como no obligatorio.
4. Error de conversión → `MethodArgumentTypeMismatchException` → **400**.

## 13. Relación con otras anotaciones
- `@PathVariable` (ruta) vs `@RequestParam` (query).
- `@DateTimeFormat` / `@NumberFormat` para formatos.
- Con `@Validated` en la clase o validación de métodos (Spring 6.1+): `@RequestParam @Min(1) int pagina`.
- `@ModelAttribute` para agrupar muchos parámetros en un objeto.

## 14. Dependencias necesarias
`spring-boot-starter-web` / `webflux`.

## 15. Alternativas
- Omitir la anotación (tipos simples se tratan como `@RequestParam` opcional).
- Objeto con `@ModelAttribute` (o sin anotación) para filtros con muchos campos.
- `Pageable` de Spring Data para `page`, `size`, `sort`.

## 16. Limitaciones
- No lee el cuerpo JSON (para eso `@RequestBody`).
- Los valores en la URL quedan en logs, historial del navegador y proxies: nunca datos sensibles.

## 17. Errores comunes
- Primitivo (`int`) con `required = false` sin default → `IllegalStateException: Optional int parameter 'x' is present but cannot be translated into a null value`.
- Fechas sin `@DateTimeFormat` en formatos no ISO → 400.
- Poner `defaultValue` y además `required = true` esperando error: el default gana.
- Enviar PAN de tarjeta o tokens en query string.

## 18. Ejemplo básico
```java
@GetMapping("/buscar")
public List<Producto> buscar(@RequestParam String q,
                             @RequestParam(defaultValue = "0") int pagina) {
    return servicio.buscar(q, pagina);
}
```

## 19. Ejemplo real
**FinTech — cotización de cambio de divisas**:
```java
@GetMapping("/api/v1/cotizaciones")
public CotizacionDto cotizar(
        @RequestParam Currency origen,                               // ?origen=USD
        @RequestParam Currency destino,                              // &destino=ARS
        @RequestParam @DecimalMin("0.01") BigDecimal importe,         // &importe=250.00
        @RequestParam(defaultValue = "VENTA") TipoCotizacion tipo,
        @RequestParam(required = false)
        @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate fechaValor) {

    return fx.cotizar(origen, destino, importe, tipo,
                      fechaValor != null ? fechaValor : LocalDate.now(reloj));
}
```
**Otro dominio — viajes**: `GET /vuelos?origen=COR&destino=MAD&fecha=2026-12-20&pasajeros=2`.

## 20. Qué ocurre si la elimino
Para tipos simples, Spring aplica un `@RequestParam` **implícito no obligatorio**: sigue funcionando, pero si el parámetro falta llega `null` (o error si es primitivo) en vez de un 400 claro. Para tipos complejos, se trata como `@ModelAttribute`.
