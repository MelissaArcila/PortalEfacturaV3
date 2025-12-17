# HU001 - Autenticación y Selección de Cliente

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
| **Arquitectura relacionada** | Portal Unificado, Gestión de Usuarios y Clientes |

---

## Contexto

Actualmente, la plataforma no cuenta con un mecanismo de autenticación que permita a los usuarios acceder al Portal Unificado de manera segura. Los usuarios pueden estar asociados a uno o más clientes (empresas que tienen contratados productos del ecosistema de facturación electrónica DIAN), y es necesario que el sistema valide sus credenciales y les permita seleccionar el contexto de cliente bajo el cual desean operar durante su sesión.

Esta funcionalidad es crítica para garantizar la seguridad, trazabilidad y correcta segregación de accesos en la plataforma, permitiendo que cada usuario trabaje únicamente con la información y funcionalidades autorizadas para el cliente seleccionado.

---

**Notas:**
- Esta historia de usuario NO incluye funcionalidades de recuperación de contraseña, doble factor de autenticación, primer acceso o cambio de cliente durante la sesión, las cuales serán abordadas en historias de usuario separadas.
- El sistema debe cumplir con los lineamientos de seguridad definidos en el documento APL03 - Estándar de Seguridad para Desarrollo de Aplicaciones.
- La auditoría de eventos de autenticación es obligatoria para cumplir con requisitos de trazabilidad y seguridad.

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Usuario** | Quiero autenticarme con mi nombre de usuario y contraseña | Para acceder de forma segura al Portal Unificado |
| Como **Usuario con acceso a múltiples clientes** | Quiero seleccionar el cliente con el cual deseo trabajar | Para operar bajo el contexto correcto y acceder a la información correspondiente al cliente seleccionado |
| Como **Usuario del Sistema** | Quiero que mi cuenta se bloquee temporalmente después de varios intentos fallidos | Para proteger mi cuenta de accesos no autorizados |
| Como **Administrador del Sistema** | Quiero que el sistema registre todos los eventos de autenticación exitosos y fallidos | Para poder auditar los accesos y detectar posibles incidentes de seguridad |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Autenticación exitosa con un solo cliente asignado | El usuario tiene credenciales válidas, su cuenta está activa, tiene asociado únicamente un cliente activo, y no ha alcanzado el límite de intentos fallidos | El usuario ingresa su nombre de usuario y contraseña correctos y presiona el botón para ingresar | El sistema valida las credenciales, identifica que el usuario tiene un solo cliente activo, crea una sesión de usuario bajo el contexto de ese cliente e ingresa automáticamente al Portal Unificado sin solicitar selección de cliente | ☐ | ☐ | ☐ |
| 2 | Autenticación exitosa con múltiples clientes asignados | El usuario tiene credenciales válidas, su cuenta está activa, tiene asociados múltiples clientes (al menos uno activo), y no ha alcanzado el límite de intentos fallidos | El usuario ingresa su nombre de usuario y contraseña correctos y presiona el botón para ingresar | El sistema valida las credenciales, identifica que el usuario tiene múltiples clientes, muestra una pantalla de selección de cliente con una lista desplegable que presenta cada cliente en formato "NIT - Nombre" permitiendo búsqueda dentro de la lista, y habilita un botón para confirmar la selección | ☐ | ☐ | ☐ |
| 3 | Selección de cliente e ingreso al sistema | El usuario ha sido autenticado exitosamente y se encuentra en la pantalla de selección de cliente | El usuario selecciona un cliente de la lista desplegable y presiona el botón para ingresar | El sistema valida que el cliente seleccionado está activo, crea una sesión de usuario bajo el contexto de cliente seleccionado e ingresa al Portal Unificado | ☐ | ☐ | ☐ |
| 4 | Credenciales incorrectas | El usuario no ha alcanzado el límite de 5 intentos fallidos | El usuario ingresa un nombre de usuario que no existe en el sistema o una contraseña incorrecta y presiona el botón para ingresar | El sistema muestra el mensaje "Credenciales incorrectas", no revela si el error fue en el usuario o en la contraseña, incrementa el contador de intentos fallidos para ese usuario y registra el evento fallido en auditoría con fecha/hora, IP local, IP pública y nombre de usuario ingresado | ☐ | ☐ | ☐ |
| 5 | Bloqueo de cuenta por intentos fallidos | El usuario ha ingresado credenciales incorrectas en 5 intentos consecutivos | El usuario intenta autenticarse por quinta vez con credenciales incorrectas | El sistema bloquea la cuenta del usuario, muestra el mensaje "Credenciales incorrectas" sin revelar que la cuenta ha sido bloqueada, registra el evento de bloqueo en auditoría con fecha/hora, IP local, IP pública y nombre de usuario, e inicia un temporizador de 30 minutos para desbloqueo automático | ☐ | ☐ | ☐ |
| 6 | Intento de autenticación con cuenta bloqueada | La cuenta del usuario está bloqueada por intentos fallidos y no han transcurrido 30 minutos desde el bloqueo | El usuario intenta autenticarse con credenciales correctas o incorrectas | El sistema muestra el mensaje "Credenciales incorrectas", mantiene la cuenta bloqueada y registra el intento en auditoría | ☐ | ☐ | ☐ |
| 7 | Desbloqueo automático de cuenta | La cuenta del usuario fue bloqueada hace 30 minutos o más | El usuario intenta autenticarse con sus credenciales correctas | El sistema desbloquea automáticamente la cuenta, reinicia el contador de intentos fallidos a cero, valida las credenciales correctamente y procede con el flujo de autenticación exitosa (escenario 1 o 2 según corresponda), registrando el desbloqueo y autenticación exitosa en auditoría | ☐ | ☐ | ☐ |
| 8 | Usuario inactivo | La cuenta del usuario existe pero está en estado inactivo | El usuario ingresa sus credenciales correctas y presiona el botón para ingresar | El sistema muestra el mensaje "Credenciales incorrectas", no permite el acceso y registra el intento en auditoría indicando que se intentó acceder con un usuario inactivo | ☐ | ☐ | ☐ |
| 9 | Cliente inactivo - Usuario con un solo cliente | El usuario tiene credenciales válidas y está activo, pero su único cliente asociado está en estado inactivo | El usuario ingresa sus credenciales correctas y presiona el botón para ingresar | El sistema muestra el mensaje "Acceso no disponible. Contacte al administrador." y registra el evento en auditoría | ☐ | ☐ | ☐ |
| 10 | Todos los clientes inactivos - Usuario con múltiples clientes | El usuario tiene credenciales válidas y está activo, pero todos sus clientes asociados están en estado inactivo | El usuario ingresa sus credenciales correctas y presiona el botón para ingresar | El sistema muestra el mensaje "Acceso no disponible. Contacte al administrador." y registra el evento en auditoría | ☐ | ☐ | ☐ |
| 11 | Selección de cliente inactivo | El usuario autenticado tiene múltiples clientes y se encuentra en la pantalla de selección | El usuario selecciona un cliente que está en estado inactivo de la lista desplegable y presiona el botón para ingresar | El sistema muestra el mensaje "Acceso no disponible. Contacte al administrador." y registra el evento en auditoría | ☐ | ☐ | ☐ |
| 12 | Registro de autenticación exitosa en auditoría | El usuario completa exitosamente el proceso de autenticación y selección de cliente (o ingreso directo si tiene un solo cliente) | El sistema crea la sesión de usuario e ingresa al Portal Unificado | El sistema registra en auditoría el evento de autenticación exitosa incluyendo: nombre de usuario, fecha y hora (con milisegundos y zona horaria), IP local, IP pública, cliente seleccionado o asignado automáticamente, y acción realizada (login exitoso) | ☐ | ☐ | ☐ |
| 13 | Lista desplegable con búsqueda | El usuario autenticado tiene múltiples clientes y se encuentra en la pantalla de selección de cliente | El usuario escribe texto en el campo de búsqueda de la lista desplegable | El sistema filtra la lista de clientes mostrando únicamente aquellos cuyo NIT o Nombre contengan el texto ingresado | ☐ | ☐ | ☐ |
| 14 | Visualización de solo clientes activos en lista de selección | El usuario autenticado tiene múltiples clientes, algunos activos y algunos inactivos, y se encuentra en la pantalla de selección | El sistema presenta la lista desplegable de clientes | El sistema muestra únicamente los clientes que están en estado activo en formato "NIT - Nombre", sin mostrar los clientes inactivos en la lista | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes
Pendiente

**Descripción funcional de las pantallas:**

**Pantalla 1 - Login:**
- Campo de texto para nombre de usuario
- Campo de texto enmascarado para contraseña
- Botón "Ingresar"
- Área para mostrar mensajes de error

**Pantalla 2 - Selección de Cliente (solo si el usuario tiene múltiples clientes activos):**
- Lista desplegable con búsqueda que muestra clientes en formato "NIT - Nombre"
- Botón "Ingresar"
- Área para mostrar mensajes de error

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Ataques de fuerza bruta** | Un atacante podría intentar adivinar credenciales mediante múltiples intentos automatizados | Alto | Media | Implementar bloqueo de cuenta después de 5 intentos fallidos y registro en auditoría de todos los intentos fallidos para detección de patrones de ataque | Mitigado |
| **Enumeración de usuarios** | Un atacante podría identificar usuarios válidos del sistema según las respuestas diferenciadas | Alto | Media | Utilizar mensaje genérico "Credenciales incorrectas" para todos los casos de fallo de autenticación sin revelar si el error fue en usuario o contraseña | Mitigado |
| **Fuga de información en auditoría** | Los logs de auditoría podrían contener información sensible accesible a personal no autorizado | Alto | Baja | Garantizar que los logs de auditoría sean almacenados de forma segura y solo sean accesibles a personal autorizado según lineamientos de seguridad | Aceptado |
| **Denegación de servicio por bloqueos masivos** | Un atacante podría bloquear múltiples cuentas de usuarios legítimos | Medio | Media | Implementar desbloqueo automático después de 30 minutos y monitoreo de patrones de bloqueo masivo en auditoría | Mitigado |
| **Usuario con múltiples clientes no puede acceder si todos están inactivos** | Un usuario legítimo podría quedar sin acceso si todos sus clientes son desactivados | Medio | Baja | Mostrar mensaje claro solicitando contactar al administrador y garantizar que existe un proceso de soporte para activación de clientes | Aceptado |

---

## Notas Adicionales

**Fuera de alcance de esta HU:**
- Recuperación de contraseña (olvidé mi contraseña)
- Doble factor de autenticación
- Comportamiento de primer acceso (cambio obligatorio de contraseña, aceptación de términos)
- Cambio de cliente sin cerrar sesión
- Retraso incremental en intentos fallidos (AP-0007)
- Gestión manual de desbloqueo de cuentas por parte de administradores

**Consideraciones de seguridad aplicables:**
- AP-0002: Credenciales únicas para cada usuario
- AP-0008: Respuestas genéricas en fallos de autenticación
- AP-0009: Bloqueo de cuenta por intentos fallidos
- AP-0011: Solicitar nombre de usuario y contraseña
- AP-0021: No permitir autenticación con credenciales expiradas, revocadas o bloqueadas
- AP-0022: Registro de eventos de seguridad (exitosos y no exitosos)
- AP-0024: Registro detallado con fecha/hora, usuario, acción, IPs
