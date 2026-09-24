# @RequestHeader

> Nivel: esencial · Categoría: web / parámetros

## 1. Nombre
`@RequestHeader`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`).

## 4. Propósito
Enlazar el valor de una **cabecera HTTP** a un parámetro del método handler (o todas las cabeceras a un `Map`/`HttpHeaders`).

## 5. Target
`ElementType.PARAMETER`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` / `name` | `String` | Nombre de la cabecera (no distingue mayúsculas). |
| `required` | `boolean` | Si es obligatoria. |
| `defaultValue` | `String` | Valor por defecto (implica `required = false`). |

## 8. Valores por defecto
`name = ""` (nombre del parámetro), `required = true`, `defaultValue = ValueConstants.DEFAULT_NONE`.

## 9. Quién la procesa
`RequestHeaderMethodArgumentResolver` (valor único) y `RequestHeaderMapMethodArgumentResolver` (`Map<String,String>`, `MultiValueMap`, `HttpHeaders`).

## 10. Cuándo se procesa
En cada petición, al resolver argumentos.

## 11. Efecto observable
`X-Correlation-Id: abc-123` → parámetro `correlationId = "abc-123"`.

## 12. Qué ocurre internamente
1. Lee `request.getHeaders(nombre)`.
2. Si falta: default o `MissingRequestHeaderException` → **400**.
3. Convierte el valor (`UUID`, `Locale`, `List<String>` separando por comas, `long`…).

## 13. Relación con otras anotaciones
`@RequestParam`, `@PathVariable`, `@CookieValue` (misma familia). En seguridad, la cabecera `Authorization` la procesa Spring Security; en el controlador se usa `@AuthenticationPrincipal`.

## 14. Dependencias necesarias
`spring-boot-starter-web` / `webflux`.

## 15. Alternativas
- `HttpServletRequest.getHeader()`.
- Un `Filter`/`HandlerInterceptor` para cabeceras transversales (correlation id → MDC para logs) en vez de repetir el parámetro en todos los métodos.

## 16. Limitaciones
- Pensada para cabeceras concretas de un endpoint; para cabeceras globales es mejor un filtro.
- El nombre del parámetro Java no puede contener guiones: con cabeceras tipo `X-Api-Key` hay que indicar `name` explícitamente.

## 17. Errores comunes
- Omitir el nombre en `@RequestHeader String xApiKey` → busca la cabecera `xApiKey`, no `X-Api-Key`.
- Leer `Authorization` a mano y validar el token en el controlador (debe hacerlo Spring Security).
- Confiar en `X-Forwarded-For` sin configurar `server.forward-headers-strategy` (suplantación de IP).

## 18. Ejemplo básico
```java
@GetMapping("/saludo")
public String saludo(@RequestHeader(name = "Accept-Language", defaultValue = "es") Locale idioma) {
    return mensajes.getMessage("saludo", null, idioma);
}
```

## 19. Ejemplo real
**FinTech — Open Banking**: los TPP (terceros autorizados) deben enviar cabeceras normativas como identificador de petición y consentimiento:
```java
@GetMapping("/open-banking/v1/cuentas/{cuentaId}/saldos")
public SaldosOpenBankingDto saldos(
        @PathVariable String cuentaId,
        @RequestHeader("X-Request-ID") UUID requestId,
        @RequestHeader("Consent-ID") String consentId,
        @RequestHeader(name = "PSU-IP-Address", required = false) String ipCliente) {

    consentimientos.validar(consentId, cuentaId, Permiso.LEER_SALDOS); // 403 si no cubre la cuenta
    return openBanking.saldos(cuentaId, requestId, ipCliente);
}
```
**Otro dominio — SaaS multi‑tenant**: `@RequestHeader("X-Tenant-Id") String tenant` para enrutar a la base de datos del cliente.

## 20. Qué ocurre si la elimino
Para tipos simples, Spring asume `@RequestParam` implícito: buscará `?requestId=` en la query y no la cabecera → valores `null` o 400.
