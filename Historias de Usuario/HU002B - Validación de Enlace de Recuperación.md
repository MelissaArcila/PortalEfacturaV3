# HU002B - Validación de Enlace de Recuperación

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
| **Arquitectura relacionada** | Portal Unificado - Módulo de Autenticación |

---

## Contexto

Esta historia cubre el **segundo componente del flujo de recuperación de contraseña**: la validación del enlace temporal que el Usuario recibió por correo electrónico.

Cuando el Usuario hace clic en el enlace recibido en su correo (generado en HU002A), el sistema debe validar que el token es válido, no ha expirado (15 minutos), no ha sido usado previamente, y no ha sido manipulado. Según el resultado de estas validaciones, el sistema debe mostrar la pantalla de restablecimiento de contraseña (HU002C) o una pantalla de error específica que explique el problema.

Esta funcionalidad es **independiente y entregable por sí misma**: valida la seguridad del enlace y previene accesos no autorizados, aunque requiere HU002A para generar los tokens y HU002C para completar el cambio de contraseña.

---

**Notas:**
- Esta HU cubre SOLO la validación del enlace y navegación a pantalla correcta. El formulario de cambio de contraseña se cubre en HU002C
- Los enlaces son válidos por 15 minutos desde su generación
- Los enlaces son de un solo uso: después de usarse exitosamente, no pueden volver a usarse
- Si el Usuario solicita un nuevo enlace (HU002A), los enlaces anteriores se invalidan automáticamente
- Todo el proceso debe ser auditable siguiendo el Estándar de Auditoría Funcional

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Usuario** | Quiero que el sistema valide que el enlace que recibí por correo es válido | Para poder acceder de forma segura al formulario de cambio de contraseña |
| Como **Usuario** | Quiero ver un mensaje claro si el enlace expiró, ya fue usado, o es inválido | Para entender qué salió mal y saber qué hacer a continuación (solicitar nuevo enlace) |
| Como **Administrador de Seguridad** | Quiero que el sistema rechace enlaces expirados, ya usados, o manipulados | Para garantizar que solo enlaces válidos y temporales permiten el restablecimiento de contraseña |
| Como **Administrador** | Quiero que todos los accesos a enlaces de recuperación (exitosos y fallidos) sean auditados | Para detectar intentos de manipulación de tokens y garantizar trazabilidad |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso a enlace válido dentro de 15 minutos | Usuario recibe correo con enlace de recuperación (HU002A) y hace clic en el enlace dentro de los 15 minutos desde su generación. El token no ha sido usado previamente | Usuario hace clic en el enlace del correo | Sistema valida: (1) Token existe en base de datos, (2) Token no ha expirado (fecha_creacion + 15 minutos > fecha_actual), (3) Token no ha sido marcado como USADO, (4) Token no ha sido INVALIDADO. Si todas las validaciones pasan, sistema navega a pantalla de "Restablecer Contraseña" (HU002C) y registra acceso exitoso en auditoría | ☐ | ☐ | ☐ |
| 2 | Acceso a enlace expirado (más de 15 minutos) | Usuario recibe correo con enlace de recuperación pero hace clic en el enlace después de 15 minutos desde su generación | Usuario hace clic en el enlace del correo | Sistema valida token y detecta que ha expirado (fecha_creacion + 15 minutos <= fecha_actual). Sistema navega a pantalla de error "Enlace Expirado". Pantalla muestra: icono de advertencia (warning_amber color Warning), título "Enlace expirado" (H4), mensaje "Este enlace ha expirado. Los enlaces de recuperación son válidos por 15 minutos.", botón primario "Solicitar nuevo enlace" (redirige a HU002A), botón secundario "Volver a inicio de sesión" (redirige a HU001). Registra evento en auditoría con severidad WARNING | ☐ | ☐ | ☐ |
| 3 | Acceso a enlace ya utilizado | Usuario ya usó el enlace exitosamente para cambiar su contraseña (HU002C) e intenta acceder al mismo enlace nuevamente | Usuario hace clic en el enlace del correo por segunda vez | Sistema valida token y detecta que ya fue usado (estado = USADO). Sistema navega a pantalla de error "Enlace Ya Utilizado". Pantalla muestra: icono de advertencia (warning_amber color Warning), título "Enlace ya utilizado" (H4), mensaje "Este enlace ya fue utilizado y no es válido. Si necesitas restablecer tu contraseña nuevamente, solicita un nuevo enlace.", botón primario "Solicitar nuevo enlace" (redirige a HU002A), botón secundario "Volver a inicio de sesión" (redirige a HU001). Registra evento en auditoría con severidad WARNING | ☐ | ☐ | ☐ |
| 4 | Acceso a enlace invalidado por nueva solicitud | Usuario solicitó un nuevo enlace de recuperación (HU002A escenario 12) lo que invalidó automáticamente los enlaces anteriores, e intenta acceder a un enlace anterior invalidado | Usuario hace clic en el enlace anterior del correo | Sistema valida token y detecta que fue invalidado (estado = INVALIDADO). Sistema navega a pantalla de error "Enlace Inválido". Pantalla muestra: icono de advertencia (error color Error), título "Enlace inválido" (H4), mensaje "Este enlace ya no es válido porque solicitaste un nuevo enlace de recuperación. Revisa tu correo para usar el enlace más reciente.", botón primario "Solicitar nuevo enlace" (redirige a HU002A), botón secundario "Volver a inicio de sesión" (redirige a HU001). Registra evento en auditoría con severidad WARNING | ☐ | ☐ | ☐ |
| 5 | Acceso a enlace con token manipulado o inválido | Usuario (o atacante) accede a URL con token que no existe en base de datos, tiene formato incorrecto (no es UUID válido), está corrupto, o ha sido manipulado | Usuario accede a URL con token inválido | Sistema intenta validar token y detecta que: (1) No existe en base de datos, O (2) Formato de token inválido (no es UUID v4), O (3) Token corrupto. Sistema navega a pantalla de error "Enlace Inválido". Pantalla muestra: icono de error (error color Error), título "Enlace inválido" (H4), mensaje "Este enlace no es válido. Verifica que lo hayas copiado correctamente del correo o solicita un nuevo enlace.", botón primario "Solicitar nuevo enlace" (redirige a HU002A), botón secundario "Volver a inicio de sesión" (redirige a HU001). Registra evento en auditoría con severidad ERROR indicando posible manipulación | ☐ | ☐ | ☐ |
| 6 | Acceso a URL sin token (parámetro faltante) | Usuario (o atacante) accede a URL de recuperación pero sin incluir el parámetro de token en la query string | Usuario accede a URL: /reset-password (sin ?token=...) | Sistema detecta que falta el parámetro de token. Sistema navega a pantalla de error "Enlace Inválido" (misma que escenario 5). Registra evento en auditoría con severidad WARNING indicando acceso sin token | ☐ | ☐ | ☐ |
| 7 | Usuario cancelado en pantalla de error | Usuario en cualquier pantalla de error (expirado, usado, inválido) | Usuario hace clic en botón "Volver a inicio de sesión" | Sistema redirige a pantalla de login (HU001). No registra evento en auditoría (acción sin impacto de seguridad) | ☐ | ☐ | ☐ |
| 8 | Usuario solicita nuevo enlace desde pantalla de error | Usuario en cualquier pantalla de error (expirado, usado, inválido) | Usuario hace clic en botón "Solicitar nuevo enlace" | Sistema redirige a pantalla de solicitud de recuperación (HU002A). No registra evento en auditoría (navegación simple) | ☐ | ☐ | ☐ |
| 9 | Tiempo de validación consistente (anti-enumeración) | Sistema valida token (válido, expirado, usado, inválido) | Sistema procesa validación | Sistema tarda tiempo similar (±100ms) en validar y responder independientemente del motivo de fallo (expirado vs usado vs inválido vs no existe). Evita que un atacante pueda determinar información sobre tokens existentes midiendo tiempos de respuesta diferentes | ☐ | ☐ | ☐ |
| 10 | Cálculo preciso de expiración considerando zona horaria | Token generado con timestamp en UTC. Usuario en zona horaria diferente accede al enlace | Sistema valida expiración | Sistema calcula expiración usando timestamps en UTC para evitar problemas de zona horaria. Validación: (timestamp_generacion_utc + 15 minutos) > timestamp_actual_utc. Garantiza que 15 minutos sean exactos independientemente de la zona horaria del usuario o servidor | ☐ | ☐ | ☐ |
| 11 | Indicador de carga durante validación | Usuario hace clic en enlace del correo | Sistema procesa validación de token (tarda 0.5-2 segundos) | Sistema muestra pantalla con indicador de carga (spinner centrado, mensaje "Validando enlace...") mientras procesa las validaciones. Una vez completado, navega a pantalla de restablecimiento (HU002C) o pantalla de error correspondiente | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 12 | Estructura obligatoria de registro de auditoría | Cualquier evento de acceso a enlace de recuperación (válido, expirado, usado, inválido) | Sistema registra evento | Registro contiene: ID Único del Evento (UUID), Tipo de Evento (AUTENTICACION_ENLACE_*), Fecha y Hora (UTC con milisegundos, ISO 8601), Usuario (nombre de usuario asociado al token, si existe), Cliente (NULL), Cliente Nombre (NULL), IP Local, IP Pública, Resultado (EXITOSO/FALLIDO), Descripción (texto descriptivo del evento), Nivel de Severidad (INFO/WARNING/ERROR), Datos Adicionales (JSON con contexto relevante) | ☐ | ☐ | ☐ |
| 13 | Auditoría de acceso exitoso a enlace válido | Usuario accede a enlace válido dentro de 15 minutos | Usuario hace clic en enlace (escenario 1) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_ENLACE_ACCEDIDO", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Usuario {usuario} accedió exitosamente a enlace de recuperación de contraseña", Datos Adicionales: {"token_id": "UUID del token", "fecha_generacion_token": "timestamp ISO 8601", "minutos_desde_generacion": 8, "tiempo_restante_minutos": 7, "ip_acceso_local": "IP local", "ip_acceso_publica": "IP pública", "ip_solicitud_original": "IP pública de cuando se solicitó"} | ☐ | ☐ | ☐ |
| 14 | Auditoría de acceso a enlace expirado | Usuario accede a enlace después de 15 minutos | Usuario hace clic en enlace (escenario 2) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_ENLACE_EXPIRADO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} intentó acceder a enlace de recuperación expirado", Datos Adicionales: {"token_id": "UUID", "fecha_generacion_token": "timestamp", "fecha_expiracion_token": "timestamp", "fecha_acceso": "timestamp", "minutos_desde_generacion": 23, "minutos_despues_expiracion": 8, "ip_acceso_local": "IP local", "ip_acceso_publica": "IP pública"} | ☐ | ☐ | ☐ |
| 15 | Auditoría de acceso a enlace ya usado | Usuario accede a enlace que ya fue utilizado exitosamente | Usuario hace clic en enlace (escenario 3) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_ENLACE_REUTILIZADO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} intentó reutilizar enlace de recuperación ya consumido", Datos Adicionales: {"token_id": "UUID", "fecha_generacion_token": "timestamp", "fecha_uso_exitoso_original": "timestamp", "ip_uso_original": "IP pública de primer uso", "ip_reuso_actual": "IP pública de intento actual", "minutos_entre_usos": 15, "usuario_mismo": true} | ☐ | ☐ | ☐ |
| 16 | Auditoría de acceso a enlace invalidado | Usuario accede a enlace que fue invalidado por nueva solicitud | Usuario hace clic en enlace (escenario 4) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_ENLACE_INVALIDADO_PREVIO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} intentó acceder a enlace invalidado por nueva solicitud", Datos Adicionales: {"token_id": "UUID", "fecha_generacion_token": "timestamp", "fecha_invalidacion": "timestamp", "token_nuevo_generado": "UUID del token más reciente", "ip_acceso_local": "IP local", "ip_acceso_publica": "IP pública"} | ☐ | ☐ | ☐ |
| 17 | Auditoría de acceso con token manipulado o inválido | Usuario o atacante accede con token que no existe, tiene formato inválido, o está corrupto | Usuario accede a URL (escenario 5) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_ENLACE_INVALIDO", Resultado: "FALLIDO", Severidad: "ERROR", Descripción: "Intento de acceso con token de recuperación inválido o manipulado", Datos Adicionales: {"token_recibido_truncado": "Primeros 10 caracteres del token", "motivo_invalido": "no_existe_en_bd" | "formato_invalido" | "corrupto", "formato_esperado": "UUID v4", "ip_acceso_local": "IP local", "ip_acceso_publica": "IP pública", "user_agent": "navegador/SO", "posible_manipulacion": true} | ☐ | ☐ | ☐ |
| 18 | Auditoría de acceso sin token | Usuario accede a URL sin parámetro de token | Usuario accede a URL (escenario 6) | Registro de auditoría: Tipo de Evento: "AUTENTICACION_ENLACE_SIN_TOKEN", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Acceso a URL de recuperación sin parámetro de token", Datos Adicionales: {"url_accedida": "/reset-password", "parametros_recibidos": "{}", "ip_acceso_local": "IP local", "ip_acceso_publica": "IP pública", "user_agent": "navegador/SO"} | ☐ | ☐ | ☐ |
| 19 | Inmutabilidad de registros de auditoría | Cualquier escenario de auditoría | Sistema genera registro | Registros de auditoría NO pueden ser modificados ni eliminados después de su creación. Solo se permiten consultas de lectura. Implementar controles que prevengan alteración de registros históricos | ☐ | ☐ | ☐ |
| 20 | Consulta de registros de auditoría para investigación | Administrador o auditor requiere investigar eventos de acceso a enlaces | Administrador accede a módulo de auditoría | Sistema permite filtrar por: rango de fechas, usuario, tipo de evento (AUTENTICACION_ENLACE_*), resultado (EXITOSO/FALLIDO), severidad (INFO/WARNING/ERROR), IP origen (local o pública), token_id específico. Permite exportar resultados en formato CSV/Excel para análisis externo. Permite correlacionar eventos: solicitud (HU002A) → acceso a enlace (HU002B) → cambio de contraseña (HU002C) por mismo token_id | ☐ | ☐ | ☐ |
| 21 | Detección de patrones de ataque | Múltiples intentos de acceso con tokens inválidos desde misma IP en corto período | Sistema detecta patrón anómalo (ej: 10+ accesos con tokens inválidos en 5 minutos desde misma IP) | Sistema genera alerta automática o registro especial de auditoría con tipo "AUTENTICACION_PATRON_ATAQUE_DETECTADO", severidad ERROR, para notificar a equipo de seguridad de posible intento de adivinación o fuerza bruta de tokens | ☐ | ☐ | ☐ |
| 22 | Retención de registros de auditoría | Registros de auditoría de validación de enlaces | Paso del tiempo | Registros se conservan por tiempo mínimo definido por regulación vigente aplicable al ecosistema de facturación electrónica DIAN. Después de este período, pueden ser archivados en almacenamiento de largo plazo o eliminados según política de retención de la organización | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

Se requieren los siguientes mockups interactivos en HTML siguiendo la Guía de Estilos (Material UI, paleta de colores Primary #4A5A9E, Secondary #D4145A):

#### 1. **HU002B-ValidatingLink.html** - Pantalla de carga durante validación

**Elementos:**
- Logotipo de CDN Facturación centrado en parte superior
- Spinner circular (CircularProgress) grande centrado (color Primary)
- Texto "Validando enlace..." (Body1, color grey-600, centrado)
- Fondo blanco limpio

**Duración:** Esta pantalla se muestra mientras se validan las condiciones del token (0.5-2 segundos aprox)

#### 2. **HU002B-LinkExpired.html** - Pantalla de enlace expirado

**Elementos:**
- Logotipo de CDN Facturación centrado en parte superior
- Icono grande centrado: warning_amber (color Warning #FF9800, tamaño 80px)
- Título: "Enlace expirado" (Tipografía H4, color Primary Dark #364378)
- Mensaje principal (Body1, color grey-700, centrado, max-width 500px):
  - "Este enlace ha expirado. Los enlaces de recuperación son válidos por 15 minutos."
- Mensaje secundario (Body2, color grey-600, centrado):
  - "Por tu seguridad, solicita un nuevo enlace para restablecer tu contraseña."
- Botón primario (contained, fullWidth en móvil, auto en desktop, gradiente Primary → Secondary):
  - Texto: "Solicitar nuevo enlace"
  - onClick: redirige a HU002A (pantalla de solicitud)
- Botón secundario (text, centrado):
  - Texto: "Volver a inicio de sesión"
  - Color: Primary
  - onClick: redirige a HU001 (login)

#### 3. **HU002B-LinkUsed.html** - Pantalla de enlace ya utilizado

**Elementos:**
- Logotipo de CDN Facturación centrado en parte superior
- Icono grande centrado: check_circle (color Success #4CAF50, tamaño 80px)
- Título: "Enlace ya utilizado" (Tipografía H4, color Primary Dark #364378)
- Mensaje principal (Body1, color grey-700, centrado, max-width 500px):
  - "Este enlace ya fue utilizado y no es válido."
- Mensaje secundario (Body2, color grey-600, centrado):
  - "Si necesitas restablecer tu contraseña nuevamente, solicita un nuevo enlace."
- Botón primario (contained, fullWidth en móvil, auto en desktop, gradiente Primary → Secondary):
  - Texto: "Solicitar nuevo enlace"
  - onClick: redirige a HU002A
- Botón secundario (text, centrado):
  - Texto: "Volver a inicio de sesión"
  - Color: Primary
  - onClick: redirige a HU001

#### 4. **HU002B-LinkInvalid.html** - Pantalla de enlace inválido (manipulado, formato incorrecto, no existe)

**Elementos:**
- Logotipo de CDN Facturación centrado en parte superior
- Icono grande centrado: error (color Error #F44336, tamaño 80px)
- Título: "Enlace inválido" (Tipografía H4, color Primary Dark #364378)
- Mensaje principal (Body1, color grey-700, centrado, max-width 500px):
  - "Este enlace no es válido."
- Mensaje secundario (Body2, color grey-600, centrado):
  - "Verifica que lo hayas copiado correctamente del correo o solicita un nuevo enlace."
- Alert de información (info severity, borde izquierdo azul):
  - "Si no solicitaste este cambio de contraseña, tu cuenta podría estar en riesgo. Contacta a soporte inmediatamente: [email/teléfono]"
- Botón primario (contained, fullWidth en móvil, auto en desktop, gradiente Primary → Secondary):
  - Texto: "Solicitar nuevo enlace"
  - onClick: redirige a HU002A
- Botón secundario (text, centrado):
  - Texto: "Volver a inicio de sesión"
  - Color: Primary
  - onClick: redirige a HU001

**Notas de diseño:**
- Todas las pantallas deben ser responsive (mobile-first)
- Espaciado vertical generoso entre elementos (24-32px)
- Iconos deben ser claramente visibles y comunicar el estado (warning, success, error)
- Botones centrados con max-width en pantallas grandes
- Mantener consistencia visual con otras pantallas del flujo (HU002A, HU002C, HU001)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Intentos de adivinación de tokens** | Un atacante podría intentar adivinar tokens válidos generando múltiples UUIDs y probándolos en la URL de recuperación | Alto | Muy Baja | Tokens generados con UUID v4 (128 bits, 2^128 combinaciones posibles, criptográficamente seguros). Probabilidad de adivinación es prácticamente cero. Registrar todos los intentos con tokens inválidos con severidad ERROR para detectar patrones de ataque. Considerar rate limiting a nivel de infraestructura para limitar intentos por IP | Mitigado |
| **Timing attacks para enumeración** | Un atacante podría medir diferencias de tiempo de respuesta entre tokens válidos (pero expirados) vs tokens inexistentes para determinar qué tokens existen en el sistema | Medio | Baja | Implementar tiempo de respuesta consistente para todos los tipos de fallo (expirado, usado, inválido, no existe) añadiendo delay artificial si es necesario. Diferencia máxima: ±100ms. Monitorear tiempos de respuesta en producción | Mitigado |
| **Reutilización de enlace por atacante que interceptó correo** | Si un atacante tiene acceso al correo del usuario (phishing, compromiso de cuenta), puede usar el enlace antes que el usuario legítimo | Alto | Baja | Enlace válido solo 15 minutos (ventana corta). Enlace de un solo uso. Registrar IP de solicitud original (HU002A) y de acceso al enlace (HU002B) para detectar cambios sospechosos de ubicación geográfica. Notificar al usuario por correo cuando contraseña sea cambiada (HU002C) | Aceptado con controles |
| **Confusión entre tipos de error** | Usuario legítimo puede no entender diferencia entre "expirado", "usado" e "inválido", causando frustración | Bajo | Media | Mensajes de error claros y diferenciados que explican exactamente qué pasó y qué debe hacer el usuario. Botón prominente "Solicitar nuevo enlace" en todas las pantallas de error para facilitar acción siguiente. Incluir información de contacto de soporte si el usuario tiene problemas | Mitigado |
| **Enlaces no expiran correctamente por problemas de zona horaria** | Diferencias de zona horaria entre servidor que genera token y servidor que valida pueden causar cálculos incorrectos de expiración | Medio | Baja | Todos los timestamps en UTC. Cálculo de expiración en UTC. Validación explícita: (timestamp_generacion_utc + 15 min) > timestamp_actual_utc. Pruebas de QA con diferentes zonas horarias | Mitigado |
| **Usuario accede a enlace desde diferente dispositivo/ubicación** | Usuario solicita enlace desde PC en oficina pero accede desde móvil en casa, generando cambio de IP que podría parecer sospechoso | Bajo | Alta | Esto es comportamiento legítimo y esperado. No bloquear por cambio de IP. Registrar ambas IPs en auditoría para análisis posterior si hay incidente. Permitir acceso mientras token sea válido independientemente de IP | Aceptado - comportamiento esperado |

---

## Notas Adicionales

### Fuera de Alcance (Out of Scope)

**Esta HU NO incluye:**
- Generación del token y envío por correo → **Cubierto en HU002A**
- Formulario de cambio de contraseña → **Cubierto en HU002C**
- Validación de requisitos de nueva contraseña → **Cubierto en HU002C**
- Cambio efectivo de contraseña → **Cubierto en HU002C**
- Invalidación de sesiones activas → **Cubierto en HU002C**
- Notificación al usuario de cambio de contraseña exitoso → **Cubierto en HU002C**

### Dependencias

**Esta HU depende de:**
- **HU002A - Solicitud de Recuperación de Contraseña**: El token que se valida en HU002B fue generado en HU002A

**Esta HU es prerequisito para:**
- **HU002C - Restablecimiento de Contraseña**: Solo si HU002B valida exitosamente el token, el usuario puede acceder al formulario de HU002C

**Flujo completo:** HU002A (solicitud + generación token) → **HU002B (validación token)** → HU002C (cambio contraseña)

### Valor de Negocio Independiente

Esta HU puede ser entregada y probada de forma independiente. Valor específico:
- ✅ Valida que el sistema protege correctamente los enlaces temporales
- ✅ Valida que tokens expirados no permiten acceso
- ✅ Valida que tokens no pueden reutilizarse (seguridad)
- ✅ Valida mensajes de error claros para usuarios
- ✅ Permite probar detección de manipulación de tokens
- ✅ Genera registros de auditoría detallados de intentos de acceso
- ✅ Puede testearse con tokens generados manualmente en base de datos sin necesidad de HU002A completamente funcional (para testing inicial)

### Testing Recomendado

Para probar esta HU de forma independiente:
1. Crear tokens de prueba manualmente en base de datos con diferentes estados (válido, expirado, usado, invalidado)
2. Probar acceso con tokens válidos, expirados (modificar fecha), ya usados (marcar como USADO)
3. Probar con tokens inexistentes, formato inválido, corruptos
4. Probar sin parámetro de token en URL
5. Verificar tiempos de respuesta consistentes
6. Verificar registros de auditoría correctos para cada tipo de evento
7. Probar navegación a pantallas correctas (restablecimiento vs errores)
8. Probar desde diferentes navegadores, dispositivos, IPs

### Referencias

- **Estándar de Auditoría Funcional**: Tipos de eventos `AUTENTICACION_ENLACE_*`, estructura obligatoria de 12 campos, niveles de severidad INFO/WARNING/ERROR
- **Estándar de Seguridad APL03**:
  - AP-0022: Registro de eventos de seguridad exitosos y no exitosos
  - AP-0024: Registro con timestamp UTC, usuario, IPs, y contexto detallado
  - AP-0025: Logs no modificables ni alterables
- **Glosario**: Usuario, Trazabilidad, No Repudio
- **Guía de Estilos**: Material UI, paleta de colores Primary/Secondary/Success/Warning/Error, tipografía Roboto, componentes CircularProgress/Button/Alert, iconografía Material Icons

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**HU Original:** HU002 - Recuperación de Contraseña (atomizada en HU002A, HU002B, HU002C)
