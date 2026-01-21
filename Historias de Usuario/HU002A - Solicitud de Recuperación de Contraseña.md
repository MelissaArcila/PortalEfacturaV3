# HU002A - Solicitud de Recuperación de Contraseña

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 1 |
| **Epic** | Gestión de Acceso y Seguridad |
| **Estado** | BORRADOR |
| **Propietario** | Por definir |
| **Arquitecto** | Por definir |
| **Desarrolladores** | Por definir |
| **Tester** | Por definir |
| **Incidentes relacionados** | N/A |
| **Arquitectura relacionada** | Portal Unificado - Módulo de Autenticación, Sistema de Notificaciones |

---

## Contexto

Los Usuarios del Portal Unificado pueden olvidar su contraseña de acceso. Esta historia cubre el **primer componente del flujo de recuperación**: la solicitud de recuperación y envío de enlace temporal por correo electrónico.

El Usuario debe poder acceder a una pantalla de solicitud desde el login, ingresar su nombre de usuario o correo, y recibir un correo con un enlace temporal válido por 15 minutos. El sistema debe validar que el Usuario exista, esté activo, tenga correo registrado, y no esté bloqueado. Además, debe limitar las solicitudes a 5 por usuario en 24 horas para prevenir abuso.

Esta funcionalidad es **independiente y entregable por sí misma**: permite a los Usuarios iniciar el proceso de recuperación y recibir el enlace, aunque las HUs HU002B y HU002C sean necesarias para completar el flujo completo de cambio de contraseña.

---

**Notas:**
- Esta HU cubre SOLO la solicitud y envío de enlace. La validación del enlace y el cambio de contraseña se cubren en HU002B y HU002C respectivamente
- El sistema NO revela si el usuario existe o no (medida de seguridad anti-enumeración)
- Usuarios bloqueados o inactivos NO reciben correo, pero el sistema muestra el mismo mensaje genérico
- Todo el proceso debe ser auditable siguiendo el Estándar de Auditoría Funcional

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Usuario** | Quiero acceder a una pantalla de "¿Olvidaste tu contraseña?" desde el login | Para poder solicitar la recuperación de mi contraseña de forma autoservicio |
| Como **Usuario** | Quiero ingresar mi nombre de usuario o correo electrónico | Para identificarme en el sistema sin necesidad de recordar mi contraseña |
| Como **Usuario** | Quiero recibir un correo electrónico con un enlace temporal y seguro | Para poder acceder al proceso de restablecimiento de contraseña de forma segura |
| Como **Administrador de Seguridad** | Quiero que el sistema limite las solicitudes de recuperación a 5 por usuario en 24 horas | Para prevenir abuso del sistema y ataques de denegación de servicio |
| Como **Administrador** | Quiero que todos los eventos de solicitud de recuperación sean auditados | Para investigar incidentes de seguridad y garantizar trazabilidad |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso a pantalla de recuperación desde login | Usuario en pantalla de inicio de sesión (HU001) | Usuario hace clic en enlace "¿Olvidaste tu contraseña?" | Sistema navega a pantalla de "Recuperación de Contraseña". Muestra título "¿Olvidaste tu contraseña?", texto explicativo, campo de texto para "Usuario o correo electrónico", botón "Enviar enlace de recuperación" (habilitado cuando campo tiene valor válido), y enlace "Volver a inicio de sesión" | ☐ | ☐ | ☐ |
| 2 | Validación de formato de entrada en tiempo real | Usuario en pantalla de solicitud de recuperación | Usuario escribe en campo "Usuario o correo electrónico" | Sistema valida formato en tiempo real. Acepta: letras, números, guiones, puntos, arroba (@). NO acepta: espacios al inicio/final, caracteres especiales no permitidos, cadena vacía. Si formato inválido, muestra error: "Ingresa un nombre de usuario o correo electrónico válido". Botón "Enviar enlace" permanece deshabilitado mientras formato sea inválido | ☐ | ☐ | ☐ |
| 3 | Solicitud exitosa con usuario activo y correo válido | Usuario activo con correo electrónico registrado y verificado ingresa su usuario o correo | Usuario hace clic en "Enviar enlace de recuperación" | Sistema muestra mensaje genérico: "Si el usuario existe, recibirás un correo con instrucciones para recuperar tu contraseña". Sistema genera token único (UUID), almacena con fecha de expiración (15 minutos), envía correo electrónico al correo registrado del usuario con enlace que incluye el token, asunto "Recuperación de contraseña - Portal Unificado CDN", y cuerpo según plantilla definida | ☐ | ☐ | ☐ |
| 4 | Contenido del correo de recuperación | Sistema envía correo de recuperación | Usuario recibe correo | Correo contiene: logo de CDN Facturación, saludo personalizado con nombre del usuario, texto explicativo, botón prominente "Restablecer mi contraseña" con enlace al token, información de seguridad (válido 15 minutos, un solo uso, no compartir), enlace alternativo en texto plano, y pie con información de contacto y aviso de correo automático | ☐ | ☐ | ☐ |
| 5 | Solicitud con usuario bloqueado (no enviar correo) | Usuario bloqueado por intentos fallidos de autenticación (ver HU001) ingresa su usuario | Usuario hace clic en "Enviar enlace de recuperación" | Sistema muestra mensaje genérico (idéntico al exitoso): "Si el usuario existe, recibirás un correo con instrucciones para recuperar tu contraseña". Sistema NO genera token. Sistema NO envía correo. Registra evento en auditoría con tipo "AUTENTICACION_RECUPERACION_BLOQUEADO", severidad WARNING | ☐ | ☐ | ☐ |
| 6 | Solicitud con usuario inactivo (no enviar correo) | Usuario en estado inactivo ingresa su usuario | Usuario hace clic en "Enviar enlace de recuperación" | Sistema muestra mensaje genérico: "Si el usuario existe, recibirás un correo con instrucciones para recuperar tu contraseña". Sistema NO genera token. Sistema NO envía correo. Registra evento en auditoría con tipo "AUTENTICACION_RECUPERACION_INACTIVO", severidad WARNING | ☐ | ☐ | ☐ |
| 7 | Solicitud con usuario sin correo registrado (no enviar correo) | Usuario activo sin correo electrónico registrado en el sistema ingresa su usuario | Usuario hace clic en "Enviar enlace de recuperación" | Sistema muestra mensaje genérico: "Si el usuario existe, recibirás un correo con instrucciones para recuperar tu contraseña". Sistema NO genera token. Sistema NO envía correo. Registra evento en auditoría con tipo "AUTENTICACION_RECUPERACION_SIN_CORREO", severidad WARNING | ☐ | ☐ | ☐ |
| 8 | Solicitud con usuario inexistente (no revelar) | Usuario ingresa nombre de usuario o correo que NO existe en el sistema | Usuario hace clic en "Enviar enlace de recuperación" | Sistema muestra mensaje genérico (idéntico al exitoso): "Si el usuario existe, recibirás un correo con instrucciones para recuperar tu contraseña". Sistema NO genera token. Sistema NO envía correo. Sistema NO registra evento en auditoría (para no registrar intentos de usuarios inexistentes). Tiempo de respuesta debe ser consistente con el caso exitoso para evitar enumeración por timing attack | ☐ | ☐ | ☐ |
| 9 | Límite de solicitudes - dentro del límite (1-4 solicitudes) | Usuario ha solicitado recuperación 1, 2, 3 o 4 veces en las últimas 24 horas | Usuario hace clic en "Enviar enlace de recuperación" | Sistema procesa la solicitud normalmente (según escenarios 3-8). Incrementa contador de solicitudes del usuario con timestamp. Permite continuar | ☐ | ☐ | ☐ |
| 10 | Límite de solicitudes excedido (5+ solicitudes en 24h) | Usuario ha solicitado recuperación 5 veces en las últimas 24 horas y ya no tiene solicitudes disponibles | Usuario intenta hacer clic en "Enviar enlace de recuperación" por 6ta vez | Sistema muestra mensaje de error específico: "Has excedido el número máximo de solicitudes de recuperación (5 en 24 horas). Por favor, intenta nuevamente más tarde o contacta a soporte." Botón "Enviar enlace" se deshabilita. Registra evento en auditoría con tipo "AUTENTICACION_RECUPERACION_LIMITE_EXCEDIDO", severidad ERROR, incluyendo IPs de intentos anteriores | ☐ | ☐ | ☐ |
| 11 | Reseteo de contador de límite después de 24 horas | Han transcurrido 24 horas desde la primera solicitud del usuario (que tenía 5 solicitudes) | Usuario intenta solicitar recuperación nuevamente | Sistema detecta que han pasado 24 horas, reinicia contador de solicitudes a cero, y procesa la solicitud normalmente según escenarios 3-8 | ☐ | ☐ | ☐ |
| 12 | Múltiples enlaces activos - invalidar anteriores | Usuario solicita nuevo enlace de recuperación mientras tiene uno o más enlaces activos (no usados y no expirados) | Usuario hace clic en "Enviar enlace de recuperación" nuevamente | Sistema invalida todos los tokens anteriores no utilizados del usuario (marca como "INVALIDADO"), genera nuevo token único válido por 15 minutos, envía nuevo correo, muestra mensaje genérico exitoso, y registra en auditoría evento "AUTENTICACION_ENLACES_INVALIDADOS" con UUIDs de tokens invalidados y nuevo token | ☐ | ☐ | ☐ |
| 13 | Volver a inicio de sesión | Usuario en pantalla de solicitud de recuperación | Usuario hace clic en enlace "Volver a inicio de sesión" | Sistema redirige a pantalla de login (HU001). No registra evento en auditoría (acción sin impacto de seguridad) | ☐ | ☐ | ☐ |
| 14 | Tiempo de respuesta consistente (anti-enumeración) | Usuario ingresa usuario válido o inválido | Usuario hace clic en "Enviar enlace de recuperación" | Sistema tarda tiempo similar (±200ms) en responder independientemente de si el usuario existe, está bloqueado, inactivo, sin correo, o no existe. Evita que un atacante pueda enumerar usuarios válidos midiendo tiempos de respuesta diferentes | ☐ | ☐ | ☐ |
| 15 | Indicador visual durante procesamiento | Usuario hace clic en "Enviar enlace de recuperación" | Sistema procesa solicitud (tarda 1-3 segundos por validaciones, generación de token, envío de correo) | Sistema muestra indicador de carga (spinner o progress circular) sobre el botón "Enviar enlace de recuperación", deshabilita el botón temporalmente, y muestra texto "Enviando..." hasta que el proceso termina y muestra el mensaje de resultado | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 16 | Estructura obligatoria de registro de auditoría | Cualquier evento de solicitud de recuperación de contraseña | Sistema registra evento | Registro contiene: ID Único del Evento (UUID), Tipo de Evento (AUTENTICACION_RECUPERACION_*), Fecha y Hora (UTC con milisegundos, ISO 8601), Usuario (nombre de usuario ingresado), Cliente (NULL - no aplica en autenticación), Cliente Nombre (NULL), IP Local, IP Pública, Resultado (EXITOSO/FALLIDO), Descripción (texto descriptivo del evento), Nivel de Severidad (INFO/WARNING/ERROR), Datos Adicionales (JSON con contexto relevante) | ☐ | ☐ | ☐ |
| 17 | Auditoría de solicitud exitosa | Usuario activo con correo registrado solicita recuperación y se envía enlace | Usuario hace clic en "Enviar enlace de recuperación" (escenario 3) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_RECUPERACION_SOLICITADA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Usuario {usuario} solicitó recuperación de contraseña exitosamente", Datos Adicionales: {"correo_destino_parcial": "u***@dominio.com", "token_id": "UUID del token generado", "tiempo_expiracion_minutos": 15, "ip_solicitud_local": "IP local", "ip_solicitud_publica": "IP pública"} | ☐ | ☐ | ☐ |
| 18 | Auditoría de solicitud fallida por usuario bloqueado | Usuario bloqueado intenta solicitar recuperación | Usuario hace clic en "Enviar enlace de recuperación" (escenario 5) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_RECUPERACION_BLOQUEADO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} bloqueado intentó solicitar recuperación de contraseña", Datos Adicionales: {"estado_usuario": "bloqueado", "motivo_bloqueo": "intentos_fallidos_autenticacion", "fecha_desbloqueo_automatico": "timestamp ISO 8601", "ip_intento_local": "IP local", "ip_intento_publica": "IP pública"} | ☐ | ☐ | ☐ |
| 19 | Auditoría de solicitud fallida por usuario inactivo | Usuario inactivo intenta solicitar recuperación | Usuario hace clic en "Enviar enlace de recuperación" (escenario 6) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_RECUPERACION_INACTIVO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} inactivo intentó solicitar recuperación de contraseña", Datos Adicionales: {"estado_usuario": "inactivo", "fecha_inactivacion": "timestamp", "ip_intento_local": "IP local", "ip_intento_publica": "IP pública"} | ☐ | ☐ | ☐ |
| 20 | Auditoría de solicitud fallida por usuario sin correo | Usuario sin correo registrado intenta solicitar recuperación | Usuario hace clic en "Enviar enlace de recuperación" (escenario 7) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_RECUPERACION_SIN_CORREO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} sin correo electrónico registrado intentó solicitar recuperación de contraseña", Datos Adicionales: {"estado_usuario": "activo", "correo_registrado": false, "ip_intento_local": "IP local", "ip_intento_publica": "IP pública"} | ☐ | ☐ | ☐ |
| 21 | Auditoría de límite de solicitudes excedido | Usuario supera 5 solicitudes de recuperación en 24 horas | Usuario hace clic en "Enviar enlace de recuperación" por 6ta vez en 24 horas (escenario 10) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_RECUPERACION_LIMITE_EXCEDIDO", Resultado: "FALLIDO", Severidad: "ERROR", Descripción: "Usuario {usuario} excedió límite de solicitudes de recuperación de contraseña (5 en 24 horas)", Datos Adicionales: {"intentos_en_periodo": 5, "periodo_horas": 24, "ip_intento_actual": "IP actual", "solicitudes_anteriores": [{"timestamp": "...", "ip_local": "...", "ip_publica": "..."}, {"timestamp": "...", "ip_local": "...", "ip_publica": "..."}, ...]} | ☐ | ☐ | ☐ |
| 22 | Auditoría de invalidación de enlaces anteriores | Usuario solicita nuevo enlace mientras tiene uno activo | Usuario hace clic en "Enviar enlace de recuperación" nuevamente (escenario 12) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_ENLACES_INVALIDADOS", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Usuario {usuario} solicitó nuevo enlace de recuperación, invalidando enlaces anteriores", Datos Adicionales: {"tokens_invalidados": ["UUID1", "UUID2"], "tokens_invalidados_count": 2, "nuevo_token_id": "UUID3", "ip_solicitud_local": "IP local", "ip_solicitud_publica": "IP pública"} | ☐ | ☐ | ☐ |
| 23 | Inmutabilidad de registros de auditoría | Cualquier escenario de auditoría | Sistema genera registro | Registros de auditoría NO pueden ser modificados ni eliminados después de su creación. Solo se permiten consultas de lectura. Implementar controles que prevengan alteración de registros históricos | ☐ | ☐ | ☐ |
| 24 | Consulta de registros de auditoría para investigación | Administrador o auditor requiere investigar eventos de solicitud de recuperación | Administrador accede a módulo de auditoría | Sistema permite filtrar por: rango de fechas, usuario, tipo de evento (AUTENTICACION_RECUPERACION_*), resultado (EXITOSO/FALLIDO), severidad (INFO/WARNING/ERROR), IP origen (local o pública). Permite exportar resultados en formato CSV/Excel para análisis externo | ☐ | ☐ | ☐ |
| 25 | Retención de registros de auditoría | Registros de auditoría de solicitud de recuperación | Paso del tiempo | Registros se conservan por tiempo mínimo definido por regulación vigente aplicable al ecosistema de facturación electrónica DIAN. Después de este período, pueden ser archivados en almacenamiento de largo plazo o eliminados según política de retención de la organización | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

Se requieren los siguientes mockups interactivos en HTML siguiendo la Guía de Estilos (Material UI, paleta de colores Primary #4A5A9E, Secondary #D4145A):

#### 1. **HU002A-ForgotPassword.html** - Pantalla de solicitud de recuperación

**Elementos:**
- Logotipo de CDN Facturación centrado en parte superior
- Título: "¿Olvidaste tu contraseña?" (Tipografía H3, color Primary Dark #364378)
- Texto descriptivo: "Ingresa tu nombre de usuario o correo electrónico y te enviaremos un enlace para recuperar tu contraseña" (Body2, color grey-700)
- Campo de texto (TextField outlined, fullWidth):
  - Label: "Usuario o correo electrónico"
  - Placeholder: "Ej: usuario@empresa.com"
  - Validación en tiempo real con mensaje de error
  - Máximo de caracteres: 100
- Botón primario (contained, fullWidth, gradiente Primary → Secondary):
  - Texto: "Enviar enlace de recuperación"
  - Deshabilitado si campo vacío o formato inválido
  - Muestra spinner cuando está procesando
- Enlace secundario centrado: "Volver a inicio de sesión" (texto underline, color Primary)
- Área de mensajes (ocultos por defecto, se muestran según resultado):
  - **Alert success**: "Si el usuario existe, recibirás un correo con instrucciones para recuperar tu contraseña" (icono check_circle)
  - **Alert error**: "Has excedido el número máximo de solicitudes de recuperación (5 en 24 horas). Por favor, intenta nuevamente más tarde o contacta a soporte." (icono error)
  - **Alert error de formato**: "Ingresa un nombre de usuario o correo electrónico válido" (icono warning)

**Estados a implementar en mockup:**
- Estado inicial (sin mensajes)
- Estado con campo llenado correctamente
- Estado procesando (spinner en botón)
- Estado éxito (mensaje genérico verde)
- Estado error por límite excedido (mensaje rojo)
- Estado error de formato (mensaje en campo)

#### 2. **HU002A-EmailTemplate.html** - Plantilla de correo electrónico

**Elementos:**
- **Encabezado**:
  - Fondo con gradiente Primary → Secondary (#4A5A9E → #D4145A)
  - Logo de CDN Facturación en blanco centrado
  - Altura: 80px
- **Cuerpo principal** (fondo blanco, padding 40px):
  - Saludo personalizado: "Hola {nombre_usuario}," (Body1 Bold, color grey-900)
  - Párrafo 1: "Recibimos una solicitud para restablecer la contraseña de tu cuenta en el Portal Unificado de CDN Facturación." (Body1, color grey-700)
  - Párrafo 2: "Haz clic en el botón a continuación para crear una nueva contraseña:" (Body1, color grey-700)
  - **Botón CTA prominente** (centered, padding 16px 32px):
    - Texto: "Restablecer mi contraseña"
    - Fondo: gradiente Primary → Secondary
    - Color texto: blanco
    - Border radius: 8px
    - Enlace: {url_con_token}
  - **Sección de seguridad** (Alert info style, fondo #E3F2FD, borde izquierdo azul):
    - Icono info circular
    - "Este enlace es válido por 15 minutos y solo puede usarse una vez."
    - "Si no solicitaste este cambio, ignora este correo y tu contraseña permanecerá sin cambios."
    - "Por tu seguridad, nunca compartas este enlace con nadie."
  - **Enlace alternativo** (Caption, color grey-600):
    - "Si el botón no funciona, copia y pega este enlace en tu navegador:"
    - {url_completa} (texto monospace, seleccionable)
- **Pie de correo** (fondo grey-100, padding 24px, borde superior grey-300):
  - "Soporte CDN Facturación" (Body2 Bold)
  - Información de contacto (email, teléfono)
  - Separador
  - Aviso legal (Caption, color grey-600): "Este es un correo automático, por favor no respondas a este mensaje."

**Nota de diseño**: El correo debe ser responsive y verse correctamente en clientes de correo web y móviles (Gmail, Outlook, Apple Mail)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Enumeración de usuarios** | Un atacante podría intentar enumerar usuarios válidos del sistema observando diferencias en las respuestas, tiempos de respuesta, o comportamiento del sistema | Alto | Media | Sistema responde con mensaje genérico idéntico para todos los casos (usuario válido, inválido, bloqueado, inactivo, sin correo). Implementar tiempo de respuesta consistente añadiendo delay artificial si es necesario. No revelar información en auditoría pública | Mitigado |
| **Ataque de denegación de servicio (DoS) mediante solicitudes masivas** | Un atacante podría enviar múltiples solicitudes de recuperación para saturar el sistema de correos, llenar la cola de emails, o bloquear usuarios legítimos mediante agotamiento de límite | Alto | Media | Límite estricto de 5 solicitudes por usuario en 24 horas. Considerar implementar CAPTCHA o mecanismo anti-bot (reCAPTCHA v3) para detectar automatización. Monitoreo de patrones anómalos (múltiples solicitudes desde misma IP a diferentes usuarios). Rate limiting a nivel de infraestructura | Mitigado |
| **Correos no entregados o en spam** | Correos de recuperación pueden ser bloqueados por filtros anti-spam, no llegar al usuario, o tardar varios minutos en entregarse | Medio | Media | Configurar correctamente SPF, DKIM, DMARC en servidor de correos. Usar servicio de envío de correos confiable. Monitorear tasa de entrega (bounce rate). Incluir en pantalla de éxito: "Si no recibes el correo en 5 minutos, revisa tu carpeta de spam o solicita uno nuevo". Registrar en auditoría si envío de correo falla técnicamente | Aceptado con monitoreo |
| **Usuario sin acceso al correo registrado** | Usuario olvida contraseña pero ya no tiene acceso al correo electrónico registrado (cambio de empleo, correo deshabilitado, cuenta cerrada) | Medio | Media | Funcionalidad de recuperación por correo no aplica en este escenario. Usuario debe contactar a soporte para verificación de identidad alternativa y actualización de correo. Incluir en pantalla mensaje: "Si no tienes acceso a tu correo registrado, contacta a soporte: [email/teléfono]" | Aceptado - fuera de alcance |
| **Manipulación del enlace o token** | Un atacante podría intentar manipular el token en el enlace para acceder a otras cuentas, o intentar generar tokens válidos | Alto | Baja | Tokens generados con UUID v4 (128 bits, criptográficamente seguros, no predecibles). Tokens almacenados hasheados en base de datos. Validación estricta de integridad y formato del token (HU002B). Registrar en auditoría con severidad ERROR cualquier intento de acceso con token manipulado o inválido | Mitigado |
| **Fuga de información en correos interceptados** | Si correo es interceptado (phishing, compromiso de cuenta de email corporativo), un atacante puede ver el enlace y usarlo para cambiar la contraseña | Alto | Baja | Enlace válido solo por 15 minutos (ventana corta). Enlace de un solo uso (se invalida después de usarse o si se solicita nuevo). Registrar IP de solicitud y de uso del enlace para detección de anomalías. Notificar al usuario por correo cuando la contraseña sea cambiada (HU002C). Considerar autenticación de dos factores (2FA) en futuras versiones | Aceptado con controles |
| **Límite bloqueando usuarios legítimos** | Usuario legítimo que olvida su contraseña múltiples veces en un día puede quedar bloqueado por límite de 5 solicitudes | Bajo | Baja | Límite de 5 solicitudes en 24 horas es razonable para uso legítimo. Mensaje de error indica claramente que debe esperar o contactar a soporte. Soporte puede manualmente resetear el contador si el usuario verifica su identidad | Aceptado |

---

## Notas Adicionales

### Fuera de Alcance (Out of Scope)

**Esta HU NO incluye:**
- Validación del enlace de recuperación recibido por correo → **Cubierto en HU002B**
- Pantallas de error por enlace expirado, usado o inválido → **Cubierto en HU002B**
- Formulario de cambio de contraseña → **Cubierto en HU002C**
- Validación de requisitos de nueva contraseña → **Cubierto en HU002C**
- Validación de no reutilización de contraseñas → **Cubierto en HU002C**
- Invalidación de sesiones activas al cambiar contraseña → **Cubierto en HU002C**
- Recuperación de contraseña mediante SMS o autenticación telefónica → Futuras versiones
- Recuperación mediante preguntas de seguridad → Futuras versiones
- Autenticación de dos factores (2FA) → Futuras versiones
- Desbloqueo de cuenta mediante recuperación de contraseña → Ver HU001 para desbloqueo automático

### Dependencias

**Esta HU depende de:**
- **HU001 - Autenticación y Selección de Cliente**: Los usuarios que solicitan recuperación deben estar registrados en el sistema definido en HU001

**Esta HU es prerequisito para:**
- **HU002B - Validación de Enlace de Recuperación**: El token generado en HU002A es validado en HU002B
- **HU002C - Restablecimiento de Contraseña**: El flujo completo requiere HU002A → HU002B → HU002C

**Sistemas externos requeridos:**
- Sistema de Notificaciones por Correo Electrónico con capacidad de envío de correos HTML y plantillas personalizables
- Módulo de Auditoría implementado según Estándar de Auditoría Funcional

### Valor de Negocio Independiente

Esta HU puede ser entregada y probada de forma independiente. Aunque el flujo completo de recuperación requiere HU002B y HU002C, la funcionalidad de HU002A tiene valor por sí misma:
- ✅ Permite a los usuarios iniciar el proceso de recuperación
- ✅ Valida que el sistema protege contra enumeración de usuarios
- ✅ Valida que el sistema limita correctamente las solicitudes (anti-DoS)
- ✅ Permite probar y ajustar la plantilla de correo electrónico
- ✅ Permite validar que el sistema de envío de correos funciona correctamente
- ✅ Genera registros de auditoría que pueden ser consultados inmediatamente

### Referencias

- **Estándar de Auditoría Funcional**: Tipos de eventos `AUTENTICACION_RECUPERACION_*`, estructura obligatoria de 12 campos, niveles de severidad INFO/WARNING/ERROR
- **Estándar de Seguridad APL03**:
  - AP-0008: Respuestas genéricas en fallos de autenticación
  - AP-0019: Garantizar que quien realiza el proceso es humano (considerar CAPTCHA)
  - AP-0022: Registro de eventos de seguridad exitosos y no exitosos
  - AP-0024: Registro con timestamp, usuario, IPs, y contexto
- **Glosario**: Usuario, Sesión de Usuario, Bloqueo de Cuenta, Estado de Usuario, Trazabilidad
- **Guía de Estilos**: Material UI, paleta de colores Primary/Secondary, tipografía Roboto, componentes TextField/Button/Alert

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**HU Original:** HU002 - Recuperación de Contraseña (atomizada en HU002A, HU002B, HU002C)
