# HU-AD004B - Flujo de Login SSO

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 2 |
| **Epic** | Integración con Active Directory mediante SCIM |
| **Estado** | BORRADOR |
| **Propietario** | Por definir |
| **Arquitecto** | Por definir |
| **Desarrolladores** | Por definir |
| **Tester** | Por definir |
| **Incidentes relacionados** | N/A |
| **Arquitectura relacionada** | Portal Unificado - Autenticación, SAML 2.0, UX |

---

## Contexto

Esta HU detalla la **experiencia de usuario completa** del flujo de autenticación Single Sign-On (SSO) mediante SAML 2.0, desde que el usuario accede al Portal hasta que queda autenticado.

El flujo cubre:
- Detección automática de usuarios con integración AD
- Redirección al IdP corporativo
- Retorno al Portal tras autenticación
- Manejo de estados durante el flujo
- Experiencia en diferentes escenarios (primera vez, sesión expirada, URL directa)

---

**Notas:**
- Flujo es transparente para el usuario (mínimos clics)
- Preserva URL destino original (RelayState) para redirigir tras login
- Muestra indicadores de carga durante redirecciones
- Maneja errores de forma amigable
- Compatible con deep links (URLs directas a recursos protegidos)

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Usuario de Cliente con AD** | Quiero un flujo de login simple y rápido usando mis credenciales corporativas | Para acceder al Portal sin fricción ni necesidad de recordar contraseñas adicionales |
| Como **Usuario** | Quiero que el Portal me lleve automáticamente a mi página de login corporativa | Para autenticarme con el sistema que ya conozco y uso diariamente |
| Como **Usuario** | Quiero regresar a la página que solicité originalmente después de autenticarme | Para continuar mi trabajo sin perder el contexto de lo que estaba haciendo |
| Como **Portal Unificado** | Quiero mostrar feedback visual durante las redirecciones SAML | Para que el usuario sepa que el proceso está en curso y no se confunda |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso a Portal sin sesión - Usuario AD | Usuario sin sesión activa accede a `https://portal.efac.com/login` | Usuario ingresa email corporativo (ej: juan@empresa.com) y hace blur/tab | Sistema detecta que email pertenece a dominio de tenant con integración AD activa (consulta BD). Oculta campo contraseña. Muestra mensaje "Su organización {Nombre Empresa} usa Single Sign-On" y botón "Continuar con {Nombre Empresa}" (contained, gradiente). Usuario hace clic en botón | ☐ | ☐ | ☐ |
| 2 | Generación de SAML AuthnRequest | Usuario hace clic en "Continuar con {Nombre Empresa}" | Sistema inicia SSO | Sistema: (1) Genera UUID para RequestID, (2) Almacena en sesión temporal: RequestID, timestamp, tenant_id, RelayState (URL destino o dashboard), (3) Construye XML SAML AuthnRequest con: ID=RequestID, IssueInstant=now, Issuer=Entity ID del Portal, AssertionConsumerServiceURL, (4) Codifica en Base64, (5) Muestra pantalla de carga "Redirigiendo a {Nombre Empresa}...", (6) Redirige navegador a IdP URL con parámetros GET: `?SAMLRequest={base64}&RelayState={url_destino}` | ☐ | ☐ | ☐ |
| 3 | Usuario en página de login del IdP | Navegador redirigido a AD FS / Azure AD | Usuario ve interfaz del IdP | Usuario ve página de login de su organización (AD FS / Azure AD). Usuario ingresa credenciales corporativas (usuario y contraseña de AD, MFA si configurado). IdP valida credenciales. Si exitoso: IdP genera SAML Response con assertion firmada | ☐ | ☐ | ☐ |
| 4 | Retorno al Portal - POST a ACS | IdP valida usuario exitosamente | IdP redirige a Portal | IdP genera formulario HTML oculto con método POST a `https://portal.efac.com/saml/{tenant_id}/acs`, campos: SAMLResponse (Base64), RelayState (URL destino). JavaScript auto-envía formulario. Usuario ve brevemente "Completando inicio de sesión..." mientras navegador POST a ACS del Portal | ☐ | ☐ | ☐ |
| 5 | Procesamiento en ACS - Validación exitosa | Portal recibe POST con SAMLResponse | Sistema procesa assertion | Sistema ejecuta validaciones (HU-AD004C): firma, timestamps, audience. Extrae NameID. Busca usuario en BD. Si todo válido: genera JWT con roles del usuario, crea sesión en BD, establece cookie HttpOnly, elimina sesión temporal de RequestID, muestra brevemente "¡Bienvenido, {Nombre Usuario}!", redirige a RelayState (URL original solicitada o dashboard). Usuario autenticado y en página destino | ☐ | ☐ | ☐ |
| 6 | Deep link con autenticación - URL directa a recurso | Usuario sin sesión accede directamente a `https://portal.efac.com/facturacion/emitir` | Sistema detecta falta de sesión | Sistema: (1) Detecta que usuario no está autenticado, (2) Guarda URL solicitada `/facturacion/emitir` como RelayState, (3) Redirige a `/login?returnUrl=/facturacion/emitir`, (4) Usuario ingresa email AD, (5) Flujo SSO se ejecuta con RelayState preservado, (6) Tras autenticación, usuario llega directamente a `/facturacion/emitir` (no a dashboard) | ☐ | ☐ | ☐ |
| 7 | Sesión expirada - Re-autenticación automática | Usuario con sesión JWT expirada intenta acceder a recurso | Sistema detecta token expirado | Sistema: (1) Detecta JWT expirado, (2) Si usuario tiene `gestionado_por_ad=true`: redirige automáticamente a flujo SSO (no muestra login), (3) IdP detecta que usuario ya tiene sesión activa corporativa (SSO session), (4) IdP devuelve assertion SIN solicitar credenciales nuevamente (silent re-authentication), (5) Usuario obtiene nueva sesión transparentemente sin ingresar contraseña | ☐ | ☐ | ☐ |
| 8 | Indicadores de carga durante flujo | Usuario en medio de flujo SSO | Redirecciones ocurren | Sistema muestra pantallas de transición: "Redirigiendo a {Nombre Empresa}..." (spinner), "Validando credenciales..." (al regresar de IdP), "¡Bienvenido!" (tras validación exitosa). Cada pantalla visible 0.5-2 segundos. Usuario entiende que proceso está en curso | ☐ | ☐ | ☐ |
| 9 | Primer login de usuario - Onboarding | Usuario autenticado por primera vez vía SSO | Sesión creada | Sistema detecta que es primera autenticación del usuario (`primer_login=true`), tras autenticación muestra pantalla de bienvenida opcional: "¡Bienvenido al Portal, {Nombre}! Esta es su primera sesión", botón "Comenzar". Usuario hace clic y llega a dashboard. Flag `primer_login` se marca como false | ☐ | ☐ | ☐ |
| 10 | Usuario sin integración AD intenta usar SSO | Usuario ingresa email de dominio sin AD configurado | Sistema valida dominio | Sistema detecta que email NO pertenece a tenant con AD activo, muestra campo contraseña normal, NO muestra opción SSO. Usuario debe autenticarse con contraseña local | ☐ | ☐ | ☐ |
| 11 | Cancelación de SSO por usuario | Usuario en página IdP decide no continuar | Usuario cierra ventana o navega atrás | Usuario regresa manualmente a Portal (cierra pestaña o botón atrás del navegador). Portal detecta que no hay SAMLResponse, muestra pantalla de login nuevamente con mensaje opcional "Inicio de sesión cancelado". Usuario puede reintentar | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 12 | Estructura obligatoria de auditoría | Eventos del flujo SSO | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_SSO_*), Fecha/Hora (UTC ISO 8601), Usuario (userName o NULL), Cliente (tenant_id), IP Local (NULL), IP Pública (IP usuario), Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 13 | Auditoría de inicio de flujo SSO | Usuario hace clic en botón SSO | AuthnRequest generado | Registro: Tipo "INTEGRACION_AD_SSO_INICIADO", Resultado "EXITOSO", Severidad "INFO", Descripción "Flujo SSO iniciado para usuario {email}", Datos Adicionales: {"tenant_id": "UUID", "email": "juan@empresa.com", "request_id": "UUID", "relay_state": "/facturacion/emitir"} | ☐ | ☐ | ☐ |
| 14 | Auditoría de retorno desde IdP | Portal recibe SAMLResponse | POST a ACS recibido | Registro: Tipo "INTEGRACION_AD_SSO_RESPUESTA_RECIBIDA", Resultado "EXITOSO", Severidad "INFO", Descripción "SAMLResponse recibido del IdP para tenant {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "nameId": "juan@empresa.com", "in_response_to": "UUID"} | ☐ | ☐ | ☐ |
| 15 | Auditoría de login completo exitoso | Usuario autenticado y sesión creada | Flujo completado | Registro: Tipo "INTEGRACION_AD_SSO_LOGIN_COMPLETADO", Resultado "EXITOSO", Severidad "INFO", Descripción "Usuario {userName} completó login SSO exitosamente", Datos Adicionales: {"user_id": "UUID", "tenant_id": "UUID", "sesion_id": "UUID", "relay_state_final": "/facturacion/emitir", "duracion_flujo_segundos": 12} | ☐ | ☐ | ☐ |
| 16 | Auditoría de cancelación de flujo | Usuario no completa SSO | Flujo abandonado | Registro: Tipo "INTEGRACION_AD_SSO_CANCELADO", Resultado "FALLIDO", Severidad "INFO", Descripción "Flujo SSO abandonado/cancelado", Datos Adicionales: {"tenant_id": "UUID", "email": "juan@empresa.com", "request_id": "UUID"} | ☐ | ☐ | ☐ |
| 17 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, tenant, usuario, evento, resultado. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU-AD004B-LoginSSO-Paso1.html** - Detección de email AD

**Elementos:**
- Logo Portal (centrado)
- Título: "Iniciar Sesión"
- Campo Email (TextField, autofocus)
- **Estado inicial**: Muestra campo Email vacío + campo Contraseña deshabilitado
- **Después de ingresar email AD**:
  - Campo Email (con checkmark verde)
  - Mensaje: "Su organización **{Nombre Empresa}** usa Single Sign-On"
  - Botón "Continuar con {Nombre Empresa}" (contained, gradiente Primary→Secondary, ícono empresa)
  - Link pequeño: "¿No es usted? Cambiar email"

#### **HU-AD004B-RedirectingIdP.html** - Redirigiendo a IdP

**Elementos:**
- Logo Portal (pequeño, esquina superior)
- Spinner/Loading (centrado, animado)
- Mensaje: "Redirigiendo a **{Nombre Empresa}**..."
- Submensaje: "Un momento por favor"
- (Esta pantalla visible ~0.5-1 segundo antes de redirect)

#### **HU-AD004B-Processing.html** - Procesando respuesta

**Elementos:**
- Logo Portal (pequeño)
- Spinner/Loading
- Mensaje: "Validando credenciales..."
- Submensaje: "Completando inicio de sesión"
- (Visible ~0.5-1 segundo mientras valida SAML Response)

#### **HU-AD004B-Welcome.html** - Bienvenida tras login exitoso

**Elementos:**
- Checkmark verde grande (icono success)
- Mensaje: "¡Bienvenido, **{Nombre Usuario}**!"
- (Visible ~1 segundo, luego redirect automático a RelayState/dashboard)

#### **HU-AD004B-Onboarding.html** - Primer login (opcional)

**Elementos:**
- Título: "¡Bienvenido al Portal!"
- Mensaje: "Esta es su primera sesión. Comencemos"
- Ilustración/imagen de bienvenida
- Botón "Comenzar" (contained)

**Flujo Visual Completo:**
```
[Login Email Input]
    ↓ (ingresa email AD)
[Email + Botón SSO]
    ↓ (clic en botón)
[Redirigiendo...]
    ↓ (redirect a IdP)
[Página IdP - AD FS/Azure]
    ↓ (usuario ingresa credenciales)
[Procesando...]
    ↓ (POST a ACS, validación)
[¡Bienvenido!]
    ↓ (auto-redirect 1 seg)
[Dashboard / URL Original]
```

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Usuario confundido por redirecciones** | Múltiples redirecciones pueden desorientar al usuario | Bajo | Media | Mostrar mensajes claros en cada paso, indicadores de carga, branding consistente, transiciones rápidas (<2 seg cada una) | Mitigado |
| **RelayState perdido durante flujo** | Bug podría hacer que usuario llegue a dashboard en vez de URL solicitada | Medio | Baja | Almacenar RelayState en sesión temporal del servidor (no solo en parámetro URL), validar que RelayState es URL interna del Portal, testing exhaustivo | Mitigado |
| **IdP no devuelve usuario al Portal** | Configuración incorrecta en IdP podría hacer que ACS URL sea incorrecta | Alto | Media | Validar metadata SAML cargado correctamente en IdP, documentación clara, testing con IdPs comunes (AD FS, Azure AD), logs detallados si ACS no recibe POST | Mitigado |
| **Silent re-authentication falla** | Si IdP no mantiene sesión, usuario debe re-autenticar frecuentemente | Bajo | Media | Documentar configuración de session timeout en IdP, JWT del Portal con duración alineada a sesión IdP (4h), permitir re-autenticación fluida | Aceptado |

---

## Notas Adicionales

### Fuera de Alcance

- Autenticación multi-tenant en una sola sesión → Cada tenant es independiente
- Selección de IdP si tenant tiene múltiples → Solo 1 IdP por tenant
- Logout single sign-out (SLO) → HU futura
- Autenticación sin redirecciones (embedded login) → SAML requiere redirecciones por seguridad

### Dependencias

**Depende de:**
- **HU-AD004A - Service Provider SAML**: Infraestructura SAML debe estar implementada
- **HU-AD001A - Configuración Cliente**: IdP URL debe estar configurado
- **HU-AD002B - POST /Users**: Usuario debe existir (sincronizado)

**Relacionada con:**
- **HU-AD004C - Validación Aserciones**: ACS valida usando esa lógica
- **HU-AD004D - Manejo Errores SAML**: Errores durante flujo se manejan allí

### Valor Independiente

- ✅ Proporciona UX completa de SSO
- ✅ Minimiza fricción para usuarios corporativos
- ✅ Preserva contexto del usuario (deep links)
- ✅ Puede probarse end-to-end con IdP de prueba

### Especificación Técnica

**Sesión Temporal para RequestID:**
```javascript
// Almacenar en Redis o sesión servidor (NO cookie cliente por seguridad)
{
  requestId: "uuid-v4",
  timestamp: "2026-01-20T10:00:00Z",
  tenantId: "tenant-uuid",
  relayState: "/facturacion/emitir",
  email: "juan@empresa.com",
  expiresAt: "2026-01-20T10:15:00Z" // 15 min
}
```

**Validación de RelayState (Seguridad):**
```javascript
// Prevenir Open Redirect
function validateRelayState(relayState) {
  if (!relayState) return '/dashboard'; // default

  // Solo permitir URLs internas del Portal
  const url = new URL(relayState, 'https://portal.efac.com');
  if (url.origin !== 'https://portal.efac.com') {
    return '/dashboard'; // Rechazar URLs externas
  }

  return url.pathname + url.search;
}
```

**Tiempos de Transición (UX):**
- Email detectado → Mostrar botón SSO: Inmediato (0ms)
- Clic en SSO → Pantalla "Redirigiendo": 500ms
- Redirect a IdP: Inmediato tras generar AuthnRequest
- POST desde IdP → Pantalla "Procesando": 500-1500ms (según validación)
- Validación exitosa → Pantalla "Bienvenido": 1000ms
- Redirect a destino: Automático tras 1 seg

**Detección de Email AD:**
```sql
-- Query para determinar si email tiene AD
SELECT t.id, t.nombre, tac.idp_url
FROM tenants t
INNER JOIN tenant_ad_configuration tac ON t.id = tac.tenant_id
INNER JOIN usuarios u ON u.userName = 'juan@empresa.com' AND u.tenant_id = t.id
WHERE tac.estado_activo = true
  AND u.gestionado_por_ad = true
  AND u.deleted_at IS NULL;
```

### Referencias

- **HU-AD004A - Service Provider SAML**: Metadata y endpoints base
- **HU-AD004C - Validación Aserciones**: Lógica de validación en ACS
- **Estándar Auditoría**: Tipos `INTEGRACION_AD_SSO_*`
- **Glosario**: SSO, RelayState, AuthnRequest, ACS, Deep Link
- **OWASP**: Open Redirect Prevention

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - SAML Authentication
