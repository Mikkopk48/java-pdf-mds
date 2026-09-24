# @ResponseBody

> Nivel: esencial · Categoría: web / respuesta

## 1. Nombre
`@ResponseBody`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`).

## 4. Propósito
Indicar que el **valor de retorno** del método se debe escribir directamente en el **cuerpo de la respuesta HTTP** (serializado con un `HttpMessageConverter`), en lugar de interpretarse como nombre de vista.

## 5. Target
`ElementType.TYPE`, `ElementType.METHOD`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`).

## 7. Atributos
No tiene atributos.

## 8. Valores por defecto
No aplica.

## 9. Quién la procesa
`RequestResponseBodyMethodProcessor` (un `HandlerMethodReturnValueHandler`) en MVC; `ResponseBodyResultHandler` en WebFlux.

## 10. Cuándo se procesa
En cada petición, tras ejecutar el método handler.

## 11. Efecto observable
El objeto devuelto aparece como JSON (o el formato negociado) en el cuerpo de la respuesta.

## 12. Qué ocurre internamente
1. `supportsReturnType()` comprueba si el método o su clase tienen `@ResponseBody`.
2. Negociación de contenido: combina `Accept` del cliente, `produces` del mapeo y tipos que los conversores pueden escribir.
3. Aplica `ResponseBodyAdvice` (hooks para envolver o modificar la respuesta).
4. El conversor escribe en el `OutputStream` de la respuesta y se marca `mavContainer.setRequestHandled(true)` (no hay vista).
5. Si ningún conversor puede producir el tipo pedido → `HttpMediaTypeNotAcceptableException` → **406**.

## 13. Relación con otras anotaciones
- Incluida en `@RestController` y `@RestControllerAdvice`.
- Contraparte de `@RequestBody`.
- `@ResponseStatus` para el código.
- Anotaciones Jackson controlan el formato (`@JsonInclude`, `@JsonFormat`).

## 14. Dependencias necesarias
`spring-boot-starter-web` / `webflux`.

## 15. Alternativas
`@RestController` (a nivel de clase); devolver `ResponseEntity<T>` (implícitamente escribe cuerpo, sin necesitar `@ResponseBody`).

## 16. Limitaciones
- `String` devuelto se escribe como texto plano (`StringHttpMessageConverter`), no como JSON entre comillas.
- Objetos grandes se serializan en memoria; para streams usa `StreamingResponseBody` o `ResponseEntity<Resource>`.

## 17. Errores comunes
- Ponerla en un `@RestController` (redundante).
- Serializar entidades con relaciones lazy fuera de transacción.
- Cambiar un método de `@Controller` a JSON y olvidar la anotación → error de vista.

## 18. Ejemplo básico
```java
@Controller
public class EstadoController {
    @GetMapping("/estado")
    @ResponseBody
    public Map<String, String> estado() {
        return Map.of("estado", "OK");
    }
}
```

## 19. Ejemplo real
**FinTech — portal web MVC (vistas Thymeleaf) con un endpoint JSON para el gráfico de gastos** consumido por JavaScript en la misma página:
```java
@Controller
@RequestMapping("/panel")
public class PanelController {

    @GetMapping
    public String panel() { return "panel/index"; }             // HTML

    @GetMapping(path = "/gastos-por-categoria", produces = MediaType.APPLICATION_JSON_VALUE)
    @ResponseBody
    public List<GastoCategoriaDto> gastos(@AuthenticationPrincipal OidcUser user,
                                          @RequestParam YearMonth mes) {
        return analitica.gastosPorCategoria(user.getSubject(), mes); // JSON para el gráfico
    }
}
```
**Otro dominio — CMS**: un `@Controller` que sirve páginas y expone `@ResponseBody` para autoguardado de borradores.

## 20. Qué ocurre si la elimino
En un `@Controller`, el valor de retorno se interpreta como **nombre de vista**: devolver un objeto provoca que se use el nombre derivado de la URL como vista y un error de plantilla no encontrada, o `Circular view path` (500).
