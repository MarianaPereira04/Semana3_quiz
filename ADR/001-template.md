# ADR-001: Refactorización del módulo de autenticación por vulnerabilidades críticas de seguridad y violaciones a principios SOLID

## Contexto

El sistema actual implementa un módulo básico de autenticación compuesto por un controlador REST, un servicio de autenticación y un repositorio que accede directamente a una base de datos PostgreSQL. Su funcionalidad principal es permitir el registro y login de usuarios mediante consultas SQL construidas manualmente.

Durante la auditoría de código (FASE 2) se identificaron múltiples problemas críticos, entre ellos: vulnerabilidad a SQL Injection por concatenación de strings en consultas, uso de MD5 como algoritmo de hashing inseguro, exposición del hash de contraseña en las respuestas del login, credenciales de base de datos hardcodeadas en el código fuente y ausencia de cierre de conexiones JDBC. Adicionalmente, se detectaron violaciones a principios Clean Code y SOLID como falta de encapsulamiento en la entidad User, bajo nivel de abstracción en el acceso a datos y nombres poco descriptivos en los parámetros.

Las pruebas funcionales (FASE 3) confirmaron que el sistema expone información sensible y que la validación de contraseñas es insuficiente, permitiendo credenciales extremadamente débiles. Aunque el sistema funciona técnicamente, su estado actual representa un riesgo alto en un entorno productivo.

Esta situación afecta directamente la seguridad de los usuarios, la integridad de los datos y la reputación del sistema, además de generar deuda técnica que impacta al equipo de desarrollo en mantenibilidad y escalabilidad futura.

---

## Decisión

Se decide realizar una refactorización estructural del módulo de autenticación aplicando principios de seguridad, Clean Code y SOLID.

1. Se reemplazará el uso de `Statement` y concatenación de strings por `PreparedStatement` o preferiblemente `Spring Data JPA`, eliminando la vulnerabilidad a SQL Injection y aplicando el principio de separación de responsabilidades (SRP) y bajo acoplamiento (DIP).

2. Se sustituirá el algoritmo MD5 por un algoritmo de hashing seguro como BCrypt, el cual incorpora salting automático y es resistente a ataques de fuerza bruta, alineándose con buenas prácticas modernas de seguridad.

3. Se eliminará cualquier exposición de información sensible en las respuestas del API. El endpoint de login devolverá únicamente un indicador de éxito y, en una evolución futura, un token de autenticación (por ejemplo JWT), aplicando el principio de mínima exposición de datos.

4. Se eliminarán credenciales hardcodeadas del repositorio y se centralizará la configuración en `application.properties` o variables de entorno, mejorando la seguridad y la portabilidad del sistema.

5. Se refactorizará la entidad `User` aplicando encapsulamiento (atributos privados con getters/setters) y se mejorarán los nombres de variables y métodos para cumplir con estándares de legibilidad y Clean Code.

Después de estos cambios, la arquitectura del módulo quedará organizada en capas claramente definidas: Controller → Service → Repository (abstraído por Spring), con responsabilidades bien delimitadas y menor acoplamiento a implementaciones concretas de infraestructura.

---

## Consecuencias

### Consecuencias positivas

- Eliminación de vulnerabilidades críticas como SQL Injection.
- Mejora significativa en la seguridad de almacenamiento de contraseñas.
- Mayor mantenibilidad y legibilidad del código.
- Mejor alineación con principios SOLID.
- Reducción de deuda técnica.
- Preparación del sistema para escalar hacia autenticación basada en tokens.

### Consecuencias negativas / riesgos

- Incremento temporal en el esfuerzo de desarrollo por la refactorización.
- Posibles regresiones si no se realizan pruebas adecuadas.
- Curva de aprendizaje si el equipo no está familiarizado con Spring Data JPA o BCrypt.
- Necesidad de migrar contraseñas existentes a un nuevo esquema de hashing.

---

## Alternativas consideradas

1. Corregir únicamente los problemas de seguridad sin refactorizar la estructura.

   Se consideró reemplazar MD5 y usar PreparedStatement sin modificar la arquitectura general. Esta opción fue descartada porque mantendría problemas de diseño, acoplamiento y deuda técnica que afectarían la mantenibilidad futura.

2. Reescribir completamente el módulo desde cero.

   Se evaluó reconstruir el sistema de autenticación utilizando un framework de seguridad completo como Spring Security. Sin embargo, esta alternativa fue descartada por el alcance del proyecto y el tiempo disponible, ya que implicaría una reestructuración más amplia de la aplicación.

3. Mantener el sistema actual al ser un entorno académico.

   Aunque el sistema cumple su función en un entorno controlado, se descartó esta alternativa debido a que el objetivo del ejercicio es identificar riesgos reales y proponer mejoras arquitectónicas fundamentadas.

---
