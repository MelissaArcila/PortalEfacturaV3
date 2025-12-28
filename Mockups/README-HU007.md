# HU007 - Contraseña Temporal y Primer Cambio Obligatorio
## Documentación de Mockups Interactivos

### Información General

| Campo | Valor |
|-------|-------|
| **Historia de Usuario** | HU007 - Contraseña Temporal y Primer Cambio Obligatorio |
| **Fecha de Creación** | 2024 |
| **Archivos Generados** | 3 mockups HTML interactivos |
| **Tecnologías** | HTML5, CSS3, JavaScript (Vanilla) |
| **Diseño Base** | Material UI + Guía de Estilos CDN Facturación |

---

## Archivos Generados

### 1. HU007-CambioContraseñaObligatorio.html
**Propósito**: Pantalla de cambio forzado de contraseña temporal en el primer inicio de sesión.

**Características Principales**:
- Cambio de contraseña obligatorio (sin opción de cancelar)
- Validación en tiempo real de requisitos de seguridad
- Indicadores visuales de fortaleza de contraseña
- Toggle de visibilidad en campos de contraseña
- 6 estados demo para pruebas

**Elementos de la Interfaz**:

#### Logo Section
- Fondo con gradiente Primary → Secondary
- Logo de CDN Facturación centrado
- Establece contexto visual de la aplicación

#### Título y Mensaje Informativo
- **Título**: "Cambio de Contraseña Requerido"
- **Alert Info** (azul): Explica que debe establecer contraseña definitiva
- Mensaje claro y tranquilizador

#### Campo "Nueva Contraseña"
- Input tipo password con toggle de visibilidad
- Icono de ojo para mostrar/ocultar
- Validación en tiempo real al escribir

**Lista de Requisitos** (actualización en tiempo real):
1. ✗/✓ Mínimo 8 caracteres
2. ✗/✓ Al menos una mayúscula (A-Z)
3. ✗/✓ Al menos una minúscula (a-z)
4. ✗/✓ Al menos un número (0-9)
5. ✗/✓ Al menos un símbolo (!@#$%^&*)
6. ✗/✓ No puede ser igual a contraseña temporal

- Cruces rojas (✗) cuando no cumple
- Checkmarks verdes (✓) cuando cumple
- Colores dinámicos en iconos y texto

**Barra de Fortaleza**:
- **Débil** (rojo): 1-3 requisitos cumplidos, 33% de ancho
- **Media** (naranja): 4-5 requisitos cumplidos, 66% de ancho
- **Fuerte** (verde): 6 requisitos cumplidos, 100% de ancho
- Animación suave de transición

#### Campo "Confirmar Nueva Contraseña"
- Input tipo password con toggle de visibilidad
- Validación de coincidencia en tiempo real
- Mensaje de error si no coincide: "Las contraseñas no coinciden"

#### Botón "Cambiar Contraseña"
- Gradiente Primary → Secondary
- Deshabilitado hasta cumplir TODOS los requisitos:
  - 6 requisitos de contraseña válidos
  - Confirmación coincide
  - Confirmación no está vacía
- Icono de lock_reset

#### Nota de Seguridad
- Box naranja informativo
- Mensaje: "Esta acción es obligatoria. No podrá acceder al portal sin establecer una contraseña segura."
- Icono de security

#### Alert de Éxito (oculto por defecto)
- Background verde
- Mensaje: "Contraseña cambiada exitosamente. Redirigiendo al portal..."
- Se muestra después de cambio exitoso
- Auto-redirige después de 2 segundos

**Estados del Demo**:

1. **Estado Inicial**: Formulario limpio
2. **Contraseña Débil**: "abc123" - solo 2-3 requisitos
3. **Contraseña Media**: "Abc123" - 4-5 requisitos
4. **Contraseña Fuerte**: "SecureP@ss123" - todos los requisitos
5. **No Coinciden**: Contraseñas diferentes en confirmación
6. **Cambio Exitoso**: Muestra alert de éxito

**Lógica de Validación**:

```javascript
const requirements = {
    length: (pwd) => pwd.length >= 8,
    uppercase: (pwd) => /[A-Z]/.test(pwd),
    lowercase: (pwd) => /[a-z]/.test(pwd),
    number: (pwd) => /[0-9]/.test(pwd),
    symbol: (pwd) => /[!@#$%^&*]/.test(pwd),
    notTemp: (pwd) => pwd !== TEMP_PASSWORD
};
```

**Cálculo de Fortaleza**:
- Cuenta requisitos válidos
- 0-3 requisitos = Débil (rojo)
- 4-5 requisitos = Media (naranja)
- 6 requisitos = Fuerte (verde)

**Validación Adicional**:
- Lista negra de contraseñas comunes (Password1!, Qwerty123!, etc.)
- Alert si intenta usar contraseña común
- Mensaje: "Esta contraseña es muy común. Por favor, elija una contraseña más segura y única."

**Flujo Post-Cambio**:
1. Mostrar alert de éxito
2. Deshabilitar formulario
3. Esperar 2 segundos
4. Redirigir según caso:
   - Si tiene múltiples clientes → HU001 Selección de Cliente
   - Si tiene un solo cliente → HU001 Ingreso Directo

---

### 2. HU007-EmailContraseñaTemporal.html
**Propósito**: Plantilla de correo electrónico HTML para envío de contraseña temporal.

**Características Principales**:
- Diseño responsive para todos los clientes de correo
- Estilos inline para compatibilidad
- Branding corporativo con gradientes
- Información clara y estructurada
- Instrucciones paso a paso
- Advertencias de seguridad destacadas

**Estructura del Correo**:

#### Header
- Gradiente Primary → Secondary
- Logo de CDN Facturación centrado
- Padding generoso para presencia visual

#### Saludo Personalizado
- "Hola {nombre_usuario},"
- Color Primary para nombre
- Tono cálido y profesional

#### Introducción
- Bienvenida al Portal Unificado
- Confirmación de creación de cuenta
- Texto claro y conciso

#### Box de Credenciales (Destacado)
- Fondo gris claro con borde
- Border-radius para suavidad
- **Usuario**: Número de identificación
- **Contraseña Temporal**:
  - Fuente monospace (Courier New)
  - Color Primary
  - Negrita
  - Tamaño 18px
  - Letter-spacing para legibilidad
- **Válida hasta**:
  - Fecha completa (DD/MM/YYYY HH:MM)
  - Color rojo (#D4145A)
  - Enfatiza urgencia

#### Instrucciones Paso a Paso
1. **Acceda al portal**:
   - Botón CTA con gradiente
   - URL visible debajo del botón
   - Color de texto oscuro para claridad

2. **Ingrese usuario y contraseña temporal**:
   - Referencia a datos arriba
   - Lenguaje simple

3. **Sistema solicitará cambio**:
   - Explica qué esperar
   - Reduce sorpresa

4. **Cree contraseña segura**:
   - Enfatiza requisitos
   - Mensaje de seguridad

#### Box de Advertencias (Naranja)
- Background #fff3e0
- Border-left naranja (#ff9800)
- Lista con bullets:
  - Contraseña de un solo uso
  - Expira en 72 horas
  - No compartir
  - Si no solicitó, contactar soporte
- Iconos de advertencia (⚠️)

#### Sección de Ayuda (Azul)
- Background #e3f2fd
- Border-left azul (#1976d2)
- **Contactos**:
  - Teléfono
  - Email
  - Horarios de atención
- Mensaje de disponibilidad del equipo

#### Footer
- Background gris claro (#f8f9fa)
- "Equipo de CDN Facturación" en negrita
- Correo del destinatario
- Aviso legal: "Este es un correo automático, no responder"

**Características Técnicas**:

**Compatibilidad con Email Clients**:
- Estilos inline para máxima compatibilidad
- Table-based layout (no flexbox/grid)
- Fuentes web-safe (Segoe UI, Tahoma, Verdana)
- Imágenes con alt text
- Ancho máximo 600px

**Responsive Design**:
```css
@media only screen and (max-width: 600px) {
    .email-body { padding: 30px 20px; }
    .credentials-box { padding: 20px; }
    .credential-value.password {
        font-size: 16px;
        word-break: break-all;
    }
}
```

**Funcionalidad Demo**:
- Botón "Llenar con datos de ejemplo"
- Botón "Limpiar"
- Calcula fecha de expiración dinámicamente (+72 horas)
- Formato de fecha localizado (español)

**Datos que se Personalizan**:
- {nombre_usuario}: Nombre completo del usuario
- {numero_identificacion}: Usuario para login
- {contraseña_temporal}: 12 caracteres generados
- {fecha_expiracion}: Fecha + 72 horas
- {url_portal_login}: URL del portal
- {correo_usuario}: Email del destinatario

**Mejores Prácticas Aplicadas**:
1. **Diseño**: Limpio, profesional, branded
2. **Accesibilidad**: Alto contraste, texto legible
3. **Seguridad**: No revelar información sensible innecesaria
4. **UX**: Instrucciones claras, CTA obvio
5. **Deliverability**: SPF/DKIM/DMARC ready

---

### 3. HU007-ContraseñaExpirada.html
**Propósito**: Pantalla de login mostrando error de contraseña temporal expirada.

**Características Principales**:
- Basado en pantalla de login estándar (HU001)
- Alert de error prominente
- Información de contacto de soporte
- Opciones de acción claras

**Elementos de la Interfaz**:

#### Logo Section
- Igual que pantalla de login normal
- Gradiente Primary → Secondary
- Logo centrado

#### Título de Login
- "Iniciar Sesión"
- Subtitle: "Portal Unificado de CDN Facturación"
- Mantiene consistencia con flujo normal

#### Alert de Error (Prominente)
- **Background**: #ffebee (rojo claro)
- **Border-left**: 4px rojo (#D4145A)
- **Animación**: Shake al cargar (0.5s)
- **Contenido**:
  - Icono: error (Material Icons)
  - Título: "Su contraseña temporal ha expirado"
  - Mensaje: Explicación clara + instrucciones
  - Color: Contraste alto para legibilidad

**Animación Shake**:
```css
@keyframes shake {
    0%, 100% { transform: translateX(0); }
    25% { transform: translateX(-10px); }
    75% { transform: translateX(10px); }
}
```

#### Formulario de Login (Deshabilitado)
- Campos con valores que causaron el error
- **Número de Identificación**: 123456789 (disabled)
- **Contraseña**: Kj8mL@pQ3z#W (disabled, type password)
- Muestra contexto del error
- Color gris para indicar estado deshabilitado

#### Botones de Acción

**Volver a Intentar** (Primario):
- Gradiente Primary → Secondary
- Icono: refresh
- Limpia formulario
- Habilita campos
- Oculta alert de error
- Enfoca primer campo

**Contactar Soporte** (Outlined):
- Borde gris, background blanco
- Icono: support_agent
- Abre canal de soporte

#### Sección de Información de Contacto
- Background azul claro (#e3f2fd)
- Border-left azul (#1976d2)
- **Título**: "Información de Contacto" con icono
- **Datos**:
  - Teléfono: +57 (1) 234-5678
  - Email: soporte@cdnfacturacion.com
  - Horario: Lunes a Viernes 8:00 AM - 6:00 PM

#### Info Box (Naranja)
- Background #fff3e0
- Border-left naranja (#ff9800)
- **Contenido**:
  - "¿Por qué expiró mi contraseña?"
  - Explicación de política de 72 horas
  - Mensaje sobre regeneración por administrador

**Flujo de Interacción**:

1. **Usuario intenta login** con contraseña temporal expirada
2. **Sistema valida** credenciales y detecta expiración
3. **Muestra esta pantalla** con error destacado
4. **Usuario puede**:
   - Volver a intentar (limpiar y reintentar con otra contraseña)
   - Contactar soporte (solicitar nueva temporal)

**Funcionalidad de Botones**:

**retry()**:
```javascript
function retry() {
    document.getElementById('idNumber').value = '';
    document.getElementById('password').value = '';
    document.getElementById('idNumber').disabled = false;
    document.getElementById('password').disabled = false;
    document.querySelector('.alert-error').style.display = 'none';
    document.getElementById('idNumber').focus();
}
```

**contactSupport()**:
- Mock: Muestra alert con descripción
- Real: Abriría formulario/chat/email
- Incluiría automáticamente:
  - ID del usuario
  - Razón: "Contraseña temporal expirada"
  - Timestamp

---

## Flujo de Navegación Completo

```
HU006-CreacionExitosa.html (Usuario creado)
    ↓
[Sistema genera contraseña temporal de 12 caracteres]
    ↓
[Sistema envía correo]
    ↓
HU007-EmailContraseñaTemporal.html (Usuario recibe correo)
    ↓
[Usuario hace clic en enlace del correo]
    ↓
HU001-Login.html (Pantalla de login)
    ↓
[Usuario ingresa ID + contraseña temporal]
    ↓
    ├─→ [Si contraseña expirada (>72h)]
    │       ↓
    │   HU007-ContraseñaExpirada.html
    │       ├─→ [Volver a intentar] → HU001-Login.html
    │       └─→ [Contactar soporte] → (Formulario/Chat)
    │
    └─→ [Si contraseña temporal válida (<72h)]
            ↓
        HU007-CambioContraseñaObligatorio.html
            ↓
        [Usuario establece nueva contraseña]
            ↓
        [Validación de requisitos]
            ↓
        [Cambio exitoso]
            ↓
        [Alert de éxito - espera 2 segundos]
            ↓
            ├─→ [Si tiene múltiples clientes] → HU001 Selección de Cliente
            └─→ [Si tiene un solo cliente] → HU001 Ingreso Directo al Portal
```

---

## Validaciones Implementadas

### Frontend (Tiempo Real)

#### Requisitos de Contraseña

**1. Longitud Mínima**:
```javascript
length: (pwd) => pwd.length >= 8
```
- Mínimo 8 caracteres
- Actualiza en tiempo real al escribir

**2. Mayúsculas**:
```javascript
uppercase: (pwd) => /[A-Z]/.test(pwd)
```
- Al menos una letra A-Z
- Regex: `/[A-Z]/`

**3. Minúsculas**:
```javascript
lowercase: (pwd) => /[a-z]/.test(pwd)
```
- Al menos una letra a-z
- Regex: `/[a-z]/`

**4. Números**:
```javascript
number: (pwd) => /[0-9]/.test(pwd)
```
- Al menos un dígito 0-9
- Regex: `/[0-9]/`

**5. Símbolos**:
```javascript
symbol: (pwd) => /[!@#$%^&*]/.test(pwd)
```
- Al menos un símbolo: !@#$%^&*
- Regex: `/[!@#$%^&*]/`

**6. No Igual a Temporal**:
```javascript
notTemp: (pwd) => pwd !== TEMP_PASSWORD
```
- Compara con contraseña temporal (almacenada en variable)
- Previene reutilización

#### Coincidencia de Confirmación

```javascript
function validateConfirmPassword() {
    const password = document.getElementById('newPassword').value;
    const confirmPassword = document.getElementById('confirmPassword').value;

    if (!confirmPassword) {
        // No mostrar error si está vacío
        return false;
    }

    if (password !== confirmPassword) {
        // Mostrar error "Las contraseñas no coinciden"
        confirmInput.classList.add('error');
        return false;
    } else {
        // Ocultar error, marcar como válido
        confirmInput.classList.add('success');
        return true;
    }
}
```

#### Lista Negra de Contraseñas Comunes

```javascript
const COMMON_PASSWORDS = [
    'Password1!', 'Qwerty123!', 'Admin123!',
    '12345678!', 'Welcome1!', 'Passw0rd!',
    'Secret123!', 'Test1234!', 'Hello123!'
];

// Validación antes de enviar
if (COMMON_PASSWORDS.includes(password)) {
    alert('Esta contraseña es muy común...');
    return;
}
```

#### Habilitación del Botón Submit

```javascript
function updateSubmitButton() {
    const allRequirementsMet =
        requirements.length(password) &&
        requirements.uppercase(password) &&
        requirements.lowercase(password) &&
        requirements.number(password) &&
        requirements.symbol(password) &&
        requirements.notTemp(password) &&
        password === confirmPassword &&
        confirmPassword.length > 0;

    submitBtn.disabled = !allRequirementsMet;
}
```

**Condiciones para habilitar**:
- ✓ 8+ caracteres
- ✓ Mayúscula
- ✓ Minúscula
- ✓ Número
- ✓ Símbolo
- ✓ No es temporal
- ✓ Confirmación coincide
- ✓ Confirmación no está vacía

---

## Lógica de Negocio Implementada

### 1. Generación de Contraseña Temporal

**Algoritmo** (documentado para backend):
```
1. Generar 4 letras mayúsculas aleatorias (A-Z)
2. Generar 4 letras minúsculas aleatorias (a-z)
3. Generar 2 dígitos aleatorios (0-9)
4. Generar 2 símbolos aleatorios (!@#$%^&*)
5. Mezclar todos los caracteres en orden aleatorio
```

**Ejemplo**: `Kj8mL@pQ3z#W`

**Características**:
- 12 caracteres total
- Criptográficamente aleatorio
- ~72 bits de entropía
- Espacio de búsqueda: (26+26+10+8)^12 ≈ 10^21 combinaciones

### 2. Expiración de Contraseña Temporal

**Duración**: 72 horas (3 días)

**Cálculo**:
```javascript
const expiryDate = new Date();
expiryDate.setHours(expiryDate.getHours() + 72);
```

**Validación en Login**:
```javascript
if (is_password_temporary === true) {
    if (current_timestamp >= password_temp_expires_at) {
        // Contraseña expirada
        return error('Contraseña temporal expirada');
    } else {
        // Contraseña válida, forzar cambio
        redirect('HU007-CambioContraseñaObligatorio.html');
    }
}
```

### 3. Forzar Cambio de Contraseña

**Implementación** (backend):
```javascript
// Después de login exitoso con temporal
if (user.is_password_temporary === true) {
    session.requires_password_change = true;
    redirect('/change-password-mandatory');
}

// Middleware en TODAS las rutas (excepto /change-password y /logout)
if (session.requires_password_change === true) {
    if (request.path !== '/change-password' && request.path !== '/logout') {
        redirect('/change-password-mandatory');
    }
}
```

**Garantiza**:
- Usuario no puede acceder al portal sin cambiar contraseña
- No puede evadir mediante manipulación de URL
- Solo puede cambiar contraseña o cerrar sesión

### 4. Cambio Exitoso de Contraseña

**Proceso**:
1. Validar todos los requisitos
2. Hash de la nueva contraseña (bcrypt/Argon2)
3. Actualizar en BD:
   ```sql
   UPDATE users SET
       password_hash = :new_hash,
       is_password_temporary = false,
       password_temp_expires_at = NULL,
       updated_at = NOW()
   WHERE user_id = :user_id
   ```
4. Actualizar sesión:
   ```javascript
   session.requires_password_change = false;
   ```
5. Registrar en auditoría
6. Enviar email de confirmación al usuario:
   - "Su contraseña fue cambiada desde IP {IP} el {fecha}"
   - "Si no fue usted, contacte soporte inmediatamente"
7. Redirigir según permisos del usuario

### 5. Invalidación de Contraseña Temporal

**Escenarios**:

**Después de cambio exitoso**:
- Flag `is_password_temporary` = false
- Contraseña temporal hasheada ya no sirve

**Regeneración por administrador**:
- Invalida contraseña temporal anterior
- Genera nueva con nueva fecha de expiración

**Expiración por tiempo**:
- Contraseña sigue en BD pero marcada como expirada
- Login rechazado si current_time >= expiry_time

---

## Datos Mock Utilizados

### Contraseña Temporal de Ejemplo
```javascript
const TEMP_PASSWORD = 'Kj8mL@pQ3z#W';
```

**Características**:
- K, j, m, L, p, Q, z, W (8 letras: 4 mayúsculas, 4 minúsculas)
- 8, 3 (2 números)
- @, # (2 símbolos)
- Total: 12 caracteres

### Usuario de Ejemplo
```javascript
{
    nombre_usuario: 'Juan Carlos Pérez López',
    numero_identificacion: '123456789',
    correo: 'juan.perez@empresa.com',
    fecha_creacion: '2024-01-15 14:30:00',
    fecha_expiracion: '2024-01-18 14:30:00' // +72 horas
}
```

### Lista Negra de Contraseñas
```javascript
const COMMON_PASSWORDS = [
    'Password1!',
    'Qwerty123!',
    'Admin123!',
    '12345678!',
    'Welcome1!',
    'Passw0rd!',
    'Secret123!',
    'Test1234!',
    'Hello123!'
];
```

**Fuente**: Top contraseñas comunes que cumplen requisitos técnicos pero son débiles.

---

## Responsive Design

### Breakpoints
- **Desktop**: > 768px (diseño completo)
- **Mobile**: ≤ 768px (adaptaciones)

### Adaptaciones Mobile

#### HU007-CambioContraseñaObligatorio.html
```css
@media (max-width: 768px) {
    .content {
        padding: 24px 20px;  /* Reduce padding */
    }

    .title {
        font-size: 20px;  /* Título más pequeño */
    }
}
```

#### HU007-EmailContraseñaTemporal.html
```css
@media only screen and (max-width: 600px) {
    .email-body {
        padding: 30px 20px;
    }

    .credentials-box {
        padding: 20px;
    }

    .credential-value.password {
        font-size: 16px;
        word-break: break-all;  /* Evita overflow */
    }
}
```

#### HU007-ContraseñaExpirada.html
```css
@media (max-width: 768px) {
    .content {
        padding: 24px 20px;
    }

    .title {
        font-size: 20px;
    }
}
```

---

## Integración con Backend

### Endpoints Esperados

#### POST /api/auth/login
**Request**:
```json
{
    "idNumber": "123456789",
    "password": "Kj8mL@pQ3z#W"
}
```

**Response (Contraseña temporal válida)**:
```json
{
    "success": true,
    "requiresPasswordChange": true,
    "redirectUrl": "/change-password-mandatory",
    "sessionToken": "jwt_token_here",
    "message": "Debe cambiar su contraseña temporal"
}
```

**Response (Contraseña temporal expirada)**:
```json
{
    "success": false,
    "error": "TEMP_PASSWORD_EXPIRED",
    "message": "Su contraseña temporal ha expirado. Por favor, contacte al administrador para solicitar una nueva.",
    "expirationDate": "2024-01-18T14:30:00Z",
    "currentDate": "2024-01-20T10:00:00Z"
}
```

#### POST /api/auth/change-password-mandatory
**Request**:
```json
{
    "newPassword": "SecureP@ss123",
    "confirmPassword": "SecureP@ss123",
    "sessionToken": "jwt_token_here"
}
```

**Response (Éxito)**:
```json
{
    "success": true,
    "message": "Contraseña cambiada exitosamente",
    "redirectUrl": "/client-selection",
    "sessionToken": "new_jwt_token_here"
}
```

**Response (Error)**:
```json
{
    "success": false,
    "error": "WEAK_PASSWORD",
    "message": "La contraseña no cumple con los requisitos de seguridad",
    "failedRequirements": ["symbol", "length"]
}
```

#### POST /api/users/{userId}/generate-temporary-password
**Request** (por Administrador):
```json
{
    "userId": "uuid-123",
    "reason": "Usuario no recibió correo inicial"
}
```

**Response**:
```json
{
    "success": true,
    "message": "Nueva contraseña temporal generada y enviada",
    "emailSent": true,
    "emailAddress": "j***@empresa.com",
    "expirationDate": "2024-01-18T14:30:00Z"
}
```

#### POST /api/users/{userId}/resend-temporary-password
**Request** (por Administrador):
```json
{
    "userId": "uuid-123"
}
```

**Response (Éxito)**:
```json
{
    "success": true,
    "message": "Correo reenviado exitosamente",
    "emailAddress": "juan.perez@empresa.com"
}
```

**Response (Expirada)**:
```json
{
    "success": false,
    "error": "TEMP_PASSWORD_EXPIRED",
    "message": "La contraseña temporal expiró. Debe generar una nueva con 'Resetear Contraseña'",
    "expirationDate": "2024-01-18T14:30:00Z"
}
```

### Eventos de Auditoría

#### Generación de Contraseña Temporal
```json
{
    "eventId": "uuid",
    "eventType": "SEGURIDAD_CONTRASENA_TEMPORAL_GENERADA",
    "timestamp": "2024-01-15T14:30:00.123Z",
    "userId": "uuid-usuario",
    "result": "EXITOSO",
    "severity": "INFO",
    "description": "Contraseña temporal generada para usuario 123456789 por Administrador admin@cdn.com",
    "additionalData": {
        "usuario_id": "uuid-usuario",
        "usuario_numero_id": "123456789",
        "usuario_nombre": "Juan Pérez",
        "correo_destino": "j***@empresa.com",
        "fecha_expiracion": "2024-01-18T14:30:00Z",
        "administrador_creador": "admin123"
    }
}
```

#### Envío Exitoso de Correo
```json
{
    "eventType": "SEGURIDAD_CONTRASENA_TEMPORAL_ENVIADA",
    "result": "EXITOSO",
    "severity": "INFO",
    "additionalData": {
        "usuario_id": "uuid",
        "correo_destino": "j***@empresa.com",
        "fecha_envio": "2024-01-15T14:30:05Z",
        "servicio_correo_respuesta": "250 OK"
    }
}
```

#### Error al Enviar Correo
```json
{
    "eventType": "SEGURIDAD_CONTRASENA_TEMPORAL_ERROR_ENVIO",
    "result": "FALLIDO",
    "severity": "ERROR",
    "additionalData": {
        "usuario_id": "uuid",
        "correo_destino": "email@domain.com",
        "error_tipo": "timeout",
        "error_mensaje": "Connection timeout after 5000ms"
    }
}
```

#### Login con Contraseña Temporal
```json
{
    "eventType": "SEGURIDAD_LOGIN_CONTRASENA_TEMPORAL",
    "result": "EXITOSO",
    "severity": "INFO",
    "description": "Usuario 123456789 autenticado con contraseña temporal - redirigido a cambio obligatorio",
    "additionalData": {
        "usuario_id": "uuid",
        "usuario_numero_id": "123456789",
        "ip_publica": "203.0.113.50",
        "ip_local": "192.168.1.100",
        "fecha_generacion_temporal": "2024-01-15T14:30:00Z",
        "dias_desde_generacion": 1,
        "cambio_obligatorio": true
    }
}
```

#### Cambio Exitoso de Contraseña
```json
{
    "eventType": "SEGURIDAD_CONTRASENA_CAMBIADA_PRIMER_LOGIN",
    "result": "EXITOSO",
    "severity": "INFO",
    "description": "Usuario 123456789 cambió contraseña temporal por contraseña definitiva",
    "additionalData": {
        "usuario_id": "uuid",
        "usuario_numero_id": "123456789",
        "ip_publica": "203.0.113.50",
        "fecha_cambio": "2024-01-16T10:15:00Z",
        "fecha_generacion_temporal": "2024-01-15T14:30:00Z",
        "tiempo_uso_temporal_horas": 19.75
    }
}
```

#### Contraseña Temporal Expirada
```json
{
    "eventType": "SEGURIDAD_LOGIN_CONTRASENA_TEMPORAL_EXPIRADA",
    "result": "FALLIDO",
    "severity": "WARNING",
    "description": "Usuario 123456789 intentó autenticarse con contraseña temporal expirada",
    "additionalData": {
        "usuario_id": "uuid",
        "fecha_generacion": "2024-01-15T14:30:00Z",
        "fecha_expiracion": "2024-01-18T14:30:00Z",
        "fecha_intento": "2024-01-20T10:00:00Z",
        "horas_desde_expiracion": 43.5
    }
}
```

---

## Patrones de Diseño Aplicados

### Colores
- **Primary**: `#4A5A9E` (Azul corporativo)
- **Secondary**: `#D4145A` (Magenta corporativo)
- **Success**: `#4caf50` (Verde Material)
- **Error**: `#D4145A` / `#ffebee` (Rojo)
- **Warning**: `#ff9800` / `#fff3e0` (Naranja)
- **Info**: `#1976d2` / `#e3f2fd` (Azul claro)

### Tipografía
- **Familia**: Roboto (Google Fonts)
- **Email**: Segoe UI, Tahoma, Verdana (web-safe)
- **Monospace** (contraseña temporal): Courier New
- **Pesos**: 300, 400, 500, 600, 700
- **Tamaños**:
  - H1: 24px
  - H3: 16px
  - Body: 14px
  - Small: 13px
  - Contraseña: 18px

### Espaciado
- **Base**: 8px
- **Padding contenido**: 32px desktop, 24px mobile
- **Gap elementos**: 8-16px
- **Margin secciones**: 24-32px

### Componentes Material UI

#### Alerts
- **Info**: Fondo #e3f2fd, borde izquierdo #1976d2
- **Success**: Fondo #e8f5e9, borde izquierdo #4caf50
- **Error**: Fondo #ffebee, borde izquierdo #D4145A
- **Warning**: Fondo #fff3e0, borde izquierdo #ff9800
- Icono a la izquierda (24px)
- Padding: 16px
- Border-radius: 4px

#### Password Input con Toggle
- Input normal con padding-right para icono
- Botón de toggle posicionado absolute
- Icono: visibility / visibility_off
- Toggle entre type="password" y type="text"

#### Progress Bar (Fortaleza)
- Height: 6px
- Background: #e0e0e0
- Fill con transición suave
- Colores según nivel:
  - Débil: #D4145A (33%)
  - Media: #ff9800 (66%)
  - Fuerte: #4caf50 (100%)

#### Requirement List
- Items con icono + texto
- Estados:
  - Invalid: close icon rojo, texto gris
  - Valid: check icon verde, texto verde
- Transiciones suaves entre estados

---

## Escenarios de Prueba

### Escenario 1: Cambio de Contraseña Exitoso
1. Abrir HU007-CambioContraseñaObligatorio.html
2. Ingresar contraseña: "MyNewP@ss123"
3. Verificar que todos los requisitos muestran ✓ verde
4. Verificar barra de fortaleza en "Fuerte" (verde, 100%)
5. Ingresar confirmación: "MyNewP@ss123"
6. Verificar que botón se habilita
7. Clic en "Cambiar Contraseña"
8. Verificar alert de éxito
9. Esperar 2 segundos
10. Verificar redirección (mock con alert)

### Escenario 2: Contraseña Débil
1. Usar demo "Contraseña Débil"
2. Verificar contraseña: "abc123"
3. Verificar requisitos:
   - ✗ Mínimo 8 caracteres (solo 6)
   - ✗ Mayúscula (no tiene)
   - ✓ Minúscula
   - ✓ Número
   - ✗ Símbolo (no tiene)
   - ✓ No es temporal
4. Verificar barra: "Débil" (rojo, 33%)
5. Verificar botón deshabilitado

### Escenario 3: Contraseñas No Coinciden
1. Usar demo "No Coinciden"
2. Nueva contraseña: "SecureP@ss123" (todos ✓)
3. Confirmación: "SecureP@ss456" (diferente)
4. Verificar error: "Las contraseñas no coinciden"
5. Verificar campo confirmación con borde rojo
6. Verificar botón deshabilitado

### Escenario 4: Intentar Reusar Contraseña Temporal
1. Ingresar nueva contraseña: "Kj8mL@pQ3z#W" (la temporal)
2. Verificar requisito "No puede ser igual a contraseña temporal" con ✗
3. Verificar botón deshabilitado
4. Cambiar a otra contraseña
5. Verificar que requisito cambia a ✓

### Escenario 5: Contraseña Común (Lista Negra)
1. Ingresar contraseña: "Password1!"
2. Verificar todos los requisitos ✓
3. Ingresar confirmación: "Password1!"
4. Botón se habilita
5. Clic en "Cambiar Contraseña"
6. Verificar alert: "Esta contraseña es muy común..."
7. Cambio no procede

### Escenario 6: Toggle de Visibilidad
1. Ingresar contraseña
2. Verificar campo muestra: "••••••••"
3. Clic en icono de ojo
4. Verificar campo muestra texto plano
5. Verificar icono cambió a visibility_off
6. Clic nuevamente
7. Verificar vuelve a ocultar

### Escenario 7: Correo de Contraseña Temporal
1. Abrir HU007-EmailContraseñaTemporal.html
2. Verificar datos prellenados
3. Verificar contraseña en monospace: "Kj8mL@pQ3z#W"
4. Verificar fecha de expiración (+72 horas)
5. Verificar enlace al portal
6. Clic en "Limpiar"
7. Verificar placeholders
8. Clic en "Llenar con datos de ejemplo"
9. Verificar datos se restauran

### Escenario 8: Contraseña Expirada
1. Abrir HU007-ContraseñaExpirada.html
2. Verificar alert de error visible
3. Verificar animación shake
4. Verificar formulario deshabilitado con valores
5. Verificar información de contacto visible
6. Clic en "Volver a Intentar"
7. Verificar formulario se habilita y limpia
8. Verificar alert de error se oculta

### Escenario 9: Estados Demo
1. Probar cada botón de demo:
   - Estado Inicial
   - Contraseña Débil
   - Contraseña Media
   - Contraseña Fuerte
   - No Coinciden
   - Cambio Exitoso
2. Verificar que cada estado muestra UI correcta

### Escenario 10: Responsive en Mobile
1. Abrir DevTools
2. Cambiar a vista móvil (375px)
3. Verificar HU007-CambioContraseñaObligatorio.html:
   - Logo visible
   - Formulario ajustado
   - Requisitos legibles
   - Botón full-width
4. Verificar HU007-EmailContraseñaTemporal.html:
   - Contraseña con word-break
   - Padding reducido
   - Botones ajustados
5. Verificar HU007-ContraseñaExpirada.html:
   - Alert legible
   - Botones apilados

---

## Mejoras Futuras (Fuera de Alcance del Mockup)

### Funcionalidades
- [ ] Autenticación de dos factores (2FA) en primer login
- [ ] Recordatorio por email 24 horas antes de expiración
- [ ] Generación de contraseña sugerida (botón "Generar contraseña segura")
- [ ] Verificación de contraseña contra base de datos de brechas (HIBP API)
- [ ] Límite de intentos de cambio de contraseña (rate limiting)
- [ ] Historial de contraseñas anteriores (prevenir reutilización de últimas 5)
- [ ] Configuración de tiempo de expiración personalizable por administrador
- [ ] Envío de SMS como alternativa/complemento al email
- [ ] Magic link como alternativa a contraseña temporal
- [ ] Notificación a administrador cuando contraseña temporal expira sin uso

### UX
- [ ] Copiar contraseña temporal con un clic en el email
- [ ] Indicador de tiempo restante para expiración en pantalla de cambio
- [ ] Tutorial interactivo en primer login
- [ ] Feedback háptico en mobile al cumplir requisitos
- [ ] Modo oscuro para pantalla de cambio
- [ ] Animaciones más elaboradas (confetti al cambiar exitosamente)
- [ ] Preview de contraseña enmascarada (mostrar solo últimos 4 caracteres)
- [ ] Sugerencias de contraseña basadas en el nombre del usuario
- [ ] Medidor de entropía de contraseña

### Técnicas
- [ ] Implementación de zxcvbn para cálculo de fortaleza real
- [ ] Web Worker para validación asíncrona sin bloquear UI
- [ ] Service Worker para funcionamiento offline
- [ ] Almacenamiento seguro de contraseña temporal en IndexedDB
- [ ] Compresión y cifrado de datos en tránsito
- [ ] WebAuthn para autenticación sin contraseña
- [ ] Passkeys como alternativa moderna

### Accesibilidad
- [ ] ARIA live regions para anunciar cambios de requisitos
- [ ] Navegación completa por teclado con indicadores visuales
- [ ] Soporte para lectores de pantalla mejorado
- [ ] Alto contraste y modo de accesibilidad
- [ ] Descripción de audio para usuarios con discapacidad visual
- [ ] Ajuste de tamaño de fuente por usuario

### Seguridad
- [ ] CAPTCHA después de 3 intentos fallidos
- [ ] Detección de comportamiento anómalo (bot detection)
- [ ] Bloqueo temporal después de intentos sospechosos
- [ ] Geolocalización y alerta de login desde ubicación inusual
- [ ] Verificación de dispositivo conocido
- [ ] Sesión segura con rotación de tokens

---

## Consideraciones de Implementación

### Generación de Contraseña Temporal

**Backend** (ejemplo en Node.js):
```javascript
const crypto = require('crypto');

function generateTemporaryPassword() {
    const uppercase = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
    const lowercase = 'abcdefghijklmnopqrstuvwxyz';
    const numbers = '0123456789';
    const symbols = '!@#$%^&*';

    const chars = [];

    // 4 mayúsculas
    for (let i = 0; i < 4; i++) {
        chars.push(uppercase[crypto.randomInt(0, uppercase.length)]);
    }

    // 4 minúsculas
    for (let i = 0; i < 4; i++) {
        chars.push(lowercase[crypto.randomInt(0, lowercase.length)]);
    }

    // 2 números
    for (let i = 0; i < 2; i++) {
        chars.push(numbers[crypto.randomInt(0, numbers.length)]);
    }

    // 2 símbolos
    for (let i = 0; i < 2; i++) {
        chars.push(symbols[crypto.randomInt(0, symbols.length)]);
    }

    // Mezclar
    for (let i = chars.length - 1; i > 0; i--) {
        const j = crypto.randomInt(0, i + 1);
        [chars[i], chars[j]] = [chars[j], chars[i]];
    }

    return chars.join('');
}
```

### Envío de Correo

**Backend** (ejemplo con Nodemailer):
```javascript
const nodemailer = require('nodemailer');

async function sendTemporaryPasswordEmail(user, tempPassword, expiryDate) {
    const transporter = nodemailer.createTransport({
        service: 'SendGrid',
        auth: {
            user: process.env.SENDGRID_USER,
            pass: process.env.SENDGRID_API_KEY
        }
    });

    const htmlTemplate = renderEmailTemplate({
        userName: user.fullName,
        userId: user.idNumber,
        tempPassword: tempPassword,
        expiryDate: expiryDate,
        portalUrl: process.env.PORTAL_URL
    });

    try {
        const info = await transporter.sendMail({
            from: '"CDN Facturación" <noreply@cdnfacturacion.com>',
            to: user.email,
            subject: 'Bienvenido al Portal Unificado CDN Facturación - Credenciales de Acceso',
            html: htmlTemplate
        });

        await auditLog('SEGURIDAD_CONTRASENA_TEMPORAL_ENVIADA', {
            userId: user.id,
            emailAddress: maskEmail(user.email),
            messageId: info.messageId,
            result: 'EXITOSO'
        });

        return { success: true, messageId: info.messageId };

    } catch (error) {
        await auditLog('SEGURIDAD_CONTRASENA_TEMPORAL_ERROR_ENVIO', {
            userId: user.id,
            emailAddress: user.email,
            error: error.message,
            result: 'FALLIDO'
        });

        throw error;
    }
}
```

### Almacenamiento Seguro

**Tabla de Usuarios** (esquema):
```sql
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    id_number VARCHAR(15) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    is_password_temporary BOOLEAN DEFAULT FALSE,
    password_temp_expires_at TIMESTAMP NULL,
    password_changed_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_temp_password ON users(is_password_temporary, password_temp_expires_at);
```

**Hash de Contraseña**:
```javascript
const bcrypt = require('bcrypt');

// Al crear contraseña temporal
const hash = await bcrypt.hash(temporaryPassword, 12);
await db.query(`
    UPDATE users SET
        password_hash = $1,
        is_password_temporary = true,
        password_temp_expires_at = NOW() + INTERVAL '72 hours'
    WHERE user_id = $2
`, [hash, userId]);

// Al cambiar contraseña
const newHash = await bcrypt.hash(newPassword, 12);
await db.query(`
    UPDATE users SET
        password_hash = $1,
        is_password_temporary = false,
        password_temp_expires_at = NULL,
        password_changed_at = NOW()
    WHERE user_id = $2
`, [newHash, userId]);
```

---

## Conclusión

Los mockups de HU007 implementan un flujo completo y seguro de gestión de contraseñas temporales y primer cambio obligatorio, siguiendo las mejores prácticas de seguridad y experiencia de usuario:

- **Seguro**: Contraseñas fuertes, expiración, cambio obligatorio, validaciones múltiples
- **Funcional**: Validación en tiempo real, feedback inmediato, estados claros
- **Interactivo**: 6 estados demo, toggle de visibilidad, barra de fortaleza dinámica
- **Accesible**: HTML semántico, contraste adecuado, mensajes claros
- **Responsive**: Adaptable a todos los dispositivos
- **Auditable**: Diseñado para integración con sistema de auditoría completo

Los mockups sirven como especificación completa para desarrollo backend y frontend, garantizando seguridad en el proceso de activación de cuentas de usuario.
