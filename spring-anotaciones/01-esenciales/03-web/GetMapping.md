# @GetMapping

> Nivel: esencial · Categoría: web / enrutamiento

## 1. Nombre
`@GetMapping`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`), desde 4.3.

## 4. Propósito
Atajo de `@RequestMapping(method = RequestMethod.GET)`: mapea peticiones **GET**, usadas para **leer** recursos sin modificar estado (seguras e idempotentes según HTTP).

## 5. Target
`ElementType.METHOD`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`). Meta‑anotada con `@RequestMapping(method = RequestMethod.GET)`.

## 7. Atributos
Todos son `@AliasFor` de `@RequestMapping`:

| Atributo | Tipo | Descripción |
|---|---|---|
| `name` | `String` | Nombre del mapeo. |
| `value` / `path` | `String[]` | Rutas. |
| `params` | `String[]` | Condiciones de parámetros. |
| `headers` | `String[]` | Condiciones de cabeceras. |
| `consumes` | `String[]` | Content-Type aceptado (raro en GET). |
| `produces` | `String[]` | Tipos producidos. |
| `version` | `String` | (Spring 7+) Versión de API. |

## 8. Valores por defecto
Todos vacíos. `@GetMapping` sin ruta → hereda la ruta de la clase.

## 9. Quién la procesa
`RequestMappingHandlerMapping` (la reconoce como `@RequestMapping` gracias a la meta‑anotación y `MergedAnnotations`).

## 10. Cuándo se procesa
Arranque (registro) y cada petición (matching).

## 11. Efecto observable
`GET /ruta` llega al método; otros métodos a esa ruta → 405 si no hay otro handler. Spring atiende también `HEAD` automáticamente para los GET.

## 12. Qué ocurre internamente
Igual que `@RequestMapping`: se sintetiza la anotación combinada (`AnnotatedElementUtils.findMergedAnnotation`) con `method = GET` fijo y se registra el `RequestMappingInfo`.

## 13. Relación con otras anotaciones
- Especialización de `@RequestMapping`.
- Con `@PathVariable`, `@RequestParam`, `@RequestHeader`, `@ModelAttribute`.
- Con caché HTTP: `ResponseEntity.ok().eTag(...)`, `CacheControl`.

## 14. Dependencias necesarias
`spring-boot-starter-web` / `webflux`.

## 15. Alternativas
`@RequestMapping(method = GET)`; `RouterFunctions.route().GET(...)`.

## 16. Limitaciones
- No debe tener cuerpo (`@RequestBody` en GET no está prohibido por Spring, pero muchos proxies y clientes lo descartan).
- No usar para operaciones con efectos (borrado, pagos): los crawlers, prefetch del navegador y reintentos automáticos pueden ejecutarlos.

## 17. Errores comunes
- GET que modifica estado (`GET /pagos/{id}/confirmar`).
- Devolver listas enormes sin paginación.
- Datos sensibles en query string (quedan en logs de acceso y proxies).

## 18. Ejemplo básico
```java
@GetMapping("/libros/{isbn}")
public Libro porIsbn(@PathVariable String isbn) {
    return servicio.buscar(isbn);
}
```

## 19. Ejemplo real
**FinTech — movimientos paginados con filtros** (el IBAN va en la ruta, no en query, y las fechas son obligatorias para acotar la consulta):
```java
@GetMapping("/api/v1/cuentas/{iban}/movimientos")
public Page<MovimientoDto> movimientos(
        @PathVariable String iban,
        @RequestParam LocalDate desde,
        @RequestParam LocalDate hasta,
        @RequestParam(required = false) TipoMovimiento tipo,
        @PageableDefault(size = 50, sort = "fechaValor", direction = Sort.Direction.DESC) Pageable pageable) {

    if (ChronoUnit.DAYS.between(desde, hasta) > 366)
        throw new RangoFechasInvalidoException("Máximo 1 año por consulta");
    return movimientos.buscar(iban, desde, hasta, tipo, pageable);
}
```
**Otro dominio — salud**: `GET /pacientes/{id}/citas?estado=PENDIENTE`.

## 20. Qué ocurre si la elimino
El método deja de estar mapeado: `GET` a esa ruta → 404 (o 405 si la ruta existe para otros métodos).
