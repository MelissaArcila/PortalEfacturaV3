# HU002C - Restablecimiento de Contraseña

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
| **Arquitectura relacionada** | Portal Unificado - Módulo de Autenticación, Gestión de Sesiones |

---

## Contexto

Esta historia cubre el **tercer y último componente del flujo de recuperación de contraseña**: el formulario de restablecimiento de contraseña y el cambio efectivo de la misma.

Después de que el Usuario accedió a un enlace válido (validado en HU002B), el sistema muestra un formulario donde el Usuario ingresa su nueva contraseña. El sistema debe validar en tiempo real que la contraseña cumple con requisitos de seguridad (longitud mínima, complejidad, no reutilización de últimas 5 contraseñas), permitir confirmar la contraseña para evitar errores de tipeo, cambiar la contraseña efectivamente, invalidar el token usado, cerrar todas las sesiones activas del usuario, y notificar al usuario por correo del cambio exitoso.

Esta funcionalidad **completa el flujo de recuperación de contraseña** y permite al Usuario recuperar el acceso a su cuenta de forma segura y autoservicio.

---

**Notas:**
- Esta HU cubre SOLO el formulario de cambio de contraseña y el cambio efectivo. La solicitud de enlace se cubre en HU002A y la validación del enlace en HU002B
- Las contraseñas deben cumplir requisitos de seguridad: mínimo 8 caracteres, mayúscula, minúscula, número, símbolo
- El sistema NO permite reutilizar las últimas 5 contraseñas del usuario
- El sistema NO permite usar la contraseña actual como nueva contraseña
- Después del cambio exitoso, todas las sesiones activas del usuario se invalidan (forzar reautenticación)
- El usuario recibe notificación por correo del cambio de contraseña
- Todo el proceso debe ser auditable siguiendo el Estándar de Auditoría Funcional

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Usuario** | Quiero ver un formulario para ingresar mi nueva contraseña | Para poder establecer una contraseña segura que recuerde |
| Como **Usuario** | Quiero ver indicadores visuales en tiempo real de los requisitos de contraseña | Para saber inmediatamente si mi contraseña cumple con los requisitos sin tener que enviar el formulario |
| Como **Usuario** | Quiero confirmar mi nueva contraseña ingresándola dos veces | Para evitar errores de tipeo y asegurarme de que recordaré la contraseña correcta |
| Como **Usuario** | Quiero poder mostrar/ocultar los caracteres de la contraseña | Para poder verificar visualmente que escribí la contraseña correcta |
| Como **Usuario** | Quiero recibir confirmación visual cuando mi contraseña sea cambiada exitosamente | Para tener certeza de que el cambio fue efectivo y poder ingresar al sistema |
| Como **Administrador de Seguridad** | Quiero que el sistema invalide automáticamente todas las sesiones activas del usuario al cambiar su contraseña | Para garantizar que si un atacante tenía acceso, sea expulsado inmediatamente |
| Como **Administrador** | Quiero que todos los eventos de cambio de contraseña (exitosos y fallidos) sean auditados | Para investigar incidentes de seguridad y garantizar trazabilidad |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Visualización del formulario de restablecimiento | Usuario accedió exitosamente a enlace válido (HU002B escenario 1) | Sistema navega a pantalla de restablecimiento | Sistema muestra formulario con: título "Restablecer contraseña", texto explicativo "Ingresa tu nueva contraseña. Debe cumplir con los requisitos de seguridad.", campo "Nueva contraseña" (tipo password con icono ojo), campo "Confirmar contraseña" (tipo password con icono ojo), indicador visual de requisitos (lista con checkmarks), barra de fortaleza de contraseña, botón "Restablecer Contraseña" (deshabilitado inicialmente), botón "Cancelar" | ☐ | ☐ | ☐ |
| 2 | Indicador visual de requisitos en tiempo real | Usuario en formulario de restablecimiento | Usuario escribe en campo "Nueva contraseña" | Sistema muestra lista de requisitos con iconos en tiempo real: "✓ Mínimo 8 caracteres" (verde si cumple, rojo si no), "✓ Al menos una mayúscula (A-Z)", "✓ Al menos una minúscula (a-z)", "✓ Al menos un número (0-9)", "✓ Al menos un símbolo (!@#$%^&*)", "✓ No puede ser igual a contraseña actual", "✓ No puede ser una de las últimas 5 contraseñas". Cada requisito se actualiza dinámicamente mientras el usuario escribe | ☐ | ☐ | ☐ |
| 3 | Barra de fortaleza de contraseña | Usuario escribe en campo "Nueva contraseña" | Sistema calcula fortaleza | Sistema muestra LinearProgress debajo del campo con colores: Débil (0-40%, rojo Error), Media (41-70%, naranja Warning), Fuerte (71-100%, verde Success). Cálculo basado en: longitud, diversidad de caracteres, no estar en lista de contraseñas comunes. Texto descriptivo: "Fortaleza: Débil/Media/Fuerte" | ☐ | ☐ | ☐ |
| 4 | Mostrar/ocultar contraseña | Usuario en formulario de restablecimiento | Usuario hace clic en icono de ojo junto al campo de contraseña | Sistema alterna entre mostrar caracteres en texto plano y ocultarlos con puntos/asteriscos. Icono cambia entre "visibility" (muestra) y "visibility_off" (oculta). Aplica a ambos campos: "Nueva contraseña" y "Confirmar contraseña" (cada uno con su propio toggle independiente) | ☐ | ☐ | ☐ |
| 5 | Validación de confirmación de contraseña en tiempo real | Usuario escribe en campo "Confirmar contraseña" | Usuario escribe y campo pierde foco (blur) | Sistema compara "Nueva contraseña" con "Confirmar contraseña". Si NO coinciden, muestra error debajo del campo: "Las contraseñas no coinciden" (color Error). Si coinciden, muestra checkmark verde. Botón "Restablecer Contraseña" permanece deshabilitado mientras no coincidan | ☐ | ☐ | ☐ |
| 6 | Cambio de contraseña exitoso | Usuario completa formulario correctamente: nueva contraseña cumple todos los requisitos, confirmación coincide, contraseña no es igual a actual ni a últimas 5 | Usuario hace clic en "Restablecer Contraseña" | Sistema: (1) Valida requisitos en backend, (2) Cambia contraseña del usuario (hashea con bcrypt/Argon2 y salt), (3) Almacena hash en historial de últimas 5 contraseñas, (4) Marca token como USADO, (5) Invalida todas las sesiones activas del usuario, (6) Envía correo de notificación al usuario, (7) Muestra mensaje de éxito: "Tu contraseña ha sido actualizada correctamente. Redirigiendo a inicio de sesión...", (8) Redirige automáticamente a login (HU001) después de 3 segundos, (9) Registra evento en auditoría | ☐ | ☐ | ☐ |
| 7 | Nueva contraseña no cumple requisitos | Usuario ingresa contraseña que NO cumple requisitos (menos de 8 caracteres, sin mayúscula, sin número, etc.) | Usuario intenta hacer clic en "Restablecer Contraseña" | Sistema mantiene botón deshabilitado mientras algún requisito no se cumpla (indicadores muestran ✗ rojos). Si el usuario intenta forzar envío, sistema valida en backend y rechaza con error: "La contraseña no cumple con los requisitos de seguridad" listando requisitos incumplidos | ☐ | ☐ | ☐ |
| 8 | Confirmación de contraseña no coincide | Usuario ingresa nueva contraseña válida pero la confirmación es diferente | Usuario hace clic en "Restablecer Contraseña" | Sistema valida y detecta que no coinciden. Muestra error debajo del campo "Confirmar contraseña": "Las contraseñas no coinciden". Botón permanece deshabilitado o muestra error si ya se habilitó. No envía al backend | ☐ | ☐ | ☐ |
| 9 | Nueva contraseña igual a contraseña actual | Usuario ingresa como nueva contraseña la misma contraseña actual que tiene | Usuario hace clic en "Restablecer Contraseña" | Sistema valida en backend (compara hash nuevo con hash actual). Muestra error: "La nueva contraseña no puede ser igual a la contraseña actual. Elige una contraseña diferente." Indicador visual de requisito "No puede ser igual a contraseña actual" muestra ✗ rojo. Registra evento en auditoría con severidad WARNING | ☐ | ☐ | ☐ |
| 10 | Nueva contraseña igual a una de las últimas 5 contraseñas | Usuario ingresa contraseña que fue utilizada en los últimos 5 cambios de contraseña | Usuario hace clic en "Restablecer Contraseña" | Sistema valida en backend (compara hash nuevo con hashes de historial de últimas 5). Muestra error: "No puedes reutilizar tus últimas 5 contraseñas. Elige una contraseña diferente." Indicador visual de requisito "No puede ser una de las últimas 5" muestra ✗ rojo. Registra evento en auditoría con severidad WARNING | ☐ | ☐ | ☐ |
| 11 | Contraseña común o en lista negra | Usuario ingresa contraseña que cumple requisitos técnicos pero es muy común (ej: "Password1!", "Qwerty123!", "Welcome2024!") | Usuario hace clic en "Restablecer Contraseña" | Sistema valida contra diccionario de contraseñas comunes (top 10,000 más usadas). Si está en lista negra, muestra error: "Esta contraseña es muy común y fácil de adivinar. Elige una contraseña más segura y única." Barra de fortaleza muestra "Débil" en rojo. Permite continuar si usuario insiste, pero con advertencia clara | ☐ | ☐ | ☐ |
| 12 | Cancelación del proceso | Usuario en formulario de restablecimiento | Usuario hace clic en botón "Cancelar" | Sistema muestra diálogo de confirmación: "¿Estás seguro que deseas cancelar el cambio de contraseña? Tu contraseña actual no será modificada." Botones: "Continuar editando", "Sí, cancelar". Si confirma, redirige a login (HU001). El token permanece válido (hasta expirar o usarse) | ☐ | ☐ | ☐ |
| 13 | Invalidación de sesiones activas | Usuario cambia contraseña exitosamente y tenía sesiones activas en múltiples dispositivos (PC, móvil, tablet) | Sistema cambia contraseña | Sistema invalida TODAS las sesiones activas del usuario: elimina/invalida todos los tokens de sesión existentes en base de datos/caché. Si el usuario está autenticado en otros dispositivos, al intentar hacer cualquier acción, el sistema detecta sesión inválida y muestra mensaje: "Tu sesión ha expirado porque la contraseña fue cambiada. Por favor, inicia sesión nuevamente." y redirige a login | ☐ | ☐ | ☐ |
| 14 | Notificación por correo de cambio exitoso | Usuario cambia contraseña exitosamente | Sistema envía correo | Sistema envía correo electrónico al correo registrado del usuario con: asunto "Contraseña actualizada - Portal Unificado CDN", cuerpo: saludo, confirmación de cambio de contraseña, fecha/hora del cambio, IP desde donde se realizó, advertencia de seguridad "Si no realizaste este cambio, contacta a soporte inmediatamente: [email/teléfono]", botón "Iniciar sesión" (enlace a login), pie de correo | ☐ | ☐ | ☐ |
| 15 | Token ya usado después de cambio exitoso | Usuario cambia contraseña exitosamente y luego intenta volver a usar el mismo enlace | Usuario vuelve a acceder al enlace del correo | Sistema detecta que token está marcado como USADO (marcado en escenario 6). Sistema navega a pantalla de error "Enlace ya utilizado" de HU002B. No permite acceder al formulario de restablecimiento nuevamente | ☐ | ☐ | ☐ |
| 16 | Indicador de carga durante procesamiento | Usuario completa formulario correctamente y hace clic en "Restablecer Contraseña" | Sistema procesa cambio (tarda 1-3 segundos por validaciones, hasheo, almacenamiento, envío de correo) | Sistema muestra indicador de carga (spinner) en el botón "Restablecer Contraseña", deshabilita el botón temporalmente, y muestra texto "Restableciendo..." hasta que el proceso termina y muestra mensaje de éxito | ☐ | ☐ | ☐ |
| 17 | Redirección automática después de éxito | Usuario ve mensaje de éxito "Tu contraseña ha sido actualizada correctamente. Redirigiendo a inicio de sesión..." | Sistema muestra mensaje | Sistema muestra contador regresivo visual (3, 2, 1) y después de 3 segundos redirige automáticamente a pantalla de login (HU001). Usuario puede hacer clic en botón "Ir a inicio de sesión ahora" para omitir espera | ☐ | ☐ | ☐ |
| 18 | Primer uso de nueva contraseña | Usuario cambió contraseña exitosamente, fue redirigido a login, e intenta autenticarse con la nueva contraseña | Usuario ingresa sus credenciales en HU001 | Sistema valida credenciales usando el nuevo hash de contraseña. Autenticación exitosa procede normalmente según flujo HU001 (usuario con un solo cliente o selección de cliente). Registra evento de autenticación exitosa en auditoría | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 19 | Estructura obligatoria de registro de auditoría | Cualquier evento de restablecimiento de contraseña (exitoso, fallido, cancelado) | Sistema registra evento | Registro contiene: ID Único del Evento (UUID), Tipo de Evento (AUTENTICACION_CONTRASENA_*), Fecha y Hora (UTC con milisegundos, ISO 8601), Usuario (nombre de usuario), Cliente (NULL), Cliente Nombre (NULL), IP Local, IP Pública, Resultado (EXITOSO/FALLIDO), Descripción (texto descriptivo del evento), Nivel de Severidad (INFO/WARNING/ERROR), Datos Adicionales (JSON con contexto relevante) | ☐ | ☐ | ☐ |
| 20 | Auditoría de cambio de contraseña exitoso | Usuario completa proceso de cambio de contraseña con enlace válido | Usuario hace clic en "Restablecer Contraseña" con datos válidos (escenario 6) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_CONTRASENA_CAMBIADA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Usuario {usuario} cambió contraseña exitosamente mediante recuperación", Datos Adicionales: {"token_id": "UUID del token usado", "metodo": "recuperacion_correo", "ip_cambio_local": "IP local", "ip_cambio_publica": "IP pública", "fortaleza_contraseña": "fuerte", "sesiones_invalidadas_count": 3, "correo_notificacion_enviado": true} | ☐ | ☐ | ☐ |
| 21 | Auditoría de intento fallido por requisitos inválidos | Usuario intenta cambiar contraseña pero no cumple requisitos de seguridad | Usuario hace clic en "Restablecer Contraseña" (escenario 7) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_CONTRASENA_REQUISITOS_INVALIDOS", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} intentó cambiar contraseña con requisitos inválidos", Datos Adicionales: {"token_id": "UUID", "requisitos_incumplidos": ["longitud_minima", "sin_mayusculas", "sin_numeros"], "fortaleza_contraseña": "debil", "ip_intento_local": "IP local", "ip_intento_publica": "IP pública"} | ☐ | ☐ | ☐ |
| 22 | Auditoría de intento fallido por contraseña igual a actual | Usuario intenta cambiar a contraseña igual a la contraseña actual | Usuario hace clic en "Restablecer Contraseña" (escenario 9) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_CONTRASENA_IGUAL_ACTUAL", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} intentó cambiar a contraseña igual a la actual", Datos Adicionales: {"token_id": "UUID", "ip_intento_local": "IP local", "ip_intento_publica": "IP pública"} | ☐ | ☐ | ☐ |
| 23 | Auditoría de intento fallido por reutilización de contraseña | Usuario intenta cambiar a contraseña igual a una de las últimas 5 | Usuario hace clic en "Restablecer Contraseña" (escenario 10) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_CONTRASENA_REUTILIZADA", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} intentó reutilizar contraseña reciente", Datos Adicionales: {"token_id": "UUID", "posicion_en_historial": 3, "politica_no_reutilizar": 5, "ip_intento_local": "IP local", "ip_intento_publica": "IP pública"} | ☐ | ☐ | ☐ |
| 24 | Auditoría de intento con contraseña común | Usuario intenta cambiar a contraseña común o en lista negra | Usuario hace clic en "Restablecer Contraseña" (escenario 11) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_CONTRASENA_COMUN_DETECTADA", Resultado: "EXITOSO" (si se permitió con advertencia), Severidad: "WARNING", Descripción: "Usuario {usuario} estableció contraseña común a pesar de advertencia", Datos Adicionales: {"token_id": "UUID", "contraseña_en_lista_negra": true, "fortaleza_contraseña": "debil", "advertencia_mostrada": true, "ip_cambio_local": "IP local", "ip_cambio_publica": "IP pública"} | ☐ | ☐ | ☐ |
| 25 | Auditoría de cancelación del proceso | Usuario cancela proceso de cambio de contraseña | Usuario confirma cancelación (escenario 12) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_CONTRASENA_CAMBIO_CANCELADO", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Usuario {usuario} canceló proceso de cambio de contraseña", Datos Adicionales: {"token_id": "UUID", "porcentaje_completado": "formulario_llenado_50%", "ip_cancelacion_local": "IP local", "ip_cancelacion_publica": "IP pública"} | ☐ | ☐ | ☐ |
| 26 | Auditoría de invalidación de sesiones | Usuario cambia contraseña exitosamente e invalida sesiones activas | Sistema invalida sesiones (escenario 13) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_SESIONES_INVALIDADAS", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Sesiones activas de usuario {usuario} invalidadas por cambio de contraseña", Datos Adicionales: {"usuario_id": "UUID", "sesiones_invalidadas_count": 3, "sesiones_invalidadas": [{"sesion_id": "UUID1", "dispositivo": "Chrome/Windows", "ultima_actividad": "timestamp", "ip": "IP1"}, {"sesion_id": "UUID2", "dispositivo": "Safari/iOS", "ultima_actividad": "timestamp", "ip": "IP2"}, ...], "motivo": "cambio_contraseña"} | ☐ | ☐ | ☐ |
| 27 | Auditoría de notificación por correo enviada | Usuario cambia contraseña exitosamente y se envía correo de notificación | Sistema envía correo (escenario 14) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_NOTIFICACION_CAMBIO_ENVIADA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Notificación de cambio de contraseña enviada a usuario {usuario}", Datos Adicionales: {"usuario_id": "UUID", "correo_destino_parcial": "u***@dominio.com", "fecha_hora_cambio": "timestamp", "ip_cambio": "IP pública", "correo_enviado_exitosamente": true} | ☐ | ☐ | ☐ |
| 28 | Inmutabilidad de registros de auditoría | Cualquier escenario de auditoría | Sistema genera registro | Registros de auditoría NO pueden ser modificados ni eliminados después de su creación. Solo se permiten consultas de lectura. Implementar controles que prevengan alteración de registros históricos | ☐ | ☐ | ☐ |
| 29 | Consulta de registros de auditoría para investigación | Administrador o auditor requiere investigar eventos de cambio de contraseña | Administrador accede a módulo de auditoría | Sistema permite filtrar por: rango de fechas, usuario, tipo de evento (AUTENTICACION_CONTRASENA_*, AUTENTICACION_SESIONES_*), resultado (EXITOSO/FALLIDO), severidad (INFO/WARNING/ERROR), IP origen, token_id específico. Permite correlacionar flujo completo: solicitud (HU002A) → validación enlace (HU002B) → cambio contraseña (HU002C) por mismo token_id. Permite exportar resultados CSV/Excel | ☐ | ☐ | ☐ |
| 30 | Retención de registros de auditoría | Registros de auditoría de cambio de contraseña | Paso del tiempo | Registros se conservan por tiempo mínimo definido por regulación vigente aplicable al ecosistema de facturación electrónica DIAN. Después de este período, pueden ser archivados en almacenamiento de largo plazo o eliminados según política de retención de la organización | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

Se requieren los siguientes mockups interactivos en HTML siguiendo la Guía de Estilos (Material UI, paleta de colores Primary #4A5A9E, Secondary #D4145A):

#### 1. **HU002C-ResetPassword.html** - Formulario de restablecimiento de contraseña

**Elementos:**
- Logotipo de CDN Facturación centrado en parte superior
- Título: "Restablecer contraseña" (Tipografía H3, color Primary Dark #364378)
- Texto descriptivo: "Ingresa tu nueva contraseña. Debe cumplir con los requisitos de seguridad." (Body2, color grey-700)

**Campo "Nueva contraseña":**
- TextField outlined, fullWidth, tipo password
- Label: "Nueva contraseña *"
- Icono de ojo (visibility/visibility_off) al final del campo para mostrar/ocultar
- Error text (si aplica) debajo del campo

**Indicador visual de requisitos** (debajo del campo "Nueva contraseña"):
- Lista de requisitos con iconos dinámicos:
  - ✓ (verde) / ✗ (rojo) "Mínimo 8 caracteres"
  - ✓ / ✗ "Al menos una mayúscula (A-Z)"
  - ✓ / ✗ "Al menos una minúscula (a-z)"
  - ✓ / ✗ "Al menos un número (0-9)"
  - ✓ / ✗ "Al menos un símbolo (!@#$%^&*)"
  - ✓ / ✗ "No puede ser igual a contraseña actual"
  - ✓ / ✗ "No puede ser una de las últimas 5 contraseñas"
- Tipografía Caption, colores: verde Success (#4CAF50) para ✓, rojo Error (#F44336) para ✗

**Barra de fortaleza** (debajo de indicadores):
- LinearProgress con buffer variant
- Colores dinámicos: rojo (débil 0-40%), naranja (media 41-70%), verde (fuerte 71-100%)
- Texto: "Fortaleza: Débil/Media/Fuerte" (Caption, color correspondiente)

**Campo "Confirmar contraseña":**
- TextField outlined, fullWidth, tipo password
- Label: "Confirmar contraseña *"
- Icono de ojo al final del campo (independiente del otro campo)
- Error text: "Las contraseñas no coinciden" (si aplica, color Error)
- Checkmark verde si coincide

**Botones:**
- Botón primario (contained, fullWidth en móvil, gradiente Primary → Secondary):
  - Texto: "Restablecer Contraseña"
  - Deshabilitado (gris) hasta que todos los requisitos se cumplan y contraseñas coincidan
  - Muestra spinner y texto "Restableciendo..." durante procesamiento
- Botón secundario (outlined):
  - Texto: "Cancelar"
  - Muestra diálogo de confirmación al hacer clic

**Estados a implementar en mockup:**
- Estado inicial (campos vacíos, indicadores rojos, botón deshabilitado)
- Estado escribiendo (algunos requisitos verdes, otros rojos, barra de fortaleza actualizándose)
- Estado todos los requisitos cumplidos (todos verdes, barra verde "Fuerte", botón habilitado)
- Estado error de confirmación (nueva contraseña válida pero confirmación no coincide)
- Estado procesando (spinner en botón)

#### 2. **HU002C-PasswordChangedSuccess.html** - Pantalla de éxito

**Elementos:**
- Logotipo de CDN Facturación centrado en parte superior
- Icono grande centrado: check_circle (color Success #4CAF50, tamaño 100px)
- Título: "¡Contraseña actualizada!" (Tipografía H3, color Primary Dark #364378)
- Mensaje principal (Body1, color grey-700, centrado, max-width 500px):
  - "Tu contraseña ha sido actualizada correctamente."
- Mensaje secundario (Body2, color grey-600, centrado):
  - "Ya puedes iniciar sesión con tu nueva contraseña."
  - "Redirigiendo a inicio de sesión en 3 segundos..."
- Contador regresivo visual (Tipografía H4, color Primary, centrado): "3" → "2" → "1"
- Botón primario (contained, gradiente Primary → Secondary):
  - Texto: "Ir a inicio de sesión ahora"
  - onClick: redirige inmediatamente a HU001 (sin esperar countdown)

**Animación:** Fade in al cargar, countdown animado

#### 3. **HU002C-ConfirmationDialog.html** - Diálogo de confirmación de cancelación

**Elementos:**
- Dialog de Material UI
- Título: "¿Cancelar cambio de contraseña?" (H6, color Primary Dark)
- Contenido (Body1, color grey-700):
  - "¿Estás seguro que deseas cancelar el cambio de contraseña?"
  - "Tu contraseña actual no será modificada."
- Botones:
  - Botón secundario (text): "Continuar editando" (color Primary, cierra diálogo)
  - Botón primario (contained): "Sí, cancelar" (color Error, redirige a login)

#### 4. **HU002C-EmailNotification.html** - Plantilla de correo de notificación

**Elementos:**
- **Encabezado** (fondo gradiente Primary → Secondary):
  - Logo CDN Facturación en blanco centrado
- **Cuerpo principal** (fondo blanco, padding 40px):
  - Saludo: "Hola {nombre_usuario}," (Body1 Bold, color grey-900)
  - Párrafo 1: "Te confirmamos que tu contraseña ha sido actualizada exitosamente." (Body1, color grey-700)
  - **Card de información** (fondo grey-100, borde Primary):
    - "Fecha y hora: {fecha_hora_cambio}"
    - "Dirección IP: {ip_publica}"
    - "Dispositivo: {navegador/SO}"
  - **Alert de seguridad** (Alert error severity, fondo #FFEBEE):
    - Icono warning
    - "Si NO realizaste este cambio, tu cuenta puede estar en riesgo. Contacta a soporte inmediatamente:"
    - Email: soporte@cdnfacturacion.com
    - Teléfono: +57 (1) 234-5678
  - Botón CTA (contained, gradiente Primary → Secondary):
    - Texto: "Iniciar sesión"
    - Enlace a login (HU001)
- **Pie de correo** (fondo grey-100):
  - "Equipo de Seguridad CDN Facturación" (Body2 Bold)
  - "Este correo es informativo y automático, no requiere respuesta."

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Usuario olvida nueva contraseña inmediatamente después de cambiarla** | Usuario cambia contraseña a algo que cree recordará, pero al intentar ingresar posteriormente la olvidó nuevamente | Medio | Media | Mostrar recomendación de usar gestor de contraseñas en pantalla de cambio. Permitir al usuario ver la contraseña antes de confirmar (icono ojo). Incluir mensaje en correo de notificación sugiriendo guardar contraseña de forma segura. Proceso de recuperación (HU002A-C) puede usarse nuevamente sin límite de tiempo | Aceptado |
| **Reutilización de contraseñas débiles o comunes** | Usuario elige contraseña que cumple requisitos técnicos pero es débil o común (ej: Password1!, Welcome2024!) | Medio | Media | Validar contra diccionario de contraseñas comunes (top 10,000). Mostrar barra de fortaleza en tiempo real. Mostrar advertencia clara si contraseña es común. Permitir continuar con advertencia (no bloquear) para no frustrar usuario, pero educarlo sobre riesgo | Mitigado |
| **Usuario no recibe correo de notificación de cambio** | Correo de notificación no llega, se va a spam, o hay error en envío. Usuario legítimo no se entera de que su contraseña fue cambiada si fue atacante | Alto | Baja | Monitorear tasa de entrega de correos. Configurar SPF/DKIM/DMARC correctamente. Registrar en auditoría si envío falla. Considerar múltiples canales de notificación en futuras versiones (SMS, notificación push). Mostrar mensaje en pantalla de éxito: "Revisa tu correo para confirmación" | Aceptado con monitoreo |
| **Sesiones no se invalidan correctamente por problemas técnicos** | Error técnico al invalidar sesiones activas, permitiendo que atacante que cambió contraseña mantenga acceso en sesiones previas del usuario legítimo | Alto | Muy Baja | Implementar invalidación robusta con transacciones. Probar exhaustivamente en diferentes escenarios. Implementar timeout de sesión absoluto como respaldo (ej: máx 8 horas). Monitorear logs de invalidación de sesiones. Si falla invalidación, registrar evento ERROR en auditoría para investigación inmediata | Mitigado |
| **Contraseña enviada sin cifrado por error de implementación** | Error de implementación podría causar que contraseña se envíe o almacene en texto plano en logs, auditoría, o base de datos | Crítico | Muy Baja | NUNCA almacenar contraseña en texto plano. Solo almacenar hash seguro (bcrypt/Argon2 con salt). Validar en code review que contraseñas no se loguean. Auditoría NO debe contener contraseñas (ni actual ni nueva). Solo registrar eventos y metadatos. Pruebas de seguridad exhaustivas antes de producción | Mitigado - crítico |
| **Historial de contraseñas insuficiente o no funciona** | Usuario podría reutilizar contraseña #6 o anterior si historial no se mantiene correctamente | Medio | Baja | Almacenar historial de últimas 5 contraseñas con hashes separados. Implementar limpieza de historial antiguo (mantener solo últimas 5). Validar en backend contra todas las 5. Pruebas exhaustivas de validación de no reutilización | Mitigado |
| **Usuario cierra navegador antes de completar redirección** | Usuario cambia contraseña exitosamente pero cierra navegador durante el countdown de 3 segundos, no ve redirección a login | Bajo | Baja | Cambio de contraseña ya es efectivo y está guardado. Usuario puede navegar a login manualmente o hacer clic en enlace del correo de notificación. Botón "Ir a inicio de sesión ahora" permite omitir countdown. No hay pérdida de funcionalidad | Aceptado - sin impacto |

---

## Notas Adicionales

### Fuera de Alcance (Out of Scope)

**Esta HU NO incluye:**
- Solicitud de enlace de recuperación → **Cubierto en HU002A**
- Validación del enlace recibido por correo → **Cubierto en HU002B**
- Cambio de contraseña desde perfil de usuario (estando ya autenticado) → HU futura
- Configuración de política de contraseñas por administrador → HU futura
- Autenticación de dos factores (2FA) → Futuras versiones
- Notificación por SMS o push → Futuras versiones
- Gestor de contraseñas integrado → Futuras versiones

### Dependencias

**Esta HU depende de:**
- **HU002A - Solicitud de Recuperación de Contraseña**: El token usado en HU002C fue generado en HU002A
- **HU002B - Validación de Enlace de Recuperación**: El token debe haber sido validado exitosamente en HU002B antes de mostrar el formulario de HU002C
- **HU001 - Autenticación y Selección de Cliente**: Después de cambiar contraseña, el usuario debe poder autenticarse con la nueva contraseña usando el flujo de HU001

**Flujo completo:** HU002A (solicitud + generación token) → HU002B (validación token) → **HU002C (cambio contraseña)** → HU001 (login con nueva contraseña)

**Sistemas externos requeridos:**
- Sistema de Gestión de Sesiones con capacidad de invalidar tokens de sesión activos
- Sistema de Notificaciones por Correo para enviar notificación de cambio exitoso
- Módulo de Auditoría implementado según Estándar de Auditoría Funcional

### Valor de Negocio Independiente

Esta HU completa el flujo de recuperación de contraseña. Valor específico:
- ✅ Permite a usuarios establecer nueva contraseña de forma segura
- ✅ Valida requisitos de seguridad en tiempo real (UX mejorada)
- ✅ Previene reutilización de contraseñas débiles
- ✅ Invalida sesiones activas para proteger contra atacantes
- ✅ Notifica al usuario del cambio para detección de compromiso
- ✅ Completa el flujo end-to-end de recuperación autoservicio
- ✅ Reduce carga de soporte (usuarios resuelven por sí mismos)

### Testing Recomendado

Para probar esta HU de forma independiente:
1. Crear tokens válidos en base de datos manualmente (si HU002A/B no están completas)
2. Probar indicadores visuales de requisitos en tiempo real
3. Probar validación de confirmación de contraseña
4. Probar validación de no reutilización (crear historial de contraseñas en BD)
5. Probar validación de contraseña igual a actual
6. Probar contraseñas comunes (con lista negra de prueba)
7. Probar cambio exitoso y verificar hash almacenado correctamente
8. Probar invalidación de sesiones (crear sesiones activas de prueba)
9. Probar envío de correo de notificación
10. Probar token marcado como USADO después de cambio
11. Probar redirección automática a login
12. Probar login con nueva contraseña (integración con HU001)
13. Verificar registros de auditoría para todos los escenarios

### Referencias

- **Estándar de Auditoría Funcional**: Tipos de eventos `AUTENTICACION_CONTRASENA_*`, estructura obligatoria de 12 campos, niveles de severidad INFO/WARNING/ERROR
- **Estándar de Seguridad APL03**:
  - AP-0039: Contraseñas almacenadas con resúmenes criptográficos (PBKDF2, bcrypt, Argon2)
  - AP-0041: No permitir reutilización de últimas 24 contraseñas (aplicamos 5 por simplicidad inicial)
  - AP-0044: Longitud mínima 8 caracteres para usuarios finales
  - AP-0045: Almacenar contraseñas con diferentes derivaciones de clave (Salt)
  - AP-0049: Invalidar sesión al cambiar contraseña
  - AP-0051: Contraseñas deben contener al menos 3 de 4 tipos de caracteres (aplicamos los 4)
  - AP-0152: Enmascarar contraseña mientras se ingresa
  - AP-0155: Fomentar uso de passphrases, no impedir contraseñas largas
- **Glosario**: Usuario, Sesión de Usuario, Trazabilidad, No Repudio
- **Guía de Estilos**: Material UI, paleta de colores Primary/Secondary/Success/Warning/Error, tipografía Roboto, componentes TextField/Button/Alert/LinearProgress/CircularProgress/Dialog, iconografía Material Icons (visibility, check_circle, error)

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**HU Original:** HU002 - Recuperación de Contraseña (atomizada en HU002A, HU002B, HU002C)
