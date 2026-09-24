# @PathVariable

> Nivel: esencial · Categoría: web / parámetros

## 1. Nombre
`@PathVariable`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`).

## 4. Propósito
Extraer un valor de una **variable de la ruta** (`/cuentas/{iban}`) e inyectarlo en un parámetro del método handler, convertido al tipo del parámetro.

## 5. Target
`ElementType.PARAMETER`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` / `name` | `String` | Nombre de la variable en la plantilla de ruta. |
| `required` | `boolean` | Si la variable es obligatoria (desde 4.3.3). |

## 8. Valores por defecto
| Atributo | Default |
|---|---|
| `value`/`name` | `""` → se usa el nombre del parámetro Java (requiere compilar con `-parameters`) |
| `required` | `true` |

## 9. Quién la procesa
`PathVariableMethodArgumentResolver` (y `PathVariableMapMethodArgumentResolver` para `Map<String,String>`).

## 10. Cuándo se procesa
En cada petición, antes de invocar el método, al resolver argumentos.

## 11. Efecto observable
`GET /cuentas/ES7921000813610123456789` → el parámetro `iban` vale `"ES7921000813610123456789"`.

## 12. Qué ocurre internamente
1. Al hacer *matching* de la ruta, el `HandlerMapping` guarda las variables en el atributo de request `HandlerMapping.URI_TEMPLATE_VARIABLES_ATTRIBUTE`.
2. El resolver lee el valor por nombre.
3. `WebDataBinder` lo convierte al tipo (`Long`, `UUID`, `LocalDate`, enums…) con el `ConversionService`.
4. Si falla la conversión → `MethodArgumentTypeMismatchException` → **400**.
5. Si falta y es `required` → `MissingPathVariableException` → **500** (se considera error de configuración del servidor).

## 13. Relación con otras anotaciones
- Rutas en `@RequestMapping`/`@GetMapping`…
- `@Valid`/constraints sobre el parámetro con `@Validated` (o validación de métodos nativa de Spring 6.1+).
- `@MatrixVariable` para parámetros tipo `;clave=valor` en segmentos.

## 14. Dependencias necesarias
`spring-boot-starter-web` / `webflux`. Spring Boot configura `-parameters` en el `spring-boot-starter-parent`.

## 15. Alternativas
`@RequestParam` (query string); inyectar `HttpServletRequest` y parsear a mano (no recomendado).

## 16. Limitaciones
- Por defecto un `{var}` no incluye `/`. Para capturar el resto de la ruta: `{*resto}` (con `PathPatternParser`).
- Valores con punto (`/archivos/{nombre}` con `informe.pdf`) funcionan en Spring 6; en versiones antiguas se truncaba la extensión.

## 17. Errores comunes
- Compilar sin `-parameters` y omitir el nombre → en Spring 6.1+ error `Name for argument of type [...] not specified, and parameter name information not available via reflection`.
- Nombre distinto entre ruta y anotación (`{id}` vs `@PathVariable("cuentaId")`).
- No validar el formato (IDs arbitrarios llegan a la base de datos). Usa regex en la ruta: `{id:\\d+}`.
- Exponer IDs secuenciales que permiten enumerar recursos de otros usuarios.

## 18. Ejemplo básico
```java
@GetMapping("/usuarios/{id}")
public Usuario porId(@PathVariable Long id) {
    return servicio.buscar(id);
}
```

## 19. Ejemplo real
**FinTech — consulta de una operación concreta de una tarjeta** con validación de formato en la ruta:
```java
@GetMapping("/api/v1/tarjetas/{tarjetaId}/operaciones/{operacionId:[0-9a-f\\-]{36}}")
public OperacionDto operacion(@PathVariable UUID tarjetaId,
                              @PathVariable UUID operacionId,
                              @AuthenticationPrincipal Jwt jwt) {
    return operaciones.buscar(jwt.getSubject(), tarjetaId, operacionId); // verifica que la tarjeta es del cliente
}
```
**Otro dominio — educación**: `GET /cursos/{curso}/alumnos/{legajo}/notas`.

## 20. Qué ocurre si la elimino
Spring trata el parámetro como un `@RequestParam` implícito (para tipos simples) y lo busca en la query string: llegará `null`, o fallará con `400 Required parameter 'id' is not present` en tipos primitivos.
