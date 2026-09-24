# @Profile

> Nivel: esencial · Categoría: configuración / entornos

## 1. Nombre
`@Profile`

## 2. Paquete
`org.springframework.context.annotation`

## 3. Framework / librería
Spring Framework (`spring-context`).

## 4. Propósito
Registrar un componente o bean **solo si ciertos perfiles están activos** (`dev`, `test`, `prod`, `sandbox`…). Permite tener implementaciones distintas por entorno.

## 5. Target
`ElementType.TYPE`, `ElementType.METHOD`.

## 6. Retention
`RetentionPolicy.RUNTIME` (`@Documented`). Meta‑anotada con `@Conditional(ProfileCondition.class)`.

## 7. Atributos
| Atributo | Tipo | Descripción |
|---|---|---|
| `value` | `String[]` | Perfiles o **expresiones de perfil**: `"prod"`, `"!prod"`, `"prod & eu"`, `"dev | test"`, `"(a & b) | c"`. Varios valores se combinan con OR. |

## 8. Valores por defecto
No tiene default: es obligatorio.

## 9. Quién la procesa
`ProfileCondition`, evaluada por `ConditionEvaluator` durante el parseo de clases de configuración y el escaneo de componentes. Consulta `Environment.acceptsProfiles(Profiles.of(...))`.

## 10. Cuándo se procesa
Arranque, al registrar definiciones de beans (fase `REGISTER_BEAN` / `PARSE_CONFIGURATION`).

## 11. Efecto observable
- Log: `The following 1 profile is active: "prod"`.
- El bean existe o no existe según el perfil activo.

## 12. Qué ocurre internamente
1. Los perfiles activos salen de `spring.profiles.active` (properties, variable `SPRING_PROFILES_ACTIVE`, `--spring.profiles.active=`, `@ActiveProfiles` en tests). Si no hay ninguno, se usa `default`.
2. `ProfileCondition.matches()` parsea las expresiones con `Profiles.of()` y pregunta al `Environment`.
3. Si no coincide, la clase/método se descarta y **no** se registra la definición.

## 13. Relación con otras anotaciones
- Es un caso particular de `@Conditional`.
- `@ActiveProfiles` (tests) activa perfiles.
- Relacionada con los ficheros `application-{perfil}.yml` y con `spring.config.activate.on-profile` en documentos YAML multi‑documento.
- Para condiciones por propiedad es mejor `@ConditionalOnProperty`.

## 14. Dependencias necesarias
`spring-context`.

## 15. Alternativas
- `@ConditionalOnProperty("feature.x.enabled")`: más granular que un perfil de entorno.
- Ficheros `application-{perfil}.yml` para cambiar valores (no implementaciones).
- *Feature flags* (Unleash, LaunchDarkly, OpenFeature).

## 16. Limitaciones
- Se evalúa al arrancar: no cambia en caliente.
- Abusar de perfiles para features lleva a combinaciones difíciles de probar.
- Las expresiones con `&` y `|` mezcladas sin paréntesis no son válidas.

## 17. Errores comunes
- Olvidar activar el perfil en producción → se carga la implementación `default` (p. ej. un mock de pagos).
- Poner `@Profile("dev")` en la clase `@SpringBootApplication`.
- Usar `@Profile("!prod")` para beans peligrosos y desplegar prod con el perfil mal escrito (`produccion`) → el bean peligroso se activa. Mejor lista positiva: `@Profile({"dev","test"})`.

## 18. Ejemplo básico
```java
@Configuration
public class EmailConfig {
    @Bean @Profile("dev")
    EmailSender consola() { return msg -> System.out.println(msg); }

    @Bean @Profile("prod")
    EmailSender smtp(JavaMailSender mail) { return new SmtpEmailSender(mail); }
}
```

## 19. Ejemplo real
**FinTech — proveedor KYC (verificación de identidad)**: sandbox en desarrollo, proveedor real en producción, y nunca real en tests automáticos:
```java
public interface VerificadorIdentidad {
    ResultadoKyc verificar(SolicitudKyc solicitud);
}

@Service
@Profile({"dev", "test", "qa"})
class VerificadorIdentidadSandbox implements VerificadorIdentidad {
    public ResultadoKyc verificar(SolicitudKyc s) {
        // DNI terminado en 0 → rechazado, resto aprobado: permite probar ambos caminos
        return s.documento().endsWith("0") ? ResultadoKyc.rechazado("SANDBOX") : ResultadoKyc.aprobado();
    }
}

@Service
@Profile("prod & !contingencia")
class VerificadorIdentidadProveedor implements VerificadorIdentidad {
    // llamada HTTP al proveedor externo con mTLS
}

@Service
@Profile("prod & contingencia")
class VerificadorIdentidadManual implements VerificadorIdentidad {
    // encola la solicitud para revisión manual si el proveedor cae
}
```
**Otro dominio — e‑commerce**: `@Profile("local")` para cargar datos de ejemplo del catálogo al arrancar.

## 20. Qué ocurre si la elimino
El bean se registra **en todos los entornos**. Si hay varias implementaciones de la misma interfaz, aparece `NoUniqueBeanDefinitionException`; si solo queda la de sandbox, podrías aprobar identidades falsas en producción.
