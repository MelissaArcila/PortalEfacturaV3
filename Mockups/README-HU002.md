# Mockups HU002 - Recuperación de Contraseña

Este documento describe los mockups interactivos creados para la **HU002 - Recuperación de Contraseña**.

## 📋 Archivos Creados

### Pantallas Web Interactivas

#### 1. **HU002-ForgotPassword.html** - Solicitud de recuperación
**Ubicación de la HU:** Escenario 1, línea 51

**Descripción:** Pantalla donde el usuario solicita el envío del enlace de recuperación.

**Elementos principales:**
- Campo de entrada para usuario/correo con validación en tiempo real
- Validación de formato de email y username
- Mensaje de éxito genérico (no revela si el usuario existe)
- Mensaje de error por límite de solicitudes excedido
- Botón deshabilitado hasta que la entrada sea válida

**Estados del demo:**
- ✅ Normal - Formulario en estado inicial
- ✅ Mensaje de Éxito - Muestra confirmación de envío
- ✅ Límite Excedido - Error de 5 solicitudes en 24h
- ✅ Entrada Inválida - Formato inválido de usuario/email

**Validaciones implementadas:**
- Email: Regex `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`
- Username: Alfanumérico con guiones y puntos, mínimo 3 caracteres

---

#### 2. **HU002-ResetPassword.html** - Restablecimiento de contraseña
**Ubicación de la HU:** Escenarios 3-8, líneas 53-58

**Descripción:** Pantalla de creación de nueva contraseña con validación completa de requisitos.

**Elementos principales:**
- Campo de nueva contraseña con indicador de visibilidad (icono de ojo)
- Validación en tiempo real de 7 requisitos:
  - ✓/✗ Mínimo 8 caracteres
  - ✓/✗ Al menos una mayúscula (A-Z)
  - ✓/✗ Al menos una minúscula (a-z)
  - ✓/✗ Al menos un número (0-9)
  - ✓/✗ Al menos un símbolo (!@#$%^&*)
  - ✓/✗ No puede ser igual a contraseña actual
  - ✓/✗ No puede ser una de las últimas 5 contraseñas
- Barra de fortaleza de contraseña (Débil/Media/Fuerte) con colores
- Campo de confirmación de contraseña con validación de coincidencia
- Botón deshabilitado hasta cumplir todos los requisitos

**Estados del demo:**
- ✅ Normal - Formulario vacío
- ✅ Contraseña Débil - Password: `pass123`
- ✅ Contraseña Media - Password: `Password123`
- ✅ Contraseña Fuerte - Password: `MySecure@Pass123`
- ✅ Contraseñas No Coinciden - Campos con valores diferentes
- ✅ Cambio Exitoso - Mensaje de confirmación y redirección

**Tecnología UI:**
- Material Icons para iconografía (visibility/visibility_off)
- Indicadores visuales con checkmarks dinámicos
- Barra de progreso lineal para fortaleza

---

#### 3. **HU002-LinkExpired.html** - Estados de enlace inválido
**Ubicación de la HU:** Escenarios 9-11, líneas 59-61

**Descripción:** Pantalla de error para enlaces expirados, ya utilizados o inválidos.

**Elementos principales:**
- Iconografía de estado con Material Icons:
  - 🟠 `warning_amber` - Enlace expirado
  - 🟢 `check_circle` - Enlace ya utilizado
  - 🔴 `error_outline` - Enlace inválido
- Título y descripción dinámicos según el estado
- Info box con colores y bordes personalizados por estado
- Botón primario "Solicitar nuevo enlace"
- Botón secundario "Volver a inicio de sesión"

**Estados del demo:**
- ✅ Enlace Expirado - >15 minutos de generado
- ✅ Enlace Ya Utilizado - Token consumido anteriormente
- ✅ Enlace Inválido - Token corrupto o manipulado

**Configuración de colores por estado:**
```javascript
expired: { icon: warning_amber, color: #ff9800 }
used:    { icon: check_circle,  color: #4caf50 }
invalid: { icon: error_outline, color: #f44336 }
```

---

### Plantillas de Correo Electrónico

#### 4. **HU002-EmailTemplate.html** - Correo de recuperación
**Ubicación de la HU:** Escenario 2, línea 52 + Mockups líneas 154-170

**Descripción:** Plantilla de correo enviada al usuario con el enlace de recuperación.

**Asunto:** `Recuperación de contraseña - Portal Unificado CDN`

**Elementos principales:**
- Header con gradiente de marca y logo
- Saludo personalizado: `Hola {nombre_usuario},`
- Mensaje explicativo del proceso
- Botón CTA prominente: "Restablecer mi contraseña"
- Sección de seguridad con información de:
  - ⏱ Validez del enlace (15 minutos)
  - 🔒 Advertencia de no compartir
  - Instrucciones si no solicitó el cambio
- Enlace alternativo en texto plano
- Footer con información de contacto y legal

**Variables dinámicas:**
- `{nombre_usuario}` - Nombre del usuario
- `{enlace_recuperacion}` - URL con token único

**Compatibilidad:**
- ✅ Diseño de tabla para Outlook
- ✅ Responsive con media queries
- ✅ Estilos inline para clientes de correo
- ✅ Reset CSS para compatibilidad

---

#### 5. **HU002-EmailPasswordChanged.html** - Correo de confirmación ⭐ NUEVO
**Ubicación de la HU:** Riesgos, línea 180 + Escenario 4, línea 184

**Descripción:** Plantilla de correo enviada al usuario confirmando el cambio exitoso de contraseña.

**Asunto:** `Contraseña actualizada - Portal Unificado CDN`

**Elementos principales:**
- Icono de éxito grande con checkmark ✓
- Mensaje de confirmación destacado en caja verde
- Tabla de detalles del cambio:
  - 📅 Fecha y hora del cambio
  - 🌐 Dirección IP desde donde se cambió
  - 📍 Ubicación aproximada
  - 💻 Dispositivo/navegador utilizado
- Sección de advertencia (caja naranja):
  - ⚠️ "¿No fuiste tú?"
  - Instrucciones de acción inmediata
  - Pasos para reportar compromiso
- Botón CTA: "Contactar a Soporte"
- Tips de seguridad con recomendaciones
- Nota de cierre de sesiones activas

**Variables dinámicas:**
- `{nombre_usuario}` - Nombre del usuario
- `{fecha_cambio}` - Timestamp del cambio
- `{ip_cambio}` - IP desde donde se realizó
- `{ubicacion_aproximada}` - Geolocalización de IP
- `{dispositivo}` - User-agent/navegador
- `{url_contacto_soporte}` - URL de soporte

**Propósito de seguridad:**
- ✅ Notificación inmediata de cambios en la cuenta
- ✅ Detección temprana de compromiso de cuenta
- ✅ Información forense para investigación
- ✅ Cumplimiento de riesgo de línea 180 (HU002)

---

## 🎨 Guía de Estilos Aplicada

Todos los mockups siguen la guía de estilos especificada en la HU002:

### Colores
- **Primary:** `#4A5A9E`
- **Secondary:** `#D4145A`
- **Gradiente de marca:** `linear-gradient(135deg, #4A5A9E 0%, #D4145A 100%)`

### Tipografía
- **Fuente:** Roboto (Google Fonts)
- **Pesos:** 300 (Light), 400 (Regular), 500 (Medium), 700 (Bold)

### Componentes Material UI
- TextField outlined con focus en Primary
- Buttons contained con gradiente
- Buttons outlined con borde Primary
- Alert success/error con colores semánticos
- LinearProgress para barra de fortaleza

### Iconografía
- **Material Icons** de Google Fonts
- Iconos: visibility, visibility_off, warning_amber, check_circle, error_outline

---

## 🔧 Funcionalidad Técnica

### Todas las pantallas incluyen:
✅ **Selector de estados demo** - Panel lateral/superior para cambiar entre estados
✅ **JavaScript funcional** - Sin errores, todos los parámetros correctamente pasados
✅ **Validaciones en tiempo real** - Feedback inmediato al usuario
✅ **Responsive design** - Media queries para móviles (<600px)
✅ **Accesibilidad** - Labels, aria-labels, navegación por teclado

### Scripts de demostración:
- Variables mockeadas para testing sin backend
- Alertas explicativas en producción
- Reemplazo automático de variables en correos
- Event handlers seguros sin dependencias externas

---

## 📂 Estructura de Archivos

```
Mockups/
├── HU001-Login.html                    # HU001 - Login
├── HU002-ForgotPassword.html           # Solicitud de recuperación
├── HU002-ResetPassword.html            # Restablecimiento de contraseña
├── HU002-LinkExpired.html              # Estados de enlace
├── HU002-EmailTemplate.html            # Correo de recuperación
├── HU002-EmailPasswordChanged.html     # Correo de confirmación ⭐
└── README-HU002.md                     # Este documento
```

---

## 🚀 Cómo Probar los Mockups

### Pantallas Web:
1. Abrir cualquier archivo `HU002-*.html` en un navegador
2. Usar el selector de estados demo (esquina superior derecha)
3. Interactuar con los formularios y ver las validaciones en tiempo real

### Correos Electrónicos:
1. Abrir `HU002-EmailTemplate.html` o `HU002-EmailPasswordChanged.html` en un navegador
2. El script de demostración reemplaza automáticamente las variables `{variable}`
3. Para testing en clientes de correo reales, usar herramientas como:
   - [Litmus](https://litmus.com/)
   - [Email on Acid](https://www.emailonacid.com/)
   - Enviar correo de prueba a múltiples clientes

---

## ✅ Cumplimiento de la HU002

| Requerimiento | Archivo | Estado |
|---------------|---------|--------|
| Escenario 1: Solicitud de recuperación | HU002-ForgotPassword.html | ✅ |
| Escenario 2: Envío de correo con enlace | HU002-EmailTemplate.html | ✅ |
| Escenarios 3-8: Restablecimiento de contraseña | HU002-ResetPassword.html | ✅ |
| Escenarios 9-11: Enlace expirado/usado/inválido | HU002-LinkExpired.html | ✅ |
| Riesgo línea 180: Notificación de cambio | HU002-EmailPasswordChanged.html | ✅ |
| Mockup 1: Pantalla de solicitud | HU002-ForgotPassword.html | ✅ |
| Mockup 2: Pantalla de restablecimiento | HU002-ResetPassword.html | ✅ |
| Mockup 3: Pantalla de enlace expirado | HU002-LinkExpired.html | ✅ |
| Mockup 4: Plantilla de correo | HU002-EmailTemplate.html | ✅ |

---

## 📝 Notas Adicionales

### Variables de correo en producción:
Los correos HTML contienen variables entre llaves `{variable}` que deben ser reemplazadas por el backend antes del envío:

**HU002-EmailTemplate.html:**
- `{nombre_usuario}`
- `{enlace_recuperacion}`

**HU002-EmailPasswordChanged.html:**
- `{nombre_usuario}`
- `{fecha_cambio}`
- `{ip_cambio}`
- `{ubicacion_aproximada}`
- `{dispositivo}`
- `{url_contacto_soporte}`

### Integración con backend:
Los mockups están listos para integrarse con:
- API REST endpoints
- Sistema de envío de correos (SMTP/SendGrid/AWS SES)
- Sistema de templates (Handlebars/Jinja2/etc)
- Base de datos para validación de historial de contraseñas

---

**Creado para:** Portal Unificado de CDN Facturación
**Historia de Usuario:** HU002 - Recuperación de Contraseña
**Versión:** 1.0
**Fecha:** Diciembre 2025
**Diseñador:** UI/UX Designer (Claude Code)
