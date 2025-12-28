# HU003 - Protección Anti-Bot con reCAPTCHA v3

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
| **Arquitectura relacionada** | Portal Unificado - Módulo de Autenticación, Sistema de Notificaciones, Módulo de Recuperación de Contraseña |

---

## Contexto

El Portal Unificado de CDN Facturación está expuesto a ataques automatizados de bots maliciosos que intentan comprometer la seguridad mediante fuerza bruta en la autenticación, solicitudes masivas de recuperación de contraseña, y otros patrones de comportamiento no humano que pueden afectar la disponibilidad del servicio y la seguridad de las cuentas de Usuario.

Aunque el sistema cuenta con mecanismos de protección como bloqueo de cuenta después de intentos fallidos (HU001) y límite de solicitudes de recuperación (HU002), estos controles reactivos solo actúan después de que el ataque ya ha impactado el sistema. Es necesario implementar una capa de protección proactiva que identifique y bloquee comportamiento automatizado antes de que alcance los sistemas críticos de autenticación.

La implementación de Google reCAPTCHA v3 permite evaluar de forma invisible y no intrusiva si las interacciones con el Portal provienen de usuarios humanos o de bots automatizados, mediante un sistema de puntuación (score) que analiza el comportamiento del visitante sin requerir interacción del usuario (sin desafíos visuales), mejorando así la experiencia de usuario mientras se mantiene un alto nivel de seguridad.

---

**Notas:**
- reCAPTCHA v3 es completamente invisible para el usuario - no requiere hacer clic en "No soy un robot" ni resolver desafíos visuales
- El sistema genera un score de 0.0 (muy probable que sea bot) a 1.0 (muy probable que sea humano) para cada interacción
- Se aplicará en los puntos críticos de entrada: autenticación y recuperación de contraseña
- Complementa pero NO reemplaza los mecanismos de bloqueo y limitación de tasa existentes
- Todo el proceso debe ser auditable siguiendo el Estándar de Auditoría Funcional
- Se debe cumplir con el estándar de seguridad APL03-AP-0019: "El sistema debe garantizar que quien realiza las acciones de registro, autenticación y restablecimiento de contraseña es un humano"

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Usuario legítimo** | Quiero que el sistema valide de forma invisible que soy humano cuando inicio sesión o solicito recuperación de contraseña | Para acceder al Portal sin fricción adicional mientras el sistema me protege contra suplantación por bots |
| Como **Administrador de Seguridad** | Quiero que el sistema detecte y bloquee automáticamente intentos de acceso provenientes de bots y scripts automatizados | Para prevenir ataques de fuerza bruta, scraping, y abuso del servicio sin afectar a usuarios legítimos |
| Como **Usuario** | Quiero recibir mensajes claros si el sistema no puede verificar que soy humano | Para entender por qué no puedo completar mi acción y qué pasos seguir (contactar soporte, intentar desde otro navegador, etc.) |
| Como **Auditor** | Quiero que todas las verificaciones anti-bot sean registradas con su resultado y score | Para investigar patrones de ataque, ajustar umbrales de detección, y garantizar cumplimiento de políticas de seguridad |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Verificación exitosa en autenticación | Usuario humano accede a pantalla de login (HU001), completa campos de usuario y contraseña | Usuario hace clic en botón "Ingresar" | Sistema ejecuta verificación anti-bot de forma invisible. Si score >= 0.5, la verificación es exitosa y el sistema procede con la validación de credenciales normalmente. El usuario NO percibe ningún paso adicional | ☐ | ☐ | ☐ |
| 2 | Verificación fallida en autenticación | Comportamiento automatizado (bot, script) intenta acceder a pantalla de login | Bot ejecuta submit del formulario de autenticación | Sistema ejecuta verificación anti-bot. Si score < 0.5, la verificación falla y el sistema muestra mensaje: "No se pudo verificar que no eres un robot. Por favor, intenta nuevamente desde un navegador actualizado o contacta a soporte." El inicio de sesión NO se procesa, independientemente de si las credenciales son correctas | ☐ | ☐ | ☐ |
| 3 | Verificación exitosa en recuperación de contraseña | Usuario humano accede a pantalla de recuperación de contraseña (HU002), ingresa usuario o correo válido | Usuario hace clic en "Enviar enlace de recuperación" | Sistema ejecuta verificación anti-bot de forma invisible. Si score >= 0.5, la verificación es exitosa y el sistema procede con el flujo normal de envío de enlace (si el usuario existe). El usuario NO percibe ningún paso adicional | ☐ | ☐ | ☐ |
| 4 | Verificación fallida en recuperación de contraseña | Bot automatizado intenta solicitar recuperación masiva de contraseñas | Bot ejecuta submit del formulario de recuperación | Sistema ejecuta verificación anti-bot. Si score < 0.5, la verificación falla y el sistema muestra mensaje: "No se pudo verificar que no eres un robot. Por favor, intenta nuevamente o contacta a soporte." NO se envía correo de recuperación, independientemente de si el usuario existe | ☐ | ☐ | ☐ |
| 5 | Mensaje de error claro y no técnico | Verificación anti-bot falla (score < 0.5) en cualquier pantalla | Usuario ve mensaje de error | Mensaje NO revela detalles técnicos sobre scores o reCAPTCHA. Mensaje es claro: "No se pudo verificar que no eres un robot. Por favor, intenta nuevamente o contacta a soporte." Incluye sugerencias: usar navegador actualizado, deshabilitar extensiones que puedan interferir, contactar soporte si persiste | ☐ | ☐ | ☐ |
| 6 | Usuario legítimo con score limítrofe (0.4-0.5) | Usuario legítimo navega con VPN, bloqueadores de anuncios, o desde conexión compartida/sospechosa, obtiene score entre 0.4 y 0.49 | Usuario intenta autenticarse o recuperar contraseña | Sistema rechaza la verificación por estar bajo el umbral de 0.5. Usuario recibe mensaje de error con orientación de contactar soporte. Sistema registra el evento en auditoría con severidad WARNING para revisión por administrador | ☐ | ☐ | ☐ |
| 7 | Indicador visual opcional durante verificación | Usuario hace clic en botón de "Ingresar" o "Enviar enlace de recuperación" | Sistema ejecuta verificación anti-bot (toma 300-1000ms) | Opcionalmente, el sistema puede mostrar indicador breve de "Verificando..." o deshabilitar temporalmente el botón para evitar doble envío. La verificación debe completarse en menos de 2 segundos en condiciones normales de red | ☐ | ☐ | ☐ |
| 8 | Error de servicio reCAPTCHA no disponible | Sistema no puede comunicarse con servicio de Google reCAPTCHA (timeout, servicio caído, error de red) | Usuario intenta autenticarse o recuperar contraseña | Sistema registra el error técnico en auditoría con severidad ERROR. Decisión de producto: (A) Permitir el acceso sin verificación anti-bot y alertar a administrador, o (B) Bloquear el acceso y mostrar mensaje: "Servicio de verificación temporalmente no disponible. Por favor, intenta en unos minutos." | ☐ | ☐ | ☐ |
| 9 | Múltiples intentos del mismo usuario legítimo | Usuario humano legítimo ha intentado autenticarse 3-4 veces en corto período (olvidó contraseña, errores de tipeo) | Usuario intenta autenticarse nuevamente | Sistema ejecuta verificación anti-bot normalmente. El score puede ser ligeramente más bajo por el patrón de intentos repetidos, pero si sigue >= 0.5, la verificación es exitosa. Los mecanismos de bloqueo de cuenta de HU001 (5 intentos fallidos) operan de forma independiente | ☐ | ☐ | ☐ |
| 10 | Comportamiento sospechoso progresivo | Usuario o bot ejecuta múltiples intentos rápidos (5+ intentos en menos de 1 minuto) desde la misma sesión/navegador | Sistema evalúa patrón de comportamiento | El score de reCAPTCHA v3 disminuye progresivamente con cada intento rápido. Eventualmente el score cae por debajo de 0.5 y las siguientes verificaciones fallan. Sistema registra patrón sospechoso en auditoría con severidad WARNING | ☐ | ☐ | ☐ |
| 11 | Usuario legítimo con JavaScript deshabilitado | Usuario accede con navegador que tiene JavaScript deshabilitado o bloqueado | Usuario intenta enviar formulario de login o recuperación | reCAPTCHA v3 requiere JavaScript para funcionar. Sistema detecta que JavaScript no está disponible y muestra mensaje: "Este sitio requiere JavaScript habilitado para verificación de seguridad. Por favor, habilita JavaScript en tu navegador o contacta a soporte." El formulario NO se procesa | ☐ | ☐ | ☐ |
| 12 | Configuración de umbral de score | Administrador de Seguridad configura el score mínimo aceptable | Administrador ajusta umbral desde 0.5 a otro valor (ej: 0.3 más permisivo, 0.7 más estricto) | Sistema aplica el nuevo umbral a todas las verificaciones subsecuentes. Cambio se registra en auditoría como evento de configuración. Nota: El valor 0.5 es el balanceado recomendado por Google | ☐ | ☐ | ☐ |
| 13 | Usuario desde ubicación geográfica inusual | Usuario legítimo viaja a otro país o accede desde ubicación geográfica diferente a la habitual | Usuario intenta autenticarse | reCAPTCHA v3 evalúa el contexto (IP, ubicación, comportamiento) y puede generar score ligeramente más bajo pero aún >= 0.5 si el comportamiento es humano. Verificación es exitosa. El cambio de ubicación NO debe por sí solo causar falla de verificación si el comportamiento es humano | ☐ | ☐ | ☐ |
| 14 | Privacidad del usuario y badge informativo | Usuario visualiza cualquier pantalla con reCAPTCHA v3 implementado | Usuario navega en la página | Sistema muestra badge pequeño en esquina inferior derecha con texto: "Este sitio está protegido por reCAPTCHA y se aplican la Política de privacidad y Términos de servicio de Google" con enlaces a políticas de Google. Badge cumple requisitos de transparencia de Google | ☐ | ☐ | ☐ |
| 15 | Acceso desde navegadores antiguos o no compatibles | Usuario accede desde navegador muy antiguo (IE 10 o anterior, versiones obsoletas de Chrome/Firefox) que no soporta completamente reCAPTCHA v3 | Usuario intenta autenticarse o recuperar contraseña | Sistema detecta incompatibilidad o reCAPTCHA retorna error. Sistema muestra mensaje: "Tu navegador no es compatible con las medidas de seguridad del sitio. Por favor, actualiza a una versión reciente de Chrome, Firefox, Edge o Safari." Proporciona enlaces a descargas de navegadores | ☐ | ☐ | ☐ |
| 16 | Combinación con bloqueo de cuenta existente | Usuario bloqueado por 5 intentos fallidos (HU001 Escenario 5) intenta autenticarse | Usuario bloqueado hace clic en "Ingresar" | Sistema ejecuta verificación anti-bot PRIMERO. Si pasa (score >= 0.5), entonces evalúa estado de bloqueo de cuenta. Muestra mensaje de cuenta bloqueada según HU001: "Tu cuenta ha sido bloqueada por múltiples intentos fallidos. Por favor, intenta nuevamente en 30 minutos o contacta a soporte." La verificación anti-bot NO desbloquea cuentas | ☐ | ☐ | ☐ |
| 17 | Combinación con límite de recuperación de contraseña | Usuario ha alcanzado límite de 5 solicitudes de recuperación en 24 horas (HU002 Escenario 15) | Usuario intenta solicitar recuperación nuevamente | Sistema ejecuta verificación anti-bot PRIMERO. Si pasa (score >= 0.5), entonces evalúa límite de solicitudes. Muestra mensaje de límite excedido según HU002: "Has excedido el número máximo de solicitudes de recuperación. Por favor, intenta nuevamente en 24 horas o contacta a soporte." | ☐ | ☐ | ☐ |
| 18 | Pruebas y ambiente de desarrollo | Equipo de desarrollo o QA necesita probar funcionalidad sin verificación real de reCAPTCHA | Equipo ejecuta pruebas en ambiente de desarrollo/staging | Sistema utiliza claves de prueba de Google reCAPTCHA que siempre retornan score de 1.0 (siempre pasa). Alternativamente, existe configuración para deshabilitar verificación en ambientes no productivos. En PRODUCCIÓN, la verificación SIEMPRE está activa | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 19 | Estructura obligatoria de registro de auditoría | Cualquier verificación anti-bot (exitosa, fallida, error de servicio) en autenticación o recuperación de contraseña | Sistema ejecuta verificación | Registro contiene: ID Único del Evento (UUID), Tipo de Evento (formato MODULO_ENTIDAD_ACCION), Fecha y Hora (UTC con milisegundos, ISO 8601), Usuario (nombre de usuario ingresado o "ANONIMO" si no identificable), Cliente (NIT - NULL si no aplica), Cliente (Nombre - NULL si no aplica), IP Local, IP Pública, Resultado (EXITOSO/FALLIDO), Descripción (texto descriptivo del evento), Nivel de Severidad (INFO/WARNING/ERROR), Datos Adicionales (JSON con score, acción, token_id si aplica) | ☐ | ☐ | ☐ |
| 20 | Auditoría de verificación exitosa en autenticación | Usuario pasa verificación anti-bot (score >= 0.5) en pantalla de login | Usuario envía formulario de autenticación | Registro de auditoría: Tipo de Evento: "SEGURIDAD_ANTIBOT_VERIFICACION_EXITOSA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Verificación anti-bot exitosa para usuario {usuario} en autenticación", Datos Adicionales: {"accion": "login", "score": 0.87, "umbral": 0.5, "navegador": "Chrome 120", "ip_publica": "IP"} | ☐ | ☐ | ☐ |
| 21 | Auditoría de verificación fallida en autenticación | Bot o comportamiento automatizado falla verificación (score < 0.5) en login | Bot intenta autenticarse | Registro de auditoría: Tipo de Evento: "SEGURIDAD_ANTIBOT_VERIFICACION_FALLIDA", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Verificación anti-bot fallida para usuario {usuario} en autenticación - posible bot detectado", Datos Adicionales: {"accion": "login", "score": 0.23, "umbral": 0.5, "motivo": "score_bajo", "navegador": "user-agent", "ip_publica": "IP", "patron_sospechoso": true} | ☐ | ☐ | ☐ |
| 22 | Auditoría de verificación exitosa en recuperación | Usuario pasa verificación anti-bot (score >= 0.5) en pantalla de recuperación de contraseña | Usuario solicita recuperación de contraseña | Registro de auditoría: Tipo de Evento: "SEGURIDAD_ANTIBOT_VERIFICACION_EXITOSA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Verificación anti-bot exitosa para usuario {usuario} en recuperación de contraseña", Datos Adicionales: {"accion": "forgot_password", "score": 0.92, "umbral": 0.5, "navegador": "Firefox 119", "ip_publica": "IP"} | ☐ | ☐ | ☐ |
| 23 | Auditoría de verificación fallida en recuperación | Bot o comportamiento automatizado falla verificación en recuperación de contraseña | Bot intenta solicitud masiva de recuperación | Registro de auditoría: Tipo de Evento: "SEGURIDAD_ANTIBOT_VERIFICACION_FALLIDA", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Verificación anti-bot fallida en recuperación de contraseña - posible ataque de denegación de servicio", Datos Adicionales: {"accion": "forgot_password", "score": 0.11, "umbral": 0.5, "motivo": "score_muy_bajo", "patron_ataque": "solicitudes_masivas", "ip_publica": "IP"} | ☐ | ☐ | ☐ |
| 24 | Auditoría de error de servicio reCAPTCHA | Servicio de Google reCAPTCHA no está disponible o retorna error | Sistema intenta verificar pero falla por error técnico | Registro de auditoría: Tipo de Evento: "SEGURIDAD_ANTIBOT_ERROR_SERVICIO", Resultado: "FALLIDO", Severidad: "ERROR", Descripción: "Error al comunicarse con servicio de verificación anti-bot para usuario {usuario}", Datos Adicionales: {"accion": "login o forgot_password", "error_tipo": "timeout", "error_mensaje": "Connection timeout after 5000ms", "ip_publica": "IP", "accion_tomada": "acceso_permitido o acceso_bloqueado"} | ☐ | ☐ | ☐ |
| 25 | Auditoría de patrón de comportamiento sospechoso | Sistema detecta múltiples verificaciones fallidas desde misma IP en corto período | 10+ verificaciones fallidas en 5 minutos desde misma IP pública | Registro de auditoría: Tipo de Evento: "SEGURIDAD_ANTIBOT_PATRON_ATAQUE", Resultado: "FALLIDO", Severidad: "ERROR", Descripción: "Patrón de ataque automatizado detectado - múltiples verificaciones anti-bot fallidas desde {IP}", Datos Adicionales: {"intentos_fallidos": 15, "periodo_minutos": 5, "ip_publica": "IP", "scores_promedio": 0.18, "acciones": ["login", "forgot_password"], "posible_ataque": "fuerza_bruta_distribuida"} | ☐ | ☐ | ☐ |
| 26 | Auditoría de usuario con score limítrofe | Usuario legítimo obtiene score entre umbral y umbral-0.1 (ej: 0.4-0.49 cuando umbral es 0.5) | Usuario es rechazado por estar bajo umbral pero cerca del límite | Registro de auditoría: Tipo de Evento: "SEGURIDAD_ANTIBOT_SCORE_LIMITROFE", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} rechazado por verificación anti-bot con score limítrofe - posible falso positivo", Datos Adicionales: {"accion": "login", "score": 0.47, "umbral": 0.5, "diferencia": -0.03, "sugerencia_revision": true, "navegador": "user-agent", "ip_publica": "IP"} | ☐ | ☐ | ☐ |
| 27 | Auditoría de JavaScript deshabilitado | Usuario intenta acceder sin JavaScript habilitado | Sistema detecta falta de JavaScript | Registro de auditoría: Tipo de Evento: "SEGURIDAD_ANTIBOT_SIN_JAVASCRIPT", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Intento de acceso sin JavaScript habilitado para usuario {usuario}", Datos Adicionales: {"accion": "login o forgot_password", "javascript_habilitado": false, "ip_publica": "IP", "navegador": "user-agent"} | ☐ | ☐ | ☐ |
| 28 | Auditoría de cambio de configuración de umbral | Administrador modifica el score umbral de verificación | Administrador cambia umbral de 0.5 a 0.3 o 0.7 | Registro de auditoría: Tipo de Evento: "SEGURIDAD_ANTIBOT_CONFIGURACION_MODIFICADA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {admin_usuario} modificó umbral de verificación anti-bot", Datos Adicionales: {"umbral_anterior": 0.5, "umbral_nuevo": 0.7, "razon_cambio": "Aumentar seguridad por detección de ataques", "fecha_cambio": "timestamp", "ip_administrador": "IP"} | ☐ | ☐ | ☐ |
| 29 | Auditoría de navegador incompatible | Usuario intenta acceder desde navegador antiguo o no compatible | Sistema detecta navegador obsoleto | Registro de auditoría: Tipo de Evento: "SEGURIDAD_ANTIBOT_NAVEGADOR_INCOMPATIBLE", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Intento de acceso desde navegador no compatible con verificación anti-bot", Datos Adicionales: {"navegador": "Internet Explorer 10", "version": "10.0", "compatible": false, "ip_publica": "IP", "usuario": "{usuario o ANONIMO}"} | ☐ | ☐ | ☐ |
| 30 | Inmutabilidad de registros de auditoría | Cualquier escenario de auditoría de verificación anti-bot | Sistema genera registro | Registros de auditoría NO pueden ser modificados ni eliminados. Solo se permiten consultas de lectura. Implementar controles que prevengan alteración de registros históricos | ☐ | ☐ | ☐ |
| 31 | Consulta de registros para análisis de seguridad | Administrador de Seguridad requiere analizar patrones de ataques o ajustar umbrales | Administrador accede a módulo de auditoría | Sistema permite filtrar por: rango de fechas, tipo de evento (SEGURIDAD_ANTIBOT_*), resultado (EXITOSO/FALLIDO), severidad (INFO/WARNING/ERROR), rango de scores (ej: 0.0-0.3 para bots evidentes), IP origen, acción (login/forgot_password). Permite exportar resultados en formato CSV/Excel. Muestra estadísticas: score promedio, tasa de rechazo, IPs más bloqueadas | ☐ | ☐ | ☐ |
| 32 | Retención de registros de auditoría | Registros de auditoría de verificaciones anti-bot | Paso del tiempo | Registros se conservan por tiempo mínimo definido por regulación (7 años para documentos fiscales, aplicar misma política para eventos de seguridad). Después de este período, pueden ser archivados en almacenamiento de largo plazo o eliminados según política de retención | ☐ | ☐ | ☐ |
| 33 | Alertas automáticas por patrones anómalos | Sistema detecta tasa inusualmente alta de verificaciones fallidas (>50 en 10 minutos) o score promedio muy bajo (<0.2) | Umbral de alerta es superado | Sistema genera alerta automática a equipo de seguridad vía correo electrónico o sistema de monitoreo. Alerta incluye: número de intentos fallidos, IPs involucradas, scores promedio, período de tiempo, posible tipo de ataque. Alerta se registra en auditoría con severidad ERROR | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

Se modificarán los mockups existentes para incluir la verificación anti-bot:

#### 1. **HU001-Login.html** (Modificación)
**Elementos a agregar:**
- Script de Google reCAPTCHA v3 en `<head>` del documento
- Indicador visual opcional de "Verificando..." durante la verificación (300-1000ms)
- Mensaje de error específico si verificación anti-bot falla: "No se pudo verificar que no eres un robot. Por favor, intenta nuevamente o contacta a soporte."
- Badge informativo en esquina inferior derecha: "Este sitio está protegido por reCAPTCHA..." con enlaces a políticas de Google
- Deshabilitar botón "Ingresar" durante verificación para evitar doble envío

#### 2. **HU002-ForgotPassword.html** (Modificación)
**Elementos a agregar:**
- Script de Google reCAPTCHA v3 en `<head>` del documento
- Indicador visual opcional de "Verificando..." durante la verificación
- Mensaje de error específico en Alert error si verificación falla: "No se pudo verificar que no eres un robot. Por favor, intenta nuevamente o contacta a soporte."
- Badge informativo en esquina inferior: "Este sitio está protegido por reCAPTCHA y se aplican la Política de privacidad y Términos de servicio de Google"
- Deshabilitar botón "Enviar enlace de recuperación" durante verificación

#### 3. **HU003-BrowserNotSupported.html** (Nuevo - Opcional)
**Elementos:**
- Pantalla informativa para usuarios con JavaScript deshabilitado o navegadores no compatibles
- Título: "Navegador no compatible"
- Mensaje: "Este sitio requiere JavaScript habilitado y un navegador moderno para garantizar la seguridad de tu cuenta."
- Instrucciones: Cómo habilitar JavaScript, enlaces a descargas de navegadores modernos (Chrome, Firefox, Edge, Safari)
- Información de contacto de soporte para asistencia

#### 4. **Panel de Administración - Configuración de Seguridad** (Nuevo - Futuro)
**Elementos:**
- Sección de configuración de reCAPTCHA
- Campo: "Score umbral mínimo" (slider de 0.0 a 1.0, valor por defecto: 0.5)
- Descripción de umbrales: "0.3 Permisivo | 0.5 Balanceado (recomendado) | 0.7 Estricto"
- Estadísticas en tiempo real: Total de verificaciones hoy, tasa de éxito, score promedio, intentos bloqueados
- Botón: "Guardar configuración" con confirmación

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Falsos positivos (usuarios legítimos bloqueados)** | Usuarios humanos legítimos pueden obtener scores bajos (<0.5) debido a VPN, bloqueadores de anuncios, comportamiento de navegación inusual, o acceso desde redes corporativas restrictivas | Alto | Media | Umbral balanceado de 0.5 (recomendado por Google). Mensaje de error con instrucciones claras de contactar soporte. Auditoría de casos limítrofes (scores 0.4-0.5) con severidad WARNING para revisión. Proceso de soporte para validación manual de usuarios bloqueados incorrectamente | Mitigado |
| **Dependencia de servicio externo (Google)** | Si el servicio de Google reCAPTCHA está caído o inalcanzable, el sistema no puede verificar a los usuarios | Alto | Baja | Implementar timeout de 5 segundos en llamada a reCAPTCHA. Definir comportamiento de fallback: (A) Permitir acceso sin verificación y alertar a administrador (prioriza disponibilidad), o (B) Bloquear acceso temporalmente (prioriza seguridad). Monitorear estado del servicio y registrar eventos con severidad ERROR | Aceptado con controles |
| **Privacidad y cumplimiento GDPR/regulaciones** | reCAPTCHA v3 de Google procesa información del usuario (IP, cookies, comportamiento de navegación) y la envía a servidores de Google, lo que puede tener implicaciones de privacidad según GDPR y regulaciones locales de protección de datos | Medio | Media | Incluir información de reCAPTCHA en Política de Privacidad del Portal. Mostrar badge obligatorio de Google en todas las páginas con reCAPTCHA. Incluir consentimiento de uso de cookies/análisis de comportamiento en términos de uso. Evaluar alternativas si la jurisdicción no permite servicios de terceros (ej: hCaptcha, Cloudflare Turnstile) | Aceptado con transparencia |
| **Evasión por bots sofisticados** | Bots avanzados con motores de navegación headless (Puppeteer, Selenium) pueden imitar comportamiento humano y obtener scores altos (>0.5), evadiendo la protección | Medio | Media | reCAPTCHA v3 NO es infalible, es una capa de defensa en profundidad. Mantener controles complementarios: bloqueo de cuenta por intentos fallidos (HU001), límite de solicitudes (HU002), monitoreo de patrones anómalos, análisis de logs de auditoría. Ajustar umbral a 0.7 si se detectan ataques sofisticados | Aceptado |
| **Costos de servicio reCAPTCHA** | Google reCAPTCHA v3 tiene cuota gratuita de 1,000,000 de evaluaciones/mes. Si el portal excede este volumen, Google puede cobrar $1 USD por cada 1,000 evaluaciones adicionales | Bajo | Baja | Para Release 1 con base de usuarios inicial, 1M evaluaciones/mes es suficiente. Monitorear uso mensual en consola de Google reCAPTCHA. Si se aproxima al límite, evaluar: (A) Actualizar plan con Google, (B) Aplicar reCAPTCHA solo en escenarios de alto riesgo, (C) Migrar a alternativa (hCaptcha es gratuito sin límites) | Aceptado |
| **Experiencia degradada para usuarios con JavaScript deshabilitado** | Usuarios con JavaScript deshabilitado (ya sea intencionalmente por seguridad, o por políticas corporativas) no pueden completar la verificación anti-bot y quedan bloqueados del portal | Medio | Baja | Mostrar mensaje claro y no técnico: "Este sitio requiere JavaScript para verificación de seguridad...". Proveer canal alternativo (soporte telefónico/email) para estos usuarios. Documentar que el Portal requiere JavaScript moderno como requisito técnico | Aceptado |
| **Ajuste incorrecto de umbral** | Administrador configura umbral muy bajo (ej: 0.2) permitiendo bots, o muy alto (ej: 0.9) bloqueando usuarios legítimos | Alto | Baja | Valor por defecto de 0.5 (balanceado, recomendado por Google). Interfaz de configuración muestra advertencias: "Umbral <0.4 puede permitir bots sofisticados" y "Umbral >0.7 puede bloquear usuarios legítimos". Requiere confirmación explícita al cambiar umbral. Auditar cambios de configuración con usuario administrador, timestamp, y razón del cambio | Mitigado |
| **Impacto en rendimiento de carga de página** | El script de reCAPTCHA v3 (aproximadamente 60-80 KB) puede incrementar el tiempo de carga inicial de las páginas de login y recuperación | Bajo | Alta | Cargar script de forma asíncrona (`async defer` en tag `<script>`). reCAPTCHA v3 está optimizado y se carga en paralelo. Impacto estimado: <500ms adicionales en conexión 3G, <100ms en conexión rápida. Beneficio de seguridad justifica el pequeño overhead de rendimiento | Aceptado |

---

## Notas Adicionales

### Fuera de Alcance (Out of Scope)
- Implementación de reCAPTCHA v2 (con desafío visual de "No soy un robot") - no es necesario con v3
- reCAPTCHA en otras pantallas del portal más allá de login y recuperación de contraseña (ej: formularios de contacto, registro de nuevos usuarios) - se evaluará en futuras HUs
- Integración con soluciones anti-bot alternativas (hCaptcha, Cloudflare Turnstile, AWS WAF) - solo Google reCAPTCHA v3 en esta HU
- Sistema de whitelist de IPs confiables que omitan verificación anti-bot - puede considerarse en futuras versiones si hay casos de uso justificados
- Análisis de machine learning personalizado sobre patrones de ataque - reCAPTCHA v3 ya provee este análisis

### Consideraciones Técnicas para Implementación (Informativas)
- **Frontend:** Cargar script de Google `https://www.google.com/recaptcha/api.js?render=SITE_KEY` de forma asíncrona. Ejecutar `grecaptcha.execute()` antes de enviar formulario. Enviar token generado al backend en campo oculto o header HTTP
- **Backend:** Validar token con API de Google `https://www.google.com/recaptcha/api/siteverify` usando Secret Key. Evaluar score retornado (0.0-1.0) contra umbral configurado (0.5). Rechazar solicitud si score < umbral
- **Claves:** Site Key (pública, va en frontend) y Secret Key (privada, solo backend). Registrar dominio en Google reCAPTCHA Admin Console: https://www.google.com/recaptcha/admin
- **Ambientes:** Usar claves de prueba de Google para desarrollo/staging que siempre retornan score 1.0. Usar claves reales solo en producción
- **Timeout:** Configurar timeout de 5 segundos en llamada a API de Google. Implementar retry 1 vez antes de considerar falla definitiva
- **Acciones:** reCAPTCHA v3 permite nombrar acciones (ej: 'login', 'forgot_password'). Usar nombres descriptivos para análisis en consola de Google
- **Logs:** NO registrar tokens de reCAPTCHA completos en logs (son efímeros, válidos 2 minutos, y ocupan espacio). Solo registrar score, acción, y resultado
- **Cache:** NO cachear resultados de verificación. Cada submit debe ejecutar nueva verificación para detectar cambios de comportamiento

### Dependencias
- **HU001 - Autenticación y Selección de Cliente**: La verificación anti-bot se implementa ANTES de la validación de credenciales en el flujo de login
- **HU002 - Recuperación de Contraseña**: La verificación anti-bot se implementa ANTES de validar formato de usuario/correo y enviar enlace de recuperación
- **Cuenta de Google Cloud o Google reCAPTCHA Admin**: Necesaria para obtener Site Key y Secret Key
- **Conexión a internet desde frontend y backend**: Para comunicarse con APIs de Google
- **Módulo de Auditoría**: Debe estar implementado para registrar todos los eventos de verificación anti-bot

### Referencias
- **Estándar de Auditoría Funcional**: Seguir estructura obligatoria de 13 campos, principios de inmutabilidad, formato de tipos de eventos (SEGURIDAD_ANTIBOT_*), niveles de severidad
- **Estándar de Seguridad APL03**:
  - AP-0019: Garantizar que quien realiza acciones es humano (CAPTCHA o retrasos)
  - AP-0022: Registro de eventos de seguridad
  - AP-0024: Registro detallado con contexto (IP, timestamps, resultados)
- **Glosario**: Usuario, Trazabilidad, No Repudio, Auditoría
- **Documentación de Google reCAPTCHA v3**: https://developers.google.com/recaptcha/docs/v3
- **Guía de estilos**: Mensajes de error en Material UI Alert (color error #f44336), badges informativos, indicadores de carga

### Valores Recomendados de Score Umbral

Según la documentación de Google reCAPTCHA v3:

| Score | Interpretación | Recomendación |
|-------|---------------|---------------|
| **0.0 - 0.1** | Muy probable que sea bot | Bloquear inmediatamente |
| **0.1 - 0.3** | Probable que sea bot | Bloquear con alta confianza |
| **0.3 - 0.5** | Comportamiento sospechoso | Aplicar controles adicionales o bloquear si umbral es 0.5 |
| **0.5 - 0.7** | Neutral - comportamiento mixto | Permitir con monitoreo (umbral recomendado: 0.5) |
| **0.7 - 0.9** | Probable que sea humano | Permitir con confianza |
| **0.9 - 1.0** | Muy probable que sea humano | Permitir con alta confianza |

**Umbral recomendado para este proyecto: 0.5** (balance entre seguridad y experiencia de usuario)

### Plan de Rollout Sugerido

1. **Fase 1 - Desarrollo y Staging (2 semanas):**
   - Implementar con claves de prueba de Google
   - Probar con diferentes navegadores, VPNs, bloqueadores
   - Validar flujo completo de auditoría
   - QA exhaustivo de casos límite

2. **Fase 2 - Producción en modo observación (2 semanas):**
   - Implementar en producción con umbral muy permisivo (0.3)
   - NO bloquear usuarios, solo registrar en auditoría
   - Analizar distribución de scores reales
   - Identificar patrones de usuarios legítimos vs bots

3. **Fase 3 - Activación gradual (1 semana):**
   - Aumentar umbral a 0.5 (balanceado)
   - Monitorear tasa de falsos positivos
   - Ajustar según métricas observadas

4. **Fase 4 - Optimización continua:**
   - Revisar logs de auditoría mensualmente
   - Ajustar umbral si es necesario según patrones de ataque
   - Capacitar a soporte para atender casos de falsos positivos
