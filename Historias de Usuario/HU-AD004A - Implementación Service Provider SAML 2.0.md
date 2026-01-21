# HU-AD004A - Implementación Service Provider SAML 2.0

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
| **Arquitectura relacionada** | Portal Unificado - Autenticación, SAML 2.0, SSO |

---

## Contexto

Esta HU implementa el **Service Provider (SP) SAML 2.0** del Portal Unificado que permite a usuarios de clientes con integración AD autenticarse mediante **Single Sign-On (SSO)** usando sus credenciales de Active Directory.

SAML 2.0 es el protocolo **obligatorio y no negociable** para autenticación de usuarios gestionados por AD. Los usuarios NO ingresan contraseña en el Portal; son redirigidos al Identity Provider (IdP) del cliente (típicamente AD FS o Azure AD) para autenticarse.

El Portal actúa como **Service Provider** que confía en aserciones SAML firmadas por el IdP del cliente.

---

**Notas:**
- SAML 2.0 es **obligatorio** para clientes con integración AD (no opcional)
- Usuarios con `gestionado_por_ad=true` SOLO pueden autenticarse vía SAML
- Portal genera metadata SAML para que cliente configure en su IdP
- Cada tenant tiene su propia configuración SAML independiente (multitenant)
- Aserciones SAML deben estar firmadas con certificado X.509 configurado en HU-AD001A

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Usuario de Cliente con AD** | Quiero autenticarme con mis credenciales de Active Directory | Para acceder al Portal sin crear contraseña adicional, usando mis credenciales corporativas |
| Como **Portal Unificado (SP)** | Quiero recibir aserciones SAML firmadas del IdP del cliente | Para autenticar usuarios confiando en la identidad validada por el Active Directory |
| Como **Administrador del Portal** | Quiero generar metadata SAML del Portal | Para entregar al cliente y configurar el Portal como Service Provider en su IdP |
| Como **Portal Unificado** | Quiero validar firma de aserciones SAML con certificado del cliente | Para garantizar que la aserción proviene del IdP legítimo y no ha sido alterada |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Generación de metadata SAML del Portal (SP) | Administrador necesita configurar IdP del cliente | Administrador accede a configuración AD del tenant | Sistema genera XML metadata SAML del Portal con: Entity ID `https://portal.efac.com/saml/{tenant_id}`, Assertion Consumer Service URL `https://portal.efac.com/saml/{tenant_id}/acs`, Single Logout Service URL, Certificado X.509 público del Portal. Permite descargar archivo XML. Proporciona instrucciones para cargar en IdP del cliente | ☐ | ☐ | ☐ |
| 2 | Pantalla de login con opción SSO | Usuario con email de dominio corporativo accede al Portal | Usuario ingresa email en login | Sistema detecta que email pertenece a tenant con integración AD activa (valida dominio), muestra botón "Iniciar Sesión con {Nombre Empresa}" en lugar de campo contraseña. Usuario hace clic en botón SSO | ☐ | ☐ | ☐ |
| 3 | Inicio de flujo SSO - SAML AuthnRequest | Usuario hace clic en botón SSO | Sistema inicia autenticación SAML | Sistema genera SAML AuthnRequest XML (protocolo SAML 2.0), codifica en Base64, redirige navegador del usuario a URL del IdP configurada en HU-AD001A con parámetros: `SAMLRequest={encoded_request}&RelayState={return_url}`. Usuario sale del Portal y llega a página de login del IdP corporativo | ☐ | ☐ | ☐ |
| 4 | Usuario se autentica en IdP corporativo | Usuario redirigido a IdP (AD FS / Azure AD) | Usuario ingresa credenciales corporativas | IdP del cliente valida credenciales contra Active Directory, genera SAML Response con aserción firmada (contiene: NameID=userName, atributos de usuario), codifica Response en Base64, redirige navegador de vuelta al Portal vía POST al Assertion Consumer Service (ACS) | ☐ | ☐ | ☐ |
| 5 | Recepción y validación de SAML Response | IdP redirige a `/saml/{tenant_id}/acs` con SAMLResponse | Portal recibe POST con SAMLResponse | Sistema: (1) Decodifica Base64, (2) Parsea XML SAML Response, (3) Valida firma XML usando certificado X.509 del cliente (configurado en HU-AD001A), (4) Verifica que aserción no ha expirado (NotBefore, NotOnOrAfter), (5) Verifica Audience = Entity ID del Portal, (6) Extrae NameID (userName) de la aserción. Si validación exitosa: continúa autenticación | ☐ | ☐ | ☐ |
| 6 | Búsqueda de usuario por NameID | Sistema extrae userName de aserción SAML | Sistema valida usuario | Sistema busca usuario en BD con `userName = NameID AND tenant_id = {tenant_id} AND gestionado_por_ad = true AND deleted_at IS NULL`. Si usuario existe y está activo: continúa creación de sesión. Si usuario no existe: error "Usuario no encontrado" (debe sincronizarse vía SCIM primero). Si usuario `active=false` o `deleted_at` presente: error "Usuario inactivo" | ☐ | ☐ | ☐ |
| 7 | Creación de sesión JWT tras SAML exitoso | Usuario validado exitosamente | Sistema crea sesión | Sistema genera JWT con: user_id, tenant_id, roles (del usuario en BD), expiración (4 horas por defecto configurado en HU-AD001A), marca sesión como `origen_saml=true`. Almacena sesión en BD para invalidación proactiva futura. Establece cookie HttpOnly con JWT. Redirige a `RelayState` (URL original solicitada) o dashboard. Usuario autenticado exitosamente | ☐ | ☐ | ☐ |
| 8 | Error de validación de firma SAML | SAML Response firmado con certificado diferente | Sistema valida firma | Sistema detecta que firma XML no coincide con certificado configurado para el tenant, rechaza autenticación, muestra mensaje "Error de autenticación: firma SAML inválida. Contacte a soporte", registra error en auditoría (CRÍTICO), NO crea sesión | ☐ | ☐ | ☐ |
| 9 | Error de aserción SAML expirada | IdP envía aserción con NotOnOrAfter antiguo | Sistema valida timestamps | Sistema detecta que timestamp actual > NotOnOrAfter de la aserción, rechaza autenticación, muestra mensaje "La sesión de autenticación ha expirado. Intente nuevamente", registra en auditoría (INFO), NO crea sesión | ☐ | ☐ | ☐ |
| 10 | Error de usuario no sincronizado | Usuario autenticado en IdP pero no existe en Portal | Sistema busca usuario | Sistema no encuentra usuario en BD (nunca fue sincronizado vía SCIM), muestra mensaje "Usuario no encontrado. Contacte al administrador para sincronización", registra en auditoría (WARNING), NO crea sesión. Administrador debe ejecutar sincronización SCIM | ☐ | ☐ | ☐ |
| 11 | Prevención de autenticación local para usuarios AD | Usuario gestionado por AD intenta login con contraseña | Usuario ingresa email y contraseña | Sistema detecta `gestionado_por_ad=true` para el email, rechaza autenticación local, muestra mensaje "Este usuario debe autenticarse vía Single Sign-On (SSO) con Active Directory", ofrece botón SSO. NO valida contraseña | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 12 | Estructura obligatoria de auditoría | Eventos de autenticación SAML | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_SAML_*), Fecha/Hora (UTC ISO 8601), Usuario (userName de aserción o NULL), Cliente (tenant_id), IP Local (NULL), IP Pública (IP del usuario), Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 13 | Auditoría de autenticación SAML exitosa | Usuario completa SSO exitosamente | Sesión creada | Registro: Tipo "INTEGRACION_AD_SAML_LOGIN_EXITOSO", Resultado "EXITOSO", Severidad "INFO", Descripción "Usuario {userName} autenticado vía SAML para tenant {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "user_id": "UUID", "userName": "juan.perez@empresa.com", "nameId": "juan.perez@empresa.com", "ip_usuario": "203.0.113.5", "sesion_id": "UUID"} | ☐ | ☐ | ☐ |
| 14 | Auditoría de firma SAML inválida | Firma de aserción no valida | Autenticación rechazada | Registro: Tipo "INTEGRACION_AD_SAML_FIRMA_INVALIDA", Resultado "FALLIDO", Severidad "ERROR", Descripción "Intento de autenticación SAML con firma inválida para tenant {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "nameId": "desconocido", "ip_usuario": "203.0.113.5", "razon": "Certificado no coincide"} | ☐ | ☐ | ☐ |
| 15 | Auditoría de aserción expirada | Aserción fuera de ventana de tiempo | Autenticación rechazada | Registro: Tipo "INTEGRACION_AD_SAML_ASERCION_EXPIRADA", Resultado "FALLIDO", Severidad "WARNING", Descripción "Aserción SAML expirada para usuario {nameId}", Datos Adicionales: {"tenant_id": "UUID", "nameId": "juan.perez@empresa.com", "notOnOrAfter": "2026-01-20T10:00:00Z", "timestamp_actual": "2026-01-20T10:05:00Z"} | ☐ | ☐ | ☐ |
| 16 | Auditoría de usuario no sincronizado | Usuario autenticado en IdP pero no en Portal | Autenticación rechazada | Registro: Tipo "INTEGRACION_AD_SAML_USUARIO_NO_SINCRONIZADO", Resultado "FALLIDO", Severidad "WARNING", Descripción "Usuario {nameId} autenticado en IdP pero no existe en Portal", Datos Adicionales: {"tenant_id": "UUID", "nameId": "nuevo@empresa.com", "accion_requerida": "Ejecutar sincronización SCIM"} | ☐ | ☐ | ☐ |
| 17 | Auditoría de metadata SAML generado | Administrador descarga metadata | Sistema genera XML | Registro: Tipo "INTEGRACION_AD_SAML_METADATA_GENERADO", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {usuario} generó metadata SAML para tenant {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "entity_id": "https://portal.efac.com/saml/{tenant_id}"} | ☐ | ☐ | ☐ |
| 18 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, tenant, usuario, evento, resultado. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU-AD004A-LoginSSO.html** - Pantalla de login con SSO

**Elementos:**
- Logo del Portal (centrado)
- Título: "Iniciar Sesión"
- Campo: Email (TextField)
- **Condicional según email:**
  - Si email NO es de tenant con AD: Muestra campo "Contraseña" + botón "Iniciar Sesión"
  - Si email ES de tenant con AD: Oculta campo contraseña, muestra:
    - Mensaje: "Su organización usa Single Sign-On"
    - Botón "Iniciar Sesión con {Nombre Empresa}" (contained, gradiente, con ícono de empresa/AD)
    - Link "¿No puede acceder? Contacte a soporte"

**Estados:**
- Email no ingresado
- Email de usuario local (sin AD) - muestra contraseña
- Email de usuario AD - muestra botón SSO
- Error "Usuario debe usar SSO"

#### **HU-AD004A-MetadataSAML.html** - Generación de metadata

**Elementos:**
- Título: "Metadata SAML - {Nombre Cliente}"
- Sección "Entity ID": `https://portal.efac.com/saml/{tenant_id}` (TextField read-only, botón "Copiar")
- Sección "Assertion Consumer Service (ACS) URL": `https://portal.efac.com/saml/{tenant_id}/acs` (TextField read-only, botón "Copiar")
- Botón "Descargar Metadata XML" (contained, ícono download) - descarga archivo sp-metadata.xml
- **Instrucciones para el cliente:**
  1. Descargue el archivo metadata XML
  2. En su IdP (AD FS / Azure AD), agregue el Portal como Relying Party Trust / Enterprise Application
  3. Cargue el metadata XML descargado
  4. Configure Name ID format como "Email Address" o "Persistent"
  5. Asegúrese de que las aserciones SAML estén firmadas con el certificado configurado en el Portal
- Link "Ver Guía Detallada de Configuración" (abre documentación)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Firma SAML no validada permite suplantación** | Si validación de firma falla, atacante podría generar aserciones falsas | Crítico | Baja | Validación estricta de firma XML con certificado X.509, rechazo total si firma inválida, auditoría de intentos fallidos, testing exhaustivo | Mitigado |
| **Certificado X.509 del IdP expirado** | Si certificado expira, todas las autenticaciones SAML fallarán | Alto | Media | Validar expiración al configurar, mostrar fecha de expiración en UI, alertas 30 días antes (HU futura), documentar renovación | Mitigado |
| **Usuario no sincronizado no puede autenticarse** | Si SCIM no sincronizó usuario, SSO falla incluso con credenciales válidas | Alto | Media | Mensaje claro indicando "Usuario no sincronizado", auditoría con warning, documentar que SCIM debe ejecutarse primero | Mitigado |
| **Relojes desincronizados causan validación de timestamps fallida** | Si reloj del IdP o Portal está desincronizado, aserciones parecen expiradas | Medio | Baja | Permitir tolerancia de ±5 minutos en validación NotBefore/NotOnOrAfter, usar NTP en servidores, validar sincronización horaria | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance

- Autenticación local (contraseña) para usuarios AD → Prohibido por diseño
- SAML Single Logout (SLO) → HU futura (opcional)
- Configuración IdP desde Portal → Cliente configura su IdP manualmente
- Soportar múltiples IdPs por tenant → Solo 1 IdP por tenant
- SAML Metadata refresh automático → Manual por administrador

### Dependencias

**Depende de:**
- **HU-AD001A - Configuración Cliente**: IdP URL y certificado deben estar configurados
- **HU-AD002B - POST /Users**: Usuarios deben existir (sincronizados vía SCIM)
- **Sistema de Sesiones JWT**: Para crear sesiones tras autenticación exitosa

**Es prerequisito para:**
- **HU-AD005A, B - Session Management**: Sesiones creadas por SAML necesitan gestión
- **HU-AD006A, B, C - Invalidación Proactiva**: Sesiones SAML pueden invalidarse

### Valor Independiente

- ✅ Permite autenticación SSO para clientes con AD
- ✅ Elimina gestión de contraseñas para usuarios corporativos
- ✅ Cumple requisito obligatorio de SAML 2.0
- ✅ Puede probarse con IdP de prueba (ej: SAML test tools)

### Especificación Técnica

**SAML 2.0 Binding:**
- HTTP-POST Binding para AuthnRequest (opcional)
- HTTP-POST Binding para Response (obligatorio)
- HTTP-Redirect Binding para AuthnRequest (recomendado)

**Algoritmos de Firma Soportados:**
- RSA-SHA256 (recomendado)
- RSA-SHA1 (legacy, deprecado pero aceptado)

**Name ID Format:**
- `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress` (preferido)
- `urn:oasis:names:tc:SAML:2.0:nameid-format:persistent`

**Modelo de Datos (adición):**
```sql
ALTER TABLE sessions ADD COLUMN origen_saml BOOLEAN DEFAULT false;
ALTER TABLE sessions ADD COLUMN saml_session_index VARCHAR(500) NULL; -- Para SLO futuro

CREATE INDEX idx_sessions_origen_saml ON sessions(origen_saml) WHERE origen_saml = true;
```

**Ejemplo de Metadata XML (Service Provider):**
```xml
<?xml version="1.0"?>
<md:EntityDescriptor xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata"
                     entityID="https://portal.efac.com/saml/{tenant_id}">
  <md:SPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
    <md:AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
                                 Location="https://portal.efac.com/saml/{tenant_id}/acs"
                                 index="1"/>
  </md:SPSSODescriptor>
</md:EntityDescriptor>
```

### Referencias

- **SAML 2.0 Core Specification**: http://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf
- **SAML 2.0 Bindings**: http://docs.oasis-open.org/security/saml/v2.0/saml-bindings-2.0-os.pdf
- **Estándar Auditoría**: Tipos `INTEGRACION_AD_SAML_*`
- **Glosario**: SAML, SSO, Service Provider, Identity Provider, Assertion, NameID
- **Biblioteca recomendada**: passport-saml (Node.js), python3-saml (Python)

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - SAML Authentication
