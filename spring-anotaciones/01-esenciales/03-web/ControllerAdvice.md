# @ControllerAdvice

> Nivel: esencial · Categoría: web / transversal

## 1. Nombre
`@ControllerAdvice`

## 2. Paquete
`org.springframework.web.bind.annotation`

## 3. Framework / librería
Spring Framework (`spring-web`).

## 4. Propósito
Declarar una clase con lógica **compartida por muchos controladores**: métodos `@ExceptionHandler` (manejo global de errores), `@InitBinder` (configuración de binding) y `@ModelAttribute` (atributos comunes del modelo).

## 5. Target
`ElementType.TYPE`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`). Meta‑anotada con `@Component`.

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `name` | `String` | (6.1+) Nombre del bean. |
| `value` / `basePackages` | `String[]` | Aplica solo a controladores de estos paquetes. |
| `basePackageClasses` | `Class<?>[]` | Paquetes por clase marcadora. |
| `assignableTypes` | `Class<?>[]` | Aplica a controladores asignables a estos tipos. |
| `annotations` | `Class<? extends Annotation>[]` | Aplica a controladores con estas anotaciones. |

## 8. Valores por defecto
Todos vacíos → aplica a **todos** los controladores.

## 9. Quién la procesa
- Detección: `ControllerAdviceBean.findAnnotatedBeans(context)`.
- Uso: `ExceptionHandlerExceptionResolver` (para `@ExceptionHandler`) y `RequestMappingHandlerAdapter` (para `@InitBinder` y `@ModelAttribute`). También recoge `RequestBodyAdvice`/`ResponseBodyAdvice` implementados por la clase.

## 10. Cuándo se procesa
- Descubrimiento: arranque (`afterPropertiesSet` de los componentes MVC).
- Ejecución: en cada petición/excepción aplicable.

## 11. Efecto observable
Una excepción lanzada en cualquier controlador se transforma en la respuesta definida en el advice.

## 12. Qué ocurre internamente
1. Se crean `ControllerAdviceBean`s con un `HandlerTypePredicate` construido con los selectores (`basePackages`, `assignableTypes`…).
2. Se ordenan por `@Order`/`Ordered`.
3. Ante una excepción, primero se buscan handlers en el propio controlador; después, en los advices cuyo predicado acepte el tipo del controlador, en orden.

## 13. Relación con otras anotaciones
- `@RestControllerAdvice` = `@ControllerAdvice` + `@ResponseBody`.
- Contiene `@ExceptionHandler`, `@InitBinder`, `@ModelAttribute`.
- `@Order` para priorizar entre varios advices.

## 14. Dependencias necesarias
`spring-boot-starter-web` / `webflux`.

## 15. Alternativas
`@RestControllerAdvice` para APIs JSON; handlers locales en cada controlador; `HandlerExceptionResolver` propio.

## 16. Limitaciones
- Sin `@ResponseBody`, los métodos que devuelven objetos se tratan como vistas (usa `ResponseEntity` o `@RestControllerAdvice`).
- No intercepta excepciones de filtros.
- Selectores evaluados en tiempo de ejecución por cada controlador: muchos advices con selectores complejos tienen coste (menor).

## 17. Errores comunes
- Usarla en una API REST y devolver POJOs → Spring intenta resolver vistas.
- Varios advices sin `@Order` con handlers solapados → comportamiento que depende del orden de registro.
- Colocarla fuera del paquete escaneado.

## 18. Ejemplo básico
```java
@ControllerAdvice
public class ErroresWeb {
    @ExceptionHandler(RecursoNoEncontrado.class)
    public String noEncontrado(Model model, RecursoNoEncontrado ex) {
        model.addAttribute("mensaje", ex.getMessage());
        return "errores/404";
    }
}
```

## 19. Ejemplo real
**FinTech — portal de banca web (MVC con vistas)**: datos comunes en todas las páginas y manejo de sesión caducada, solo para controladores del portal:
```java
@ControllerAdvice(basePackages = "com.neobank.portal.web")
@Order(Ordered.HIGHEST_PRECEDENCE)
public class PortalAdvice {

    private final NotificacionesService notificaciones;

    public PortalAdvice(NotificacionesService notificaciones) { this.notificaciones = notificaciones; }

    @ModelAttribute("avisosPendientes")
    public long avisos(@AuthenticationPrincipal OidcUser user) {
        return user == null ? 0 : notificaciones.pendientes(user.getSubject());
    }

    @InitBinder
    public void binder(WebDataBinder binder) {
        binder.registerCustomEditor(String.class, new StringTrimmerEditor(true)); // "  " → null
    }

    @ExceptionHandler(OperacionExpiradaException.class)
    public String operacionExpirada(RedirectAttributes flash) {
        flash.addFlashAttribute("error", "La operación expiró por seguridad. Vuelva a iniciarla.");
        return "redirect:/panel";
    }
}
```
**Otro dominio — intranet**: advice que añade el menú según el rol del usuario a todas las vistas.

## 20. Qué ocurre si la elimino
Sus handlers dejan de aplicarse: las excepciones que manejaba llegan como 500 o a `/error`; los atributos de modelo comunes desaparecen de las vistas (`null` en plantillas); el `@InitBinder` global deja de ejecutarse.
