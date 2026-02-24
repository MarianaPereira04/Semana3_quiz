# FASE 1 - Levantamiento del ambiente
![Texto alternativo](./imagenes/Levantamiento.png)

Levantamiento exitoso

---

# FASE 2 - Auditoria del código

Ahora a partir de los distintos archivos del código se muestra a continuación los hallazgos de problemas encontrados teniendo en cuenta principios Clean Code, SOLID y Seguridad Básica.

## Tabla de Hallazgos — Auditoría Clean Code y Seguridad

| # | Descripción del problema | Archivo | Línea aprox. | Principio violado | Riesgo |
|---|--------------------------|---------|--------------|------------------|--------|
| 1 | Construcción de consulta SQL mediante concatenación de strings (posible SQL Injection) | UserRepository.java | ~20 | Seguridad básica (SQL Injection) | Alto |
| 2 | Inserción SQL con concatenación directa de datos del usuario | UserRepository.java | ~34 | Seguridad básica (SQL Injection) | Alto |
| 3 | Uso de MD5 para hashing de contraseñas (algoritmo inseguro y obsoleto) | AuthService.java | ~63 | Seguridad básica (hashing débil) | Alto |
| 4 | Exposición del hash de la contraseña en la respuesta del login | AuthService.java | ~28 y ~35 | Principio de mínima exposición de datos | Alto |
| 5 | Credenciales de base de datos hardcodeadas en el repositorio | UserRepository.java | ~12-14 | Seguridad / Clean Code | Alto |
| 6 | Atributos públicos en la entidad User (violación de encapsulamiento) | User.java | ~4-6 | Clean Code / OOP | Medio |
| 7 | Controlador usa nombres de parámetros poco descriptivos (u, p, e) | AuthController.java | ~20 y ~27 | Naming (Clean Code) | Bajo |
| 8 | Falta de separación adecuada de responsabilidades en acceso a datos (uso directo de DriverManager) | UserRepository.java | ~17 | SRP / DIP (SOLID) | Medio |
| 9 | No se cierran conexiones, Statements ni ResultSet (posible fuga de recursos) | UserRepository.java | ~16-29 | Buenas prácticas / manejo de recursos | Medio |
| 10 | Validación de contraseña extremadamente débil (solo longitud > 3) | AuthService.java | ~44 | Seguridad básica | Medio |


---


# FASE 3 — Pruebas Funcionales

Se realizaron pruebas manuales utilizando Postman contra la API levantada en localhost:8080.

---

## 🧪 Prueba 1 — Login válido

**Petición:**

POST http://localhost:8080/login?u=admin&p=12345

**Respuesta:**
Status: 200 OK

{
  "ok": true,
  "user": "admin",
  "hash": "827ccb0eea8a706c4c34a16891f84e7b"
}

### 🔎 Análisis

- La autenticación fue exitosa.
- Se retorna el campo "hash" correspondiente al MD5 de la contraseña.
- El servidor devuelve información sensible al cliente.

### ⚠ Problema detectado

El hash de la contraseña no debería enviarse en la respuesta.

Aunque esté cifrada con MD5, sigue siendo información sensible que puede:
- Facilitar ataques de fuerza bruta
- Permitir ataques de rainbow tables
- Exponer lógica interna del sistema

### ✅ Conclusión

La autenticación funciona, pero existe una vulnerabilidad de exposición de información sensible.

Riesgo: Alto

---

## 🧪 Prueba 2 — SQL Injection

**Petición:**

POST http://localhost:8080/login?u=admin'--&p=cualquiercosa

**Respuesta:**
Status: 200 OK

{
  "ok": false,
  "hash": "f73862908453012d17eb1d60240d95d1"
}

### 🔎 Análisis

- El sistema no permitió el acceso.
- Sin embargo, la consulta SQL sigue siendo vulnerable.
- El parámetro se concatena directamente en la consulta.

Aunque en este caso específico no logró autenticarse, el sistema sigue siendo vulnerable a ataques de inyección SQL más elaborados.

En producción esto podría permitir:
- Bypass de autenticación
- Manipulación de consultas
- Exposición o modificación de datos

### ⚠ Problema detectado

La construcción de consultas mediante concatenación de strings es una vulnerabilidad crítica.

### ✅ Conclusión

Existe una vulnerabilidad potencial de SQL Injection aunque no se haya explotado exitosamente en esta prueba.

Riesgo: Alto

---

## 🧪 Prueba 3 — Registro con contraseña débil

### Primer intento:

POST /register?u=test&p=123&e=test@test.com

Respuesta:
Status: 200 OK

{
  "ok": false
}

### Segundo intento:

POST /register?u=test&p=1234&e=test@test.com

Respuesta:
Status: 200 OK

{
  "ok": true,
  "user": "test"
}

### 🔎 Análisis

- La contraseña "123" fue rechazada por tener longitud menor a 4.
- La contraseña "1234" fue aceptada.
- La única validación implementada es p.length() > 3.

### ⚠ Problema detectado

La validación es extremadamente débil.  
Una contraseña de 4 caracteres es insegura y vulnerable a ataques de fuerza bruta.

No se validan:
- Complejidad
- Longitud mínima adecuada
- Caracteres especiales
- Mayúsculas/minúsculas

### ✅ Conclusión

El sistema permite contraseñas inseguras, lo que representa un riesgo de seguridad medio.

Riesgo: Medio