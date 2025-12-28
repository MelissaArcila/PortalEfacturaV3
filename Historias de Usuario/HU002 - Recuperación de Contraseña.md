# HU002 - Recuperación de Contraseña

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

Los Usuarios del Portal Unificado de CDN Facturación pueden olvidar su contraseña de acceso, lo que les impide autenticarse y operar en el sistema. Es necesario proporcionar un mecanismo seguro, autoservicio y auditable para que los Usuarios puedan restablecer su contraseña sin intervención del área de soporte.

El proceso debe garantizar la identidad del Usuario mediante validación del correo electrónico registrado en el sistema, asegurar que el enlace de recuperación sea temporal y de un solo uso, aplicar políticas de seguridad en la nueva contraseña, y registrar todos los eventos para cumplir con los estándares de auditoría y trazabilidad definidos en el ecosistema de facturación electrónica.

---

**Notas:**
- Este proceso aplica únicamente para Usuarios activos con correo electrónico registrado y verificado
- Usuarios bloqueados o inactivos NO pueden utilizar este mecanismo y deben contactar a soporte
- La recuperación de contraseña no desbloquea cuentas bloqueadas por intentos fallidos (ver HU001)
- Se debe aplicar el estándar de seguridad APL03 para protección de credenciales
- Todo el proceso debe ser auditable siguiendo el Estándar de Auditoría Funcional

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Usuario** | Quiero poder solicitar la recuperación de mi contraseña cuando la olvide | Para poder restablecer el acceso al Portal Unificado sin necesidad de contactar a soporte |
| Como **Usuario** | Quiero recibir un enlace temporal y seguro por correo electrónico | Para garantizar que solo yo pueda restablecer mi contraseña |
| Como **Usuario** | Quiero que el sistema valide que mi nueva contraseña cumpla con los requisitos de seguridad | Para asegurar que mi cuenta esté protegida contra accesos no autorizados |
| Como **Administrador** | Quiero que todos los eventos de recuperación de contraseña sean auditados | Para investigar incidentes de seguridad y garantizar trazabilidad |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Solicitud de recuperación exitosa | Usuario activo en pantalla de inicio de sesión, hace clic en "¿Olvidaste tu contraseña?" | Usuario ingresa su nombre de usuario o correo electrónico y hace clic en "Enviar enlace de recuperación" | Sistema muestra mensaje: "Si el usuario existe, recibirás un correo con instrucciones para recuperar tu contraseña". No revelar si el usuario existe o no | ☐ | ☐ | ☐ |
| 2 | Generación y envío de enlace temporal | Sistema valida que el usuario existe, está activo y tiene correo registrado | Sistema genera enlace temporal único | Sistema envía correo electrónico con enlace válido por 15 minutos. Asunto: "Recuperación de contraseña - Portal Unificado CDN". Cuerpo: incluye enlace, tiempo de expiración, advertencia de no compartir el enlace, información de contacto si no solicitó el cambio | ☐ | ☐ | ☐ |
| 3 | Acceso a enlace válido | Usuario recibe correo y hace clic en el enlace dentro de los 15 minutos | Usuario accede al enlace | Sistema muestra pantalla de "Restablecer Contraseña" con campos: nueva contraseña, confirmar contraseña, indicador visual de requisitos de contraseña, botón "Restablecer Contraseña" | ☐ | ☐ | ☐ |
| 4 | Cambio de contraseña exitoso | Usuario en pantalla de restablecimiento con enlace válido | Usuario ingresa nueva contraseña que cumple requisitos, confirma contraseña idéntica, y hace clic en "Restablecer Contraseña" | Sistema cambia la contraseña, invalida el enlace, muestra mensaje de éxito: "Tu contraseña ha sido actualizada correctamente. Redirigiendo a inicio de sesión...", y redirige automáticamente a pantalla de autenticación después de 3 segundos | ☐ | ☐ | ☐ |
| 5 | Validación de requisitos de contraseña | Usuario en pantalla de restablecimiento | Usuario ingresa nueva contraseña que NO cumple requisitos (menos de 8 caracteres, sin mayúsculas, sin números, sin símbolos) | Sistema muestra error en tiempo real (mientras escribe) debajo del campo: "La contraseña debe tener: mínimo 8 caracteres ✗, al menos una mayúscula ✗, al menos una minúscula ✓, al menos un número ✗, al menos un símbolo ✗". Botón "Restablecer Contraseña" permanece deshabilitado | ☐ | ☐ | ☐ |
| 6 | Confirmación de contraseña no coincide | Usuario en pantalla de restablecimiento | Usuario ingresa nueva contraseña válida pero la confirmación no coincide | Sistema muestra error debajo del campo "Confirmar contraseña": "Las contraseñas no coinciden". Botón permanece deshabilitado | ☐ | ☐ | ☐ |
| 7 | Nueva contraseña igual a la actual | Usuario en pantalla de restablecimiento | Usuario ingresa como nueva contraseña la misma contraseña actual | Sistema muestra error: "La nueva contraseña no puede ser igual a la contraseña actual". Botón permanece deshabilitado | ☐ | ☐ | ☐ |
| 8 | Nueva contraseña igual a contraseña reciente | Usuario en pantalla de restablecimiento | Usuario ingresa contraseña que fue utilizada en los últimos 5 cambios de contraseña | Sistema muestra error: "No puedes reutilizar tus últimas 5 contraseñas". Botón permanece deshabilitado | ☐ | ☐ | ☐ |
| 9 | Enlace expirado (más de 15 minutos) | Usuario hace clic en enlace después de 15 minutos de generado | Usuario accede al enlace | Sistema muestra pantalla de error: "Este enlace ha expirado. Por favor, solicita uno nuevo." con botón "Volver a inicio de sesión" | ☐ | ☐ | ☐ |
| 10 | Enlace ya utilizado | Usuario utiliza enlace que ya fue usado para cambiar contraseña exitosamente | Usuario accede al enlace | Sistema muestra pantalla de error: "Este enlace ya fue utilizado y no es válido. Si necesitas restablecer tu contraseña nuevamente, solicita un nuevo enlace." con botón "Volver a inicio de sesión" | ☐ | ☐ | ☐ |
| 11 | Enlace inválido o manipulado | Usuario accede a enlace con token inválido, corrupto o manipulado | Usuario accede al enlace | Sistema muestra pantalla de error: "Este enlace no es válido. Verifica que lo hayas copiado correctamente o solicita uno nuevo." con botón "Volver a inicio de sesión" | ☐ | ☐ | ☐ |
| 12 | Usuario bloqueado intenta recuperar contraseña | Usuario bloqueado por intentos fallidos de autenticación (ver HU001) ingresa su usuario en pantalla de recuperación | Usuario hace clic en "Enviar enlace de recuperación" | Sistema muestra mensaje genérico (no revela que está bloqueado): "Si el usuario existe, recibirás un correo con instrucciones para recuperar tu contraseña". Sistema NO envía correo. Sistema registra el intento en auditoría como WARNING | ☐ | ☐ | ☐ |
| 13 | Usuario inactivo intenta recuperar contraseña | Usuario inactivo ingresa su usuario en pantalla de recuperación | Usuario hace clic en "Enviar enlace de recuperación" | Sistema muestra mensaje genérico: "Si el usuario existe, recibirás un correo con instrucciones para recuperar tu contraseña". Sistema NO envía correo. Sistema registra el intento en auditoría como WARNING | ☐ | ☐ | ☐ |
| 14 | Usuario sin correo registrado | Usuario activo sin correo electrónico registrado en el sistema ingresa su usuario | Usuario hace clic en "Enviar enlace de recuperación" | Sistema muestra mensaje genérico: "Si el usuario existe, recibirás un correo con instrucciones para recuperar tu contraseña". Sistema NO envía correo. Sistema registra el intento en auditoría como WARNING | ☐ | ☐ | ☐ |
| 15 | Límite de solicitudes de recuperación excedido | Usuario ha solicitado recuperación de contraseña 5 veces en las últimas 24 horas | Usuario intenta solicitar recuperación nuevamente | Sistema muestra error: "Has excedido el número máximo de solicitudes de recuperación. Por favor, intenta nuevamente en 24 horas o contacta a soporte." No permite enviar más solicitudes | ☐ | ☐ | ☐ |
| 16 | Validación de formato de correo/usuario | Usuario en pantalla de solicitud de recuperación | Usuario ingresa formato inválido (espacios, caracteres especiales no permitidos, vacío) | Sistema valida formato en tiempo real y muestra error: "Ingresa un nombre de usuario o correo electrónico válido". Botón "Enviar enlace de recuperación" permanece deshabilitado | ☐ | ☐ | ☐ |
| 17 | Múltiples enlaces activos | Usuario solicita nuevo enlace de recuperación mientras tiene uno activo sin usar | Usuario hace clic en "Enviar enlace de recuperación" nuevamente | Sistema invalida todos los enlaces anteriores no utilizados, genera nuevo enlace único válido por 15 minutos, y envía nuevo correo. Muestra mensaje: "Si el usuario existe, recibirás un correo con instrucciones para recuperar tu contraseña" | ☐ | ☐ | ☐ |
| 18 | Indicador visual de requisitos de contraseña | Usuario en pantalla de restablecimiento | Usuario comienza a escribir en el campo "Nueva contraseña" | Sistema muestra indicador visual en tiempo real con checkmarks (✓) y cruces (✗): "Mínimo 8 caracteres ✗/✓", "Al menos una mayúscula ✗/✓", "Al menos una minúscula ✗/✓", "Al menos un número ✗/✓", "Al menos un símbolo (!@#$%^&*) ✗/✓", "No puede ser igual a contraseña actual ✗/✓", "No puede ser una de las últimas 5 contraseñas ✗/✓" | ☐ | ☐ | ☐ |
| 19 | Campo de contraseña con visibilidad | Usuario en pantalla de restablecimiento | Usuario escribe contraseña | Sistema muestra icono de "ojo" junto al campo de contraseña. Al hacer clic, muestra/oculta los caracteres de la contraseña para facilitar validación visual | ☐ | ☐ | ☐ |
| 20 | Cancelación del proceso | Usuario en pantalla de restablecimiento con enlace válido | Usuario hace clic en "Cancelar" o enlace "Volver a inicio de sesión" | Sistema redirige a pantalla de inicio de sesión. El enlace permanece válido hasta su expiración o uso exitoso | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 21 | Estructura obligatoria de registro de auditoría | Cualquier evento de recuperación de contraseña (solicitud, enlace enviado, acceso a enlace, cambio exitoso, cambio fallido, enlace expirado, límite excedido) | Sistema registra evento | Registro contiene: ID Único del Evento (UUID), Tipo de Evento (formato MODULO_ENTIDAD_ACCION), Fecha y Hora (UTC con milisegundos, ISO 8601), Usuario (nombre de usuario), Cliente (NIT - si aplica, NULL si no), Cliente (Nombre - si aplica, NULL si no), IP Local, IP Pública, Resultado (EXITOSO/FALLIDO), Descripción (texto descriptivo del evento), Nivel de Severidad (INFO/WARNING/ERROR), Datos Adicionales (JSON con contexto relevante) | ☐ | ☐ | ☐ |
| 22 | Auditoría de solicitud de recuperación exitosa | Usuario activo solicita recuperación de contraseña y se envía enlace | Usuario hace clic en "Enviar enlace de recuperación" | Registro de auditoría: Tipo de Evento: "AUTENTICACION_RECUPERACION_SOLICITADA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Usuario {usuario} solicitó recuperación de contraseña exitosamente", Datos Adicionales: {"correo_destino": "u***@dominio.com", "tiempo_expiracion_minutos": 15, "ip_solicitud": "IP"} | ☐ | ☐ | ☐ |
| 23 | Auditoría de solicitud fallida por usuario bloqueado | Usuario bloqueado intenta solicitar recuperación | Usuario hace clic en "Enviar enlace de recuperación" | Registro de auditoría: Tipo de Evento: "AUTENTICACION_RECUPERACION_BLOQUEADO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} bloqueado intentó solicitar recuperación de contraseña", Datos Adicionales: {"motivo_bloqueo": "intentos_fallidos", "fecha_desbloqueo_automatico": "timestamp"} | ☐ | ☐ | ☐ |
| 24 | Auditoría de solicitud fallida por usuario inactivo | Usuario inactivo intenta solicitar recuperación | Usuario hace clic en "Enviar enlace de recuperación" | Registro de auditoría: Tipo de Evento: "AUTENTICACION_RECUPERACION_INACTIVO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} inactivo intentó solicitar recuperación de contraseña", Datos Adicionales: {"estado_usuario": "inactivo"} | ☐ | ☐ | ☐ |
| 25 | Auditoría de solicitud fallida por usuario sin correo | Usuario sin correo registrado intenta solicitar recuperación | Usuario hace clic en "Enviar enlace de recuperación" | Registro de auditoría: Tipo de Evento: "AUTENTICACION_RECUPERACION_SIN_CORREO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} sin correo electrónico registrado intentó solicitar recuperación de contraseña", Datos Adicionales: {"estado_usuario": "activo", "correo_registrado": false} | ☐ | ☐ | ☐ |
| 26 | Auditoría de acceso a enlace de recuperación | Usuario accede a enlace de recuperación válido | Usuario hace clic en el enlace recibido por correo | Registro de auditoría: Tipo de Evento: "AUTENTICACION_ENLACE_ACCEDIDO", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Usuario {usuario} accedió a enlace de recuperación de contraseña", Datos Adicionales: {"token_id": "UUID del token", "tiempo_restante_minutos": 10, "ip_acceso": "IP"} | ☐ | ☐ | ☐ |
| 27 | Auditoría de acceso a enlace expirado | Usuario accede a enlace después de 15 minutos | Usuario hace clic en enlace expirado | Registro de auditoría: Tipo de Evento: "AUTENTICACION_ENLACE_EXPIRADO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} intentó acceder a enlace de recuperación expirado", Datos Adicionales: {"token_id": "UUID", "fecha_generacion": "timestamp", "fecha_expiracion": "timestamp", "fecha_acceso": "timestamp"} | ☐ | ☐ | ☐ |
| 28 | Auditoría de acceso a enlace ya utilizado | Usuario accede a enlace que ya fue usado exitosamente | Usuario hace clic en enlace ya usado | Registro de auditoría: Tipo de Evento: "AUTENTICACION_ENLACE_REUTILIZADO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} intentó reutilizar enlace de recuperación ya consumido", Datos Adicionales: {"token_id": "UUID", "fecha_uso_original": "timestamp", "ip_uso_original": "IP original", "ip_reuso": "IP actual"} | ☐ | ☐ | ☐ |
| 29 | Auditoría de acceso a enlace inválido | Usuario accede a enlace con token inválido o manipulado | Usuario hace clic en enlace inválido | Registro de auditoría: Tipo de Evento: "AUTENTICACION_ENLACE_INVALIDO", Resultado: "FALLIDO", Severidad: "ERROR", Descripción: "Intento de acceso con enlace de recuperación inválido para usuario {usuario}", Datos Adicionales: {"token_recibido": "token truncado", "ip_acceso": "IP", "posible_manipulacion": true} | ☐ | ☐ | ☐ |
| 30 | Auditoría de cambio de contraseña exitoso | Usuario completa proceso de cambio de contraseña con enlace válido | Usuario hace clic en "Restablecer Contraseña" con datos válidos | Registro de auditoría: Tipo de Evento: "AUTENTICACION_CONTRASENA_CAMBIADA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Usuario {usuario} cambió contraseña exitosamente mediante recuperación", Datos Adicionales: {"token_id": "UUID", "metodo": "recuperacion_correo", "ip_cambio": "IP"} | ☐ | ☐ | ☐ |
| 31 | Auditoría de cambio de contraseña fallido por requisitos | Usuario intenta cambiar contraseña pero no cumple requisitos | Usuario hace clic en "Restablecer Contraseña" con contraseña inválida | Registro de auditoría: Tipo de Evento: "AUTENTICACION_CONTRASENA_REQUISITOS_INVALIDOS", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} intentó cambiar contraseña con requisitos inválidos", Datos Adicionales: {"requisitos_incumplidos": ["longitud_minima", "sin_mayusculas", "sin_numeros"], "ip_intento": "IP"} | ☐ | ☐ | ☐ |
| 32 | Auditoría de cambio de contraseña fallido por reutilización | Usuario intenta cambiar a contraseña igual a una de las últimas 5 | Usuario hace clic en "Restablecer Contraseña" con contraseña reutilizada | Registro de auditoría: Tipo de Evento: "AUTENTICACION_CONTRASENA_REUTILIZADA", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} intentó reutilizar contraseña reciente", Datos Adicionales: {"posicion_en_historial": 3, "politica_no_reutilizar": 5, "ip_intento": "IP"} | ☐ | ☐ | ☐ |
| 33 | Auditoría de límite de solicitudes excedido | Usuario supera 5 solicitudes de recuperación en 24 horas | Usuario hace clic en "Enviar enlace de recuperación" por 6ta vez en 24 horas | Registro de auditoría: Tipo de Evento: "AUTENTICACION_RECUPERACION_LIMITE_EXCEDIDO", Resultado: "FALLIDO", Severidad: "ERROR", Descripción: "Usuario {usuario} excedió límite de solicitudes de recuperación de contraseña (5 en 24 horas)", Datos Adicionales: {"intentos_en_periodo": 5, "periodo_horas": 24, "ip_intento": "IP", "intentos_anteriores": [{"timestamp": "...", "ip": "..."}, ...]} | ☐ | ☐ | ☐ |
| 34 | Auditoría de invalidación de enlaces anteriores | Usuario solicita nuevo enlace mientras tiene uno activo | Usuario hace clic en "Enviar enlace de recuperación" nuevamente | Registro de auditoría: Tipo de Evento: "AUTENTICACION_ENLACES_INVALIDADOS", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Usuario {usuario} solicitó nuevo enlace de recuperación, invalidando enlaces anteriores", Datos Adicionales: {"tokens_invalidados": ["UUID1", "UUID2"], "nuevo_token": "UUID3", "ip_solicitud": "IP"} | ☐ | ☐ | ☐ |
| 35 | Inmutabilidad de registros de auditoría | Cualquier escenario de auditoría | Sistema genera registro | Registros de auditoría NO pueden ser modificados ni eliminados. Solo se permiten consultas de lectura. Implementar controles que prevengan alteración de registros históricos | ☐ | ☐ | ☐ |
| 36 | Consulta de registros de auditoría para investigación | Administrador o auditor requiere investigar eventos de recuperación de contraseña | Administrador accede a módulo de auditoría | Sistema permite filtrar por: rango de fechas, usuario, tipo de evento (AUTENTICACION_RECUPERACION_*, AUTENTICACION_CONTRASENA_*, AUTENTICACION_ENLACE_*), resultado (EXITOSO/FALLIDO), severidad (INFO/WARNING/ERROR), IP origen. Permite exportar resultados en formato CSV/Excel para análisis | ☐ | ☐ | ☐ |
| 37 | Retención de registros de auditoría | Registros de auditoría de recuperación de contraseña | Paso del tiempo | Registros se conservan por tiempo mínimo definido por regulación (7 años para documentos fiscales, aplicar misma política para eventos de seguridad). Después de este período, pueden ser archivados en almacenamiento de largo plazo o eliminados según política de retención | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

Se requieren los siguientes mockups interactivos en HTML siguiendo la Guía de Estilos (Material UI, paleta de colores Primary #4A5A9E, Secondary #D4145A):

#### 1. **HU002-ForgotPassword.html** - Pantalla de solicitud de recuperación
**Elementos:**
- Logotipo de CDN Facturación centrado en parte superior
- Título: "¿Olvidaste tu contraseña?" (Tipografía H3, color Primary Dark)
- Texto descriptivo: "Ingresa tu nombre de usuario o correo electrónico y te enviaremos un enlace para recuperar tu contraseña" (Body2, color gris-700)
- Campo de texto (TextField outlined): "Usuario o correo electrónico" con validación en tiempo real
- Botón primario (contained): "Enviar enlace de recuperación" (gradiente Primary → Secondary, deshabilitado si campo vacío o inválido)
- Enlace secundario: "Volver a inicio de sesión" (texto, color Primary)
- Mensaje de éxito (Alert success, oculto por defecto): "Si el usuario existe, recibirás un correo con instrucciones para recuperar tu contraseña"
- Mensaje de error por límite (Alert error, oculto por defecto): "Has excedido el número máximo de solicitudes de recuperación. Por favor, intenta nuevamente en 24 horas o contacta a soporte"

#### 2. **HU002-ResetPassword.html** - Pantalla de restablecimiento de contraseña
**Elementos:**
- Logotipo de CDN Facturación centrado en parte superior
- Título: "Restablecer contraseña" (Tipografía H3, color Primary Dark)
- Texto descriptivo: "Ingresa tu nueva contraseña. Debe cumplir con los requisitos de seguridad." (Body2, color gris-700)
- Campo de texto tipo password (TextField outlined): "Nueva contraseña" con icono de ojo para mostrar/ocultar
- Indicador visual de requisitos en tiempo real:
  - Lista con checkmarks (✓ verde) y cruces (✗ rojo) según cumplimiento:
    - "Mínimo 8 caracteres"
    - "Al menos una mayúscula (A-Z)"
    - "Al menos una minúscula (a-z)"
    - "Al menos un número (0-9)"
    - "Al menos un símbolo (!@#$%^&*)"
    - "No puede ser igual a contraseña actual"
    - "No puede ser una de las últimas 5 contraseñas"
- Campo de texto tipo password (TextField outlined): "Confirmar contraseña" con icono de ojo
- Mensaje de error de confirmación (texto rojo, oculto por defecto): "Las contraseñas no coinciden"
- Barra de fortaleza de contraseña (LinearProgress con colores: débil=error, media=warning, fuerte=success)
- Botón primario (contained): "Restablecer Contraseña" (gradiente Primary → Secondary, deshabilitado hasta que todos los requisitos se cumplan)
- Botón secundario (outlined): "Cancelar"
- Mensaje de éxito (Alert success, oculto por defecto): "Tu contraseña ha sido actualizada correctamente. Redirigiendo a inicio de sesión..." con icono de éxito

#### 3. **HU002-LinkExpired.html** - Pantalla de enlace expirado/inválido/usado
**Elementos:**
- Logotipo de CDN Facturación centrado en parte superior
- Icono de advertencia (warning_amber) grande en color Warning
- Título dinámico (H3, color Primary Dark):
  - "Enlace expirado" (si >15 minutos)
  - "Enlace ya utilizado" (si ya fue usado)
  - "Enlace inválido" (si token inválido)
- Texto descriptivo dinámico (Body1, color gris-700):
  - "Este enlace ha expirado. Por favor, solicita uno nuevo."
  - "Este enlace ya fue utilizado y no es válido. Si necesitas restablecer tu contraseña nuevamente, solicita un nuevo enlace."
  - "Este enlace no es válido. Verifica que lo hayas copiado correctamente o solicita uno nuevo."
- Botón primario (contained): "Solicitar nuevo enlace" (redirección a HU002-ForgotPassword)
- Botón secundario (text): "Volver a inicio de sesión"

#### 4. **HU002-EmailTemplate.html** - Plantilla de correo electrónico
**Elementos:**
- Encabezado con logo de CDN Facturación y fondo con gradiente Primary → Secondary
- Saludo personalizado: "Hola {nombre_usuario},"
- Cuerpo del mensaje:
  - "Recibimos una solicitud para restablecer la contraseña de tu cuenta en el Portal Unificado de CDN Facturación."
  - "Haz clic en el botón a continuación para crear una nueva contraseña:"
- Botón prominente (CTA): "Restablecer mi contraseña" (enlace al token de recuperación)
- Información de seguridad:
  - "Este enlace es válido por 15 minutos y solo puede usarse una vez."
  - "Si no solicitaste este cambio, ignora este correo y tu contraseña permanecerá sin cambios."
  - "Por tu seguridad, nunca compartas este enlace con nadie."
- Enlace alternativo (texto): Si el botón no funciona, copia y pega este enlace: {url_completa}
- Pie de correo:
  - "Soporte CDN Facturación"
  - Información de contacto
  - Aviso legal: "Este es un correo automático, por favor no respondas a este mensaje."

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Enumeración de usuarios** | Un atacante podría intentar enumerar usuarios válidos del sistema observando diferencias en las respuestas o tiempos de respuesta entre usuarios existentes e inexistentes | Alto | Media | Sistema responde con mensaje genérico idéntico independientemente de si el usuario existe o no: "Si el usuario existe, recibirás un correo con instrucciones". Tiempo de respuesta consistente (delay artificial si es necesario) | Mitigado |
| **Ataque de denegación de servicio (DoS) mediante solicitudes masivas** | Un atacante podría enviar múltiples solicitudes de recuperación para saturar el sistema de correos o bloquear usuarios legítimos | Alto | Media | Límite de 5 solicitudes por usuario en 24 horas. Implementar CAPTCHA o mecanismo anti-bot en pantalla de solicitud. Monitoreo de patrones anómalos (múltiples solicitudes desde misma IP) | Mitigado |
| **Intercepción de enlace de recuperación** | Si un atacante tiene acceso al correo del usuario (phishing, compromiso de cuenta de correo), puede interceptar el enlace y cambiar la contraseña | Alto | Baja | Enlace válido solo por 15 minutos. Enlace de un solo uso. Notificar al usuario por correo cuando la contraseña sea cambiada exitosamente. Registrar IP de acceso y cambio para detección de anomalías. Considerar autenticación de dos factores (2FA) en futuras versiones | Aceptado con controles |
| **Reutilización de contraseñas débiles o comunes** | Usuario puede ingresar contraseña que cumple requisitos técnicos pero es débil o común (ej: Password1!, Qwerty123!) | Medio | Media | Validar contra diccionario de contraseñas comunes (ej: lista de top 10,000 contraseñas más usadas). Mostrar barra de fortaleza de contraseña en tiempo real. Rechazar contraseñas en lista negra con mensaje: "Esta contraseña es muy común, elige una más segura" | Mitigado |
| **Manipulación del token de recuperación** | Un atacante podría intentar manipular o predecir tokens para acceder a cuentas sin autorización | Alto | Baja | Tokens generados con UUID v4 (128 bits, criptográficamente seguros, no predecibles). Validación estricta de integridad del token. Registro de auditoría de accesos inválidos con nivel ERROR para detección de ataques | Mitigado |
| **Usuario sin acceso a correo registrado** | Usuario olvida contraseña pero ya no tiene acceso al correo electrónico registrado en el sistema (cambio de empleo, correo deshabilitado) | Medio | Media | Escenario fuera de alcance de autoservicio. Usuario debe contactar a soporte para verificación de identidad alternativa (documento de identidad, verificación telefónica, etc.) y actualización de correo. Incluir link de "No tengo acceso a mi correo" que muestre información de contacto de soporte | Aceptado |
| **Sincronización de cambio de contraseña en múltiples sesiones activas** | Si el usuario tiene sesiones activas en múltiples dispositivos cuando cambia su contraseña, esas sesiones deberían ser invalidadas por seguridad | Medio | Baja | Al cambiar contraseña exitosamente, invalidar todos los tokens de sesión activos del usuario (cerrar todas las sesiones abiertas). Forzar nueva autenticación en todos los dispositivos. Notificar al usuario que deberá iniciar sesión nuevamente en todos sus dispositivos | Mitigado |
| **Fuga de información en registros de auditoría** | Registros de auditoría podrían contener información sensible (contraseñas en claro, tokens completos) si no se diseñan correctamente | Alto | Baja | Registros NO deben contener: contraseñas (ni actual ni nueva), tokens completos de recuperación. Solo registrar: eventos, resultados, IPs, timestamps, identificadores de tokens truncados. Control de acceso estricto a módulo de auditoría (solo administradores autorizados) | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance (Out of Scope)
- Recuperación de contraseña mediante SMS o autenticación telefónica
- Recuperación mediante preguntas de seguridad
- Autenticación de dos factores (2FA) para validación adicional durante recuperación (se considerará en futuras versiones)
- Desbloqueo de cuenta mediante proceso de recuperación (cuentas bloqueadas requieren soporte o desbloqueo automático según HU001)
- Cambio de correo electrónico asociado a la cuenta desde pantalla de recuperación
- Gestión de contraseñas para usuarios administradores (podría tener flujo diferente con aprobaciones)

### Consideraciones Técnicas para Implementación (Informativas)
- Integrar con servicio de envío de correos electrónicos con plantillas HTML responsivas
- Considerar uso de cola de mensajes (queue) para envío asíncrono de correos y evitar timeout en UI
- Almacenar hash de contraseñas con algoritmo seguro (bcrypt, Argon2) con salt único por usuario
- Almacenar historial de últimas 5 contraseñas hasheadas para validación de no reutilización
- Tokens de recuperación deben almacenarse hasheados en base de datos con fecha de expiración e indicador de uso
- Implementar índices en tablas de auditoría para consultas eficientes por fecha, usuario, tipo de evento
- Considerar limpieza automática (job programado) de tokens expirados y no usados después de 24-48 horas

### Dependencias
- **HU001 - Autenticación y Selección de Cliente**: Recuperación de contraseña se aplica a usuarios que existen en el sistema definido en HU001
- **Sistema de Notificaciones por Correo**: Debe existir capacidad de envío de correos electrónicos con plantillas personalizables
- **Módulo de Auditoría**: Debe estar implementado el sistema de auditoría siguiendo el Estándar de Auditoría Funcional

### Referencias
- **Estándar de Auditoría Funcional**: Seguir estructura obligatoria de 13 campos, principios de inmutabilidad, formato de tipos de eventos, niveles de severidad
- **Estándar de Seguridad APL03**: Aplicar controles de seguridad para credenciales, bloqueos, auditoría de eventos de seguridad
- **Glosario**: Usuario, Sesión de Usuario, Bloqueo de Cuenta, Trazabilidad, No Repudio
- **Guía de Estilos**: Material UI, paleta de colores Primary/Secondary, tipografía Roboto, componentes TextField/Button/Alert/LinearProgress
