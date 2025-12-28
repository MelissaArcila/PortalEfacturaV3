# HU007 - Contraseña Temporal y Primer Cambio Obligatorio

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
| **Arquitectura relacionada** | Portal Unificado - Módulo de Autenticación, Gestión de Usuarios, Sistema de Notificaciones |

---

## Contexto

Cuando un Administrador del Portal crea un nuevo Usuario en el sistema (HU006 - Creación de Usuarios), el Usuario necesita recibir credenciales de acceso para poder autenticarse por primera vez. Por razones de seguridad, no es apropiado que el Administrador conozca o defina la contraseña definitiva del Usuario, ni que la contraseña se transmita por canales inseguros como mensajes de texto o comunicación verbal.

La solución es generar automáticamente una contraseña temporal segura y enviarla al correo electrónico registrado del Usuario. Esta contraseña temporal debe ser de un solo uso y debe expirar después de un período razonable (72 horas) para minimizar riesgos de seguridad. Cuando el Usuario inicia sesión por primera vez con su contraseña temporal, el sistema debe detectar esta condición y forzarlo a cambiar inmediatamente su contraseña por una nueva que solo él conozca, antes de permitirle acceder al Portal.

Este flujo garantiza que solo el Usuario legítimo (quien tiene acceso al correo registrado) puede activar su cuenta, y que las contraseñas nunca son compartidas ni conocidas por terceros, cumpliendo con las mejores prácticas de seguridad y el estándar APL03.

---

**Notas:**
- La contraseña temporal se genera AUTOMÁTICAMENTE al crear el Usuario exitosamente (HU006)
- La contraseña temporal es alfanumérica aleatoria de 12 caracteres (mayúsculas, minúsculas, números, símbolos)
- Se envía SOLO por correo electrónico al correo registrado del Usuario
- El Usuario DEBE tener correo electrónico registrado para recibir la contraseña temporal
- La contraseña temporal expira en 72 horas (3 días) desde su generación
- Al autenticarse con contraseña temporal (HU001), el sistema FUERZA cambio inmediato
- La nueva contraseña debe cumplir requisitos de seguridad (similar a HU002)
- No se puede reutilizar la contraseña temporal como nueva contraseña
- Todo el proceso debe ser auditable siguiendo el Estándar de Auditoría Funcional
- Si el correo falla al enviar, se debe informar al Administrador y registrar en auditoría

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador del Portal** | Quiero que al crear un Usuario exitosamente se genere automáticamente una contraseña temporal segura | Para que el Usuario pueda iniciar sesión sin que yo tenga que conocer o definir su contraseña |
| Como **Administrador del Portal** | Quiero que la contraseña temporal se envíe automáticamente por correo electrónico al Usuario | Para que solo el Usuario legítimo (quien tiene acceso al correo) pueda activar su cuenta |
| Como **Usuario recién creado** | Quiero recibir un correo con mi contraseña temporal e instrucciones claras | Para poder iniciar sesión por primera vez de forma segura |
| Como **Usuario** | Quiero que el sistema me obligue a cambiar mi contraseña temporal por una nueva en el primer inicio de sesión | Para garantizar que solo yo conozca mi contraseña y nadie más tenga acceso a mi cuenta |
| Como **Usuario** | Quiero que el sistema valide que mi nueva contraseña cumple con requisitos de seguridad | Para asegurar que mi cuenta esté protegida contra accesos no autorizados |
| Como **Administrador de Seguridad** | Quiero que las contraseñas temporales expiren después de 72 horas | Para minimizar el riesgo de que alguien use una contraseña interceptada después de un período razonable |
| Como **Auditor** | Quiero que se registre la generación, envío, uso y cambio de contraseñas temporales | Para garantizar trazabilidad completa del proceso de activación de cuentas y detectar posibles brechas de seguridad |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Generación automática de contraseña temporal al crear Usuario | Administrador del Portal ha completado creación de Usuario (HU006), Usuario tiene correo electrónico registrado | Sistema confirma creación exitosa del Usuario | Sistema genera automáticamente contraseña temporal aleatoria de 12 caracteres: 4 letras mayúsculas (A-Z), 4 letras minúsculas (a-z), 2 números (0-9), 2 símbolos (!@#$%^&*). Contraseña es criptográficamente aleatoria y única. Sistema marca la contraseña como "temporal" en base de datos, establece fecha de expiración: fecha_actual + 72 horas, y almacena hash de la contraseña (nunca en texto plano) | ☐ | ☐ | ☐ |
| 2 | Envío de correo con contraseña temporal | Sistema ha generado contraseña temporal exitosamente | Sistema procesa envío de correo | Sistema envía correo electrónico al correo registrado del Usuario con: Asunto: "Bienvenido al Portal Unificado CDN Facturación - Credenciales de Acceso", Cuerpo: Saludo personalizado con nombre del Usuario, contraseña temporal visible en texto monospace, instrucciones paso a paso para primer inicio de sesión, tiempo de expiración (72 horas), advertencia de seguridad (no compartir contraseña), enlace al portal de inicio de sesión, información de contacto de soporte. Plantilla HTML profesional con logo y colores corporativos | ☐ | ☐ | ☐ |
| 3 | Envío exitoso de correo electrónico | Sistema intenta enviar correo con contraseña temporal | Servicio de correo procesa envío | Sistema recibe confirmación de envío exitoso del servicio de correo. Muestra mensaje al Administrador en pantalla de creación exitosa: "¡Usuario creado exitosamente! Se ha enviado un correo con la contraseña temporal a [correo_usuario]. El usuario debe cambiar su contraseña en el primer inicio de sesión." Registra evento de envío exitoso en auditoría | ☐ | ☐ | ☐ |
| 4 | Error al enviar correo electrónico | Sistema intenta enviar correo pero servicio de correo falla (timeout, servidor caído, correo inválido) | Servicio de correo retorna error | Sistema detecta fallo en envío. Muestra mensaje al Administrador: "Usuario creado exitosamente, pero ocurrió un error al enviar el correo con la contraseña temporal. Por favor, contacte al usuario por otro medio o genere una nueva contraseña temporal desde la opción 'Resetear Contraseña'." Registra error de envío en auditoría con severidad ERROR. Usuario queda creado pero SIN contraseña temporal enviada | ☐ | ☐ | ☐ |
| 5 | Usuario sin correo electrónico registrado | Administrador intenta crear Usuario (HU006) sin proporcionar correo electrónico | Sistema valida datos antes de crear | Sistema muestra advertencia en pantalla de resumen (HU006): "Este usuario no tiene correo electrónico registrado. No se podrá enviar contraseña temporal automáticamente. Deberá configurar la contraseña manualmente después de la creación." Permite crear Usuario pero NO genera ni envía contraseña temporal. Usuario queda en estado "Sin contraseña" y Administrador debe usar función "Establecer Contraseña Manual" | ☐ | ☐ | ☐ |
| 6 | Primer inicio de sesión con contraseña temporal válida | Usuario recién creado accede a pantalla de login (HU001), contraseña temporal NO ha expirado (< 72 horas) | Usuario ingresa número de identificación y contraseña temporal recibida por correo, hace clic en "Ingresar" | Sistema valida credenciales, detecta que la contraseña es temporal (flag "es_temporal = true" en BD), valida que NO ha expirado (fecha_actual < fecha_expiracion). Muestra mensaje informativo: "Bienvenido al Portal Unificado. Por seguridad, debe cambiar su contraseña temporal por una nueva." Redirige AUTOMÁTICAMENTE a pantalla de "Cambio de Contraseña Obligatorio" SIN permitir acceso al Portal | ☐ | ☐ | ☐ |
| 7 | Visualización de pantalla de cambio de contraseña obligatorio | Usuario autenticado con contraseña temporal, redirigido a pantalla de cambio | Sistema carga pantalla | Sistema muestra pantalla de "Cambio de Contraseña Obligatorio" con: Título: "Cambio de Contraseña Requerido", Mensaje: "Por seguridad, debe establecer una nueva contraseña. Esta será su contraseña definitiva para acceder al Portal.", Campo: "Nueva Contraseña" (tipo password con icono mostrar/ocultar), Indicador visual de requisitos en tiempo real (similar a HU002), Campo: "Confirmar Nueva Contraseña" (tipo password), Botón: "Cambiar Contraseña" (deshabilitado hasta cumplir requisitos), NO tiene botón "Cancelar" (acción es obligatoria) | ☐ | ☐ | ☐ |
| 8 | Indicador visual de requisitos de contraseña | Usuario en pantalla de cambio obligatorio, escribe en campo "Nueva Contraseña" | Usuario ingresa caracteres | Sistema muestra indicador visual en tiempo real con checkmarks (✓ verde) y cruces (✗ rojo): "Mínimo 8 caracteres ✗/✓", "Al menos una mayúscula ✗/✓", "Al menos una minúscula ✗/✓", "Al menos un número ✗/✓", "Al menos un símbolo (!@#$%^&*) ✗/✓", "No puede ser igual a contraseña temporal ✗/✓". Botón "Cambiar Contraseña" se habilita solo cuando TODOS los requisitos muestran ✓ | ☐ | ☐ | ☐ |
| 9 | Validación de requisitos de nueva contraseña | Usuario completa campo "Nueva Contraseña" | Usuario escribe y hace blur del campo | Sistema valida que la contraseña cumple: (1) Mínimo 8 caracteres, (2) Al menos una mayúscula (A-Z), (3) Al menos una minúscula (a-z), (4) Al menos un número (0-9), (5) Al menos un símbolo (!@#$%^&*), (6) NO es igual a la contraseña temporal recibida. Si no cumple, muestra errores específicos y deshabilita botón "Cambiar Contraseña" | ☐ | ☐ | ☐ |
| 10 | Confirmación de nueva contraseña no coincide | Usuario ingresa nueva contraseña válida pero confirmación no coincide | Usuario completa campo "Confirmar Nueva Contraseña" con texto diferente | Sistema detecta que contraseñas no coinciden. Muestra error debajo del campo "Confirmar Nueva Contraseña": "Las contraseñas no coinciden". Botón "Cambiar Contraseña" permanece deshabilitado | ☐ | ☐ | ☐ |
| 11 | Nueva contraseña igual a contraseña temporal | Usuario intenta usar la misma contraseña temporal como nueva contraseña | Usuario ingresa en "Nueva Contraseña" el mismo valor de la temporal | Sistema detecta coincidencia con contraseña temporal. Muestra error: "No puede usar la contraseña temporal como su nueva contraseña. Debe establecer una contraseña diferente." Botón permanece deshabilitado | ☐ | ☐ | ☐ |
| 12 | Cambio de contraseña exitoso en primer inicio de sesión | Usuario cumple todos los requisitos, contraseñas coinciden | Usuario hace clic en "Cambiar Contraseña" | Sistema valida todos los requisitos, guarda hash de la nueva contraseña en base de datos, marca flag "es_temporal = false", elimina fecha de expiración, reinicia contador de intentos fallidos a 0, registra cambio en auditoría. Muestra mensaje de éxito: "Contraseña cambiada exitosamente. Redirigiendo al portal..." Espera 2 segundos y redirige automáticamente a flujo de selección de cliente (HU001 Escenario 2 o 3) o ingreso directo si tiene un solo cliente (HU001 Escenario 1) | ☐ | ☐ | ☐ |
| 13 | Intento de inicio de sesión con contraseña temporal expirada | Usuario intenta autenticarse con contraseña temporal más de 72 horas después de su generación | Usuario ingresa número ID y contraseña temporal en pantalla de login, hace clic en "Ingresar" | Sistema valida credenciales, detecta que contraseña es temporal, valida fecha de expiración (fecha_actual >= fecha_expiracion). Muestra mensaje: "Su contraseña temporal ha expirado. Por favor, contacte al administrador para solicitar una nueva." NO permite acceso. Registra intento con contraseña expirada en auditoría con severidad WARNING | ☐ | ☐ | ☐ |
| 14 | Regeneración de contraseña temporal por Administrador | Usuario no recibió correo, correo expiró, o necesita nueva contraseña temporal | Administrador accede a detalle de Usuario y hace clic en "Resetear Contraseña" (genera nueva temporal) | Sistema invalida contraseña temporal anterior (si existe), genera nueva contraseña temporal aleatoria (mismo algoritmo de 12 caracteres), establece nueva fecha de expiración (fecha_actual + 72 horas), envía correo con nueva contraseña temporal al correo registrado. Muestra confirmación al Administrador: "Nueva contraseña temporal generada y enviada a [correo]". Registra regeneración en auditoría | ☐ | ☐ | ☐ |
| 15 | Usuario intenta acceder al Portal sin cambiar contraseña temporal | Usuario autenticado con contraseña temporal intenta navegar directamente a URL del Portal mediante manipulación | Usuario escribe URL directa del Portal en navegador | Sistema valida en cada request que el Usuario tiene flag "es_temporal = false". Si detecta "es_temporal = true", redirige FORZOSAMENTE a pantalla de "Cambio de Contraseña Obligatorio" sin permitir acceso. Muestra mensaje: "Debe cambiar su contraseña temporal antes de acceder al sistema" | ☐ | ☐ | ☐ |
| 16 | Barra de fortaleza de contraseña | Usuario escribe en campo "Nueva Contraseña" | Sistema analiza fortaleza en tiempo real | Sistema muestra barra de progreso visual con colores: Rojo (Débil: solo cumple 1-2 requisitos), Naranja (Media: cumple 3-4 requisitos), Verde (Fuerte: cumple todos los 5+ requisitos). Texto debajo: "Débil", "Media", "Fuerte". Ayuda al Usuario a elegir contraseña segura | ☐ | ☐ | ☐ |
| 17 | Campo de contraseña con visibilidad toggle | Usuario en pantalla de cambio obligatorio | Usuario hace clic en icono "ojo" junto al campo de contraseña | Sistema alterna entre mostrar contraseña en texto plano y ocultar con asteriscos (•••). Icono cambia de "ojo abierto" a "ojo cerrado" según estado. Facilita verificación visual de lo que el Usuario escribió | ☐ | ☐ | ☐ |
| 18 | Timeout de sesión durante cambio de contraseña | Usuario autenticado con contraseña temporal deja la pantalla abierta sin interactuar por 15 minutos | Pasa tiempo sin actividad | Sistema mantiene la sesión activa durante el proceso de cambio obligatorio (NO cierra sesión automáticamente). Muestra advertencia después de 10 minutos de inactividad: "Por seguridad, cambie su contraseña pronto. Si no hay actividad, deberá iniciar sesión nuevamente." Si pasa 15 minutos, cierra sesión y redirige a login mostrando mensaje: "Su sesión expiró por inactividad. Inicie sesión nuevamente para cambiar su contraseña" | ☐ | ☐ | ☐ |
| 19 | Múltiples intentos fallidos de cambio de contraseña | Usuario intenta cambiar contraseña 10 veces pero siempre con errores de validación | Usuario hace clic en "Cambiar Contraseña" con datos inválidos repetidamente | Sistema NO bloquea al Usuario por intentos fallidos en pantalla de cambio obligatorio (a diferencia de login). Permite intentos ilimitados hasta que cumpla requisitos. Cada intento fallido muestra mensajes de error específicos para guiar al Usuario. NO registra cada intento en auditoría (solo el cambio exitoso final) | ☐ | ☐ | ☐ |
| 20 | Contraseña temporal usada después de haber sido cambiada | Usuario ya cambió su contraseña temporal por una nueva, intenta autenticarse nuevamente con la contraseña temporal antigua | Usuario ingresa número ID y contraseña temporal en login | Sistema valida credenciales, detecta que la contraseña temporal fue invalidada (ya se cambió). Muestra mensaje genérico: "Credenciales incorrectas". NO revela que era una contraseña temporal antigua. Registra intento en auditoría como autenticación fallida normal | ☐ | ☐ | ☐ |
| 21 | Validación de contraseña no en lista negra | Usuario intenta usar contraseña común o débil (ej: "Password1!", "Qwerty123!") | Usuario ingresa contraseña en "Nueva Contraseña" | Sistema valida contra lista de contraseñas comunes (top 1000 contraseñas más usadas). Si está en lista negra, muestra error: "Esta contraseña es muy común. Por favor, elija una contraseña más segura y única." Botón permanece deshabilitado | ☐ | ☐ | ☐ |
| 22 | Correo electrónico no entregado (bounce) | Sistema envía correo con contraseña temporal pero correo del Usuario no existe o está lleno | Servicio de correo retorna bounce notification | Sistema recibe notificación de bounce. Registra evento en auditoría con severidad ERROR: "Correo no entregado a [correo]". Administrador puede consultar estado de envío en detalle de Usuario. Muestra advertencia en perfil: "Correo de contraseña temporal no entregado - verificar correo registrado" | ☐ | ☐ | ☐ |
| 23 | Reenvío de correo con contraseña temporal | Usuario no recibió correo (filtro spam, error de digitación del correo) | Administrador hace clic en "Reenviar Correo de Contraseña Temporal" en detalle de Usuario | Sistema verifica que existe contraseña temporal activa y NO expirada. Reenvía el mismo correo con la misma contraseña temporal (NO genera nueva). Muestra confirmación: "Correo reenviado a [correo]". Si contraseña expiró, muestra error: "La contraseña temporal expiró. Debe generar una nueva con 'Resetear Contraseña'". Registra reenvío en auditoría | ☐ | ☐ | ☐ |
| 24 | Contador de tiempo restante para expiración | Usuario visualiza correo con contraseña temporal | Correo muestra información de expiración | Correo muestra texto: "Esta contraseña temporal es válida por 72 horas desde su generación. Fecha de expiración: [DD/MM/YYYY HH:MM]". Usuario sabe exactamente cuándo debe cambiar su contraseña antes de que expire | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 25 | Estructura obligatoria de registro de auditoría | Cualquier evento relacionado con contraseñas temporales (generación, envío, uso, cambio, expiración, regeneración) | Sistema registra evento | Registro contiene: ID Único del Evento (UUID), Tipo de Evento (formato MODULO_ENTIDAD_ACCION), Fecha y Hora (UTC con milisegundos, ISO 8601), Usuario (número de identificación del usuario afectado), Cliente (NULL), Cliente (Nombre - NULL), IP Local, IP Pública, Resultado (EXITOSO/FALLIDO), Descripción (texto descriptivo), Nivel de Severidad (INFO/WARNING/ERROR), Datos Adicionales (JSON con contexto: correo destino, fecha expiración, etc.) | ☐ | ☐ | ☐ |
| 26 | Auditoría de generación de contraseña temporal | Administrador crea Usuario exitosamente con correo registrado | Sistema genera contraseña temporal | Registro de auditoría: Tipo de Evento: "SEGURIDAD_CONTRASENA_TEMPORAL_GENERADA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Contraseña temporal generada para usuario {usuario_id} por Administrador {admin}", Datos Adicionales: {"usuario_id": "UUID", "usuario_numero_id": "123456789", "usuario_nombre": "Juan Pérez", "correo_destino": "j***@empresa.com" (parcialmente oculto), "fecha_expiracion": "timestamp +72h", "administrador_creador": "admin123"} | ☐ | ☐ | ☐ |
| 27 | Auditoría de envío exitoso de correo | Sistema envía correo con contraseña temporal y servicio confirma entrega | Servicio de correo retorna éxito | Registro de auditoría: Tipo de Evento: "SEGURIDAD_CONTRASENA_TEMPORAL_ENVIADA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Correo con contraseña temporal enviado exitosamente a usuario {usuario_id}", Datos Adicionales: {"usuario_id": "UUID", "correo_destino": "j***@empresa.com", "fecha_envio": "timestamp", "servicio_correo_respuesta": "250 OK"} | ☐ | ☐ | ☐ |
| 28 | Auditoría de error al enviar correo | Sistema intenta enviar correo pero servicio falla | Servicio de correo retorna error | Registro de auditoría: Tipo de Evento: "SEGURIDAD_CONTRASENA_TEMPORAL_ERROR_ENVIO", Resultado: "FALLIDO", Severidad: "ERROR", Descripción: "Error al enviar correo con contraseña temporal a usuario {usuario_id}", Datos Adicionales: {"usuario_id": "UUID", "correo_destino": "email@domain.com", "error_tipo": "timeout", "error_mensaje": "Connection timeout after 5000ms", "fecha_intento": "timestamp"} | ☐ | ☐ | ☐ |
| 29 | Auditoría de primer inicio de sesión con contraseña temporal | Usuario se autentica por primera vez con contraseña temporal válida | Usuario ingresa credenciales temporales en login | Registro de auditoría: Tipo de Evento: "SEGURIDAD_LOGIN_CONTRASENA_TEMPORAL", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Usuario {usuario_id} autenticado con contraseña temporal - redirigido a cambio obligatorio", Datos Adicionales: {"usuario_id": "UUID", "usuario_numero_id": "123456789", "ip_publica": "IP", "ip_local": "IP", "fecha_generacion_temporal": "timestamp", "dias_desde_generacion": 1, "cambio_obligatorio": true} | ☐ | ☐ | ☐ |
| 30 | Auditoría de cambio exitoso de contraseña temporal | Usuario completa cambio de contraseña obligatorio exitosamente | Usuario hace clic en "Cambiar Contraseña" con datos válidos | Registro de auditoría: Tipo de Evento: "SEGURIDAD_CONTRASENA_CAMBIADA_PRIMER_LOGIN", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Usuario {usuario_id} cambió contraseña temporal por contraseña definitiva exitosamente", Datos Adicionales: {"usuario_id": "UUID", "usuario_numero_id": "123456789", "ip_publica": "IP", "fecha_cambio": "timestamp", "fecha_generacion_temporal": "timestamp", "tiempo_uso_temporal_horas": 8} | ☐ | ☐ | ☐ |
| 31 | Auditoría de intento con contraseña temporal expirada | Usuario intenta autenticarse con contraseña temporal expirada (>72 horas) | Usuario ingresa credenciales en login | Registro de auditoría: Tipo de Evento: "SEGURIDAD_LOGIN_CONTRASENA_TEMPORAL_EXPIRADA", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario_id} intentó autenticarse con contraseña temporal expirada", Datos Adicionales: {"usuario_id": "UUID", "usuario_numero_id": "123456789", "fecha_generacion": "timestamp", "fecha_expiracion": "timestamp", "fecha_intento": "timestamp", "horas_desde_expiracion": 24, "ip_publica": "IP"} | ☐ | ☐ | ☐ |
| 32 | Auditoría de regeneración de contraseña temporal | Administrador regenera contraseña temporal para Usuario | Administrador hace clic en "Resetear Contraseña" | Registro de auditoría: Tipo de Evento: "SEGURIDAD_CONTRASENA_TEMPORAL_REGENERADA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {admin} regeneró contraseña temporal para usuario {usuario_id}", Datos Adicionales: {"usuario_id": "UUID", "administrador_regenerador": "admin123", "correo_destino": "j***@empresa.com", "fecha_expiracion_nueva": "timestamp +72h", "razon": "Usuario no recibió correo inicial"} | ☐ | ☐ | ☐ |
| 33 | Auditoría de reenvío de correo con contraseña temporal | Administrador reenvía correo con contraseña temporal existente | Administrador hace clic en "Reenviar Correo" | Registro de auditoría: Tipo de Evento: "SEGURIDAD_CONTRASENA_TEMPORAL_REENVIADA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {admin} reenviò correo con contraseña temporal a usuario {usuario_id}", Datos Adicionales: {"usuario_id": "UUID", "administrador": "admin123", "correo_destino": "email@domain.com", "fecha_reenvio": "timestamp", "contraseña_original_fecha": "timestamp"} | ☐ | ☐ | ☐ |
| 34 | Auditoría de bounce de correo electrónico | Servicio de correo notifica que correo no fue entregado (bounce) | Sistema recibe notificación de bounce | Registro de auditoría: Tipo de Evento: "SEGURIDAD_CONTRASENA_TEMPORAL_BOUNCE", Resultado: "FALLIDO", Severidad: "ERROR", Descripción: "Correo con contraseña temporal no entregado a usuario {usuario_id} - bounce recibido", Datos Adicionales: {"usuario_id": "UUID", "correo_destino": "email@invalido.com", "bounce_razon": "User mailbox full", "fecha_bounce": "timestamp"} | ☐ | ☐ | ☐ |
| 35 | Auditoría de intento con contraseña temporal después de cambio | Usuario intenta usar contraseña temporal después de haberla cambiado | Usuario ingresa contraseña temporal antigua en login | Registro de auditoría: Tipo de Evento: "AUTENTICACION_FALLIDA_CREDENCIALES", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Intento de autenticación con credenciales incorrectas (posible uso de contraseña temporal invalidada)", Datos Adicionales: {"usuario_numero_id": "123456789", "ip_publica": "IP", "fecha_cambio_contraseña": "timestamp cuando cambió"} | ☐ | ☐ | ☐ |
| 36 | Inmutabilidad de registros de auditoría | Cualquier escenario de auditoría de contraseñas temporales | Sistema genera registro | Registros de auditoría NO pueden ser modificados ni eliminados. Solo se permiten consultas de lectura. Implementar controles que prevengan alteración de registros históricos | ☐ | ☐ | ☐ |
| 37 | Consulta de registros de auditoría de contraseñas temporales | Administrador o Auditor requiere revisar historial de contraseñas temporales | Administrador accede a módulo de auditoría | Sistema permite filtrar por: rango de fechas, usuario afectado, tipo de evento (SEGURIDAD_CONTRASENA_TEMPORAL_*), resultado (EXITOSO/FALLIDO), severidad (INFO/WARNING/ERROR), administrador que generó/regeneró. Permite exportar resultados en CSV/Excel. Muestra: fecha/hora, usuario, evento, administrador, resultado, correo destino (parcialmente oculto) | ☐ | ☐ | ☐ |
| 38 | Retención de registros de auditoría | Registros de auditoría de contraseñas temporales | Paso del tiempo | Registros se conservan por tiempo mínimo de 7 años (política estándar de auditoría). Después pueden ser archivados o eliminados según política de retención | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

Se requieren los siguientes mockups interactivos en HTML siguiendo la Guía de Estilos (Material UI, paleta de colores Primary #4A5A9E, Secondary #D4145A):

#### 1. **HU007-CambioContraseñaObligatorio.html** - Pantalla de cambio de contraseña forzado
**Elementos:**
- Logotipo de CDN Facturación centrado en parte superior
- Título: "Cambio de Contraseña Requerido" (Tipografía H3, color Primary Dark)
- Mensaje informativo (Alert info): "Por seguridad, debe establecer una nueva contraseña. Esta será su contraseña definitiva para acceder al Portal Unificado."
- Campo "Nueva Contraseña" (TextField outlined, tipo password, icono ojo para mostrar/ocultar)
- Indicador visual de requisitos en tiempo real (lista con checkmarks/cruces):
  - "Mínimo 8 caracteres ✗/✓"
  - "Al menos una mayúscula (A-Z) ✗/✓"
  - "Al menos una minúscula (a-z) ✗/✓"
  - "Al menos un número (0-9) ✗/✓"
  - "Al menos un símbolo (!@#$%^&*) ✗/✓"
  - "No puede ser igual a contraseña temporal ✗/✓"
- Barra de fortaleza de contraseña (LinearProgress con colores: débil=error, media=warning, fuerte=success)
- Campo "Confirmar Nueva Contraseña" (TextField outlined, tipo password, icono ojo)
- Mensaje de error si no coinciden (texto rojo): "Las contraseñas no coinciden"
- Botón "Cambiar Contraseña" (contained, gradiente Primary → Secondary, deshabilitado hasta cumplir requisitos)
- NO tiene botón "Cancelar" (acción es obligatoria)
- Mensaje de éxito (Alert success, oculto por defecto): "Contraseña cambiada exitosamente. Redirigiendo al portal..."

#### 2. **HU007-EmailContraseñaTemporal.html** - Plantilla de correo electrónico
**Elementos:**
- Encabezado con logo de CDN Facturación y fondo con gradiente Primary → Secondary
- Saludo personalizado: "Hola {nombre_usuario},"
- Cuerpo del mensaje:
  - "Bienvenido al Portal Unificado de CDN Facturación. Su cuenta ha sido creada exitosamente."
  - "A continuación encontrará sus credenciales de acceso:"
- Sección destacada con fondo gris claro:
  - "**Usuario**: {numero_identificacion}"
  - "**Contraseña Temporal**: {contraseña_temporal}" (texto monospace, color Primary, negrita)
  - "**Válida hasta**: {fecha_expiracion} (72 horas)"
- Instrucciones numeradas:
  1. "Acceda al portal en: {url_portal_login}"
  2. "Ingrese su usuario y contraseña temporal"
  3. "El sistema le solicitará cambiar su contraseña por una nueva"
  4. "Cree una contraseña segura que solo usted conozca"
- Advertencias de seguridad (Alert warning):
  - "⚠ Esta contraseña es de un solo uso y expirará en 72 horas"
  - "⚠ No comparta esta contraseña con nadie"
  - "⚠ Si no solicitó esta cuenta, contacte inmediatamente a soporte"
- Sección de ayuda:
  - "¿Necesita ayuda? Contáctenos:"
  - Teléfono, Email, Horarios de atención
- Pie de correo:
  - "Equipo de CDN Facturación"
  - Aviso legal: "Este es un correo automático, por favor no responda a este mensaje."

#### 3. **HU007-ContraseñaExpirada.html** - Mensaje de contraseña temporal expirada
**Elementos:**
- Pantalla de login normal (HU001)
- Alert error visible: "Su contraseña temporal ha expirado. Por favor, contacte al administrador para solicitar una nueva."
- Información de contacto de soporte
- Botón "Volver a intentar" (limpia formulario)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Correo enviado a dirección incorrecta** | Si el correo electrónico del Usuario fue mal digitado durante creación (ej: juan@empesa.com en lugar de juan@empresa.com), la contraseña temporal llega a persona equivocada o no llega a nadie | Alto | Media | Validar formato de correo en creación de usuario (HU006) con checksum visual. Mostrar correo en pantalla de resumen antes de crear. Enviar correo de confirmación después de creación con link "No fui yo, reportar" para detectar errores. Administrador puede ver estado de entrega (enviado/bounce/leído) en detalle de usuario | Mitigado |
| **Contraseña temporal interceptada** | Si el correo es interceptado por atacante (compromiso de cuenta de email del usuario), el atacante puede activar la cuenta antes que el usuario legítimo | Alto | Baja | Contraseña temporal expira en 72 horas. Sistema registra IP de primer login y cambio de contraseña en auditoría. Enviar notificación secundaria después de cambio exitoso: "Su contraseña fue cambiada desde IP {IP} el {fecha}". Si usuario no lo hizo, puede reportar compromiso. Considerar 2FA en futuras versiones | Aceptado con controles |
| **Usuario nunca recibe el correo (filtro spam)** | Correo con contraseña temporal cae en carpeta de spam/correo no deseado del usuario y nunca lo ve. Usuario no puede activar su cuenta | Medio | Alta | Configurar SPF, DKIM, DMARC en servidor de correo para mejorar deliverability. Usar dominio corporativo verificado. Plantilla HTML sin elementos que disparen spam (imágenes externas limitadas, texto/HTML ratio balanceado). Administrador puede reenviar correo o regenerar contraseña. Incluir en correo: "Si no ve este correo, revise su carpeta de spam" | Mitigado |
| **Generación de contraseñas débiles o predecibles** | Si el algoritmo de generación aleatoria no es criptográficamente seguro, atacante podría predecir contraseñas temporales | Alto | Muy Baja | Usar generador criptográficamente seguro (ej: crypto.randomBytes en Node.js, secrets module en Python). 12 caracteres con 4 mayúsculas + 4 minúsculas + 2 números + 2 símbolos = ~72 bits de entropía. Espacio de búsqueda: (26+26+10+8)^12 ≈ 10^21 combinaciones. Tiempo de vida 72 horas hace inviable fuerza bruta | Mitigado |
| **Usuario olvida cambiar contraseña antes de 72 horas** | Usuario recibe correo pero no lo lee a tiempo, contraseña expira, no puede acceder, debe contactar administrador | Bajo | Media | Período de 72 horas es razonable (3 días). Correo muestra claramente fecha de expiración. Administrador puede regenerar fácilmente desde opción "Resetear Contraseña". Considerar email de recordatorio 24 horas antes de expiración (feature futura) | Aceptado |
| **Contraseña temporal muy compleja para recordar/escribir** | Usuario ve contraseña en correo de 12 caracteres alfanuméricos con símbolos, tiene dificultad para escribirla correctamente (errores de tipeo) | Bajo | Alta | Correo muestra contraseña en fuente monospace para claridad. Permitir copiar/pegar desde correo (no deshabilitar). Permitir ver contraseña en campo de login (icono ojo). Diseño de contraseña evita caracteres ambiguos (0 vs O, 1 vs l vs I). Múltiples intentos no bloquean cuenta en pantalla de cambio obligatorio | Mitigado |
| **Sesión secuestrada durante cambio de contraseña** | Si atacante secuestra sesión después de login con contraseña temporal pero antes de cambio, puede establecer contraseña que el usuario legítimo no conoce | Alto | Muy Baja | Usar HTTPS obligatorio. Cookies de sesión con flags Secure y HttpOnly. Regenerar session ID después de cambio de contraseña exitoso. Registrar IP de cambio en auditoría. Notificar al usuario por correo después de cambio exitoso con IP y fecha | Mitigado |
| **Usuario sin correo electrónico no puede activar cuenta** | Si usuario no tiene correo registrado, Administrador debe establecer contraseña manualmente, lo que expone la contraseña a terceros | Medio | Baja | Validar en HU006 que correo es recomendado (aunque opcional). Mostrar advertencia clara si se crea sin correo. Proceso manual alternativo: Administrador puede usar función "Establecer Contraseña Temporal Verbal" que genera contraseña corta (8 caracteres) válida 24 horas para comunicar en persona/teléfono | Aceptado con proceso alternativo |
| **Múltiples regeneraciones de contraseña temporal (abuso)** | Administrador malicioso o usuario solicita regenerar contraseña temporal múltiples veces inundando correo del usuario | Bajo | Baja | Registrar cada regeneración en auditoría con administrador que la solicitó. Implementar límite razonable: máximo 5 regeneraciones en 24 horas por usuario. Mostrar advertencia después de 3ra regeneración: "Este usuario ya tuvo 3 contraseñas temporales generadas hoy. ¿Está seguro?" | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance (Out of Scope)
- Autenticación de dos factores (2FA) para primer inicio de sesión (se evaluará en Release 2)
- Recuperación de contraseña olvidada para usuarios ya activos (se cubre en HU002 - Recuperación de Contraseña)
- Cambio de contraseña voluntario (no forzado) por el usuario (se cubrirá en HU008 - Gestión de Contraseñas de Usuario)
- Políticas de rotación de contraseñas (ej: forzar cambio cada 90 días) - se evaluará en futuras versiones
- Historial de contraseñas anteriores para prevenir reutilización (se cubre parcialmente: solo previene reusar contraseña temporal)
- Notificaciones push o SMS con contraseña temporal (solo correo electrónico en esta HU)
- Autenticación sin contraseña (passwordless) con magic links o biometría (futuro)

### Consideraciones Técnicas para Implementación (Informativas)
- **Generación de contraseña aleatoria:** Usar generador criptográficamente seguro. Algoritmo: generar 4 letras mayúsculas random + 4 minúsculas + 2 dígitos + 2 símbolos de set (!@#$%^&*), luego mezclar orden aleatoriamente. Ejemplo: `Kj8mL@pQ3z#W`
- **Almacenamiento:** NUNCA guardar contraseña temporal en texto plano. Almacenar hash (bcrypt/Argon2) igual que contraseñas normales. Campos en tabla usuarios: `password_hash`, `is_password_temporary` (boolean), `password_temp_expires_at` (timestamp nullable)
- **Validación de expiración:** En cada login, si `is_password_temporary = true`, validar `CURRENT_TIMESTAMP < password_temp_expires_at`. Si expiró, rechazar con mensaje específico
- **Forzar cambio:** Después de login exitoso con temporal, crear sesión pero con flag `requires_password_change = true`. Middleware debe redirigir TODAS las requests (excepto /change-password y /logout) a pantalla de cambio obligatorio
- **Envío de correo:** Usar cola de mensajes (ej: RabbitMQ, Redis Queue) para envío asíncrono. No bloquear creación de usuario si correo tarda. Implementar retry 3 veces con backoff exponencial si falla. Registrar estado de envío en tabla `email_logs`
- **Plantilla de correo:** HTML responsivo con fallback a texto plano. Inline CSS para compatibilidad con clientes de correo. Usar servicio transaccional (SendGrid, AWS SES, Mailgun) no correo corporativo normal
- **Notificación de cambio exitoso:** Después de cambiar contraseña, enviar correo secundario: "Su contraseña fue cambiada exitosamente el {fecha} desde IP {IP}. Si no fue usted, contacte soporte inmediatamente." Ayuda a detectar compromisos
- **Lista negra de contraseñas:** Validar contra top 1000 contraseñas comunes. Archivos disponibles: SecLists, HIBP. Check simple: `if (new_password in blacklist) reject()`
- **Bounce handling:** Implementar webhook para recibir notificaciones de bounce del servicio de correo. Actualizar tabla de usuario con flag `email_bounced = true`. Administrador ve advertencia en detalle

### Dependencias
- **HU001 - Autenticación y Selección de Cliente**: El flujo de login debe detectar si contraseña es temporal y redirigir a cambio obligatorio
- **HU006 - Creación de Usuarios y Asignación de Permisos**: Proceso de creación exitosa dispara generación y envío de contraseña temporal
- **HU002 - Recuperación de Contraseña**: Requisitos de contraseña son similares (reutilizar validación)
- **Sistema de Notificaciones por Correo**: Debe existir capacidad de envío de correos con plantillas HTML personalizables
- **Módulo de Auditoría**: Para registrar eventos de generación, envío, uso, cambio de contraseñas temporales

### Referencias
- **Estándar de Auditoría Funcional**: Tipos de eventos `SEGURIDAD_CONTRASENA_TEMPORAL_*`, severidades INFO/WARNING/ERROR
- **Estándar de Seguridad APL03**:
  - AP-0039: Almacenamiento seguro de credenciales (hash, no texto plano)
  - AP-0011: Autenticación fuerte (contraseña segura)
  - AP-0022: Registro de eventos de seguridad
  - AP-0024: Registro detallado con contexto (IP, timestamps)
- **Glosario**: Usuario, Contraseña Temporal, Primer Inicio de Sesión, Cambio de Contraseña Obligatorio, Expiración de Contraseña
- **Guía de Estilos**: Material UI, paleta Primary/Secondary, indicadores visuales de requisitos, barras de fortaleza, campos password con toggle visibilidad
- **NIST SP 800-63B**: Guía de autenticación digital (requisitos de contraseñas, entropía, expiración)
