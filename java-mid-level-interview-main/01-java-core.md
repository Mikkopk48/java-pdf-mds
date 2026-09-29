## ☕ Java Core y Fundamentos

[⬆️ Volver al índice](./README.md)

---

## 📑 Contenidos de esta sección

### 🔹 Conceptos Básicos
1. [¿Cuál es la diferencia entre == y equals()?](#1-cuál-es-la-diferencia-entre--y-equals-en-java)
2. [¿Qué es el contrato entre hashCode() y equals()?](#2-qué-es-el-contrato-entre-hashcode-y-equals-por-qué-es-importante)
3. [¿Qué es la inmutabilidad?](#3-qué-es-la-inmutabilidad-por-qué-string-es-inmutable)
4. [¿Qué es el String Pool?](#4-qué-es-el-string-pool-cómo-funciona)
5. [¿Qué es Autoboxing y Unboxing?](#5-qué-es-autoboxing-y-unboxing-cuáles-son-sus-riesgos)
6. [¿Cuáles son los tipos primitivos?](#6-cuáles-son-los-tipos-de-datos-primitivos-en-java)
7. [Checked vs Unchecked Exceptions](#7-cuál-es-la-diferencia-entre-checked-y-unchecked-exceptions)
8. [final vs finally vs finalize](#8-cuál-es-la-diferencia-entre-final-finally-y-finalize)
9. [¿Qué son los Java Records?](#9-qué-son-los-java-records-y-cuándo-usarlos-java-14)
10. [¿Qué es el paso por valor?](#10-qué-es-el-paso-por-valor-en-java-java-pasa-objetos-por-referencia)
11. [var keyword (Java 10+)](#11-qué-es-la-inferencia-de-tipos-var-y-cuándo-usarla-java-10)
12. [¿Qué es el Garbage Collector?](#12-qué-es-el-garbage-collector-cómo-funciona)
13. [¿Qué son las clases Wrapper?](#13-qué-son-las-clases-wrapper-para-qué-sirven)

### 🔹 Programación Orientada a Objetos (OOP)
14. [Pilares de la POO](#14-cuáles-son-los-4-pilares-de-la-programación-orientada-a-objetos)
15. [Clase Abstracta vs Interface](#15-cuál-es-la-diferencia-entre-clase-abstracta-e-interface)
16. [¿Java soporta herencia múltiple?](#16-java-soporta-herencia-múltiple-cómo-se-soluciona)
17. [Sobrecarga vs Sobrescritura](#17-cuál-es-la-diferencia-entre-sobrecarga-overloading-y-sobrescritura-overriding)
18. [Modificadores de acceso](#18-cuáles-son-los-modificadores-de-acceso-en-java)
19. [static vs non-static](#19-cuál-es-la-diferencia-entre-miembros-static-y-no-static)
20. [Composición vs Herencia](#20-composición-vs-herencia-cuándo-usar-cada-una)

### 🔹 Colecciones (Collections Framework)
21. [Jerarquía de Collections](#21-cuál-es-la-jerarquía-de-la-collections-framework)
22. [ArrayList vs LinkedList](#22-cuándo-usar-arraylist-vs-linkedlist)
23. [HashMap vs TreeMap vs LinkedHashMap](#23-qué-diferencias-hay-entre-hashmap-treemap-y-linkedhashmap)
24. [¿Qué es un HashSet?](#24-qué-es-un-hashset-cómo-funciona-internamente)
25. [Diferencia entre List, Set y Map](#25-cuál-es-la-diferencia-entre-list-set-y-map)
26. [ConcurrentModificationException](#26-qué-es-concurrentmodificationexception-y-cómo-evitarla)

### 🔹 Java 8+ Features
27. [¿Qué es Optional?](#27-qué-es-optional-y-por-qué-debemos-usarlo)
28. [¿Qué es Streams API?](#28-qué-es-streams-api-cuáles-son-sus-características)
29. [map() vs flatMap()](#29-cuál-es-la-diferencia-entre-map-y-flatmap)
30. [Interfaces funcionales](#30-qué-es-una-interfaz-funcional-menciona-las-más-comunes)
31. [Method references](#31-qué-son-las-method-references-tipos-y-ejemplos)
32. [Collectors](#32-qué-es-el-collectors-y-cuáles-son-los-más-utilizados)
33. [Expresiones lambda](#33-qué-son-las-expresiones-lambda-cuál-es-su-sintaxis)

### 🔹 Concurrencia Básica
34. [synchronized vs volatile](#34-cuál-es-la-diferencia-entre-synchronized-y-volatile)
35. [¿Qué es ExecutorService?](#35-qué-es-executorservice-cómo-se-usa)
36. [¿Qué es un deadlock?](#36-qué-es-un-deadlock-y-cómo-evitarlo)
37. [Thread vs Runnable](#37-qué-diferencia-hay-entre-thread-y-runnable)

### 🔹 Clases y Métodos Útiles
38. [Clases de utilidad](#38-qué-son-las-clases-de-utilidad-en-java-menciona-las-más-importantes)
39. [try-with-resources](#39-qué-es-try-with-resources-y-para-qué-sirve)

### 🔹 Conceptos Avanzados
40. [Generics y Wildcards](#40-qué-son-los-generics-y-wildcards-en-java)
41. [Type Erasure](#41-qué-es-type-erasure-y-sus-limitaciones)
42. [Reflection API](#42-qué-es-reflection-y-cuándo-usarlo)
43. [Anotaciones personalizadas](#43-cómo-crear-anotaciones-personalizadas)
44. [ClassLoader y carga de clases](#44-qué-es-un-classloader-y-cómo-funciona)
45. [Weak, Soft y Phantom References](#45-qué-son-las-referencias-débiles-soft-weak-phantom)
46. [Fork/Join Framework](#46-qué-es-el-forkjoin-framework)
47. [CompletableFuture](#47-qué-es-completablefuture-y-programación-asíncrona)
48. [Sealed Classes (Java 17+)](#48-qué-son-las-sealed-classes-java-17)
49. [Pattern Matching (Java 16+)](#49-qué-es-pattern-matching-en-java)
50. [Text Blocks (Java 15+)](#50-qué-son-los-text-blocks-java-15)
51. [Virtual Threads (Java 21+)](#51-qué-son-los-virtual-threads-java-21)

---

### 🔹 Conceptos Básicos

#### 1. ¿Cuál es la diferencia entre == y equals() en Java?

**Respuesta:**

La diferencia fundamental está en **qué comparan**:

- **==** (Operador de igualdad):
  - Para **tipos primitivos** (`int`, `double`, `char`, `boolean`): Compara directamente los **valores**.
  - Para **objetos**: Compara las **referencias de memoria** (direcciones). Es decir, verifica si ambas variables apuntan al mismo objeto en memoria.

- **equals()** (Método):
  - Es un método definido en `java.lang.Object` y heredado por todas las clases.
  - Por defecto, `equals()` se comporta como `==` (compara referencias).
  - Sin embargo, muchas clases como `String`, `Integer`, `Date`, `List` lo **sobrescriben** para comparar el **contenido/estado** del objeto.

**Ejemplo:**
```java
String s1 = new String("Java");
String s2 = new String("Java");
String s3 = s1;

System.out.println(s1 == s2);      // false (diferentes referencias)
System.out.println(s1.equals(s2)); // true (mismo contenido)
System.out.println(s1 == s3);      // true (misma referencia)
```

**Best Practice:** Para comparar objetos, siempre usa `equals()`. Solo usa `==` cuando necesites verificar si son el mismo objeto en memoria o cuando trabajes con primitivos.

---

#### 2. ¿Qué es el contrato entre hashCode() y equals()? ¿Por qué es importante?

**Respuesta:**

El **contrato** establece reglas que deben cumplirse cuando sobrescribes estos métodos:

**Reglas del contrato:**
1. Si dos objetos son **iguales** según `equals()` → **deben** devolver el mismo `hashCode()`.
2. Si dos objetos tienen el mismo `hashCode()` → **no necesariamente** son iguales (pueden tener colisiones).
3. Si un objeto no cambia, su `hashCode()` debe ser **consistente** (devolver siempre el mismo valor).

**¿Por qué es importante?**

Las colecciones basadas en hash (`HashMap`, `HashSet`, `Hashtable`, `LinkedHashMap`) dependen de este contrato:

1. Cuando insertas un objeto, la colección usa `hashCode()` para calcular en qué "bucket" (cubeta) guardarlo.
2. Cuando buscas un objeto, usa `hashCode()` para encontrar el bucket correcto.
3. Dentro del bucket, usa `equals()` para encontrar el objeto exacto.

**Problema si rompes el contrato:**
```java
public class Person {
    private String name;
    
    public Person(String name) {
        this.name = name;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return Objects.equals(name, person.name);
    }
    // ❌ ERROR: No sobrescribimos hashCode()
    // Esto viola el contrato: objetos iguales deben tener el mismo hashCode
}

Person p1 = new Person("Juan");
Person p2 = new Person("Juan");

Set<Person> set = new HashSet<>();
set.add(p1);
System.out.println(set.contains(p2)); // false ❌ (debería ser true)
// El problema: equals() dice que son iguales, pero hashCode() es diferente
```

**Best Practice:** Usa las herramientas de tu IDE o `Objects.hash()` y `Objects.equals()` para generar estos métodos correctamente:

```java
@Override
public int hashCode() {
    return Objects.hash(name);
}
```

---

#### 3. ¿Qué es la inmutabilidad? ¿Por qué String es inmutable?

**Respuesta:**

Un objeto es **inmutable** si su estado (valores de sus campos) **no puede cambiar** después de ser creado.

**Características de objetos inmutables:**
- No tienen setters.
- Todos sus campos son `final`.
- Los campos son tipos primitivos o referencias a objetos inmutables.
- La clase es `final` (no puede ser heredada).

**¿Por qué String es inmutable?**

1. **Seguridad**: Los Strings se usan como claves de HashMap, parámetros de métodos, nombres de archivos, URLs, etc. Si fueran mutables, cambiar un String podría romper estas estructuras.

2. **Thread-Safety**: Múltiples hilos pueden compartir el mismo String sin necesidad de sincronización.

3. **String Pool (Interning)**: Java mantiene un pool de Strings literales para ahorrar memoria. Si `String` fuera mutable, cambiar un String afectaría a todos los que comparten la misma referencia.

4. **Caché del hashCode**: Como no cambia, el hash se calcula una vez y se reutiliza.

**Ejemplo:**
```java
String s = "Hola";
s = s + " Mundo";  // No modifica "Hola", crea un nuevo String "Hola Mundo"
```

**Otras clases inmutables en Java:**
- Wrappers: `Integer`, `Long`, `Double`, `Boolean`
- Fecha/Hora: `LocalDate`, `LocalDateTime`, `Instant`
- Números: `BigDecimal`, `BigInteger`

**Cómo crear una clase inmutable:**
```java
public final class Person {
    private final String name;
    private final int age;
    
    public Person(String name, int age) {
        // Validación defensiva
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be null or empty");
        }
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative");
        }
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    // No setters
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Person)) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

---

#### 4. ¿Qué es el String Pool? ¿Cómo funciona?

**Respuesta:**

El **String Pool** (también llamado String Intern Pool) es un área especial de memoria en el **Heap** donde Java almacena instancias únicas de Strings literales para optimizar el uso de memoria.

**¿Cómo funciona?**

Cuando creas un String literal, Java primero verifica si ya existe en el pool:
- Si existe → devuelve la referencia existente.
- Si no existe → crea el String y lo añade al pool.

**Ejemplo:**
```java
// Strings literales se agregan automáticamente al pool
String s1 = "Java";        // Crea "Java" en el String Pool
String s2 = "Java";        // Reutiliza el mismo objeto del pool
String s3 = new String("Java"); // Crea un NUEVO objeto en el Heap (fuera del pool)

System.out.println(s1 == s2);  // true (misma referencia del pool)
System.out.println(s1 == s3);  // false (s3 está en el Heap, no en el pool)
System.out.println(s1.equals(s3)); // true (mismo contenido)

// intern() busca o agrega el String al pool y devuelve su referencia
String s4 = s3.intern();   // Devuelve la referencia del pool (s1)
System.out.println(s1 == s4);  // true (ambos apuntan al mismo objeto del pool)
```

**Ventajas:**
- Ahorra memoria al reutilizar Strings comunes.
- Comparaciones rápidas con `==` para Strings literales.
- Ideal para Strings que se repiten frecuentemente.

**Desventajas:**
- El pool consume espacio del Heap.
- `intern()` puede ser costoso si el pool es grande.
- No recomendado para Strings únicos o dinámicos.

**Ubicación:**
- Java 7+: String Pool está en el Heap.
- Java 6 y anterior: Estaba en PermGen (memoria no GC, causaba problemas).

---

#### 5. ¿Qué es Autoboxing y Unboxing? ¿Cuáles son sus riesgos?

**Respuesta:**

**Autoboxing** y **Unboxing** son conversiones automáticas entre tipos primitivos y sus clases Wrapper correspondientes, introducidas en Java 5.

**Autoboxing:** Conversión automática de primitivo → Wrapper
```java
int primitivo = 10;
Integer wrapper = primitivo;  // Autoboxing (equivale a Integer.valueOf(10))
```

**Unboxing:** Conversión automática de Wrapper → primitivo
```java
Integer wrapper = 20;
int primitivo = wrapper;  // Unboxing (equivale a wrapper.intValue())
```

**Riesgos y problemas comunes:**

1. **NullPointerException** (el más común):
```java
Integer numero = null;
int valor = numero;  // ❌ NullPointerException al hacer unboxing

// ✅ SOLUCIÓN: Siempre validar null antes de unboxing
Integer numero = obtenerNumero();
int valor = (numero != null) ? numero : 0;  // Valor por defecto

// ✅ Mejor aún: usar Optional para API más expresiva
Optional<Integer> optNumero = Optional.ofNullable(obtenerNumero());
int valor = optNumero.orElse(0);
```

2. **Problemas de rendimiento** en bucles:
```java
// ❌ MAL: Autoboxing/unboxing en cada iteración (impacto en performance)
Integer suma = 0;
for (int i = 0; i < 1000; i++) {
    suma += i;  // Unboxing de suma, operación, Autoboxing del resultado
}

// ✅ BIEN: Usa primitivos para operaciones intensivas
int suma = 0;
for (int i = 0; i < 1000; i++) {
    suma += i;
}
```

3. **Comparaciones inesperadas** con objetos cacheados:
```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b);  // true (están en la caché -128 a 127)

Integer c = 128;
Integer d = 128;
System.out.println(c == d);  // false (fuera de la caché, diferentes objetos)
```

**Best Practices:**
- Usa primitivos cuando no necesites `null`.
- Siempre verifica `null` antes del unboxing.
- Usa `equals()` para comparar Wrappers, no `==`.
- Evita Wrappers en código crítico de rendimiento.

---

#### 6. ¿Cuáles son los tipos de datos primitivos en Java?

**Respuesta:**

Java tiene **8 tipos primitivos**. Los más utilizados son:

| Tipo | Tamaño | Valor por defecto | Uso común |
|------|--------|-------------------|-----------|
| **int** | 32 bits | 0 | Enteros generales |
| **long** | 64 bits | 0L | Números grandes, timestamps, IDs |
| **double** | 64 bits | 0.0d | Decimales (estándar) |
| **boolean** | JVM-dependent | false | Banderas, condiciones |
| **char** | 16 bits | '\u0000' | Caracteres Unicode |
| **byte** | 8 bits | 0 | Datos binarios, streams |
| **short** | 16 bits | 0 | Raramente usado |
| **float** | 32 bits | 0.0f | Raramente usado (usa double) |

**Notas clave:**
- **int** es el tipo entero por defecto, **double** el decimal por defecto.
- **long** es ideal para IDs, timestamps: `long userId = 123456789L;`
- **boolean** no tiene tamaño garantizado por especificación.
- Evita **float** a menos que tengas restricciones de memoria; **double** es el estándar.

---

#### 7. ¿Cuál es la diferencia entre Checked y Unchecked Exceptions?

**Respuesta:**

La diferencia principal está en **cuándo se verifican** y **si estás obligado a manejarlas**.

**Checked Exceptions (Verificadas en tiempo de compilación):**

- El compilador te **obliga** a manejarlas con `try-catch` o declararlas con `throws`.
- Heredan de `Exception` (pero NO de `RuntimeException`).
- Representan **condiciones recuperables** o errores **esperables** en el negocio.

**Ejemplos comunes:**
- `IOException` (problemas de I/O: archivo no encontrado, red caída).
- `SQLException` (problemas con BD).
- `ClassNotFoundException`
- `ParseException`

```java
// ❌ No compila sin try-catch o throws
public void leerArchivo() {
    FileReader reader = new FileReader("file.txt"); // Error de compilación
}

// ✅ Opción 1: try-catch
public void leerArchivo() {
    try {
        FileReader reader = new FileReader("file.txt");
    } catch (FileNotFoundException e) {
        log.error("Archivo no encontrado", e);
    }
}

// ✅ Opción 2: throws (delega el manejo)
public void leerArchivo() throws FileNotFoundException {
    FileReader reader = new FileReader("file.txt");
}
```

**Unchecked Exceptions (NO verificadas):**

- **NO** estás obligado a manejarlas.
- Heredan de `RuntimeException`.
- Representan **errores de programación** o condiciones **irrecuperables**.

**Ejemplos comunes:**
- `NullPointerException` (objeto null inesperado).
- `IllegalArgumentException` (argumento inválido).
- `IndexOutOfBoundsException` (índice fuera de rango).
- `ArithmeticException` (división por cero).
- `ClassCastException` (cast inválido).

```java
// Compila sin problemas (aunque puede fallar en runtime)
public int dividir(int a, int b) {
    return a / b;  // Puede lanzar ArithmeticException si b=0
}

// Best practice: Validar antes
public int dividir(int a, int b) {
    if (b == 0) {
        throw new IllegalArgumentException("Divisor no puede ser cero");
    }
    return a / b;
}
```

**¿Cuándo usar cada una?**

| Tipo | Cuándo usar | Ejemplo |
|------|-------------|----------|
| **Checked** | Errores **recuperables** del negocio o externos | Usuario no encontrado, archivo no existe, timeout de API |
| **Unchecked** | Errores de **programación** o violaciones de contrato | Parámetro null, índice inválido, estado inconsistente |

**Best Practices:**
- ✅ Usa **Checked** para errores que el llamador **debe** manejar (ej: `UserNotFoundException`).
- ✅ Usa **Unchecked** para errores que indican bugs (ej: `IllegalArgumentException`).
- ✅ No abuses de Checked Exceptions; Spring y frameworks modernos prefieren Unchecked.
- ✅ Documenta en Javadoc las Unchecked Exceptions que tu método puede lanzar.

**Ejemplo en Spring:**
```java
// ✅ Unchecked custom exception
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

@GetMapping("/users/{id}")
public UserDTO getUser(@PathVariable Long id) {
    return userRepository.findById(id)
        .map(this::toDTO)
        .orElseThrow(() -> new ResourceNotFoundException("Usuario no encontrado"));
}
```

---

#### 8. ¿Cuál es la diferencia entre final, finally y finalize?

**Respuesta:**

Son tres palabras clave con propósitos **completamente diferentes**:

**1. final (modificador):**

Hace que algo sea **inmutable o no sobrescribible**.

**Usos:**

**a) Variable final:** No se puede reasignar.
```java
final int MAX = 100;
MAX = 200;  // ❌ Error de compilación

final List<String> lista = new ArrayList<>();
lista.add("Hola");  // ✅ OK (el contenido puede cambiar)
lista = new ArrayList<>();  // ❌ Error (no puedes reasignar la referencia)
```

**b) Método final:** No se puede sobrescribir en subclases.
```java
public class Padre {
    public final void metodoFinal() {
        // Implementación fija
    }
}

public class Hijo extends Padre {
    @Override
    public void metodoFinal() {  // ❌ Error de compilación
    }
}
```

**c) Clase final:** No se puede heredar.
```java
public final class String {  // Por eso String no se puede extender
}

public class MiString extends String {  // ❌ Error de compilación
}
```

**Best practice:** Usa `final` en variables que no deben cambiar y en parámetros de método para mayor claridad.

**2. finally (bloque try-catch):**

Bloque que **siempre se ejecuta**, haya o no excepción.

```java
try {
    int resultado = 10 / 0;  // Lanza ArithmeticException
    return resultado;
} catch (ArithmeticException e) {
    System.out.println("Error: " + e.getMessage());
    return -1;
} finally {
    System.out.println("Esto SIEMPRE se ejecuta");  // Se ejecuta incluso si hay return
}
```

**Casos de uso:**
- Cerrar recursos (archivos, conexiones BD) → Hoy se prefiere **try-with-resources**.
- Logging.
- Limpieza garantizada.

**Nota:** `finally` se ejecuta incluso si hay `return` en el `try` o `catch`. La única excepción es `System.exit()`.

**3. finalize() (método obsoleto ❌ Deprecated en Java 9):**

Método llamado por el **Garbage Collector** antes de destruir un objeto. **NO lo uses**.

```java
@Override
protected void finalize() throws Throwable {
    System.out.println("Objeto siendo destruido");
    super.finalize();
}
```

**Problemas de finalize():**
- No sabes **cuándo** se ejecutará (o si se ejecutará).
- Degrada el rendimiento del GC.
- Puede causar memory leaks si no se implementa bien.

**Alternativa moderna (Java 9+):** Usa **Cleaner** o implementa `AutoCloseable` con try-with-resources.

**Resumen:**

| Palabra | Tipo | Propósito |
|---------|------|-----------|
| **final** | Modificador | Inmutabilidad, no sobrescritura |
| **finally** | Bloque | Ejecución garantizada después de try-catch |
| **finalize()** | Método | ❌ Obsoleto, limpieza antes de GC (no usar) |

---

#### 9. ¿Qué son los Java Records y cuándo usarlos? (Java 14+)

**Respuesta:**

Los **Records** son un tipo especial de clase introducido en **Java 14** (standard en Java 16) para modelar **datos inmutables** de forma concisa. Actúan como "portadores de datos transparentes".

**Diferencias con una clase normal:**

- **Sintaxis reducida**: Boilerplate mínimo (no necesitas escribir constructor, getters, equals, hashCode, toString).
- **Inmutabilidad automática**: Todos los campos son `private final`.
- **Generación automática** de:
  - Constructor canónico con todos los campos.
  - Métodos `equals()`, `hashCode()`, `toString()`.
  - Métodos de acceso (sin prefijo `get`).
- **No tienen setters** (son inmutables).
- **No pueden heredar** de otras clases (pero pueden implementar interfaces).

**Ejemplo comparativo:**

```java
// ❌ Antes: Clase tradicional (verbosa)
public class Point {
    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    public int getX() { return x; }
    public int getY() { return y; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Point)) return false;
        Point point = (Point) o;
        return x == point.x && y == point.y;
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }
    
    @Override
    public String toString() {
        return "Point{x=" + x + ", y=" + y + "}";
    }
}

// ✅ Con Record (1 línea)
public record Point(int x, int y) {}
```

**Uso:**

```java
Point p1 = new Point(10, 20);
System.out.println(p1.x());          // 10 (nota: x(), no getX())
System.out.println(p1.y());          // 20
System.out.println(p1);              // Point[x=10, y=20] (toString automático)

Point p2 = new Point(10, 20);
System.out.println(p1.equals(p2));   // true (equals automático)
```

**¿Cuándo usarlos?**

✅ **Ideales para:**
- **DTOs (Data Transfer Objects)** en APIs REST.
- **Respuestas de API** (objetos JSON).
- **Value Objects** en DDD.
- **Claves de mapas** (porque equals/hashCode están implementados correctamente).
- Cualquier dato inmutable de solo lectura.

```java
// ✅ Perfecto para DTOs
public record UserDTO(Long id, String name, String email) {}

@GetMapping("/users/{id}")
public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
    User user = userService.findById(id);
    return ResponseEntity.ok(new UserDTO(user.getId(), user.getName(), user.getEmail()));
}
```

❌ **NO usarlos cuando:**
- Necesitas **herencia** (los records son `final` implícitamente).
- Necesitas **mutabilidad** (setters).
- Necesitas **JPA entities** con Hibernate (aunque técnicamente funcionan, no es recomendado por limitaciones de lazy loading).

**Personalizaciones permitidas:**

Puedes añadir métodos, constructores adicionales, validaciones:

```java
public record Range(int min, int max) {
    // Constructor compacto (validación)
    public Range {
        if (min > max) {
            throw new IllegalArgumentException("min debe ser <= max");
        }
    }
    
    // Métodos adicionales
    public boolean contains(int value) {
        return value >= min && value <= max;
    }
    
    // Constructor alternativo
    public Range(int max) {
        this(0, max);
    }
}
```

**Ventajas sobre Lombok @Data:**
- No necesitas dependencias externas.
- Parte del lenguaje (mejor soporte de IDEs y compilador).
- Semántica clara: "esto es un dato inmutable".

**Record vs clase tradicional vs Lombok:**

| Feature | Record | Clase normal | Lombok @Data |
|---------|--------|--------------|---------------|
| Sintaxis | ⭐⭐⭐ Concisa | ❌ Verbosa | ⭐⭐⭐ Concisa |
| Inmutabilidad | ✅ Por defecto | ⚠️ Manual | ⚠️ Opcional (@Value) |
| Dependencias | ✅ Ninguna | ✅ Ninguna | ❌ Lombok library |
| Herencia | ❌ No permite | ✅ Sí | ✅ Sí |
| JPA/Hibernate | ⚠️ Limitado | ✅ Full support | ✅ Full support |

**Best Practices:**
- ✅ Usa Records para **DTOs y Value Objects**.
- ✅ Usa clases normales para **entidades JPA**.
- ✅ Combina Records con **Pattern Matching** (Java 16+) para código más expresivo.

```java
// Pattern matching con Records (Java 16+)
public String formatShape(Shape shape) {
    return switch (shape) {
        case Circle(double radius) -> "Circle with radius " + radius;
        case Rectangle(double width, double height) -> "Rectangle " + width + "x" + height;
    };
}
```

---

#### 10. ¿Qué es el paso por valor en Java? ¿Java pasa objetos por referencia?

**Respuesta:**

**Java SIEMPRE pasa por VALOR**, pero esto se malinterpreta frecuentemente.

**Para primitivos:** Se copia el valor.
```java
void modificar(int x) {
    x = 100;  // Solo modifica la copia local
}

int num = 10;
modificar(num);
System.out.println(num);  // 10 (no cambió)
```

**Para objetos:** Se copia la **referencia** (dirección de memoria), no el objeto.
```java
void modificar(List<String> lista) {
    lista.add("nuevo");  // ✅ Modifica el objeto original (porque tenemos su dirección)
    lista = new ArrayList<>();  // ❌ Solo cambia la copia local de la referencia
}

List<String> miLista = new ArrayList<>();
modificar(miLista);
System.out.println(miLista.size());  // 1 (se añadió "nuevo")
```

**Conclusión:** Java pasa la **referencia por valor**. Puedes modificar el objeto al que apunta la referencia, pero no puedes hacer que la variable original apunte a un objeto diferente.

---

#### 11. ¿Qué es la inferencia de tipos (var) y cuándo usarla? (Java 10+)

**Respuesta:**

La palabra clave **`var`** permite al compilador **inferir automáticamente el tipo** de una variable local basándose en el valor asignado, introducida en **Java 10**.

**Sintaxis básica:**

```java
// Antes de Java 10
ArrayList<String> nombres = new ArrayList<String>();
Map<String, List<Integer>> mapa = new HashMap<String, List<Integer>>();

// Con var (Java 10+)
var nombres = new ArrayList<String>();  // Compilador infiere ArrayList<String>
var mapa = new HashMap<String, List<Integer>>();  // Infiere HashMap<String, List<Integer>>
```

**¿Cómo funciona?**

El compilador **determina el tipo en compile-time** mirando el lado derecho de la asignación. No es tipado dinámico (como JavaScript); sigue siendo **fuertemente tipado**.

```java
var numero = 10;           // Infiere int
var texto = "Hola";        // Infiere String
var lista = new ArrayList<User>();  // Infiere ArrayList<User>
var precio = 19.99;        // Infiere double

// El tipo queda fijado
numero = "texto";  // ❌ Error de compilación: incompatible types
```

**✅ Cuándo usar var (Mejora legibilidad):**

**1. Cuando el tipo es obvio por el lado derecho:**
```java
// ❌ Verboso y redundante
Map<String, List<Order>> ordersByCustomer = new HashMap<String, List<Order>>();

// ✅ Conciso y claro
var ordersByCustomer = new HashMap<String, List<Order>>();
```

**2. Con constructores o factory methods claros:**
```java
var user = new User("Juan", "juan@mail.com");
var connection = DriverManager.getConnection(url);
var builder = new StringBuilder();
```

**3. En loops (especialmente con tipos complejos):**
```java
// ❌ Verboso
for (Map.Entry<String, List<Order>> entry : orderMap.entrySet()) {
    // ...
}

// ✅ Más limpio
for (var entry : orderMap.entrySet()) {
    String customer = entry.getKey();
    List<Order> orders = entry.getValue();
}
```

**4. Con try-with-resources:**
```java
// ✅ Reduce boilerplate
try (var reader = new BufferedReader(new FileReader("file.txt"))) {
    var line = reader.readLine();
}
```

**❌ Cuándo NO usar var (Oscurece el código):**

**1. Cuando el tipo no es obvio:**
```java
// ❌ ¿Qué tipo retorna?
var result = processData();
var value = calculate();

// ✅ Explícito
User result = processData();
BigDecimal value = calculate();
```

**2. Con literales numéricos ambiguos:**
```java
var x = 10;      // ❌ ¿int? ¿long? No está claro
var price = 100; // ❌ Infiere int, pero querías BigDecimal?

// ✅ Explícito
long x = 10L;
BigDecimal price = new BigDecimal("100");
```

**3. Inicialización con null:**
```java
var user = null;  // ❌ Error de compilación: cannot infer type
```

**4. Variables sin inicialización:**
```java
var name;  // ❌ Error: cannot use 'var' on variable without initializer
name = "Juan";
```

**5. En campos de clase o parámetros de métodos:**
```java
public class User {
    var name = "Juan";  // ❌ Error: 'var' is not allowed here
}

public void process(var data) {  // ❌ Error: 'var' is not allowed here
}
```

**Limitaciones importantes:**

```java
// ❌ No funciona con:
var x;                    // Sin inicialización
var y = null;             // null literal
var z = {1, 2, 3};       // Array initializer
var lambda = x -> x * 2;  // Lambda sin contexto

// ✅ Soluciones:
int x = 0;
String y = null;
int[] z = {1, 2, 3};
Function<Integer, Integer> lambda = x -> x * 2;
```

**Ejemplo real en Spring Boot:**

```java
@GetMapping("/users")
public ResponseEntity<Page<UserDTO>> getUsers(Pageable pageable) {
    // ✅ Uso apropiado de var
    var userPage = userRepository.findAll(pageable);
    var dtoPage = userPage.map(this::toDTO);
    return ResponseEntity.ok(dtoPage);
}

// vs sin var (más verboso)
public ResponseEntity<Page<UserDTO>> getUsers(Pageable pageable) {
    Page<User> userPage = userRepository.findAll(pageable);
    Page<UserDTO> dtoPage = userPage.map(this::toDTO);
    return ResponseEntity.ok(dtoPage);
}
```

**Best Practices:**

- ✅ Usa `var` cuando **reduce ruido visual** sin sacrificar claridad.
- ✅ Usa `var` con **nombres de variables descriptivos** (compensa la falta de tipo explícito).
- ✅ En equipos, establece **guías de estilo** sobre el uso de `var`.
- ❌ No uses `var` para ahorrar tipeo; úsalo para **mejorar legibilidad**.
- ❌ Si dudas si el tipo es claro, **no uses var**.

**Impacto en rendimiento:**
- ⚡ **Ninguno**. `var` es **solo azúcar sintáctico** en compile-time. El bytecode generado es idéntico.

**Compatibilidad:**
- Requiere **Java 10+** (2018).
- No afecta bytecode, pero el código fuente no compilará en Java 9 o anterior.

**Resumen:**

| Situación | Usar var | Ejemplo |
|-----------|----------|---------|
| Tipo obvio en lado derecho | ✅ | `var user = new User()` |
| Loop con tipos complejos | ✅ | `for (var entry : map.entrySet())` |
| Tipo inferido no claro | ❌ | `var x = calculate()` |
| Literales ambiguos | ❌ | `var price = 100` |
| Campos de clase | ❌ | `class User { var name; }` |
| Parámetros de métodos | ❌ | `void method(var x)` |

---

#### 12. ¿Qué es el Garbage Collector? ¿Cómo funciona?

**Respuesta:**

El **Garbage Collector (GC)** es un proceso automático que libera memoria al eliminar objetos que ya no tienen referencias (son inalcanzables).

**¿Cómo saber si un objeto es elegible para GC?**
Un objeto puede ser recolectado cuando:
- No hay referencias activas apuntando a él.
- Todas las referencias son nulas.
- Las referencias solo existen dentro de un bloque que ya terminó.

**Generaciones de memoria (Generational GC):**

1. **Young Generation:**
   - **Eden Space**: Donde nacen los objetos nuevos.
   - **Survivor Spaces (S0, S1)**: Objetos que sobreviven a un GC menor.
   - **Minor GC**: Limpieza rápida y frecuente.

2. **Old Generation (Tenured):**
   - Objetos que han sobrevivido varios Minor GCs.
   - **Major GC / Full GC**: Limpieza más costosa y menos frecuente.

3. **Metaspace (Java 8+):**
   - Reemplaza PermGen.
   - Guarda metadata de clases.

**Tipos de GC:**
- **Serial GC**: Un solo hilo (apps pequeñas).
- **Parallel GC**: Múltiples hilos (throughput).
- **G1 GC**: Recolector por regiones (balance latencia/throughput). Default en Java 9+.
- **ZGC / Shenandoah**: Baja latencia (<10ms pauses).

**¿Puedes forzar el GC?**
`System.gc()` sugiere al GC que corra, pero no garantiza nada. **No recomendado** en producción.

---

#### 13. ¿Qué son las clases Wrapper? ¿Para qué sirven?

**Respuesta:**

Las **clases Wrapper** son clases que "envuelven" tipos primitivos en objetos. Cada primitivo tiene su Wrapper:

| Primitivo | Wrapper |
|-----------|---------|
| byte | Byte |
| short | Short |
| int | Integer |
| long | Long |
| float | Float |
| double | Double |
| char | Character |
| boolean | Boolean |

**¿Por qué existen?**

1. **Colecciones solo aceptan objetos**: `List<int>` ❌ → `List<Integer>` ✅
2. **Métodos de utilidad**: `Integer.parseInt()`, `Double.valueOf()`, `Integer.toBinaryString()`
3. **Necesidad de null**: Los primitivos no pueden ser `null`, los Wrappers sí.
4. **Constantes útiles**: `Integer.MAX_VALUE`, `Double.NaN`, `Integer.MIN_VALUE`

**Caché de Wrappers:**
Java cachea algunos valores para optimizar memoria:
- `Integer`, `Short`, `Byte`, `Character`: -128 a 127
- `Long`: -128 a 127
- `Boolean`: true y false

```java
Integer a = 100;
Integer b = 100;
System.out.println(a == b);  // true (cacheado)

Integer c = 200;
Integer d = 200;
System.out.println(c == d);  // false (no cacheado, diferentes objetos)
```

---

### 🔹 Programación Orientada a Objetos (OOP)

#### 14. ¿Cuáles son los 4 pilares de la Programación Orientada a Objetos?

**Respuesta:**

Los **4 pilares fundamentales de la POO** son:

**1. Encapsulación (Encapsulation)**

Ocultar los detalles internos de implementación y exponer solo lo necesario mediante modificadores de acceso.

```java
public class CuentaBancaria {
    private double saldo;  // Oculto, no accesible directamente
    
    public double getSaldo() {
        return saldo;
    }
    
    public void depositar(double monto) {
        if (monto > 0) {  // Validación controlada
            this.saldo += monto;
        }
    }
    
    public boolean retirar(double monto) {
        if (monto > 0 && saldo >= monto) {
            saldo -= monto;
            return true;
        }
        return false;
    }
}
```

**Beneficios:**
- Control sobre cómo se accede/modifica el estado interno
- Validación de datos
- Facilita el mantenimiento (cambiar implementación sin afectar clientes)

---

**2. Herencia (Inheritance)**

Mecanismo para crear nuevas clases basadas en clases existentes, heredando atributos y comportamientos.

```java
public class Animal {
    protected String nombre;
    
    public void comer() {
        System.out.println(nombre + " está comiendo");
    }
}

public class Perro extends Animal {
    public void ladrar() {
        System.out.println(nombre + " está ladrando");
    }
    
    @Override
    public void comer() {
        System.out.println(nombre + " está comiendo croquetas");
    }
}
```

**Beneficios:**
- Reutilización de código
- Jerarquías lógicas (relación "es-un")
- Polimorfismo

---

**3. Polimorfismo (Polymorphism)**

Capacidad de un objeto de tomar múltiples formas. Permite tratar objetos de diferentes clases de manera uniforme.

**Polimorfismo en tiempo de compilación (Sobrecarga - Overloading):**
```java
public class Calculadora {
    public int sumar(int a, int b) {
        return a + b;
    }
    
    public double sumar(double a, double b) {
        return a + b;
    }
    
    public int sumar(int a, int b, int c) {
        return a + b + c;
    }
}
```

**Polimorfismo en tiempo de ejecución (Sobrescritura - Overriding):**
```java
public class Main {
    public static void main(String[] args) {
        Animal animal = new Perro();  // Polimorfismo
        animal.comer();  // Llama al método de Perro (no Animal)
        
        List<Animal> animales = Arrays.asList(
            new Perro(),
            new Gato(),
            new Pajaro()
        );
        
        for (Animal a : animales) {
            a.comer();  // Comportamiento diferente según el tipo real
        }
    }
}
```

**Beneficios:**
- Flexibilidad en el diseño
- Código más genérico y reutilizable
- Extensibilidad (agregar nuevos tipos sin modificar código existente)

---

**4. Abstracción (Abstraction)**

Ocultar detalles complejos y mostrar solo la funcionalidad esencial. Se implementa con clases abstractas e interfaces.

```java
// Interfaz: contrato puro (qué hace)
public interface Vehiculo {
    void arrancar();
    void detener();
    int getVelocidadMaxima();
}

// Clase abstracta: implementación parcial
public abstract class VehiculoTerrestre implements Vehiculo {
    protected int ruedas;
    
    public abstract void cambiarMarcha(int marcha);
    
    @Override
    public void detener() {
        System.out.println("Frenando vehículo terrestre");
    }
}

// Implementación concreta
public class Automovil extends VehiculoTerrestre {
    @Override
    public void arrancar() {
        System.out.println("Arrancando motor del automóvil");
    }
    
    @Override
    public void cambiarMarcha(int marcha) {
        System.out.println("Cambiando a marcha " + marcha);
    }
    
    @Override
    public int getVelocidadMaxima() {
        return 200;
    }
}
```

**Beneficios:**
- Reduce complejidad
- Permite diseñar en términos de "qué hace" no "cómo lo hace"
- Facilita cambios de implementación sin afectar clientes

---

**Resumen de los 4 pilares:**

| Pilar | Propósito | Implementación |
|-------|-----------|----------------|
| **Encapsulación** | Ocultar detalles internos | `private`, `protected`, getters/setters |
| **Herencia** | Reutilizar código | `extends` |
| **Polimorfismo** | Múltiples formas | Sobrecarga, Sobrescritura, Interfaces |
| **Abstracción** | Simplificar complejidad | `abstract`, `interface` |

---

#### 15. ¿Cuál es la diferencia entre Clase Abstracta e Interface?

**Respuesta:**

Ambas definen contratos que otras clases deben cumplir, pero tienen diferencias importantes:

**Clase Abstracta:**
- Puede tener **métodos abstractos** (sin implementación) y **concretos** (con implementación)
- Puede tener **variables de instancia** (campos)
- Puede tener **constructores**
- Soporta **modificadores de acceso** en métodos
- Una clase solo puede **extender una** clase abstracta (herencia simple)
- Usa `extends`

**Interface (Interfaz):**
- Todos los métodos son **públicos y abstractos** por defecto (antes de Java 8)
- Desde Java 8: puede tener **default** y **static** methods con implementación
- Desde Java 9: puede tener **private** methods
- Todos los campos son `public static final` (constantes)
- **No tiene constructores**
- Una clase puede **implementar múltiples** interfaces
- Usa `implements`

**Comparación:**

```java
// ========== CLASE ABSTRACTA ==========
public abstract class Animal {
    // ✅ Variables de instancia
    protected String nombre;
    private int edad;
    
    // ✅ Constructor
    public Animal(String nombre) {
        this.nombre = nombre;
    }
    
    // ✅ Método concreto (con implementación)
    public void dormir() {
        System.out.println(nombre + " está durmiendo");
    }
    
    // ✅ Método abstracto (sin implementación)
    public abstract void hacerSonido();
    
    // ✅ Modificadores de acceso variados
    protected void respirar() {
        System.out.println("Respirando...");
    }
}

// Uso
public class Perro extends Animal {
    public Perro(String nombre) {
        super(nombre);
    }
    
    @Override
    public void hacerSonido() {
        System.out.println("Guau guau");
    }
}

// ========== INTERFACE ==========
public interface Volador {
    // ✅ Constantes (public static final implícito)
    int ALTURA_MAXIMA = 10000;
    
    // ✅ Método abstracto
    void volar();
    
    // ✅ Default method (Java 8+)
    default void aterrizar() {
        System.out.println("Aterrizando...");
    }
    
    // ✅ Static method (Java 8+)
    static void mostrarInfo() {
        System.out.println("Interface para objetos voladores");
    }
    
    // ✅ Private method (Java 9+)
    private void validarAltura(int altura) {
        if (altura > ALTURA_MAXIMA) {
            throw new IllegalArgumentException("Altura muy alta");
        }
    }
}

public interface Nadador {
    void nadar();
}

// ✅ Implementar múltiples interfaces
public class Pato extends Animal implements Volador, Nadador {
    public Pato(String nombre) {
        super(nombre);
    }
    
    @Override
    public void hacerSonido() {
        System.out.println("Cuac cuac");
    }
    
    @Override
    public void volar() {
        System.out.println("Volando bajo");
    }
    
    @Override
    public void nadar() {
        System.out.println("Nadando en el lago");
    }
}
```

**¿Cuándo usar cada una?**

| Usa Clase Abstracta cuando: | Usa Interface cuando: |
|------------------------------|----------------------|
| Compartes código común entre clases relacionadas | Defines un contrato/capacidad sin relación jerárquica |
| Necesitas campos no estáticos o no finales | Necesitas herencia múltiple de tipos |
| Necesitas modificadores de acceso distintos de public | Defines comportamiento que puede ser implementado por cualquier clase |
| Clases herederas están fuertemente relacionadas | Quieres definir "capacidades" (Serializable, Comparable, Runnable) |

**Ejemplos del mundo real:**

```java
// ✅ Clase abstracta: jerarquía fuerte, código compartido
abstract class HttpServlet {
    protected void service(HttpRequest req, HttpResponse res) {
        // Lógica común de manejo de requests
    }
    protected abstract void doGet(HttpRequest req, HttpResponse res);
    protected abstract void doPost(HttpRequest req, HttpResponse res);
}

// ✅ Interface: contrato/capacidad, sin relación jerárquica
interface Comparable<T> {
    int compareTo(T o);
}

interface Serializable {  // Marker interface (vacía, solo marca capacidad)
}

// Una clase puede ser comparable Y serializable
public class Usuario implements Comparable<Usuario>, Serializable {
    @Override
    public int compareTo(Usuario otro) {
        return this.nombre.compareTo(otro.nombre);
    }
}
```

**Tabla resumen:**

| Característica | Clase Abstracta | Interface |
|----------------|-----------------|-----------|
| **Herencia** | Simple (extends una) | Múltiple (implements varias) |
| **Métodos** | Abstractos y concretos | Abstractos, default, static, private |
| **Campos** | Cualquier tipo | Solo constantes (public static final) |
| **Constructores** | Sí | No |
| **Modificadores** | public, protected, private | Solo public (métodos abstractos) |
| **Cuándo usar** | Relación "es-un" + código compartido | Contrato/capacidad "puede-hacer" |

---

#### 16. ¿Java soporta herencia múltiple? ¿Cómo se soluciona?

**Respuesta:**

**Java NO soporta herencia múltiple de clases**, pero **SÍ soporta herencia múltiple de interfaces**.

**¿Por qué no permite herencia múltiple de clases?**

Para evitar el **problema del diamante (Diamond Problem)**: ambigüedad cuando dos clases padres tienen el mismo método.

```java
// ❌ ESTO NO COMPILA EN JAVA
class A {
    public void metodo() {
        System.out.println("Método de A");
    }
}

class B {
    public void metodo() {
        System.out.println("Método de B");
    }
}

// ❌ ERROR: Java no permite heredar de múltiples clases
class C extends A, B {  // ❌ Error de compilación
    // ¿Cuál metodo() se hereda? ¿A o B? → Ambigüedad
}
```

---

**Solución 1: Herencia múltiple de INTERFACES** ✅

```java
interface Volador {
    void volar();
}

interface Nadador {
    void nadar();
}

// ✅ OK: Implementar múltiples interfaces
class Pato implements Volador, Nadador {
    @Override
    public void volar() {
        System.out.println("Pato volando");
    }
    
    @Override
    public void nadar() {
        System.out.println("Pato nadando");
    }
}
```

---

**Problema del diamante con interfaces (Java 8+)**

Desde Java 8, las interfaces pueden tener **default methods**, lo que reintroduce el problema del diamante:

```java
interface A {
    default void metodo() {
        System.out.println("Método de A");
    }
}

interface B {
    default void metodo() {
        System.out.println("Método de B");
    }
}

// ❌ ERROR: Ambigüedad con default methods
class C implements A, B {  // Error: C inherits unrelated defaults for metodo() from A and B
}
```

**Solución: Sobrescribir explícitamente**

```java
class C implements A, B {
    @Override
    public void metodo() {
        // Opción 1: Elegir explícitamente una implementación
        A.super.metodo();  // Llama al default de A
        
        // Opción 2: Llamar a ambas
        // A.super.metodo();
        // B.super.metodo();
        
        // Opción 3: Implementación propia
        // System.out.println("Método de C");
    }
}
```

---

**Solución 2: COMPOSICIÓN sobre herencia** ✅

En lugar de heredar, **contener** instancias de las clases necesarias:

```java
class Motor {
    public void arrancar() {
        System.out.println("Motor arrancando");
    }
}

class SistemaElectrico {
    public void encenderLuces() {
        System.out.println("Luces encendidas");
    }
}

// ✅ Composición: "tiene-un" en lugar de "es-un"
class Automovil {
    private Motor motor;
    private SistemaElectrico sistemaElectrico;
    
    public Automovil() {
        this.motor = new Motor();
        this.sistemaElectrico = new SistemaElectrico();
    }
    
    public void arrancar() {
        motor.arrancar();
        sistemaElectrico.encenderLuces();
    }
}
```

**Ventajas de la composición:**
- Más flexible (puedes cambiar componentes en runtime)
- Menos acoplamiento
- Evita jerarquías complejas
- Principio: **"Favor composition over inheritance"**

---

**Resumen:**

| Tipo de herencia | Java lo soporta | Ejemplo |
|------------------|-----------------|---------|
| **Herencia simple de clases** | ✅ Sí | `class B extends A` |
| **Herencia múltiple de clases** | ❌ No | `class C extends A, B` ❌ |
| **Herencia múltiple de interfaces** | ✅ Sí | `class C implements A, B` ✅ |
| **Composición** | ✅ Sí (recomendado) | `class C { A a; B b; }` ✅ |

---

#### 17. ¿Cuál es la diferencia entre Sobrecarga (Overloading) y Sobrescritura (Overriding)?

**Respuesta:**

**Sobrecarga (Overloading)** y **Sobrescritura (Overriding)** son dos formas de polimorfismo, pero funcionan de manera diferente:

---

**SOBRECARGA (Method Overloading)**

Múltiples métodos con el **mismo nombre** pero **diferentes parámetros** en la **misma clase**.

**Características:**
- Mismo nombre, diferente firma (parámetros)
- Puede cambiar el tipo de retorno
- Puede cambiar modificadores de acceso
- Puede lanzar diferentes excepciones
- Ocurre en **tiempo de compilación** (polimorfismo estático)
- En la **misma clase** o en clase padre/hija

```java
public class Calculadora {
    // ✅ Sobrecarga: mismo nombre, diferentes parámetros
    
    public int sumar(int a, int b) {
        return a + b;
    }
    
    public double sumar(double a, double b) {
        return a + b;
    }
    
    public int sumar(int a, int b, int c) {
        return a + b + c;
    }
    
    public String sumar(String a, String b) {
        return a + b;
    }
}

// Uso
Calculadora calc = new Calculadora();
calc.sumar(5, 3);           // Llama a sumar(int, int)
calc.sumar(5.5, 3.2);       // Llama a sumar(double, double)
calc.sumar(1, 2, 3);        // Llama a sumar(int, int, int)
calc.sumar("Hola", "Mundo"); // Llama a sumar(String, String)
```

**Reglas de la sobrecarga:**
1. ✅ Debe cambiar el **número** o **tipo** de parámetros
2. ✅ Puede cambiar el tipo de retorno
3. ✅ Puede cambiar modificadores de acceso
4. ❌ NO puede cambiar solo el tipo de retorno

```java
// ❌ ERROR: No puede diferenciar solo por tipo de retorno
public int calcular(int a) { return a * 2; }
public double calcular(int a) { return a * 2.0; }  // ❌ Error de compilación
```

---

**SOBRESCRITURA (Method Overriding)**

Una subclase proporciona una **implementación específica** de un método que ya está definido en su superclase.

**Características:**
- Mismo nombre, **misma firma** (parámetros)
- **Mismo tipo de retorno** o subtipo (covariante desde Java 5)
- **No puede reducir** la visibilidad (public → protected ❌)
- **No puede lanzar** excepciones checked más amplias
- Ocurre en **tiempo de ejecución** (polimorfismo dinámico)
- Entre clase **padre e hija** (herencia)
- Requiere `@Override` (buena práctica)

```java
public class Animal {
    public void hacerSonido() {
        System.out.println("Animal hace un sonido");
    }
    
    public Animal reproducir() {
        return new Animal();
    }
}

public class Perro extends Animal {
    // ✅ Sobrescritura: misma firma que el padre
    @Override
    public void hacerSonido() {
        System.out.println("Guau guau");
    }
    
    // ✅ Covariant return type (puede retornar un subtipo)
    @Override
    public Perro reproducir() {
        return new Perro();
    }
}

// Uso (polimorfismo en acción)
Animal animal = new Perro();
animal.hacerSonido();  // Imprime "Guau guau" (usa la versión del Perro)
```

**Reglas de la sobrescritura:**
1. ✅ Debe tener la **misma firma** (nombre + parámetros)
2. ✅ Mismo tipo de retorno o subtipo (covariante)
3. ✅ No puede ser más restrictivo en acceso
4. ✅ No puede lanzar excepciones checked más amplias
5. ❌ NO puede sobrescribir métodos `final`, `static` o `private`

```java
public class Padre {
    public void metodo() throws IOException { }
    public final void metodoFinal() { }
    private void metodoPrivado() { }
    public static void metodoEstatico() { }
}

public class Hijo extends Padre {
    // ✅ OK: Excepción más específica
    @Override
    public void metodo() throws FileNotFoundException { }
    
    // ❌ ERROR: No puede sobrescribir método final
    @Override
    public void metodoFinal() { }  // ❌ Error
    
    // ✅ OK: No es sobrescritura (método nuevo, el padre no lo expone)
    private void metodoPrivado() { }
    
    // ⚠️ NO es sobrescritura, es HIDING (ocultar método estático)
    public static void metodoEstatico() { }
}
```

---

**Comparación:**

| Característica | Sobrecarga (Overloading) | Sobrescritura (Overriding) |
|----------------|--------------------------|----------------------------|
| **Objetivo** | Múltiples versiones de un método | Redefinir comportamiento heredado |
| **Parámetros** | Deben ser diferentes | Deben ser iguales |
| **Tipo retorno** | Puede cambiar | Debe ser igual o subtipo |
| **Acceso** | Puede cambiar | No puede ser más restrictivo |
| **Contexto** | Misma clase o herencia | Solo con herencia |
| **Binding** | Compile-time (estático) | Runtime (dinámico) |
| **@Override** | No aplica | Recomendado |
| **Métodos** | Cualquiera | No final, static, private |

---

**Ejemplo completo:**

```java
public class Ejemplo {
    // ========== SOBRECARGA ==========
    public void procesar(int n) {
        System.out.println("Procesando int: " + n);
    }
    
    public void procesar(String s) {
        System.out.println("Procesando String: " + s);
    }
    
    public void procesar(int a, int b) {
        System.out.println("Procesando dos ints: " + a + ", " + b);
    }
}

public class SubEjemplo extends Ejemplo {
    // ========== SOBRESCRITURA ==========
    @Override
    public void procesar(int n) {
        System.out.println("SubEjemplo procesando int: " + n);
    }
    
    // ========== SOBRECARGA (nueva versión) ==========
    public void procesar(double d) {
        System.out.println("Procesando double: " + d);
    }
}

// Uso
Ejemplo ej = new SubEjemplo();
ej.procesar(10);        // Sobrescritura: "SubEjemplo procesando int: 10"
ej.procesar("hola");    // Sobrecarga heredada: "Procesando String: hola"
ej.procesar(5, 7);      // Sobrecarga heredada: "Procesando dos ints: 5, 7"

SubEjemplo sub = new SubEjemplo();
sub.procesar(3.14);     // Sobrecarga nueva: "Procesando double: 3.14"
```

---

#### 18. ¿Cuáles son los modificadores de acceso en Java?

**Respuesta:**

Los **modificadores de acceso** controlan la visibilidad de clases, métodos y variables. Java tiene **4 niveles de acceso**:

| Modificador | Misma clase | Mismo package | Subclase (otro package) | Otros packages |
|-------------|-------------|---------------|-------------------------|----------------|
| **private** | ✅ | ❌ | ❌ | ❌ |
| **default** (sin modificador) | ✅ | ✅ | ❌ | ❌ |
| **protected** | ✅ | ✅ | ✅ | ❌ |
| **public** | ✅ | ✅ | ✅ | ✅ |

---

**1. private - Más restrictivo**

Solo accesible dentro de la **misma clase**.

```java
public class Usuario {
    private String password;  // Solo accesible en Usuario
    
    private void validarPassword() {
        // Método privado, solo se usa internamente
    }
    
    public void cambiarPassword(String nuevaPassword) {
        validarPassword();  // ✅ OK: misma clase
        this.password = nuevaPassword;
    }
}

public class Main {
    public static void main(String[] args) {
        Usuario user = new Usuario();
        user.password = "123";  // ❌ ERROR: password is private
        user.validarPassword(); // ❌ ERROR: method is private
    }
}
```

**Cuándo usar:**
- Variables de instancia (encapsulación)
- Métodos auxiliares internos
- Implementación que no debe exponerse

---

**2. default (package-private) - Sin modificador**

Accesible dentro del **mismo package**.

```java
// archivo: com/example/Usuario.java
package com.example;

class UsuarioInterno {  // Sin modificador = package-private
    String nombre;  // Sin modificador = package-private
    
    void metodoInterno() {
        // Accesible en todo el package
    }
}

// archivo: com/example/Servicio.java
package com.example;

public class Servicio {
    public void procesar() {
        UsuarioInterno user = new UsuarioInterno();  // ✅ OK: mismo package
        user.nombre = "Juan";  // ✅ OK
        user.metodoInterno();  // ✅ OK
    }
}

// archivo: com.otro.Main.java
package com.otro;

import com.example.UsuarioInterno;

public class Main {
    public void test() {
        UsuarioInterno user = new UsuarioInterno();  // ❌ ERROR: otro package
    }
}
```

**Cuándo usar:**
- Clases auxiliares solo usadas dentro del package
- API interna del package que no quieres exponer

---

**3. protected**

Accesible en:
- **Mismo package**
- **Subclases** (incluso en otros packages)

```java
// archivo: com/example/Animal.java
package com.example;

public class Animal {
    protected String especie;
    
    protected void respirar() {
        System.out.println("Respirando...");
    }
}

// archivo: com/example/Perro.java
package com.example;

public class Perro extends Animal {
    public void mostrar() {
        especie = "Canino";  // ✅ OK: subclase mismo package
        respirar();  // ✅ OK
    }
}

// archivo: com/otro/Gato.java
package com.otro;

import com.example.Animal;

public class Gato extends Animal {
    public void mostrar() {
        especie = "Felino";  // ✅ OK: subclase otro package
        respirar();  // ✅ OK
    }
}

// archivo: com/otro/Main.java
package com.otro;

import com.example.Animal;

public class Main {
    public void test() {
        Animal animal = new Animal();
        animal.especie = "X";  // ❌ ERROR: no es subclase
        animal.respirar();  // ❌ ERROR
        
        Gato gato = new Gato();
        gato.especie = "Y";  // ❌ ERROR: protected solo accesible en la subclase misma
    }
}
```

**Importante:** El acceso `protected` desde otra package **solo funciona dentro del código de la subclase**, no desde fuera.

**Cuándo usar:**
- Métodos/campos que las subclases necesitan modificar
- API para extensión (herencia)

---

**4. public - Menos restrictivo**

Accesible desde **cualquier lugar**.

```java
public class ServicioPublico {
    public String mensaje;
    
    public void metodoPublico() {
        // Accesible desde cualquier clase
    }
}

// Cualquier clase puede acceder
ServicioPublico servicio = new ServicioPublico();
servicio.mensaje = "Hola";
servicio.metodoPublico();
```

**Cuándo usar:**
- API pública de tu aplicación/librería
- Métodos que deben ser accesibles globalmente
- DTOs, entidades que se comparten

---

**Reglas especiales para clases:**

```java
// ✅ Clase pública (debe estar en archivo Usuario.java)
public class Usuario {
}

// ✅ Clase package-private (puede estar en cualquier archivo)
class UsuarioInterno {
}

// ❌ NO existen clases private o protected en nivel superior
private class Invalida {  // ❌ Error
}

// ✅ Clases internas SÍ pueden ser private o protected
public class Externa {
    private class Interna {  // ✅ OK
    }
    
    protected class OtraInterna {  // ✅ OK
    }
}
```

---

**Best Practices:**

1. **Principio de mínimo privilegio**: Usa el modificador más restrictivo posible
2. **Encapsulación**: Variables siempre `private`, acceso via getters/setters
3. **API clara**: Solo expón (`public`) lo que realmente necesita ser usado externamente
4. **Herencia**: Usa `protected` para métodos que subclases deben poder sobrescribir

```java
// ✅ BUEN diseño
public class CuentaBancaria {
    private double saldo;  // Privado, no se puede modificar directamente
    
    public double getSaldo() {
        return saldo;
    }
    
    public void depositar(double monto) {
        if (monto > 0) {
            saldo += monto;
        }
    }
    
    protected void validarTransaccion(double monto) {
        // Para que subclases puedan personalizarlo
    }
}
```

---

#### 19. ¿Cuál es la diferencia entre miembros static y no-static?

**Respuesta:**

La diferencia clave está en **a qué pertenecen** y **cómo se accede a ellos**.

**Miembros no-static (de instancia):**
- Pertenecen a una **instancia específica** del objeto
- Cada objeto tiene su propia copia
- Requieren crear un objeto para acceder
- Pueden acceder a miembros static y no-static

**Miembros static (de clase):**
- Pertenecen a la **clase**, no a instancias individuales
- Compartidos entre todas las instancias
- Se pueden acceder sin crear un objeto
- Solo pueden acceder directamente a otros miembros static

---

**Variables:**

```java
public class Contador {
    // Variable NO-STATIC (de instancia)
    private int contadorInstancia = 0;
    
    // Variable STATIC (de clase)
    private static int contadorGlobal = 0;
    
    public Contador() {
        contadorInstancia++;  // Cada objeto tiene su propio contador
        contadorGlobal++;     // Compartido por todos los objetos
    }
    
    public void mostrarContadores() {
        System.out.println("Instancia: " + contadorInstancia);
        System.out.println("Global: " + contadorGlobal);
    }
}

// Uso
Contador c1 = new Contador();
c1.mostrarContadores();  // Instancia: 1, Global: 1

Contador c2 = new Contador();
c2.mostrarContadores();  // Instancia: 1, Global: 2

Contador c3 = new Contador();
c3.mostrarContadores();  // Instancia: 1, Global: 3
```

---

**Métodos:**

```java
public class Utilidades {
    // Método NO-STATIC (necesita instancia)
    public String saludar(String nombre) {
        return "Hola, " + nombre;
    }
    
    // Método STATIC (no necesita instancia)
    public static int sumar(int a, int b) {
        return a + b;
    }
    
    // Método STATIC
    public static void main(String[] args) {
        // ✅ Método static: acceso directo (sin crear objeto)
        int resultado = sumar(5, 3);
        int resultado2 = Utilidades.sumar(10, 20);
        
        // ✅ Método no-static: requiere instancia
        Utilidades util = new Utilidades();
        String saludo = util.saludar("Juan");
        
        // ❌ ERROR: No puedes llamar método de instancia desde contexto static
        // String saludo2 = saludar("Pedro");  // ❌ Error
    }
}
```

---

**Restricciones de métodos static:**

```java
public class Ejemplo {
    private int valorInstancia = 10;
    private static int valorEstatico = 20;
    
    // Método NO-STATIC
    public void metodoInstancia() {
        // ✅ Puede acceder a miembros de instancia
        System.out.println(valorInstancia);
        
        // ✅ Puede acceder a miembros static
        System.out.println(valorEstatico);
        
        // ✅ Puede llamar métodos static
        metodoEstatico();
        
        // ✅ Puede usar 'this'
        this.otroMetodoInstancia();
    }
    
    public void otroMetodoInstancia() {
    }
    
    // Método STATIC
    public static void metodoEstatico() {
        // ❌ NO puede acceder a miembros de instancia
        // System.out.println(valorInstancia);  // ❌ Error
        
        // ✅ Puede acceder a miembros static
        System.out.println(valorEstatico);
        
        // ❌ NO puede llamar métodos de instancia directamente
        // metodoInstancia();  // ❌ Error
        
        // ✅ Puede llamar métodos de instancia si crea un objeto
        Ejemplo obj = new Ejemplo();
        obj.metodoInstancia();  // ✅ OK
        
        // ❌ NO puede usar 'this' ni 'super'
        // this.otroMetodoEstatico();  // ❌ Error
    }
    
    public static void otroMetodoEstatico() {
    }
}
```

---

**Casos de uso comunes:**

**1. Constantes (static final):**
```java
public class Matematica {
    public static final double PI = 3.14159;
    public static final int MAX_VALUE = Integer.MAX_VALUE;
}

// Uso
double area = Matematica.PI * radio * radio;
```

**2. Métodos utilitarios:**
```java
public class StringUtils {
    public static boolean isEmpty(String str) {
        return str == null || str.isEmpty();
    }
    
    public static String capitalizar(String str) {
        if (isEmpty(str)) return str;
        return str.substring(0, 1).toUpperCase() + str.substring(1);
    }
}

// Uso
if (StringUtils.isEmpty(nombre)) {
    // ...
}
```

**3. Singleton (instancia única compartida):**
```java
public class Configuracion {
    private static Configuracion instancia;
    
    private Configuracion() {
        // Constructor privado
    }
    
    public static Configuracion obtenerInstancia() {
        if (instancia == null) {
            instancia = new Configuracion();
        }
        return instancia;
    }
}

// Uso
Configuracion config = Configuracion.obtenerInstancia();
```

**4. Factory methods:**
```java
public class Usuario {
    private String nombre;
    private String email;
    
    private Usuario(String nombre, String email) {
        this.nombre = nombre;
        this.email = email;
    }
    
    // Factory method static
    public static Usuario crear(String nombre, String email) {
        // Validación, lógica adicional
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Email inválido");
        }
        return new Usuario(nombre, email);
    }
    
    public static Usuario crearDesdeJson(String json) {
        // Parsing JSON
        return new Usuario("nombre", "email");
    }
}

// Uso
Usuario user = Usuario.crear("Juan", "juan@mail.com");
```

---

**Bloques de inicialización:**

```java
public class Ejemplo {
    private int valorInstancia;
    private static int valorEstatico;
    
    // Bloque de inicialización STATIC (se ejecuta una vez al cargar la clase)
    static {
        System.out.println("Inicialización static");
        valorEstatico = 100;
    }
    
    // Bloque de inicialización de INSTANCIA (se ejecuta cada vez que se crea un objeto)
    {
        System.out.println("Inicialización de instancia");
        valorInstancia = 50;
    }
    
    public Ejemplo() {
        System.out.println("Constructor");
    }
    
    public static void main(String[] args) {
        // Salida:
        // Inicialización static  (una sola vez)
        // Inicialización de instancia
        // Constructor
        new Ejemplo();
        
        // Salida:
        // Inicialización de instancia
        // Constructor
        new Ejemplo();
    }
}
```

---

**Resumen:**

| Característica | Static | No-static (Instancia) |
|----------------|--------|----------------------|
| **Pertenece a** | La clase | Cada objeto |
| **Memoria** | Una copia compartida | Una copia por objeto |
| **Acceso** | `NombreClase.miembro` o `objeto.miembro` | Solo `objeto.miembro` |
| **Contexto** | No puede acceder a miembros de instancia | Puede acceder a todo |
| **Uso de `this`** | ❌ No | ✅ Sí |
| **Cuándo usar** | Utilidades, constantes, factory methods | Estado del objeto, comportamiento específico |

---

#### 20. Composición vs Herencia: ¿Cuándo usar cada una?

**Respuesta:**

**Composición** y **Herencia** son dos formas de reutilizar código, pero tienen filosofías diferentes.

**Herencia (is-a):** "es un"
- Una clase **extiende** otra
- Relación fuerte y rígida
- Comparte comportamiento y estado

**Composición (has-a):** "tiene un"
- Una clase **contiene** instancias de otras clases
- Relación flexible
- Delega responsabilidades

---

**Herencia - Relación "es-un"**

```java
// Perro ES UN Animal
public class Animal {
    protected String nombre;
    
    public void comer() {
        System.out.println(nombre + " está comiendo");
    }
    
    public void dormir() {
        System.out.println(nombre + " está durmiendo");
    }
}

public class Perro extends Animal {
    public void ladrar() {
        System.out.println(nombre + " está ladrando");
    }
    
    @Override
    public void comer() {
        System.out.println(nombre + " está comiendo croquetas");
    }
}

// Uso
Perro perro = new Perro();
perro.nombre = "Firulais";
perro.comer();    // Heredado (sobrescrito)
perro.dormir();   // Heredado
perro.ladrar();   // Propio
```

**Ventajas de herencia:**
- ✅ Reutilización directa de código
- ✅ Polimorfismo (tratar Perro como Animal)
- ✅ Jerarquías lógicas claras

**Desventajas de herencia:**
- ❌ Acoplamiento fuerte (cambios en padre afectan hijos)
- ❌ Jerarquías rígidas (difícil cambiar)
- ❌ Solo herencia simple en Java
- ❌ Expone implementación interna del padre

---

**Composición - Relación "tiene-un"**

```java
// Motor es un componente
public class Motor {
    private int potencia;
    
    public void arrancar() {
        System.out.println("Motor arrancando (" + potencia + " HP)");
    }
    
    public void detener() {
        System.out.println("Motor detenido");
    }
}

// Transmisión es un componente
public class Transmision {
    private String tipo;
    
    public void cambiarMarcha(int marcha) {
        System.out.println("Cambiando a marcha " + marcha);
    }
}

// Automóvil TIENE UN Motor y TIENE UNA Transmisión
public class Automovil {
    private Motor motor;              // Composición
    private Transmision transmision;  // Composición
    private String marca;
    
    public Automovil(String marca, int potenciaMotor) {
        this.marca = marca;
        this.motor = new Motor(potenciaMotor);
        this.transmision = new Transmision("Automática");
    }
    
    // Delegar comportamiento a los componentes
    public void arrancar() {
        motor.arrancar();
        System.out.println(marca + " listo para conducir");
    }
    
    public void cambiarMarcha(int marcha) {
        transmision.cambiarMarcha(marcha);
    }
    
    public void detener() {
        motor.detener();
    }
}

// Uso
Automovil auto = new Automovil("Toyota", 150);
auto.arrancar();
auto.cambiarMarcha(3);
auto.detener();
```

**Ventajas de composición:**
- ✅ Bajo acoplamiento (componentes independientes)
- ✅ Flexibilidad (puedes cambiar componentes en runtime)
- ✅ Más fácil de testear (puedes mockear componentes)
- ✅ Evita jerarquías complejas
- ✅ Puedes "simular" herencia múltiple

**Desventajas de composición:**
- ❌ Más código boilerplate (delegación manual)
- ❌ Menos intuitivo para relaciones naturales "es-un"

---

**Ejemplo: Problema con herencia excesiva**

```java
// ❌ MAL diseño con herencia
public class Empleado {
    protected String nombre;
    protected double salario;
    
    public void trabajar() {
        System.out.println(nombre + " está trabajando");
    }
}

public class EmpleadoConBeneficios extends Empleado {
    protected double seguroMedico;
    protected double fondoPensiones;
}

public class EmpleadoConAuto extends EmpleadoConBeneficios {
    protected String modeloAuto;
}

// ⚠️ Problema: ¿Qué pasa si quiero un empleado con auto pero sin beneficios?
// ⚠️ Jerarquía rígida y limitada
```

**✅ Mejor diseño con composición:**

```java
// Componentes independientes
public class Beneficios {
    private double seguroMedico;
    private double fondoPensiones;
    
    public void calcularBeneficios() {
        // Lógica de beneficios
    }
}

public class AsignacionAuto {
    private String modelo;
    private String placa;
    
    public void asignarAuto(String modelo) {
        this.modelo = modelo;
    }
}

// Empleado flexible con componentes opcionales
public class Empleado {
    private String nombre;
    private double salario;
    private Beneficios beneficios;        // Opcional
    private AsignacionAuto auto;          // Opcional
    
    public Empleado(String nombre, double salario) {
        this.nombre = nombre;
        this.salario = salario;
    }
    
    public void agregarBeneficios(Beneficios beneficios) {
        this.beneficios = beneficios;
    }
    
    public void asignarAuto(AsignacionAuto auto) {
        this.auto = auto;
    }
}

// Uso flexible
Empleado emp1 = new Empleado("Juan", 50000);
Empleado emp2 = new Empleado("María", 60000);
emp2.agregarBeneficios(new Beneficios());
emp2.asignarAuto(new AsignacionAuto());

Empleado emp3 = new Empleado("Pedro", 55000);
emp3.asignarAuto(new AsignacionAuto());  // Auto sin beneficios ✅
```

---

**¿Cuándo usar cada una?**

**Usa HERENCIA cuando:**
- ✅ Existe una relación "es-un" **natural y verdadera**
- ✅ La subclase es una **especialización** del padre
- ✅ Necesitas polimorfismo (tratar objetos de tipos diferentes de manera uniforme)
- ✅ La jerarquía es **estable** y no cambiará frecuentemente

**Ejemplos válidos:**
- `Perro extends Animal` (un perro ES un animal)
- `ArrayList extends AbstractList` (es una lista)
- `FileNotFoundException extends IOException` (es una excepción de IO)

---

**Usa COMPOSICIÓN cuando:**
- ✅ Existe una relación "tiene-un"
- ✅ Necesitas **flexibilidad** para cambiar comportamiento
- ✅ Quieres **reutilizar funcionalidad** sin heredar toda la clase
- ✅ La relación no es permanente o puede cambiar en runtime
- ✅ Necesitas simular "herencia múltiple"

**Ejemplos válidos:**
- `Automovil` tiene un `Motor` (no ES un motor)
- `Empleado` tiene `Beneficios` (no ES beneficios)
- `Usuario` tiene `Direccion` (no ES una dirección)

---

**Principio de diseño:**

> **"Favor composition over inheritance"** (Gang of Four - Design Patterns)

La composición es generalmente más flexible y mantenible. **Usa herencia con moderación** y solo cuando realmente represente una relación "es-un" natural.

---

**Ejemplo del mundo real (Spring Framework):**

```java
// Spring usa composición extensivamente
@RestController
public class UsuarioController {
    // Composición: Controller TIENE Service, Repository
    private final UsuarioService usuarioService;
    private final EmailService emailService;
    private final AuditoriaService auditoriaService;
    
    public UsuarioController(
        UsuarioService usuarioService,
        EmailService emailService,
        AuditoriaService auditoriaService
    ) {
        this.usuarioService = usuarioService;
        this.emailService = emailService;
        this.auditoriaService = auditoriaService;
    }
    
    @PostMapping("/usuarios")
    public ResponseEntity<Usuario> crear(@RequestBody UsuarioDTO dto) {
        Usuario usuario = usuarioService.crear(dto);
        emailService.enviarBienvenida(usuario.getEmail());
        auditoriaService.registrar("USUARIO_CREADO", usuario.getId());
        return ResponseEntity.ok(usuario);
    }
}

// ✅ Fácil de testear (puedes mockear cada servicio)
// ✅ Flexible (puedes inyectar diferentes implementaciones)
// ✅ Desacoplado (cada servicio es independiente)
```

---

### 🔹 Colecciones (Collections Framework)

#### 21. ¿Cuál es la jerarquía de la Collections Framework?

**Respuesta:**

```
                    Iterable
                       |
                   Collection
                  /     |     \
               Set    List    Queue
              / | \     |      /  \
     HashSet  |  \   ArrayList  PriorityQueue
              |   \    |
         LinkedHashSet \  LinkedList
              |         \     |
          TreeSet    Vector  Deque
                              |
                         ArrayDeque
                         LinkedList

                    Map (no hereda de Collection)
                   / | \
          HashMap  |  TreeMap
                   |
            LinkedHashMap
```

**Interfaces principales:**

- **Collection**: Interface raíz para grupos de objetos.
- **List**: Colección ordenada (secuencia), permite duplicados.
- **Set**: No permite duplicados.
- **Queue**: Cola FIFO (First In First Out).
- **Deque**: Cola de doble extremo.
- **Map**: Almacena pares clave-valor.

---

#### 22. ¿Cuándo usar ArrayList vs LinkedList?

**Respuesta:**

| Característica | ArrayList | LinkedList |
|----------------|-----------|------------|
| **Estructura interna** | Array dinámico redimensionable | Lista doblemente enlazada (nodos) |
| **Acceso por índice** | **O(1)** - Muy rápido | **O(n)** - Debe recorrer nodos |
| **Insertar/eliminar al final** | **O(1)** amortizado | **O(1)** |
| **Insertar/eliminar al inicio** | **O(n)** - Desplaza elementos | **O(1)** |
| **Insertar/eliminar en medio** | **O(n)** - Desplaza elementos | **O(1)** si tienes la referencia |
| **Uso de memoria** | Más eficiente | Mayor overhead (punteros prev/next) |
| **Iteración** | Más rápida (localidad de caché) | Más lenta (saltos de memoria) |
| **Búsqueda** | **O(n)** | **O(n)** |

**¿Cuándo usar cada una?**

**ArrayList:**
- ✅ Acceso frecuente por índice (`get(i)`)
- ✅ Iteración frecuente
- ✅ Añadir al final (`add()`)
- ✅ Caso de uso más común (95% de los casos)

**LinkedList:**
- ✅ Inserciones/eliminaciones frecuentes al inicio
- ✅ Implementar pilas, colas (aunque `ArrayDeque` es mejor)
- ✅ No conoces el tamaño final
- ❌ Raramente es la mejor opción en la práctica

**Ejemplo práctico:**
```java
// ✅ BIEN: Acceso aleatorio
List<String> lista = new ArrayList<>();
lista.add("A");
lista.add("B");
String elemento = lista.get(0);  // O(1)

// ❌ MAL: LinkedList sin razón
List<String> lista = new LinkedList<>();  // Solo si realmente necesitas inserciones al inicio
```

**Conclusión:** **Usa ArrayList por defecto**. Solo usa LinkedList si tienes una razón específica.

---

#### 23. ¿Qué diferencias hay entre HashMap, TreeMap y LinkedHashMap?

**Respuesta:**

| Característica | HashMap | TreeMap | LinkedHashMap |
|----------------|---------|---------|---------------|
| **Orden** | No garantiza orden | Ordenado por clave | Orden de inserción |
| **Estructura** | Tabla hash + listas/árboles | Árbol Rojo-Negro | Hash + lista enlazada |
| **Performance get/put** | **O(1)** promedio | **O(log n)** | **O(1)** promedio |
| **Permite null** | 1 clave null, valores null ✅ | ❌ No (clave null) | ✅ |
| **Thread-safe** | ❌ | ❌ | ❌ |
| **Uso de memoria** | Medio | Alto (nodos árbol) | Alto (punteros extra) |

**HashMap:**
```java
Map<String, Integer> map = new HashMap<>();
map.put("C", 3);
map.put("A", 1);
map.put("B", 2);
// Orden impredecible: {A=1, B=2, C=3} o {C=3, B=2, A=1}
```

**TreeMap:**
```java
Map<String, Integer> map = new TreeMap<>();
map.put("C", 3);
map.put("A", 1);
map.put("B", 2);
// Siempre ordenado: {A=1, B=2, C=3}

// Métodos adicionales:
map.firstKey();  // "A"
map.lastKey();   // "C"
map.headMap("B"); // {A=1}
```

**LinkedHashMap:**
```java
Map<String, Integer> map = new LinkedHashMap<>();
map.put("C", 3);
map.put("A", 1);
map.put("B", 2);
// Orden de inserción: {C=3, A=1, B=2}

// Útil para LRU Cache:
Map<String, Integer> lru = new LinkedHashMap<>(16, 0.75f, true); // accessOrder=true
```

**¿Cuándo usar cada uno?**

- **HashMap**: Caso general (95% de los casos). Máximo rendimiento.
- **TreeMap**: Necesitas orden natural o personalizado. Operaciones de rango.
- **LinkedHashMap**: Necesitas mantener orden de inserción. Cachés LRU.

---

#### 17. ¿Qué es un HashSet? ¿Cómo funciona internamente?

**Respuesta:**

`HashSet` es una implementación de `Set` que **no permite duplicados** y **no garantiza orden**.

**Funcionamiento interno:**
Internamente, `HashSet` usa un `HashMap`:
```java
// Simplificado
public class HashSet<E> {
    private HashMap<E, Object> map;
    private static final Object PRESENT = new Object();
    
    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }
    
    public boolean contains(Object o) {
        return map.containsKey(o);
    }
}
```

Los elementos del Set son las **claves** del HashMap interno, y todos comparten el mismo valor dummy (`PRESENT`).

**Características:**
- **Performance**: O(1) para add, remove, contains (promedio).
- **No permite duplicados**: Usa `hashCode()` y `equals()` para verificar.
- **Permite un null**.
- **No thread-safe**.

**Ejemplo:**
```java
Set<String> set = new HashSet<>();
set.add("Java");
set.add("Python");
set.add("Java");  // No se añade (duplicado)

System.out.println(set.size());  // 2
System.out.println(set.contains("Java"));  // true - O(1)
```

**Variantes:**
- **LinkedHashSet**: Mantiene orden de inserción.
- **TreeSet**: Mantiene orden natural (usa TreeMap internamente).

---

#### 18. ¿Cuál es la diferencia entre List, Set y Map?

**Respuesta:**

| Característica | List | Set | Map |
|----------------|------|-----|-----|
| **Duplicados** | ✅ Permite | ❌ No permite | Claves únicas, valores duplicados ✅ |
| **Orden** | Sí (índice) | Depende de implementación | Depende de implementación |
| **Acceso** | Por índice | Solo iteración | Por clave |
| **Null** | Múltiples nulls | 1 null (HashSet) | 1 clave null (HashMap) |
| **Implementaciones** | ArrayList, LinkedList | HashSet, TreeSet | HashMap, TreeMap |

**Cuándo usar cada una:**

**List** - Secuencia ordenada con acceso por índice:
```java
List<String> nombres = new ArrayList<>();
nombres.add("Ana");
nombres.add("Ana");  // ✅ Permite duplicados
System.out.println(nombres.get(0));  // Acceso por índice
```

**Set** - Colección única sin duplicados:
```java
Set<String> emails = new HashSet<>();
emails.add("user@mail.com");
emails.add("user@mail.com");  // Ignorado
System.out.println(emails.size());  // 1
```

**Map** - Asociación clave-valor:
```java
Map<String, Integer> edades = new HashMap<>();
edades.put("Ana", 25);
edades.put("Luis", 30);
System.out.println(edades.get("Ana"));  // 25
```

---

#### 19. ¿Qué es ConcurrentModificationException y cómo evitarla?

**Respuesta:**

Es una **excepción** que se lanza cuando modificas una colección mientras la estás iterando (con un iterador clásico o for-each).

**Ejemplo del problema:**
```java
List<String> lista = new ArrayList<>(Arrays.asList("A", "B", "C"));

// ❌ MAL: ConcurrentModificationException
for (String s : lista) {
    if (s.equals("B")) {
        lista.remove(s);  // Modificación durante iteración
    }
}
```

**Soluciones:**

**1. Usar Iterator explícito:**
```java
List<String> lista = new ArrayList<>(Arrays.asList("A", "B", "C"));
Iterator<String> it = lista.iterator();
while (it.hasNext()) {
    String s = it.next();
    if (s.equals("B")) {
        it.remove();  // ✅ Usa el remove del iterator
    }
}
```

**2. Usar removeIf (Java 8+):**
```java
lista.removeIf(s -> s.equals("B"));  // ✅ Forma más limpia
```

**3. Crear lista auxiliar:**
```java
List<String> aEliminar = new ArrayList<>();
for (String s : lista) {
    if (s.equals("B")) {
        aEliminar.add(s);
    }
}
lista.removeAll(aEliminar);
```

**4. Usar índice inverso (solo para List):**
```java
for (int i = lista.size() - 1; i >= 0; i--) {
    if (lista.get(i).equals("B")) {
        lista.remove(i);
    }
}
```

**5. Colecciones concurrentes:**
```java
List<String> lista = new CopyOnWriteArrayList<>(Arrays.asList("A", "B", "C"));
for (String s : lista) {
    if (s.equals("B")) {
        lista.remove(s);  // ✅ Thread-safe, pero costoso
    }
}
```

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

### 🔹 Java 8+ Features

#### 27. ¿Qué es Optional y por qué debemos usarlo?

**Respuesta:**

`Optional<T>` es una **clase contenedora** introducida en Java 8 que puede contener o no un valor no nulo. Su propósito es hacer explícito en la API que un método puede no devolver un valor, evitando `NullPointerException`.

**Problema sin Optional:**
```java
public String obtenerNombre(int id) {
    // Puede devolver null, pero no es obvio
    return baseDatos.findById(id);
}

String nombre = obtenerNombre(1);
System.out.println(nombre.toUpperCase());  // ❌ Posible NPE
```

**Solución con Optional:**
```java
public Optional<String> obtenerNombre(int id) {
    String nombre = baseDatos.findById(id);
    return Optional.ofNullable(nombre);  // Explícito: puede no haber valor
}

// Formas de usar:
Optional<String> opt = obtenerNombre(1);

// 1. Verificar y obtener
if (opt.isPresent()) {
    System.out.println(opt.get());
}

// 2. Valor por defecto
String nombre = opt.orElse("Desconocido");

// 3. Lanzar excepción
String nombre = opt.orElseThrow(() -> new NotFoundException());

// 4. Ejecutar si presente (funcional)
opt.ifPresent(n -> System.out.println(n.toUpperCase()));

// 5. Transformar
Optional<Integer> longitud = opt.map(String::length);

// 6. FlatMap para evitar Optional<Optional<T>>
Optional<String> ciudad = opt.flatMap(nombre -> obtenerCiudad(nombre));
```

**Métodos principales:**

| Método | Descripción |
|--------|-------------|
| `Optional.of(value)` | Crea Optional con valor (lanza NPE si es null) |
| `Optional.ofNullable(value)` | Crea Optional, acepta null |
| `Optional.empty()` | Crea Optional vacío |
| `isPresent()` | ¿Tiene valor? |
| `isEmpty()` | ¿Está vacío? (Java 11+) |
| `get()` | Obtiene valor (lanza NoSuchElementException si vacío) |
| `orElse(defaultValue)` | Valor por defecto |
| `orElseGet(Supplier)` | Valor por defecto lazy |
| `orElseThrow()` | Lanza excepción si vacío |
| `ifPresent(Consumer)` | Ejecuta acción si hay valor |
| `map(Function)` | Transforma el valor |
| `flatMap(Function)` | Transforma y aplana |
| `filter(Predicate)` | Filtra el valor |

**Best Practices:**
- ❌ NO uses `Optional` como parámetro de método.
- ❌ NO uses `Optional` en campos de clase.
- ✅ USA `Optional` como tipo de retorno.
- ❌ NO hagas `opt.get()` sin verificar (derrota el propósito).
- ✅ USA `orElse()`, `orElseGet()`, `ifPresent()`.

---

#### 21. ¿Qué es Streams API? ¿Cuáles son sus características?

**Respuesta:**

**Streams API** (Java 8+) permite procesar secuencias de elementos de forma **declarativa** (qué hacer, no cómo hacerlo) usando programación funcional.

**Características principales:**

1. **No es una estructura de datos**: Es una vista de datos (de colecciones, arrays, archivos, etc.).
2. **Lazy evaluation**: Las operaciones intermedias no se ejecutan hasta que hay una operación terminal.
3. **Consumibles**: Un Stream solo se puede usar una vez.
4. **Posiblemente infinitos**: Pueden representar secuencias infinitas.
5. **Pipelines**: Se encadenan operaciones.

**Tipos de operaciones:**

**Operaciones Intermedias** (devuelven Stream, son lazy):
- `filter(Predicate)`: Filtra elementos.
- `map(Function)`: Transforma elementos 1:1.
- `flatMap(Function)`: Transforma y aplana 1:N.
- `distinct()`: Elimina duplicados.
- `sorted()`: Ordena.
- `limit(n)`: Limita a n elementos.
- `skip(n)`: Salta n elementos.
- `peek(Consumer)`: Acción sin modificar (debug).

**Operaciones Terminales** (ejecutan el pipeline, devuelven resultado):
- `forEach(Consumer)`: Acción por elemento.
- `collect(Collector)`: Agrupa resultados.
- `toList()`: Convierte a lista (Java 16+).
- `count()`: Cuenta elementos.
- `reduce(BinaryOperator)`: Reduce a un solo valor.
- `anyMatch(Predicate)`: ¿Alguno cumple?
- `allMatch(Predicate)`: ¿Todos cumplen?
- `noneMatch(Predicate)`: ¿Ninguno cumple?
- `findFirst()`: Primer elemento.
- `findAny()`: Cualquier elemento.
- `min(Comparator)` / `max(Comparator)`: Mínimo/máximo.

**Ejemplos:**
```java
List<String> nombres = Arrays.asList("Ana", "Luis", "María", "Juan", "Pedro");

// Filtrar y transformar
List<String> resultado = nombres.stream()
    .filter(n -> n.length() > 3)      // Intermedia
    .map(String::toUpperCase)          // Intermedia
    .sorted()                          // Intermedia
    .collect(Collectors.toList());     // Terminal

// Contar
long count = nombres.stream()
    .filter(n -> n.startsWith("A"))
    .count();

// Verificar
boolean hayNombresCortos = nombres.stream()
    .anyMatch(n -> n.length() < 4);

// Reducir
Optional<String> concatenado = nombres.stream()
    .reduce((a, b) -> a + ", " + b);
```

**Paralelización:**
```java
nombres.parallelStream()  // Procesamiento paralelo
    .filter(n -> n.length() > 3)
    .collect(Collectors.toList());
```

---

#### 29. ¿Cuál es la diferencia entre map() y flatMap()?

**Respuesta:**

**map()**: Transforma cada elemento **1 a 1**. Aplica una función que devuelve un valor.

**flatMap()**: Transforma cada elemento **1 a N** y **aplana** el resultado. Aplica una función que devuelve un Stream, y luego combina todos los streams en uno solo.

**Ejemplo visual:**

```java
// map: 1 -> 1
[1, 2, 3]
    .map(n -> n * 2)
-> [2, 4, 6]

// flatMap: 1 -> N (y aplana)
[1, 2, 3]
    .flatMap(n -> Stream.of(n, n * 2))
-> [1, 2, 2, 4, 3, 6]
```

**Caso práctico:**

```java
List<String> frases = Arrays.asList("Hola mundo", "Java Streams");

// ❌ map devuelve Stream<String[]>
Stream<String[]> palabrasArray = frases.stream()
    .map(frase -> frase.split(" "));  // Cada frase se convierte en String[]

// ✅ flatMap devuelve Stream<String> (aplana)
List<String> todasLasPalabras = frases.stream()
    .flatMap(frase -> Arrays.stream(frase.split(" ")))
    .collect(Collectors.toList());
// Resultado: ["Hola", "mundo", "Java", "Streams"]
```

**Otro ejemplo: Optional anidado**
```java
// Tenemos usuarios con direcciones opcionales
Optional<Usuario> usuario = obtenerUsuario(1);

// ❌ map devuelve Optional<Optional<String>>
Optional<Optional<String>> ciudadMal = usuario
    .map(u -> u.getCiudad());  // getCiudad() devuelve Optional<String>

// ✅ flatMap devuelve Optional<String>
Optional<String> ciudadBien = usuario
    .flatMap(u -> u.getCiudad());
```

**Regla mnemotécnica:**
- **map**: Cuando tu función devuelve un **valor directo** (T).
- **flatMap**: Cuando tu función devuelve un **contenedor** (Stream<T>, Optional<T>).

---

#### 23. ¿Qué es una interfaz funcional? Menciona las más comunes

**Respuesta:**

Una **interfaz funcional** es una interfaz que tiene **exactamente un método abstracto** (SAM - Single Abstract Method). Puede tener múltiples métodos default o static.

Se usa como base para **expresiones lambda** y **method references**.

**Anotación:** `@FunctionalInterface` (opcional pero recomendada).

**Interfaces funcionales comunes en java.util.function:**

| Interfaz | Método | Descripción | Ejemplo |
|----------|--------|-------------|---------|
| **Predicate\<T>** | `boolean test(T t)` | Evalúa condición | `n -> n > 10` |
| **Function<T, R>** | `R apply(T t)` | Transforma T en R | `s -> s.length()` |
| **Consumer\<T>** | `void accept(T t)` | Consume (no devuelve) | `s -> System.out.println(s)` |
| **Supplier\<T>** | `T get()` | Provee un valor | `() -> new ArrayList<>()` |
| **UnaryOperator\<T>** | `T apply(T t)` | Function<T, T> | `n -> n * 2` |
| **BinaryOperator\<T>** | `T apply(T t1, T t2)` | Combina dos T | `(a, b) -> a + b` |
| **BiFunction<T, U, R>** | `R apply(T t, U u)` | Dos entradas, una salida | `(a, b) -> a + b` |
| **BiConsumer<T, U>** | `void accept(T t, U u)` | Consume dos valores | `(k, v) -> map.put(k, v)` |

**Ejemplos de uso:**

```java
// Predicate
Predicate<Integer> esPar = n -> n % 2 == 0;
System.out.println(esPar.test(4));  // true

// Function
Function<String, Integer> longitud = s -> s.length();
System.out.println(longitud.apply("Java"));  // 4

// Consumer
Consumer<String> imprimir = s -> System.out.println(s);
imprimir.accept("Hola");  // Imprime "Hola"

// Supplier
Supplier<Double> random = () -> Math.random();
System.out.println(random.get());  // Número aleatorio

// BinaryOperator
BinaryOperator<Integer> suma = (a, b) -> a + b;
System.out.println(suma.apply(5, 3));  // 8
```

**Crear tu propia interfaz funcional:**
```java
@FunctionalInterface
public interface Calculadora {
    int calcular(int a, int b);
    
    // Métodos default y static están permitidos
    default void info() {
        System.out.println("Soy una calculadora");
    }
}

// Uso
Calculadora suma = (a, b) -> a + b;
System.out.println(suma.calcular(5, 3));  // 8
```

---

#### 31. ¿Qué son las method references? Tipos y ejemplos

**Respuesta:**

Las **method references** son una forma abreviada de escribir lambdas que solo llaman a un método existente. Hacen el código más legible.

**Sintaxis:** `Clase::metodo` o `instancia::metodo`

**Tipos de Method References:**

**1. Referencia a método estático:** `Clase::metodoEstatico`
```java
// Lambda
Function<String, Integer> lambda = s -> Integer.parseInt(s);

// Method reference
Function<String, Integer> ref = Integer::parseInt;

// Uso en Stream
List<String> numeros = Arrays.asList("1", "2", "3");
List<Integer> ints = numeros.stream()
    .map(Integer::parseInt)
    .collect(Collectors.toList());
```

**2. Referencia a método de instancia de un objeto particular:** `instancia::metodo`
```java
String prefijo = "Hola ";

// Lambda
Function<String, String> lambda = s -> prefijo.concat(s);

// Method reference
Function<String, String> ref = prefijo::concat;

System.out.println(ref.apply("Mundo"));  // "Hola Mundo"
```

**3. Referencia a método de instancia de un objeto arbitrario:** `Clase::metodoInstancia`
```java
// Lambda
Function<String, String> lambda = s -> s.toUpperCase();

// Method reference
Function<String, String> ref = String::toUpperCase;

// Uso
List<String> palabras = Arrays.asList("java", "python", "c++");
palabras.stream()
    .map(String::toUpperCase)  // Cada string llama su propio toUpperCase
    .forEach(System.out::println);
```

**4. Referencia a constructor:** `Clase::new`
```java
// Lambda
Supplier<List<String>> lambda = () -> new ArrayList<>();

// Method reference
Supplier<List<String>> ref = ArrayList::new;

// Con parámetros
Function<Integer, List<String>> ref2 = ArrayList::new;  // ArrayList(int capacity)

// Uso en Stream
List<String> nombres = Arrays.asList("Ana", "Luis");
List<Persona> personas = nombres.stream()
    .map(Persona::new)  // Llama al constructor Persona(String nombre)
    .collect(Collectors.toList());
```

**Comparación:**

```java
// Todas estas formas son equivalentes:

// 1. Clase anónima
list.forEach(new Consumer<String>() {
    public void accept(String s) {
        System.out.println(s);
    }
});

// 2. Lambda
list.forEach(s -> System.out.println(s));

// 3. Method reference
list.forEach(System.out::println);
```

**Cuándo usar:**
- ✅ USA method reference si solo llamas a un método.
- ❌ USA lambda si necesitas lógica adicional.

---

#### 25. ¿Qué es el Collectors y cuáles son los más utilizados?

**Respuesta:**

`Collectors` es una clase de utilidad que proporciona implementaciones de `Collector` para operaciones de reducción comunes en Streams.

**Collectors más utilizados:**

**1. toList() / toSet() / toCollection():**
```java
List<String> lista = stream.collect(Collectors.toList());
Set<String> set = stream.collect(Collectors.toSet());
ArrayList<String> arrayList = stream.collect(Collectors.toCollection(ArrayList::new));
```

**2. toMap():**
```java
// Map<id, persona>
Map<Integer, Persona> mapa = personas.stream()
    .collect(Collectors.toMap(
        Persona::getId,      // Key
        persona -> persona   // Value
    ));

// Manejar claves duplicadas
Map<String, Integer> mapa = personas.stream()
    .collect(Collectors.toMap(
        Persona::getNombre,
        Persona::getEdad,
        (edad1, edad2) -> edad1  // En caso de duplicado, mantener el primero
    ));
```

**3. joining():**
```java
String resultado = nombres.stream()
    .collect(Collectors.joining());  // "AnaLuisMaria"

String resultado = nombres.stream()
    .collect(Collectors.joining(", "));  // "Ana, Luis, Maria"

String resultado = nombres.stream()
    .collect(Collectors.joining(", ", "[", "]"));  // "[Ana, Luis, Maria]"
```

**4. groupingBy():**
```java
// Agrupar personas por ciudad
Map<String, List<Persona>> porCiudad = personas.stream()
    .collect(Collectors.groupingBy(Persona::getCiudad));

// Contar por grupo
Map<String, Long> conteo = personas.stream()
    .collect(Collectors.groupingBy(
        Persona::getCiudad,
        Collectors.counting()
    ));

// Múltiples niveles
Map<String, Map<Integer, List<Persona>>> grupos = personas.stream()
    .collect(Collectors.groupingBy(
        Persona::getCiudad,
        Collectors.groupingBy(Persona::getEdad)
    ));
```

**5. partitioningBy():**
```java
// Divide en dos grupos: true/false
Map<Boolean, List<Persona>> particion = personas.stream()
    .collect(Collectors.partitioningBy(p -> p.getEdad() >= 18));

List<Persona> mayores = particion.get(true);
List<Persona> menores = particion.get(false);
```

**6. counting():**
```java
Long total = stream.collect(Collectors.counting());
```

**7. summingInt() / averagingInt() / summarizingInt():**
```java
// Sumar
Integer sumaEdades = personas.stream()
    .collect(Collectors.summingInt(Persona::getEdad));

// Promedio
Double promedioEdad = personas.stream()
    .collect(Collectors.averagingInt(Persona::getEdad));

// Estadísticas completas
IntSummaryStatistics stats = personas.stream()
    .collect(Collectors.summarizingInt(Persona::getEdad));
System.out.println(stats.getMax());
System.out.println(stats.getAverage());
```

**8. maxBy() / minBy():**
```java
Optional<Persona> mayor = personas.stream()
    .collect(Collectors.maxBy(Comparator.comparing(Persona::getEdad)));
```

---

#### 33. ¿Qué son las expresiones lambda? ¿Cuál es su sintaxis?

**Respuesta:**

Una **expresión lambda** es una función anónima (sin nombre) que se puede pasar como argumento. Introducidas en Java 8, permiten escribir código más conciso y funcional.

**Sintaxis:**
```
(parámetros) -> { cuerpo }
```

**Variaciones:**

```java
// Sin parámetros
() -> System.out.println("Hola")
() -> 42
() -> { return 42; }

// Un parámetro (paréntesis opcionales)
x -> x * 2
(x) -> x * 2
x -> { return x * 2; }

// Múltiples parámetros
(x, y) -> x + y
(x, y) -> { return x + y; }
(String s, Integer i) -> s.length() + i  // Con tipos explícitos

// Bloque de código
(x, y) -> {
    int suma = x + y;
    System.out.println(suma);
    return suma;
}
```

**Reglas:**
- Si hay **una sola expresión**, las llaves `{}` y `return` son opcionales.
- Si hay **un solo parámetro**, los paréntesis son opcionales.
- Los **tipos** de parámetros son opcionales (inferencia de tipos).

**Antes de Java 8 vs Con Lambda:**

```java
// Antes: Clase anónima
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("Clicked!");
    }
});

// Con Lambda
button.addActionListener(e -> System.out.println("Clicked!"));

// Antes: Comparator
Collections.sort(personas, new Comparator<Persona>() {
    @Override
    public int compare(Persona p1, Persona p2) {
        return p1.getNombre().compareTo(p2.getNombre());
    }
});

// Con Lambda
Collections.sort(personas, (p1, p2) -> p1.getNombre().compareTo(p2.getNombre()));

// Aún mejor: Method reference
Collections.sort(personas, Comparator.comparing(Persona::getNombre));
```

**Alcance (Scope):**
Las lambdas pueden capturar variables del contexto externo, pero deben ser **efectivamente finales**:

```java
int factor = 10;

Function<Integer, Integer> lambda = x -> x * factor;  // ✅ factor es efectivamente final

factor = 20;  // ❌ ERROR: factor ya no es efectivamente final
```

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

### 🔹 Concurrencia Básica

#### 34. ¿Cuál es la diferencia entre synchronized y volatile?

**Respuesta:**

Ambos mecanismos garantizan **visibilidad de memoria** entre hilos, pero tienen diferencias cruciales:

| Característica | synchronized | volatile |
|----------------|-------------|----------|
| **Visibilidad** | ✅ Sí | ✅ Sí |
| **Atomicidad** | ✅ Sí | ❌ No |
| **Bloqueo** | Sí (mutex) | No |
| **Performance** | Más lento | Más rápido |
| **Alcance** | Bloque/método | Solo variable |

**synchronized:**
- Garantiza que **solo un hilo** ejecute el bloque/método a la vez.
- Garantiza **atomicidad** (operaciones compuestas son indivisibles).
- Garantiza **visibilidad** (cambios son visibles a otros hilos).
- Puede causar **contención** (hilos esperando).

```java
public class Contador {
    private int cuenta = 0;
    
    // ✅ Thread-safe: solo un hilo a la vez
    public synchronized void incrementar() {
        cuenta++;  // Esta operación es atómica aquí
    }
    
    // O sincronizar solo el bloque crítico
    public void incrementar2() {
        synchronized(this) {
            cuenta++;
        }
    }
}
```

**volatile:**
- Solo garantiza **visibilidad** (lectura/escritura directa de/a memoria principal).
- **NO** garantiza atomicidad (operaciones como `i++` no son thread-safe).
- Útil para **flags booleanos** o **variables de una sola escritura**.
- Más rápido que synchronized (no hay bloqueo).

```java
public class Worker {
    private volatile boolean running = true;  // ✅ Siempre lee el valor actual
    
    public void run() {
        while (running) {  // Otros hilos verán el cambio inmediatamente
            // trabajo...
        }
    }
    
    public void stop() {
        running = false;  // Visible para todos los hilos
    }
}

// ❌ MAL: volatile no hace i++ atómico
private volatile int contador = 0;
public void incrementar() {
    contador++;  // ❌ NO thread-safe (read-modify-write)
}
```

**Cuándo usar cada uno:**
- **synchronized**: Operaciones compuestas, secciones críticas, múltiples variables relacionadas.
- **volatile**: Variables booleanas (flags), referencias a objetos inmutables, double-checked locking.

---

#### 28. ¿Qué es ExecutorService? ¿Cómo se usa?

**Respuesta:**

`ExecutorService` es una interfaz de alto nivel para manejar **pools de hilos** (ThreadPool) y ejecutar tareas asíncronas. Evita crear hilos manualmente con `new Thread()`, lo cual es costoso y difícil de gestionar.

**Ventajas:**
- **Reutilización de hilos**: Evita el overhead de crear/destruir hilos.
- **Control de concurrencia**: Limita cuántos hilos pueden ejecutarse simultáneamente.
- **Gestión de tareas**: Puedes enviar tareas y obtener resultados (`Future`).
- **Shutdown ordenado**: Cierra el pool correctamente.

**Tipos de ExecutorService:**

```java
// 1. FixedThreadPool: N hilos fijos, reutilizables
ExecutorService executor = Executors.newFixedThreadPool(4);

// 2. CachedThreadPool: Crea hilos según necesidad, reutiliza si están libres
ExecutorService executor = Executors.newCachedThreadPool();

// 3. SingleThreadExecutor: Un solo hilo (garantiza orden FIFO)
ExecutorService executor = Executors.newSingleThreadExecutor();

// 4. ScheduledExecutorService: Para tareas programadas/periódicas
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);
```

**Ejemplo de uso:**

```java
ExecutorService executor = Executors.newFixedThreadPool(3);

// Enviar tareas sin resultado
executor.submit(() -> {
    System.out.println("Tarea ejecutada por: " + Thread.currentThread().getName());
});

// Enviar tareas con resultado (Callable)
Future<Integer> future = executor.submit(() -> {
    Thread.sleep(1000);
    return 42;
});

try {
    Integer resultado = future.get();  // Bloquea hasta obtener resultado
    System.out.println("Resultado: " + resultado);
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}

// IMPORTANTE: Siempre cerrar el executor
executor.shutdown();  // No acepta más tareas, espera a que terminen las actuales
// executor.shutdownNow();  // Intenta cancelar tareas en ejecución
```

**Ejemplo con múltiples tareas:**

```java
ExecutorService executor = Executors.newFixedThreadPool(4);

List<Callable<String>> tareas = Arrays.asList(
    () -> procesarUsuario(1),
    () -> procesarUsuario(2),
    () -> procesarUsuario(3)
);

// invokeAll: ejecuta todas y espera a que terminen
List<Future<String>> resultados = executor.invokeAll(tareas);

for (Future<String> future : resultados) {
    System.out.println(future.get());
}

executor.shutdown();
```

**Best Practices:**
- Usa try-with-resources o always shutdown.
- `newFixedThreadPool` es el más común para apps server-side.
- Para CPU-intensive: pool size = número de cores.
- Para I/O-intensive: pool size > número de cores.

---

#### 29. ¿Qué es un deadlock y cómo evitarlo?

**Respuesta:**

Un **deadlock** (interbloqueo) ocurre cuando dos o más hilos se quedan **esperando mutuamente** para liberar recursos, y ninguno puede avanzar. El programa se congela.

**Condiciones para un deadlock (deben cumplirse las 4):**
1. **Exclusión mutua**: Recursos no compartibles.
2. **Hold and wait**: Un hilo retiene un recurso y espera otro.
3. **No preemption**: No se puede forzar la liberación del recurso.
4. **Espera circular**: Hilo A espera a B, B espera a A.

**Ejemplo de deadlock:**

```java
public class DeadlockDemo {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void metodo1() {
        synchronized(lock1) {
            System.out.println("Hilo1: tengo lock1");
            Thread.sleep(100);
            
            synchronized(lock2) {  // Espera lock2
                System.out.println("Hilo1: tengo lock2");
            }
        }
    }
    
    public void metodo2() {
        synchronized(lock2) {
            System.out.println("Hilo2: tengo lock2");
            Thread.sleep(100);
            
            synchronized(lock1) {  // Espera lock1 ← DEADLOCK
                System.out.println("Hilo2: tengo lock1");
            }
        }
    }
}

// Hilo1 ejecuta metodo1, Hilo2 ejecuta metodo2 → DEADLOCK
```

**¿Cómo evitarlo?**

**1. Orden consistente de locks:**
```java
// ✅ Siempre adquirir locks en el mismo orden
public void metodo1() {
    synchronized(lock1) {
        synchronized(lock2) {
            // ...
        }
    }
}

public void metodo2() {
    synchronized(lock1) {  // Mismo orden que metodo1
        synchronized(lock2) {
            // ...
        }
    }
}
```

**2. Usar tryLock con timeout:**
```java
Lock lock1 = new ReentrantLock();
Lock lock2 = new ReentrantLock();

public void metodo() {
    try {
        if (lock1.tryLock(50, TimeUnit.MILLISECONDS)) {
            try {
                if (lock2.tryLock(50, TimeUnit.MILLISECONDS)) {
                    try {
                        // Trabajo crítico
                    } finally {
                        lock2.unlock();
                    }
                }
            } finally {
                lock1.unlock();
            }
        }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
}
```

**3. Evitar locks anidados:**
- Redeseña para usar un solo lock o estructuras thread-safe.

**4. Usar colecciones concurrentes:**
```java
// En lugar de HashMap + synchronized
ConcurrentHashMap<K, V> map = new ConcurrentHashMap<>();
```

---

#### 37. ¿Qué diferencia hay entre Thread y Runnable?

**Respuesta:**

Ambos se usan para ejecutar código en paralelo, pero tienen diferencias importantes:

| Característica | Thread | Runnable |
|----------------|--------|----------|
| **Tipo** | Clase | Interfaz |
| **Herencia** | Extiende Thread (solo herencia) | Implementa Runnable (múltiples interfaces) |
| **Reutilización** | Menos flexible | Más flexible (composición) |
| **Best Practice** | ❌ No recomendado | ✅ Recomendado |

**Usando Thread:**
```java
class MiHilo extends Thread {
    @Override
    public void run() {
        System.out.println("Ejecutando en: " + currentThread().getName());
    }
}

// Uso
MiHilo hilo = new MiHilo();
hilo.start();  // ❌ Problema: no puedes extender otra clase
```

**Usando Runnable (mejor):**
```java
class MiTarea implements Runnable {
    @Override
    public void run() {
        System.out.println("Ejecutando en: " + Thread.currentThread().getName());
    }
}

// Uso
Thread hilo = new Thread(new MiTarea());
hilo.start();

// O con lambda (Java 8+)
Thread hilo = new Thread(() -> {
    System.out.println("Tarea lambda");
});
hilo.start();
```

**¿Por qué preferir Runnable?**

1. **Separación de responsabilidades**: La tarea (qué hacer) está separada del mecanismo de ejecución (cómo ejecutar).
2. **Composición sobre herencia**: Puedes extender otra clase.
3. **Reutilización**: Una misma tarea puede ejecutarse en múltiples hilos.
4. **Integración con ExecutorService**:

```java
ExecutorService executor = Executors.newFixedThreadPool(3);
executor.submit(new MiTarea());  // ✅ Funciona con Runnable
executor.shutdown();
```

**Callable (aún mejor para tareas que devuelven resultado):**
```java
Callable<Integer> tarea = () -> {
    Thread.sleep(1000);
    return 42;
};

ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(tarea);
Integer resultado = future.get();  // Bloquea y obtiene resultado
```

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

### 🔹 Clases y Métodos Útiles

#### 38. ¿Qué son las clases de utilidad en Java? Menciona las más importantes

**Respuesta:**

Las **clases de utilidad** son clases con métodos estáticos que proveen funcionalidades comunes. Típicamente son `final` con constructor privado.

**Principales clases de utilidad:**

**1. java.util.Collections:**
```java
List<Integer> lista = new ArrayList<>(Arrays.asList(3, 1, 2));

Collections.sort(lista);                    // Ordenar
Collections.reverse(lista);                 // Invertir
Collections.shuffle(lista);                 // Aleatorio
Collections.max(lista);                     // Máximo valor
Collections.frequency(lista, 2);            // Contar apariciones

List<String> inmutable = Collections.unmodifiableList(lista);
List<String> sincronizada = Collections.synchronizedList(lista);
```

**2. java.util.Arrays:**
```java
int[] arr = {3, 1, 2};

Arrays.sort(arr);                          // Ordenar
Arrays.binarySearch(arr, 2);               // Buscar (O(log n))
Arrays.toString(arr);                      // "[1, 2, 3]"
Arrays.equals(arr1, arr2);                 // Comparar contenido
Arrays.fill(arr, 0);                       // Llenar con valor

List<Integer> lista = Arrays.asList(1, 2, 3);  // Array a List
```

**3. java.util.Objects (Java 7+):**
```java
// Evitar NPE
Objects.requireNonNull(obj, "No puede ser null");

// Comparación null-safe
Objects.equals(obj1, obj2);                // No lanza NPE si alguno es null

// HashCode
Objects.hash(campo1, campo2, campo3);      // Para sobrescribir hashCode()

// ToString
Objects.toString(obj, "default");          // Devuelve "default" si obj es null
```

**4. java.lang.Math:**
```java
Math.max(5, 10);                           // 10
Math.min(5, 10);                           // 5
Math.abs(-5);                              // 5
Math.pow(2, 3);                            // 8.0
Math.sqrt(16);                             // 4.0
Math.random();                             // [0.0, 1.0)
Math.round(4.6);                           // 5
```

**5. java.util.UUID:**
```java
UUID id = UUID.randomUUID();
String uuid = id.toString();  // "550e8400-e29b-41d4-a716-446655440000"
```

**6. java.time (Java 8+):**
```java
LocalDate.now();
LocalDateTime.now();
Instant.now();
Duration.between(start, end);
```

---

#### 39. ¿Qué es try-with-resources y para qué sirve?

**Respuesta:**

**Try-with-resources** (Java 7+) es una sintaxis que garantiza el **cierre automático** de recursos que implementan `AutoCloseable` o `Closeable`, incluso si ocurre una excepción.

**Problema sin try-with-resources:**
```java
// ❌ Código verboso y propenso a errores
BufferedReader reader = null;
try {
    reader = new BufferedReader(new FileReader("file.txt"));
    String linea = reader.readLine();
    System.out.println(linea);
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (reader != null) {
        try {
            reader.close();  // Puede lanzar IOException
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**Solución con try-with-resources:**
```java
// ✅ Limpio y seguro
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
    String linea = reader.readLine();
    System.out.println(linea);
} catch (IOException e) {
    e.printStackTrace();
}
// reader.close() se llama automáticamente
```

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

[🏠 Volver al Inicio](./README.md) | [Siguiente: Spring Boot ➡️](./02-spring-boot.md)

**Múltiples recursos:**
```java
try (
    FileInputStream fis = new FileInputStream("input.txt");
    FileOutputStream fos = new FileOutputStream("output.txt");
    BufferedReader reader = new BufferedReader(new InputStreamReader(fis));
    BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(fos))
) {
    String linea;
    while ((linea = reader.readLine()) != null) {
        writer.write(linea);
        writer.newLine();
    }
} catch (IOException e) {
    e.printStackTrace();
}
// Todos los recursos se cierran en orden inverso automáticamente
```

**Tu propia clase AutoCloseable:**
```java
public class MiRecurso implements AutoCloseable {
    public void hacerAlgo() {
        System.out.println("Trabajando...");
    }
    
    @Override
    public void close() {
        System.out.println("Liberando recursos");
    }
}

// Uso
try (MiRecurso recurso = new MiRecurso()) {
    recurso.hacerAlgo();
}  // close() se llama automáticamente
```

**Recursos comunes que lo soportan:**
- Streams (InputStream, OutputStream)
- Readers/Writers
- Connection, Statement, ResultSet (JDBC)
- Scanner
- ExecutorService (con ayuda)

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

### 🔹 Conceptos Avanzados

#### 40. ¿Qué son los Generics y Wildcards en Java?

**Respuesta:**

**Generics** permiten escribir código **type-safe** que funciona con diferentes tipos, sin sacrificar seguridad en tiempo de compilación.

**Beneficios:**
1. Seguridad en tiempo de compilación (evita ClassCastException)
2. Elimina necesidad de casting
3. Código reutilizable

**Sintaxis básica:**
```java
// Clase genérica
public class Box<T> {
    private T contenido;
    
    public void set(T contenido) {
        this.contenido = contenido;
    }
    
    public T get() {
        return contenido;
    }
}

// Uso
Box<String> stringBox = new Box<>();
stringBox.set("Hola");
String valor = stringBox.get();  // Sin casting

Box<Integer> intBox = new Box<>();
intBox.set(42);
```

**Métodos genéricos:**
```java
public class Utilidades {
    // Método genérico estático
    public static <T> void imprimir(T elemento) {
        System.out.println(elemento);
    }
    
    // Método con bounded type parameter
    public static <T extends Comparable<T>> T maximo(T a, T b) {
        return a.compareTo(b) > 0 ? a : b;
    }
}

// Uso
Utilidades.imprimir("Texto");
Utilidades.imprimir(123);
Integer max = Utilidades.maximo(5, 10);
```

**Wildcards (Comodines):**

```java
// 1. Unbounded Wildcard <?>
// Acepta cualquier tipo
public void imprimirLista(List<?> lista) {
    for (Object obj : lista) {
        System.out.println(obj);
    }
}
// Puedes leer como Object, pero NO puedes agregar (excepto null)

// 2. Upper Bounded Wildcard <? extends T>
// Acepta T y subclases (Producer/Reads)
public double sumarNumeros(List<? extends Number> numeros) {
    double suma = 0;
    for (Number num : numeros) {
        suma += num.doubleValue();
    }
    return suma;
}
// Funciona con List<Integer>, List<Double>, etc.

// 3. Lower Bounded Wildcard <? super T>
// Acepta T y superclases (Consumer/Writes)
public void agregarEnteros(List<? super Integer> lista) {
    lista.add(1);
    lista.add(2);
}
// Funciona con List<Integer>, List<Number>, List<Object>
```

**PECS Principle** (Producer Extends, Consumer Super):
```java
// Producer: usa "extends" cuando LEES del genérico
public void copiarDe(List<? extends Number> origen) {
    Number num = origen.get(0);  // ✅ Puedes leer
    // origen.add(1);             // ❌ No puedes escribir
}

// Consumer: usa "super" cuando ESCRIBES en el genérico
public void copiarA(List<? super Integer> destino) {
    destino.add(42);              // ✅ Puedes escribir
    // Integer num = destino.get(0); // ❌ Solo obtienes Object
}
```

**Bounded Type Parameters:**
```java
// Múltiples bounds
public class Repositorio<T extends Serializable & Comparable<T>> {
    public void guardar(T entidad) {
        // T es Serializable Y Comparable
    }
}
```

**Best Practices:**
- Usa `<T>` para tipos genéricos de clase/método
- Usa `<E>` para elementos de colecciones
- Usa `<K, V>` para claves y valores (Map)
- Prefiere wildcards en parámetros de métodos para mayor flexibilidad

---

#### 41. ¿Qué es Type Erasure y sus limitaciones?

**Respuesta:**

**Type Erasure** es el mecanismo que usa Java para implementar generics manteniendo compatibilidad con código pre-Java 5. El compilador **borra** la información de tipos genéricos en tiempo de ejecución, reemplazándola con sus bounds o `Object`.

**¿Cómo funciona?**

```java
// En código fuente
public class Box<T> {
    private T contenido;
    
    public void set(T valor) {
        this.contenido = valor;
    }
}

// Después del Type Erasure (bytecode)
public class Box {
    private Object contenido;  // T → Object
    
    public void set(Object valor) {
        this.contenido = valor;
    }
}
```

**Con bounded types:**
```java
// Código fuente
public class NumeroBox<T extends Number> {
    private T valor;
}

// Después del erasure
public class NumeroBox {
    private Number valor;  // T extends Number → Number
}
```

**Limitaciones del Type Erasure:**

**1. No puedes instanciar tipos genéricos:**
```java
public class Contenedor<T> {
    private T[] array;
    
    public Contenedor() {
        // ❌ Error: Cannot create a generic array
        // array = new T[10];
        
        // ✅ Workaround
        array = (T[]) new Object[10];
    }
}
```

**2. No puedes usar instanceof con tipos parametrizados:**
```java
public <T> void verificar(Object obj) {
    // ❌ Error: Cannot perform instanceof check
    // if (obj instanceof T) { }
    
    // ❌ Error
    // if (obj instanceof List<String>) { }
    
    // ✅ Solo puedes verificar el tipo raw
    if (obj instanceof List) {
        List<?> lista = (List<?>) obj;
    }
}
```

**3. No puedes crear arrays de tipos parametrizados:**
```java
// ❌ Error
List<String>[] arrayDeListas = new List<String>[10];

// ✅ Workaround
List<String>[] arrayDeListas = (List<String>[]) new List<?>[10];
```

**4. No puedes usar primitivos como type parameters:**
```java
// ❌ Error
List<int> lista;

// ✅ Usa wrappers
List<Integer> lista;
```

**5. No puedes sobrecargar métodos con diferentes tipos genéricos:**
```java
public class Procesador {
    // ❌ Error: Ambos métodos tienen la misma firma después del erasure
    public void procesar(List<String> lista) { }
    public void procesar(List<Integer> lista) { }
    // Ambos se convierten en: procesar(List)
}
```

**6. No puedes hacer catch de tipos parametrizados:**
```java
// ❌ Error
try {
    // código
} catch (T e) {  // No permitido
}
```

**Consecuencias prácticas:**
- Heap pollution warnings al mezclar raw types y generics
- Necesidad de casting en algunos casos
- Imposible obtener información del tipo en runtime

**Workarounds comunes:**
```java
// Pasar Class<T> para tener información en runtime
public class Repositorio<T> {
    private final Class<T> tipo;
    
    public Repositorio(Class<T> tipo) {
        this.tipo = tipo;
    }
    
    public T crear() throws Exception {
        return tipo.getDeclaredConstructor().newInstance();
    }
    
    public boolean esInstancia(Object obj) {
        return tipo.isInstance(obj);
    }
}

// Uso
Repositorio<Usuario> repo = new Repositorio<>(Usuario.class);
Usuario usuario = repo.crear();
```

---

#### 42. ¿Qué es Reflection y cuándo usarlo?

**Respuesta:**

**Reflection API** permite inspeccionar y modificar el comportamiento de clases, métodos, campos y constructores **en tiempo de ejecución**, incluso si son privados.

**¿Para qué se usa?**
- Frameworks (Spring, Hibernate, JUnit)
- Serialización/Deserialización (Jackson, Gson)
- Dependency Injection
- Testing (acceder a campos privados)
- Proxies dinámicos

**Operaciones principales:**

**1. Obtener información de clase:**
```java
Class<?> clazz = String.class;
// O desde instancia
Class<?> clazz = "Hola".getClass();
// O por nombre
Class<?> clazz = Class.forName("java.lang.String");

// Información básica
System.out.println(clazz.getName());           // java.lang.String
System.out.println(clazz.getSimpleName());     // String
System.out.println(clazz.getPackage());        // java.lang
System.out.println(clazz.getSuperclass());     // class java.lang.Object
```

**2. Inspeccionar métodos:**
```java
public class Usuario {
    private String nombre;
    public void saludar() { }
    private void metodoPrivado() { }
}

Class<?> clazz = Usuario.class;

// Métodos públicos (incluidos heredados)
Method[] metodos = clazz.getMethods();

// Todos los métodos declarados (incluidos privados)
Method[] todosMetodos = clazz.getDeclaredMethods();

// Obtener método específico
Method metodo = clazz.getMethod("saludar");
Method privado = clazz.getDeclaredMethod("metodoPrivado");
```

**3. Invocar métodos:**
```java
public class Calculadora {
    public int sumar(int a, int b) {
        return a + b;
    }
    
    private String secreto() {
        return "Información privada";
    }
}

Calculadora calc = new Calculadora();
Class<?> clazz = calc.getClass();

// Invocar método público
Method sumar = clazz.getMethod("sumar", int.class, int.class);
int resultado = (int) sumar.invoke(calc, 5, 3);  // 8

// Invocar método privado
Method secreto = clazz.getDeclaredMethod("secreto");
secreto.setAccessible(true);  // ⚠️ Bypasear seguridad
String valor = (String) secreto.invoke(calc);
```

**4. Acceder a campos:**
```java
public class Usuario {
    private String nombre;
    public int edad;
}

Usuario user = new Usuario();
Class<?> clazz = user.getClass();

// Acceder a campo privado
Field nombreField = clazz.getDeclaredField("nombre");
nombreField.setAccessible(true);  // ⚠️ Bypasear private
nombreField.set(user, "Juan");    // Modificar valor
String nombre = (String) nombreField.get(user);  // Leer valor

// Campo público
Field edadField = clazz.getField("edad");
edadField.set(user, 30);
```

**5. Crear instancias:**
```java
// Constructor sin parámetros
Class<?> clazz = Usuario.class;
Usuario user = (Usuario) clazz.getDeclaredConstructor().newInstance();

// Constructor con parámetros
Constructor<?> constructor = clazz.getConstructor(String.class, int.class);
Usuario user2 = (Usuario) constructor.newInstance("Ana", 25);
```

**6. Inspeccionar anotaciones:**
```java
@Entity
@Table(name = "usuarios")
public class Usuario {
    @Id
    private Long id;
    
    @Column(name = "nombre_completo")
    private String nombre;
}

Class<?> clazz = Usuario.class;

// Verificar si tiene anotación
if (clazz.isAnnotationPresent(Entity.class)) {
    Entity entity = clazz.getAnnotation(Entity.class);
}

// Anotaciones en campos
Field campo = clazz.getDeclaredField("nombre");
if (campo.isAnnotationPresent(Column.class)) {
    Column col = campo.getAnnotation(Column.class);
    System.out.println(col.name());  // "nombre_completo"
}
```

**Ejemplo práctico - Mapper genérico:**
```java
public class SimpleMapper {
    public static <T> T mapear(Object origen, Class<T> destino) throws Exception {
        T instancia = destino.getDeclaredConstructor().newInstance();
        
        for (Field campoOrigen : origen.getClass().getDeclaredFields()) {
            campoOrigen.setAccessible(true);
            
            try {
                Field campoDestino = destino.getDeclaredField(campoOrigen.getName());
                campoDestino.setAccessible(true);
                
                Object valor = campoOrigen.get(origen);
                campoDestino.set(instancia, valor);
            } catch (NoSuchFieldException e) {
                // Campo no existe en destino, ignorar
            }
        }
        
        return instancia;
    }
}
```

**⚠️ Desventajas de Reflection:**
1. **Performance**: 10-100x más lento que código directo
2. **Seguridad**: Bypasea encapsulación y control de acceso
3. **Type Safety**: Pierdes verificación en tiempo de compilación
4. **Mantenimiento**: Errores en runtime, no en compile-time

**Best Practices:**
- Úsalo solo cuando sea absolutamente necesario
- Cachea `Method`, `Field`, `Constructor` objects
- Usa alternativas cuando sea posible (interfaces, lambdas)
- Ten cuidado con `setAccessible(true)` en producción

---

#### 43. ¿Cómo crear anotaciones personalizadas?

**Respuesta:**

Las **anotaciones personalizadas** permiten agregar metadatos a tu código que pueden ser procesados en tiempo de compilación o ejecución.

**Estructura básica:**
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MiAnotacion {
    String valor();
    int prioridad() default 1;
}
```

**Meta-anotaciones importantes:**

**@Target** - Dónde puede usarse:
```java
@Target(ElementType.TYPE)          // Clases, interfaces, enums
@Target(ElementType.METHOD)        // Métodos
@Target(ElementType.FIELD)         // Campos
@Target(ElementType.PARAMETER)     // Parámetros de métodos
@Target(ElementType.CONSTRUCTOR)   // Constructores
@Target({ElementType.METHOD, ElementType.FIELD})  // Múltiples
```

**@Retention** - Cuándo está disponible:
```java
@Retention(RetentionPolicy.SOURCE)    // Solo en código fuente (ej: @Override)
@Retention(RetentionPolicy.CLASS)     // En .class, no en runtime (default)
@Retention(RetentionPolicy.RUNTIME)   // Disponible en runtime vía Reflection
```

**Ejemplos prácticos:**

**1. Validación personalizada:**
```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface NotEmpty {
    String message() default "El campo no puede estar vacío";
}

public class Usuario {
    @NotEmpty(message = "El nombre es obligatorio")
    private String nombre;
    
    @NotEmpty
    private String email;
}

// Procesador
public class Validador {
    public static void validar(Object obj) throws Exception {
        for (Field campo : obj.getClass().getDeclaredFields()) {
            if (campo.isAnnotationPresent(NotEmpty.class)) {
                campo.setAccessible(true);
                Object valor = campo.get(obj);
                
                if (valor == null || valor.toString().isEmpty()) {
                    NotEmpty anotacion = campo.getAnnotation(NotEmpty.class);
                    throw new IllegalArgumentException(anotacion.message());
                }
            }
        }
    }
}
```

**2. Logging automático:**
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Loggable {
    String value() default "INFO";
}

public class UserService {
    @Loggable("DEBUG")
    public void crearUsuario(String nombre) {
        // lógica
    }
}

// AOP Proxy para interceptar
public class LoggingProxy implements InvocationHandler {
    private final Object target;
    
    public LoggingProxy(Object target) {
        this.target = target;
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (method.isAnnotationPresent(Loggable.class)) {
            Loggable log = method.getAnnotation(Loggable.class);
            System.out.println("[" + log.value() + "] Ejecutando: " + method.getName());
        }
        return method.invoke(target, args);
    }
}
```

**3. Medición de performance:**
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MedirTiempo {
    boolean async() default false;
}

// Procesador con AOP
@Aspect
@Component
public class PerformanceAspect {
    @Around("@annotation(medirTiempo)")
    public Object medirTiempo(ProceedingJoinPoint joinPoint, MedirTiempo medirTiempo) throws Throwable {
        long inicio = System.currentTimeMillis();
        
        Object resultado = joinPoint.proceed();
        
        long fin = System.currentTimeMillis();
        System.out.println(joinPoint.getSignature() + " tardó: " + (fin - inicio) + "ms");
        
        return resultado;
    }
}
```

**4. Configuración de roles:**
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequiereRol {
    String[] value();
}

@RestController
public class AdminController {
    @RequiereRol({"ADMIN", "SUPERUSER"})
    @DeleteMapping("/usuarios/{id}")
    public void eliminarUsuario(@PathVariable Long id) {
        // Solo admins
    }
}
```

**Anotaciones con arrays y enums:**
```java
public enum Nivel {
    LOW, MEDIUM, HIGH
}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Configuracion {
    String nombre();
    String[] autores() default {};
    Nivel nivel() default Nivel.MEDIUM;
    Class<? extends Procesador> procesador();
}

@Configuracion(
    nombre = "MiConfig",
    autores = {"Juan", "Ana"},
    nivel = Nivel.HIGH,
    procesador = MiProcesador.class
)
public class MiClase {
}
```

**Best Practices:**
- Usa nombres descriptivos
- Proporciona defaults razonables
- Documenta con JavaDoc
- Mantén las anotaciones simples
- Considera usar anotaciones estándar cuando existan (JSR-303, etc.)

---

#### 44. ¿Qué es un ClassLoader y cómo funciona?

**Respuesta:**

Un **ClassLoader** es responsable de **cargar clases** dinámicamente en la JVM. Java usa un sistema jerárquico de ClassLoaders que sigue el principio de **delegación**.

**Jerarquía de ClassLoaders:**

```
Bootstrap ClassLoader (C++ nativo)
    ↓ (carga clases core de Java: rt.jar)
Extension/Platform ClassLoader
    ↓ (carga extensiones: lib/ext/)
Application/System ClassLoader
    ↓ (carga tu código: classpath)
Custom ClassLoader
    (opcional: plugins, hot reload)
```

**Principio de Delegación:**

Cuando se solicita una clase:
1. El ClassLoader **delega** a su padre primero
2. Si el padre no la encuentra, intenta cargarla él mismo
3. Si no puede, lanza `ClassNotFoundException`

```java
// Orden de búsqueda para cargar "com.ejemplo.MiClase"
1. Bootstrap ClassLoader busca en rt.jar
2. Extension ClassLoader busca en lib/ext/
3. Application ClassLoader busca en classpath
4. Custom ClassLoader (si existe)
```

**Ventajas de la delegación:**
- Evita cargar clases duplicadas
- Seguridad: no puedes reemplazar clases core
- Consistencia: misma clase siempre cargada por el mismo loader

**Obtener ClassLoaders:**
```java
// ClassLoader actual
ClassLoader loader = MiClase.class.getClassLoader();

// ClassLoader del sistema
ClassLoader systemLoader = ClassLoader.getSystemClassLoader();

// ClassLoader padre
ClassLoader parent = loader.getParent();

// ClassLoader de contexto del thread
ClassLoader contextLoader = Thread.currentThread().getContextClassLoader();
```

**Cargar clases manualmente:**
```java
ClassLoader loader = MiClase.class.getClassLoader();

// Cargar clase
Class<?> clazz = loader.loadClass("com.ejemplo.OtraClase");

// Crear instancia
Object instancia = clazz.getDeclaredConstructor().newInstance();
```

**Custom ClassLoader:**
```java
public class MiClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String nombre) throws ClassNotFoundException {
        try {
            // Cargar bytes de la clase desde algún lugar
            // (archivo, red, base de datos, etc.)
            byte[] bytes = cargarBytesDeClase(nombre);
            
            // Definir clase desde bytes
            return defineClass(nombre, bytes, 0, bytes.length);
        } catch (Exception e) {
            throw new ClassNotFoundException(nombre, e);
        }
    }
    
    private byte[] cargarBytesDeClase(String nombre) throws IOException {
        String path = nombre.replace('.', '/') + ".class";
        InputStream is = getResourceAsStream(path);
        
        ByteArrayOutputStream buffer = new ByteArrayOutputStream();
        int data;
        while ((data = is.read()) != -1) {
            buffer.write(data);
        }
        
        return buffer.toByteArray();
    }
}

// Uso
MiClassLoader loader = new MiClassLoader();
Class<?> clazz = loader.loadClass("com.ejemplo.Plugin");
```

**Casos de uso prácticos:**

**1. Hot Reload (recarga sin reiniciar):**
```java
public class HotReloader {
    public Object recargar(String className) throws Exception {
        // Nuevo ClassLoader cada vez
        URLClassLoader loader = new URLClassLoader(
            new URL[]{new File("./plugins/").toURI().toURL()},
            this.getClass().getClassLoader()
        );
        
        Class<?> clazz = loader.loadClass(className);
        return clazz.getDeclaredConstructor().newInstance();
    }
}
```

**2. Cargar plugins dinámicamente:**
```java
public interface Plugin {
    void ejecutar();
}

public class PluginLoader {
    public Plugin cargarPlugin(File jarFile) throws Exception {
        URL[] urls = {jarFile.toURI().toURL()};
        URLClassLoader loader = new URLClassLoader(urls);
        
        // Cargar clase del plugin
        Class<?> clazz = loader.loadClass("com.plugins.MiPlugin");
        
        return (Plugin) clazz.getDeclaredConstructor().newInstance();
    }
}
```

**3. Isolation (aislamiento de dependencias):**
```java
// Cada plugin con su propio ClassLoader
// Evita conflictos de versiones de librerías
public class PluginManager {
    private Map<String, URLClassLoader> loaders = new HashMap<>();
    
    public void cargarPlugin(String nombre, File jar) throws Exception {
        URLClassLoader loader = new URLClassLoader(
            new URL[]{jar.toURI().toURL()},
            null  // Sin padre, aislado
        );
        loaders.put(nombre, loader);
    }
}
```

**Problemas comunes:**

**ClassNotFoundException vs NoClassDefFoundError:**
```java
// ClassNotFoundException: clase no encontrada en classpath
try {
    Class.forName("com.ejemplo.Inexistente");
} catch (ClassNotFoundException e) {
    // Clase no existe o no está en classpath
}

// NoClassDefFoundError: clase existía en compile-time pero no en runtime
// O hubo un error durante la inicialización estática
public class Prueba {
    static {
        throw new RuntimeException();  // Causa NoClassDefFoundError después
    }
}
```

**Memory Leaks con ClassLoaders:**
- Cada clase mantiene referencia a su ClassLoader
- Si no se liberan referencias, el ClassLoader no se puede GC
- Problema común en servidores de aplicaciones con hot redeploy

**Best Practices:**
- Usa el ClassLoader del contexto del thread para cargar recursos
- Cierra URLClassLoaders cuando termines
- Ten cuidado con ThreadLocal y ClassLoader leaks
- Evita custom ClassLoaders a menos que sea necesario

---

#### 38. ¿Qué son las referencias débiles (Soft, Weak, Phantom)?

**Respuesta:**

Java proporciona **4 tipos de referencias** con diferente comportamiento ante el Garbage Collector:

**1. Strong Reference (referencia fuerte) - Default:**
```java
Object obj = new Object();  // Strong reference
// El GC NUNCA recolecta objetos con strong references
```

**2. Soft Reference (referencia suave):**
```java
import java.lang.ref.SoftReference;

Object obj = new Object();
SoftReference<Object> softRef = new SoftReference<>(obj);
obj = null;  // Solo queda soft reference

// El GC recolecta solo si NECESITA memoria
Object retrieved = softRef.get();  // Puede devolver null si fue recolectado
```

**Uso: Cachés que pueden liberar memoria si es necesario**
```java
public class ImageCache {
    private Map<String, SoftReference<Image>> cache = new HashMap<>();
    
    public Image getImage(String path) {
        SoftReference<Image> ref = cache.get(path);
        
        if (ref != null) {
            Image img = ref.get();
            if (img != null) {
                return img;  // Cache hit
            }
        }
        
        // Cache miss o fue recolectada
        Image img = cargarImagen(path);
        cache.put(path, new SoftReference<>(img));
        return img;
    }
}
```

**3. Weak Reference (referencia débil):**
```java
import java.lang.ref.WeakReference;

Object obj = new Object();
WeakReference<Object> weakRef = new WeakReference<>(obj);
obj = null;

// El GC recolecta en el PRÓXIMO ciclo, sin importar memoria
Object retrieved = weakRef.get();  // Puede ser null
```

**Uso: WeakHashMap para cachés que no deben prevenir GC**
```java
// WeakHashMap: las keys son WeakReferences
Map<User, UserMetadata> metadata = new WeakHashMap<>();

User user = new User("Juan");
metadata.put(user, new UserMetadata());

// Cuando user no tiene más referencias fuertes,
// la entrada se elimina automáticamente del map
user = null;
System.gc();
// metadata.size() eventualmente será 0
```

**4. Phantom Reference (referencia fantasma):**
```java
import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;

ReferenceQueue<Object> queue = new ReferenceQueue<>();
Object obj = new Object();
PhantomReference<Object> phantomRef = new PhantomReference<>(obj, queue);

obj = null;

// phantomRef.get() SIEMPRE devuelve null
// Se usa para saber CUÁNDO un objeto fue recolectado
```

**Uso: Cleanup de recursos nativos**
```java
public class NativeResourceCleaner {
    private static final ReferenceQueue<NativeResource> queue = new ReferenceQueue<>();
    private static final Map<PhantomReference<NativeResource>, Long> cleaners = new ConcurrentHashMap<>();
    
    static {
        // Thread limpiador
        Thread cleanerThread = new Thread(() -> {
            while (true) {
                try {
                    PhantomReference<NativeResource> ref = 
                        (PhantomReference<NativeResource>) queue.remove();
                    
                    Long handle = cleaners.remove(ref);
                    if (handle != null) {
                        liberarRecursoNativo(handle);
                    }
                } catch (InterruptedException e) {
                    break;
                }
            }
        });
        cleanerThread.setDaemon(true);
        cleanerThread.start();
    }
    
    public static void registrar(NativeResource resource, long handle) {
        PhantomReference<NativeResource> ref = 
            new PhantomReference<>(resource, queue);
        cleaners.put(ref, handle);
    }
    
    private static native void liberarRecursoNativo(long handle);
}
```

**Comparación:**

| Tipo | ¿Cuándo se recolecta? | get() después GC | Uso típico |
|------|----------------------|------------------|------------|
| **Strong** | Nunca (mientras haya referencia) | Siempre disponible | Uso normal |
| **Soft** | Solo si falta memoria | Puede devolver null | Cachés sensibles a memoria |
| **Weak** | Siguiente GC | Puede devolver null | Listeners, WeakHashMap |
| **Phantom** | Después de finalización | Siempre null | Cleanup de recursos |

**Ejemplo práctico - Sistema de listeners:**
```java
public class EventBus {
    // Listeners como WeakReferences para evitar memory leaks
    private List<WeakReference<EventListener>> listeners = new ArrayList<>();
    
    public void registrar(EventListener listener) {
        listeners.add(new WeakReference<>(listener));
    }
    
    public void notificar(Event event) {
        // Limpiar referencias muertas
        listeners.removeIf(ref -> ref.get() == null);
        
        // Notificar listeners vivos
        for (WeakReference<EventListener> ref : listeners) {
            EventListener listener = ref.get();
            if (listener != null) {
                listener.onEvent(event);
            }
        }
    }
}

// Uso
EventListener listener = new MyListener();
eventBus.registrar(listener);

// Cuando listener sale de scope, se elimina automáticamente
listener = null;
System.gc();
```

**Best Practices:**
- Usa **Strong** para referencias normales (99% de los casos)
- Usa **Soft** para cachés que pueden liberar memoria
- Usa **Weak** para evitar memory leaks en listeners/observadores
- Usa **Phantom** solo para cleanup de recursos nativos
- Siempre verifica `ref.get() != null` antes de usar

---

#### 39. ¿Qué es el Fork/Join Framework?

**Respuesta:**

El **Fork/Join Framework** (Java 7+) es un framework para **paralelizar tareas recursivas** usando el patrón **divide y conquista**. Usa un pool de threads con **work-stealing** para máxima eficiencia.

**Conceptos clave:**

- **Fork**: Divide una tarea grande en subtareas más pequeñas
- **Join**: Espera y combina los resultados de las subtareas
- **Work-Stealing**: Threads ociosos "roban" trabajo de otros threads

**Clases principales:**

```java
// Tarea que devuelve resultado
RecursiveTask<T>

// Tarea sin resultado
RecursiveAction

// Pool especializado
ForkJoinPool
```

**Ejemplo 1 - Sumar array en paralelo:**
```java
import java.util.concurrent.RecursiveTask;
import java.util.concurrent.ForkJoinPool;

public class SumaParalela extends RecursiveTask<Long> {
    private static final int THRESHOLD = 10_000;
    private final long[] array;
    private final int inicio;
    private final int fin;
    
    public SumaParalela(long[] array, int inicio, int fin) {
        this.array = array;
        this.inicio = inicio;
        this.fin = fin;
    }
    
    @Override
    protected Long compute() {
        int longitud = fin - inicio;
        
        // Caso base: procesar secuencialmente
        if (longitud <= THRESHOLD) {
            return sumarSecuencial();
        }
        
        // Caso recursivo: dividir en 2
        int mitad = inicio + longitud / 2;
        
        SumaParalela izquierda = new SumaParalela(array, inicio, mitad);
        SumaParalela derecha = new SumaParalela(array, mitad, fin);
        
        // Fork: ejecutar izquierda en paralelo
        izquierda.fork();
        
        // Computar derecha en este thread
        long resultadoDerecha = derecha.compute();
        
        // Join: esperar resultado de izquierda
        long resultadoIzquierda = izquierda.join();
        
        return resultadoIzquierda + resultadoDerecha;
    }
    
    private long sumarSecuencial() {
        long suma = 0;
        for (int i = inicio; i < fin; i++) {
            suma += array[i];
        }
        return suma;
    }
}

// Uso
long[] array = new long[1_000_000];
Arrays.fill(array, 1);

ForkJoinPool pool = new ForkJoinPool();
SumaParalela tarea = new SumaParalela(array, 0, array.length);
long resultado = pool.invoke(tarea);
System.out.println("Suma: " + resultado);
```

**Ejemplo 2 - Procesamiento paralelo sin resultado:**
```java
public class ProcesadorImagenes extends RecursiveAction {
    private static final int THRESHOLD = 1000;
    private final BufferedImage imagen;
    private final int inicioY;
    private final int finY;
    
    public ProcesadorImagenes(BufferedImage imagen, int inicioY, int finY) {
        this.imagen = imagen;
        this.inicioY = inicioY;
        this.finY = finY;
    }
    
    @Override
    protected void compute() {
        if (finY - inicioY <= THRESHOLD) {
            // Procesar secuencialmente
            for (int y = inicioY; y < finY; y++) {
                for (int x = 0; x < imagen.getWidth(); x++) {
                    // Aplicar filtro
                    int rgb = imagen.getRGB(x, y);
                    int nuevo = aplicarFiltro(rgb);
                    imagen.setRGB(x, y, nuevo);
                }
            }
        } else {
            // Dividir en 2
            int mitad = inicioY + (finY - inicioY) / 2;
            invokeAll(
                new ProcesadorImagenes(imagen, inicioY, mitad),
                new ProcesadorImagenes(imagen, mitad, finY)
            );
        }
    }
    
    private int aplicarFiltro(int rgb) {
        // Filtro grayscale simple
        int r = (rgb >> 16) & 0xFF;
        int g = (rgb >> 8) & 0xFF;
        int b = rgb & 0xFF;
        int gray = (r + g + b) / 3;
        return (gray << 16) | (gray << 8) | gray;
    }
}
```

**Patrones comunes:**

**Pattern 1: Fork izquierda, compute derecha, join izquierda**
```java
TareaIzquierda tareaIzq = new TareaIzquierda();
TareaDerecha tareaDer = new TareaDerecha();

tareaIzq.fork();              // Asíncrono
T resultado2 = tareaDer.compute();  // Síncrono en este thread
T resultado1 = tareaIzq.join();     // Esperar

return combinar(resultado1, resultado2);
```

**Pattern 2: invokeAll para múltiples subtareas**
```java
List<SubTarea> subtareas = crearSubtareas();
invokeAll(subtareas);  // Fork y join automático

for (SubTarea tarea : subtareas) {
    procesar(tarea.getRawResult());
}
```

**Common Pool vs Custom Pool:**
```java
// Usar common pool (recomendado)
ForkJoinPool.commonPool().invoke(tarea);

// O simplemente
tarea.invoke();  // Usa common pool automáticamente

// Custom pool (solo si necesitas configuración específica)
ForkJoinPool customPool = new ForkJoinPool(8);  // 8 threads
customPool.invoke(tarea);
customPool.shutdown();
```

**¿Cuándo usar Fork/Join?**

✅ **Bueno para:**
- Algoritmos recursivos (merge sort, quick sort)
- Procesamiento de grandes arrays/colecciones
- Operaciones CPU-intensive que pueden dividirse
- Cuando las subtareas son balanceadas

❌ **NO usar para:**
- I/O operations (bloquean threads)
- Tareas no balanceadas
- Pocas subtareas (overhead del framework)
- Algoritmos no recursivos (usa ExecutorService normal)

**Comparación con Streams paralelos:**
```java
// Fork/Join manual
ForkJoinPool pool = new ForkJoinPool();
SumaParalela tarea = new SumaParalela(array, 0, array.length);
long suma = pool.invoke(tarea);

// Streams paralelos (usa Fork/Join internamente)
long suma = Arrays.stream(array).parallel().sum();
```

Los **parallel streams** usan Fork/Join internamente, así que para casos simples son más convenientes.

**Best Practices:**
- Ajusta THRESHOLD según el tamaño del problema
- Evita sincronización compartida en subtareas
- Usa `invokeAll()` para múltiples forks
- Prefiere parallel streams para casos simples
- Mide el performance (paralelismo tiene overhead)

---

#### 40. ¿Qué es CompletableFuture y programación asíncrona?

**Respuesta:**

**CompletableFuture** (Java 8+) es una mejora de `Future` que permite **programación asíncrona** y **reactiva** con un API fluido para encadenar operaciones, manejar errores y combinar resultados.

**Diferencia con Future tradicional:**

```java
// Future tradicional (Java 5) - Bloqueante
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<String> future = executor.submit(() -> {
    Thread.sleep(1000);
    return "Resultado";
});

String resultado = future.get();  // ❌ Bloquea el thread actual
```

```java
// CompletableFuture (Java 8+) - Non-blocking
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    sleep(1000);
    return "Resultado";
});

future.thenAccept(resultado -> {
    System.out.println(resultado);  // ✅ Callback sin bloquear
});
```

**Crear CompletableFutures:**

```java
// 1. Completado inmediatamente
CompletableFuture<String> completado = CompletableFuture.completedFuture("valor");

// 2. Tarea asíncrona con resultado (supplyAsync)
CompletableFuture<Integer> futuro = CompletableFuture.supplyAsync(() -> {
    // Código intensivo
    return 42;
});

// 3. Tarea asíncrona sin resultado (runAsync)
CompletableFuture<Void> futuro = CompletableFuture.runAsync(() -> {
    System.out.println("Tarea asíncrona");
});

// 4. Manual
CompletableFuture<String> futuro = new CompletableFuture<>();
// ...en otro thread
futuro.complete("valor");  // Completa manualmente
```

**Transformaciones (map-like):**

```java
CompletableFuture<Integer> futuro = CompletableFuture.supplyAsync(() -> 10);

// thenApply - transforma el resultado (síncrono)
CompletableFuture<String> transformado = futuro.thenApply(num -> "Número: " + num);

// thenApplyAsync - transforma en otro thread
CompletableFuture<String> transformado = futuro.thenApplyAsync(num -> {
    // Operación pesada
    return "Número: " + num;
});
```

**Consumir resultado (forEach-like):**

```java
CompletableFuture<String> futuro = CompletableFuture.supplyAsync(() -> "Hola");

// thenAccept - consume sin devolver nada
futuro.thenAccept(valor -> {
    System.out.println(valor);
});

// thenRun - ejecuta acción sin usar el resultado
futuro.thenRun(() -> {
    System.out.println("Completado");
});
```

**Composición (flatMap-like):**

```java
// thenCompose - encadena CompletableFutures (evita nesting)
CompletableFuture<Usuario> futuro = obtenerUserId(1)
    .thenCompose(id -> obtenerUsuario(id))
    .thenCompose(usuario -> obtenerPerfil(usuario));

// Sin thenCompose (anidado)
CompletableFuture<CompletableFuture<Usuario>> anidado = obtenerUserId(1)
    .thenApply(id -> obtenerUsuario(id));  // ❌ Anidado
```

**Combinación de múltiples futuros:**

```java
CompletableFuture<String> futuro1 = CompletableFuture.supplyAsync(() -> "Hola");
CompletableFuture<String> futuro2 = CompletableFuture.supplyAsync(() -> "Mundo");

// thenCombine - combina 2 futuros
CompletableFuture<String> combinado = futuro1.thenCombine(futuro2, 
    (resultado1, resultado2) -> resultado1 + " " + resultado2);

// allOf - espera que TODOS completen
CompletableFuture<Void> todos = CompletableFuture.allOf(futuro1, futuro2);
todos.thenRun(() -> {
    System.out.println("Todos completados");
});

// anyOf - espera que CUALQUIERA complete
CompletableFuture<Object> cualquiera = CompletableFuture.anyOf(futuro1, futuro2);
```

**Manejo de errores:**

```java
CompletableFuture<Integer> futuro = CompletableFuture.supplyAsync(() -> {
    if (Math.random() > 0.5) {
        throw new RuntimeException("Error");
    }
    return 42;
});

// exceptionally - proporciona valor por defecto
futuro.exceptionally(ex -> {
    System.err.println("Error: " + ex.getMessage());
    return -1;  // Valor por defecto
});

// handle - maneja tanto éxito como error
futuro.handle((resultado, excepcion) -> {
    if (excepcion != null) {
        return -1;
    }
    return resultado * 2;
});

// whenComplete - ejecuta código sin cambiar el resultado
futuro.whenComplete((resultado, excepcion) -> {
    if (excepcion != null) {
        logger.error("Falló", excepcion);
    } else {
        logger.info("Éxito: " + resultado);
    }
});
```

**Ejemplo práctico - Llamadas API paralelas:**

```java
public class ServicioUsuarios {
    public CompletableFuture<Usuario> obtenerUsuario(Long id) {
        return CompletableFuture.supplyAsync(() -> {
            // Llamada HTTP
            return httpClient.get("/usuarios/" + id);
        });
    }
    
    public CompletableFuture<List<Pedido>> obtenerPedidos(Long userId) {
        return CompletableFuture.supplyAsync(() -> {
            return httpClient.get("/pedidos?userId=" + userId);
        });
    }
    
    public CompletableFuture<Perfil> obtenerPerfil(Long userId) {
        return CompletableFuture.supplyAsync(() -> {
            return httpClient.get("/perfiles/" + userId);
        });
    }
    
    // Obtener todo en paralelo
    public CompletableFuture<UsuarioCompleto> obtenerUsuarioCompleto(Long id) {
        CompletableFuture<Usuario> futurousuario = obtenerUsuario(id);
        CompletableFuture<List<Pedido>> futuroPedidos = obtenerPedidos(id);
        CompletableFuture<Perfil> futuroPerfil = obtenerPerfil(id);
        
        return CompletableFuture.allOf(futuroUsuario, futuroPedidos, futuroPerfil)
            .thenApply(v -> {
                Usuario usuario = futuroUsuario.join();
                List<Pedido> pedidos = futuroPedidos.join();
                Perfil perfil = futuroPerfil.join();
                
                return new UsuarioCompleto(usuario, pedidos, perfil);
            });
    }
}

// Uso
servicio.obtenerUsuarioCompleto(1L)
    .thenAccept(usuarioCompleto -> {
        System.out.println(usuarioCompleto);
    })
    .exceptionally(ex -> {
        System.err.println("Error: " + ex);
        return null;
    });
```

**Timeouts:**

```java
CompletableFuture<String> futuro = CompletableFuture.supplyAsync(() -> {
    sleep(5000);
    return "Resultado";
});

// Java 9+: Timeout
futuro.orTimeout(3, TimeUnit.SECONDS)
    .exceptionally(ex -> {
        System.err.println("Timeout!");
        return "Valor por defecto";
    });

// Java 9+: Valor por defecto si tarda mucho
futuro.completeOnTimeout("Timeout", 3, TimeUnit.SECONDS);
```

**Thread Pools personalizados:**

```java
// Por defecto usa ForkJoinPool.commonPool()
// Para I/O operations, usa un pool dedicado
Executor ioPool = Executors.newFixedThreadPool(10);

CompletableFuture<String> futuro = CompletableFuture.supplyAsync(() -> {
    // I/O operation
    return leerArchivo();
}, ioPool);
```

**Best Practices:**
- Usa para operaciones I/O (HTTP, BD, archivos)
- NO uses para CPU-intensive (usa Fork/Join o parallel streams)
- Siempre maneja errores con `exceptionally` o `handle`
- Usa thread pool dedicado para I/O
- Evita `get()` (es bloqueante), prefiere callbacks
- Compón operaciones con `thenCompose`, `thenCombine`

---

#### 41. ¿Qué son las Sealed Classes (Java 17+)?

**Respuesta:**

**Sealed Classes** (clases selladas) permiten **controlar explícitamente** qué clases pueden extender o implementar una clase/interfaz. Esto proporciona más control que `public` (cualquiera) o `final` (nadie).

**Sintaxis:**
```java
// Clase sellada que permite solo 3 subclases
public sealed class Forma
    permits Circulo, Rectangulo, Triangulo {
    // código común
}

// Subclases permitidas deben ser: final, sealed o non-sealed
public final class Circulo extends Forma {
    private double radio;
}

public final class Rectangulo extends Forma {
    private double ancho, alto;
}

public sealed class Triangulo extends Forma 
    permits TrianguloEquilatero, TrianguloIsosceles {
}

public final class TrianguloEquilatero extends Triangulo { }
public final class TrianguloIsosceles extends Triangulo { }
```

**Reglas de subclases:**

```java
// 1. final - No permite más subclases
public final class Circulo extends Forma { }

// 2. sealed - Sellada a su vez, con su propio permits
public sealed class Triangulo extends Forma 
    permits TrianguloEquilatero { }

// 3. non-sealed - Abre la jerarquía de nuevo
public non-sealed class Poligono extends Forma { }

// Ahora cualquiera puede extender Poligono
public class MiPoligono extends Poligono { }
```

**Interfaces selladas:**
```java
public sealed interface Transporte
    permits Coche, Bicicleta, Tren {
    void mover();
}

public final class Coche implements Transporte {
    @Override
    public void mover() { }
}

public final class Bicicleta implements Transporte {
    @Override
    public void mover() { }
}

public non-sealed class Tren implements Transporte {
    @Override
    public void mover() { }
}
```

**¿Para qué sirven?**

**1. Domain modeling preciso:**
```java
public sealed interface Resultado<T>
    permits Exito, Error {
}

public record Exito<T>(T valor) implements Resultado<T> { }
public record Error<T>(String mensaje) implements Resultado<T> { }

// Ahora puedes estar seguro de que solo hay 2 tipos de Resultado
public <T> void procesar(Resultado<T> resultado) {
    switch (resultado) {
        case Exito<T> exito -> System.out.println(exito.valor());
        case Error<T> error -> System.err.println(error.mensaje());
        // No necesitas default, el compilador sabe que son los únicos casos
    }
}
```

**2. Pattern Matching exhaustivo (Java 17+):**
```java
public sealed interface Pago
    permits PagoTarjeta, PagoPayPal, PagoCripto {
}

public record PagoTarjeta(String numero) implements Pago { }
public record PagoPayPal(String email) implements Pago { }
public record PagoCripto(String wallet) implements Pago { }

public void procesarPago(Pago pago) {
    // ✅ Pattern matching exhaustivo sin default
    String resultado = switch (pago) {
        case PagoTarjeta t -> "Tarjeta: " + t.numero();
        case PagoPayPal p -> "PayPal: " + p.email();
        case PagoCripto c -> "Cripto: " + c.wallet();
        // No necesita default, el compilador verifica todos los casos
    };
    
    System.out.println(resultado);
}
```

**3. Algebraic Data Types (ADTs):**
```java
// AST (Abstract Syntax Tree) de expresiones
public sealed interface Expresion
    permits Numero, Suma, Multiplicacion {
}

public record Numero(int valor) implements Expresion { }
public record Suma(Expresion izq, Expresion der) implements Expresion { }
public record Multiplicacion(Expresion izq, Expresion der) implements Expresion { }

// Evaluador type-safe
public int evaluar(Expresion expr) {
    return switch (expr) {
        case Numero n -> n.valor();
        case Suma s -> evaluar(s.izq()) + evaluar(s.der());
        case Multiplicacion m -> evaluar(m.izq()) * evaluar(m.der());
    };
}

// Uso
Expresion expr = new Suma(
    new Numero(10),
    new Multiplicacion(new Numero(5), new Numero(2))
);
int resultado = evaluar(expr);  // 10 + (5 * 2) = 20
```

**4. API design segura:**
```java
public sealed interface HttpResponse
    permits OkResponse, ErrorResponse, NotFoundResponse {
}

public record OkResponse(String body) implements HttpResponse { }
public record ErrorResponse(int codigo, String mensaje) implements HttpResponse { }
public record NotFoundResponse() implements HttpResponse { }

// Los usuarios de tu API saben exactamente qué esperar
public void manejarRespuesta(HttpResponse response) {
    switch (response) {
        case OkResponse ok -> procesarExito(ok.body());
        case ErrorResponse err -> manejarError(err.codigo(), err.mensaje());
        case NotFoundResponse nf -> mostrar404();
    }
}
```

**Ventajas:**

1. **Type Safety**: El compilador verifica exhaustividad
2. **Mejor modelado**: Jerarquías cerradas y controladas
3. **Refactoring seguro**: Si agregas un subtipo, el compilador te avisa dónde actualizar
4. **Documentación**: Los desarrolladores ven todas las variantes posibles

**Comparación con enums:**

```java
// Enum: Tipos con instancias fijas
public enum DiaSemana {
    LUNES, MARTES, MIERCOLES  // Instancias fijas
}

// Sealed: Tipos con subclases fijas, cada instancia puede tener estado
public sealed interface Animal permits Perro, Gato { }
public record Perro(String nombre, String raza) implements Animal { }
public record Gato(String nombre, int vidas) implements Animal { }
// Puedes crear infinitas instancias de Perro/Gato
```

**Best Practices:**
- Combina con `record` para clases de datos inmutables
- Usa para modelar uniones de tipos (Either, Result, Option)
- Ideal para ASTs, estados de máquina, eventos
- Las subclases deben estar en el mismo paquete (o submódulo)
- Considera si un `enum` simple es suficiente antes de usar sealed

---

#### 42. ¿Qué es Pattern Matching en Java?

**Respuesta:**

**Pattern Matching** (Java 16+) simplifica la verificación de tipos y casting, haciendo el código más conciso y seguro. Evoluciona el tradicional `instanceof` + cast manual.

**Pattern Matching para instanceof (Java 16+):**

```java
// ❌ Forma tradicional (Java <16)
if (obj instanceof String) {
    String str = (String) obj;  // Cast manual redundante
    System.out.println(str.length());
}

// ✅ Pattern matching (Java 16+)
if (obj instanceof String str) {
    System.out.println(str.length());  // str ya está casted
}
```

**Scope de la variable:**
```java
if (obj instanceof String str) {
    System.out.println(str.toUpperCase());  // ✅ str disponible aquí
} else {
    // str no disponible aquí
}

// Con &&
if (obj instanceof String str && str.length() > 5) {
    System.out.println(str);  // ✅ str disponible
}

// Con ||
if (obj instanceof String str || str.length() > 5) {  // ❌ Error
    // str no está garantizado aquí
}
```

**Ejemplo práctico:**
```java
public String describir(Object obj) {
    if (obj instanceof Integer num) {
        return "Entero: " + num;
    } else if (obj instanceof String str && !str.isEmpty()) {
        return "Texto: " + str.toUpperCase();
    } else if (obj instanceof List<?> lista && lista.size() > 0) {
        return "Lista de " + lista.size() + " elementos";
    } else {
        return "Desconocido";
    }
}
```

**Pattern Matching en Switch (Java 17+, Java 21 estable):**

```java
// ❌ Forma tradicional
public String clasificar(Object obj) {
    if (obj instanceof Integer) {
        return "Es un número";
    } else if (obj instanceof String) {
        return "Es texto";
    } else if (obj instanceof List) {
        return "Es una lista";
    } else {
        return "Desconocido";
    }
}

// ✅ Switch con pattern matching (Java 17+)
public String clasificar(Object obj) {
    return switch (obj) {
        case Integer num -> "Número: " + num;
        case String str -> "Texto: " + str.toUpperCase();
        case List<?> lista -> "Lista de " + lista.size() + " elementos";
        case null -> "Null";
        default -> "Desconocido";
    };
}
```

**Guarded Patterns (with when):**

```java
public String procesar(Object obj) {
    return switch (obj) {
        case Integer num when num > 0 -> "Positivo: " + num;
        case Integer num when num < 0 -> "Negativo: " + num;
        case Integer num -> "Cero";
        case String str when str.length() > 10 -> "Texto largo: " + str;
        case String str -> "Texto corto: " + str;
        case null -> "Null";
        default -> "Otro tipo";
    };
}
```

**Pattern Matching con Sealed Classes:**

```java
public sealed interface Forma
    permits Circulo, Rectangulo, Triangulo { }

public record Circulo(double radio) implements Forma { }
public record Rectangulo(double ancho, double alto) implements Forma { }
public record Triangulo(double base, double altura) implements Forma { }

// ✅ Exhaustivo sin default
public double calcularArea(Forma forma) {
    return switch (forma) {
        case Circulo c -> Math.PI * c.radio() * c.radio();
        case Rectangulo r -> r.ancho() * r.alto();
        case Triangulo t -> (t.base() * t.altura()) / 2;
        // No necesita default, el compilador sabe que son todos los casos
    };
}
```

**Nested Pattern Matching (Java 19+ preview, Java 21):**

```java
public record Punto(int x, int y) { }
public record Circulo(Punto centro, double radio) { }

// Patrón anidado
public String describir(Object obj) {
    return switch (obj) {
        case Circulo(Punto(int x, int y), double r) ->
            "Círculo en (" + x + "," + y + ") con radio " + r;
        default -> "No es un círculo";
    };
}
```

**Record Patterns (Java 19+ preview, Java 21):**

```java
public record Usuario(String nombre, int edad) { }

public String procesar(Object obj) {
    return switch (obj) {
        case Usuario(String nombre, int edad) when edad >= 18 ->
            nombre + " es mayor de edad";
        case Usuario(String nombre, int edad) ->
            nombre + " es menor de edad";
        default -> "No es usuario";
    };
}

// Uso
procesar(new Usuario("Juan", 25));  // "Juan es mayor de edad"
```

**Ejemplo completo - Evaluador de expresiones:**

```java
public sealed interface Expr { }
public record Numero(int valor) implements Expr { }
public record Suma(Expr izq, Expr der) implements Expr { }
public record Mult(Expr izq, Expr der) implements Expr { }

public int evaluar(Expr expr) {
    return switch (expr) {
        case Numero(int valor) -> valor;
        case Suma(Expr izq, Expr der) -> evaluar(izq) + evaluar(der);
        case Mult(Expr izq, Expr der) -> evaluar(izq) * evaluar(der);
    };
}

// Uso
Expr expr = new Suma(
    new Numero(10),
    new Mult(new Numero(5), new Numero(2))
);
int resultado = evaluar(expr);  // 20
```

**Ejemplo práctico - HTTP Handler:**

```java
public sealed interface HttpRequest
    permits GetRequest, PostRequest, PutRequest { }

public record GetRequest(String path, Map<String, String> params) 
    implements HttpRequest { }
    
public record PostRequest(String path, String body) 
    implements HttpRequest { }
    
public record PutRequest(String path, String body, String contentType) 
    implements HttpRequest { }

public Response manejar(HttpRequest request) {
    return switch (request) {
        case GetRequest(String path, var params) ->
            handleGet(path, params);
            
        case PostRequest(String path, String body) 
            when path.startsWith("/api/") ->
            handleApiPost(path, body);
            
        case PostRequest(String path, String body) ->
            handlePost(path, body);
            
        case PutRequest(String path, String body, String ct) 
            when "application/json".equals(ct) ->
            handleJsonPut(path, body);
            
        case PutRequest(String path, String body, String ct) ->
            handlePut(path, body, ct);
    };
}
```

**Best Practices:**
- Usa con sealed classes para exhaustividad
- Combina con records para destructuring limpio
- Usa guarded patterns (`when`) para condiciones adicionales
- Prefiere switch expressions sobre statements
- Evita lógica compleja en los patterns

---

#### 50. ¿Qué son los Text Blocks (Java 15+)?

**Respuesta:**

**Text Blocks** son literales de String multilínea que simplifican el trabajo con texto formateado (JSON, SQL, HTML, etc.) sin necesidad de concatenación o escapes complicados.

**Sintaxis:**
```java
// Delimitado por triple comilla """
String textBlock = """
    Contenido del text block
    con múltiples líneas
    """;
```

**Comparación con Strings tradicionales:**

```java
// ❌ Forma tradicional - Difícil de leer y mantener
String json = "{\n" +
              "  \"nombre\": \"Juan\",\n" +
              "  \"edad\": 30,\n" +
              "  \"ciudad\": \"Madrid\"\n" +
              "}";

// ✅ Text Block - Legible y mantenible
String json = """
    {
      "nombre": "Juan",
      "edad": 30,
      "ciudad": "Madrid"
    }
    """;
```

**Características:**

**1. Indentación automática:**
```java
public void ejemplo() {
    // La indentación se calcula desde la """ de cierre
    String sql = """
        SELECT id, nombre, email
        FROM usuarios
        WHERE edad > 18
          AND ciudad = 'Madrid'
        ORDER BY nombre
        """;
    // El espacio común se elimina automáticamente
}
```

**2. Escapes simplificados:**
```java
// ❌ Tradicional con muchos escapes
String path = "C:\\Users\\Juan\\Documents\\file.txt";

// ✅ Text block (\ sigue siendo necesario en Windows paths)
String html = """
    <html>
        <body>
            <h1>Título</h1>
            <p class="texto">Contenido con "comillas"</p>
        </body>
    </html>
    """;
// No necesitas escapar " dentro del text block
```

**3. Control de nueva línea final:**
```java
// Sin nueva línea al final
String sinNuevaLinea = """
    Línea 1
    Línea 2\
    """;  // El \ al final elimina la nueva línea

// Con nueva línea al final (default)
String conNuevaLinea = """
    Línea 1
    Línea 2
    """;
```

**4. Concatenación:**
```java
String nombre = "Juan";
int edad = 30;

// Con formatted() o format()
String mensaje = """
    Hola %s,
    Tienes %d años
    """.formatted(nombre, edad);

// Con String.format()
String mensaje = String.format("""
    Hola %s,
    Tienes %d años
    """, nombre, edad);
```

**Ejemplos prácticos:**

**SQL Queries:**
```java
public List<Usuario> buscarUsuarios(String ciudad, int edadMinima) {
    String sql = """
        SELECT 
            u.id,
            u.nombre,
            u.email,
            u.fecha_registro
        FROM usuarios u
        WHERE u.ciudad = ?
          AND u.edad >= ?
          AND u.activo = true
        ORDER BY u.nombre ASC
        """;
    
    return jdbcTemplate.query(sql, 
        new Object[]{ciudad, edadMinima},
        new UsuarioRowMapper());
}
```

**JSON:**
```java
public String crearJsonUsuario(Usuario usuario) {
    return """
        {
          "id": %d,
          "nombre": "%s",
          "email": "%s",
          "roles": ["USER", "ADMIN"],
          "activo": %b,
          "metadata": {
            "createdAt": "%s",
            "lastLogin": "%s"
          }
        }
        """.formatted(
            usuario.getId(),
            usuario.getNombre(),
            usuario.getEmail(),
            usuario.isActivo(),
            usuario.getCreatedAt(),
            usuario.getLastLogin()
        );
}
```

**HTML Templates:**
```java
public String generarPaginaHTML(String titulo, String contenido) {
    return """
        <!DOCTYPE html>
        <html lang="es">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>%s</title>
            <style>
                body {
                    font-family: Arial, sans-serif;
                    margin: 20px;
                }
                .content {
                    background-color: #f0f0f0;
                    padding: 15px;
                    border-radius: 5px;
                }
            </style>
        </head>
        <body>
            <h1>%s</h1>
            <div class="content">
                %s
            </div>
        </body>
        </html>
        """.formatted(titulo, titulo, contenido);
}
```

**Logs estructurados:**
```java
public void logError(Exception e, String contexto) {
    String logMessage = """
        ═══════════════════════════════════════
        ERROR DETECTADO
        ═══════════════════════════════════════
        Contexto: %s
        Excepción: %s
        Mensaje: %s
        Stack Trace:
        %s
        ═══════════════════════════════════════
        """.formatted(
            contexto,
            e.getClass().getName(),
            e.getMessage(),
            Arrays.toString(e.getStackTrace())
        );
    
    logger.error(logMessage);
}
```

**Regex patterns complejos:**
```java
// Más legible que una sola línea
String emailPattern = """
    ^[a-zA-Z0-9._%+-]+
    @[a-zA-Z0-9.-]+
    \\.[a-zA-Z]{2,6}$
    """.replaceAll("\\s+", "");  // Eliminar espacios/newlines

Pattern pattern = Pattern.compile(emailPattern);
```

**Tests con datos de ejemplo:**
```java
@Test
void testParserJSON() {
    String jsonInput = """
        {
          "usuarios": [
            {
              "id": 1,
              "nombre": "Juan",
              "roles": ["ADMIN", "USER"]
            },
            {
              "id": 2,
              "nombre": "Ana",
              "roles": ["USER"]
            }
          ]
        }
        """;
    
    List<Usuario> usuarios = jsonParser.parse(jsonInput);
    assertEquals(2, usuarios.size());
}
```

**Best Practices:**
- Usa para SQL, JSON, HTML, XML, YAML
- Usa `.formatted()` o `String.format()` para interpolación
- Ten cuidado con la indentación (la """ de cierre determina el nivel)
- Combina con `stripIndent()` si necesitas control fino
- No uses para strings cortos de una línea
- Para JSON complejo, considera librerías (Jackson, Gson)

---

#### 44. ¿Qué son los Virtual Threads (Java 21+)?

**Respuesta:**

**Virtual Threads** (Project Loom) son threads **livianos** gestionados por la JVM (no por el OS) que permiten ejecutar millones de tareas concurrentes sin el overhead de threads tradicionales.

**Problema con threads tradicionales (Platform Threads):**

```java
// ❌ Platform threads son caros
// 1 thread del OS ≈ 1-2 MB de memoria
// Máximo ~10,000 threads antes de problemas de performance

ExecutorService executor = Executors.newFixedThreadPool(10);
for (int i = 0; i < 1_000_000; i++) {
    executor.submit(() -> {
        // Tarea que hace I/O (espera)
        llamadaHTTP();
    });
}
// ❌ Solo 10 threads procesan 1M de tareas → muy lento
```

**Solución con Virtual Threads:**

```java
// ✅ Virtual threads son livianos
// 1 virtual thread ≈ pocos KB de memoria
// Puedes crear millones sin problema

try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 1_000_000; i++) {
        executor.submit(() -> {
            // Tarea que hace I/O
            llamadaHTTP();
        });
    }
}
// ✅ 1M de virtual threads ejecutan concurrentemente
```

**¿Cómo funcionan?**

- Virtual threads **no** están vinculados a un thread del OS 1:1
- La JVM **multiplexa** muchos virtual threads en pocos platform threads (carriers)
- Cuando un virtual thread se **bloquea** (I/O, sleep), la JVM lo "desmonta" del platform thread
- Otro virtual thread puede usar ese platform thread
- Ideal para workloads **I/O-bound** (HTTP, BD, archivos)

**Crear Virtual Threads:**

```java
// 1. Usando Thread.startVirtualThread()
Thread vt = Thread.startVirtualThread(() -> {
    System.out.println("Virtual thread: " + Thread.currentThread());
});
vt.join();

// 2. Usando Thread.ofVirtual()
Thread vt = Thread.ofVirtual().start(() -> {
    System.out.println("Tarea");
});

// 3. Usando ExecutorService (recomendado)
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> {
        System.out.println("Tarea en virtual thread");
    });
}

// 4. Usando ThreadFactory
ThreadFactory factory = Thread.ofVirtual().factory();
Thread vt = factory.newThread(() -> {
    System.out.println("Tarea");
});
vt.start();
```

**Ejemplo práctico - Servidor HTTP:**

```java
// ❌ Platform threads - Limitado por thread pool
@Bean
public TomcatProtocolHandlerCustomizer<?> protocolHandlerPlatform() {
    return protocolHandler -> {
        protocolHandler.setExecutor(Executors.newFixedThreadPool(200));
        // Máximo 200 requests concurrentes
    };
}

// ✅ Virtual threads - Sin límite práctico
@Bean
public TomcatProtocolHandlerCustomizer<?> protocolHandlerVirtual() {
    return protocolHandler -> {
        protocolHandler.setExecutor(Executors.newVirtualThreadPerTaskExecutor());
        // Millones de requests concurrentes posibles
    };
}
```

**Ejemplo - Llamadas API paralelas:**

```java
public class ServicioExterno {
    public String llamarAPI(int id) throws Exception {
        // Simular llamada HTTP que tarda 1 segundo
        Thread.sleep(1000);
        return "Resultado " + id;
    }
}

// ❌ Secuencial - Tarda 10 segundos
public List<String> obtenerDatosSecuencial() throws Exception {
    List<String> resultados = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        resultados.add(llamarAPI(i));
    }
    return resultados;  // 10 segundos
}

// ✅ Paralelo con Virtual Threads - Tarda ~1 segundo
public List<String> obtenerDatosParalelo() throws Exception {
    try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
        List<Future<String>> futures = new ArrayList<>();
        
        for (int i = 0; i < 10; i++) {
            int id = i;
            futures.add(executor.submit(() -> llamarAPI(id)));
        }
        
        return futures.stream()
            .map(f -> {
                try {
                    return f.get();
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            })
            .toList();
    }  // ~1 segundo (todas en paralelo)
}
```

**Structured Concurrency (Java 21+ preview):**

```java
import java.util.concurrent.StructuredTaskScope;

public String obtenerDatosUsuario(Long id) throws Exception {
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        // Lanzar tareas concurrentes
        Future<Usuario> futuroUsuario = scope.fork(() -> obtenerUsuario(id));
        Future<List<Pedido>> futuroPedidos = scope.fork(() -> obtenerPedidos(id));
        Future<Perfil> futuroPerfil = scope.fork(() -> obtenerPerfil(id));
        
        // Esperar que todas completen
        scope.join();           // Espera todas
        scope.throwIfFailed();  // Lanza si alguna falló
        
        // Obtener resultados
        Usuario usuario = futuroUsuario.resultNow();
        List<Pedido> pedidos = futuroPedidos.resultNow();
        Perfil perfil = futuroPerfil.resultNow();
        
        return combinar(usuario, pedidos, perfil);
    }  // Scope se cierra automáticamente, cancelando tareas pendientes
}
```

**Scoped Values (reemplazo de ThreadLocal):**

```java
// ❌ ThreadLocal con Virtual Threads puede causar problemas de memoria
private static final ThreadLocal<String> userId = new ThreadLocal<>();

// ✅ ScopedValue - Optimizado para Virtual Threads
private static final ScopedValue<String> userId = ScopedValue.newInstance();

public void procesar(String user) {
    ScopedValue.where(userId, user).run(() -> {
        // userId disponible aquí
        System.out.println("Usuario: " + userId.get());
        
        metodoAnidado();  // También tiene acceso
    });
    // userId ya no disponible aquí
}

public void metodoAnidado() {
    System.out.println("Usuario en método anidado: " + userId.get());
}
```

**¿Cuándo usar Virtual Threads?**

✅ **Ideal para:**
- Aplicaciones I/O-bound (HTTP, BD, archivos)
- Microservicios con muchas llamadas externas
- Servidores web/API con alta concurrencia
- Procesamiento de streams/colas con esperas

❌ **NO usar para:**
- Tareas CPU-intensive (usa ForkJoin o parallel streams)
- Código con synchronized (puede causar pinning)
- Cuando ya usas programación reactiva (WebFlux)

**Pitfalls y consideraciones:**

**1. Thread Pinning (el virtual thread no puede desmontarse):**
```java
// ❌ synchronized puede causar pinning
synchronized (lock) {
    llamadaIO();  // Virtual thread queda "pinned" durante el I/O
}

// ✅ Usa ReentrantLock en su lugar
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    llamadaIO();  // Virtual thread puede desmontarse durante I/O
} finally {
    lock.unlock();
}
```

**2. ThreadLocal puede consumir memoria:**
```java
// ❌ ThreadLocal con millones de virtual threads
private static final ThreadLocal<LargeObject> data = new ThreadLocal<>();

// ✅ Usa ScopedValue
private static final ScopedValue<LargeObject> data = ScopedValue.newInstance();
```

**3. Pooling innecesario:**
```java
// ❌ NO uses thread pools con virtual threads
ExecutorService executor = Executors.newFixedThreadPool(100);
// Derrotas el propósito de virtual threads

// ✅ Crea virtual threads bajo demanda
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
```

**Comparación:**

| Aspecto | Platform Threads | Virtual Threads |
|---------|-----------------|-----------------|
| **Memoria** | ~1-2 MB cada uno | ~pocos KB cada uno |
| **Cantidad** | ~10,000 máx | Millones |
| **Creación** | Cara (~1ms) | Barata (~1μs) |
| **Ideal para** | CPU-intensive | I/O-intensive |
| **Bloqueante** | Desperdicia thread | Thread se desmonta |

**Best Practices:**
- Usa `Executors.newVirtualThreadPerTaskExecutor()`
- Evita synchronized, usa `ReentrantLock`
- Evita ThreadLocal, usa `ScopedValue`
- No hagas pooling de virtual threads
- Usa para I/O, no para CPU
- Combina con Structured Concurrency para código más limpio

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

[🏠 Volver al Inicio](./README.md) | [Siguiente: Spring Boot ➡️](./02-spring-boot.md)
