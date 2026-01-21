# HU-AD004D - Manejo de Errores SAML

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

Esta HU define cómo el Portal debe **manejar y comunicar errores** que ocurren durante el flujo de autenticación SAML 2.0, tanto errores técnicos (validación, configuración) como errores del Identity Provider (usuario no autorizado, credenciales incorrectas).

El manejo de errores debe:
- Proporcionar mensajes claros y accionables al usuario
- NO exponer información técnica sensible que pueda ayudar a atacantes
- Registrar detalles técnicos completos en auditoría para debugging
- Diferenciar entre errores recuperables (reintentar) y no recuperables (contactar soporte)

---

**Notas:**
- Mensajes al usuario son genéricos para seguridad (no revelar detalles internos)
- Logs técnicos contienen información detallada para administradores
- Errores se categorizan por severidad y acción requerida
- Algunos errores del IdP se reciben en SAML Response (StatusCode)

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Usuario** | Quiero mensajes de error claros cuando SSO falla | Para entender qué salió mal y qué hacer (reintentar vs contactar soporte) |
| Como **Portal Unificado** | Quiero registrar errores técnicos detallados en auditoría | Para que administradores puedan diagnosticar problemas de configuración o ataques |
| Como **Administrador del Portal** | Quiero distinguir entre errores del usuario vs problemas de configuración | Para priorizar acciones correctivas (renovar certificado vs instruir al usuario) |
| Como **Portal Unificado** | Quiero NO exponer detalles técnicos a usuarios finales | Para prevenir que atacantes obtengan información sobre infraestructura o configuración |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Error de IdP - StatusCode en SAML Response | IdP envía Response con Status != Success | Sistema procesa Response | Sistema parsea `<StatusCode>` de SAML Response. Valores comunes: `urn:oasis:names:tc:SAML:2.0:status:Responder` (error IdP), `urn:oasis:names:tc:SAML:2.0:status:AuthnFailed` (credenciales incorrectas), `urn:oasis:names:tc:SAML:2.0:status:NoAuthnContext`. Sistema extrae `<StatusMessage>` opcional. Muestra al usuario: "No se pudo autenticar. {StatusMessage genérico}". Registra StatusCode y StatusMessage completo en auditoría. Botón "Intentar Nuevamente" | ☐ | ☐ | ☐ |
| 2 | Error de credenciales incorrectas (AuthnFailed) | Usuario ingresa contraseña incorrecta en IdP | IdP devuelve AuthnFailed | Sistema detecta `StatusCode = AuthnFailed`. Muestra mensaje: "Las credenciales ingresadas son incorrectas. Verifique su usuario y contraseña." con botón "Intentar Nuevamente". Registra en auditoría (INFO, no es error de configuración). NO bloquea cuenta (eso es responsabilidad del IdP/AD) | ☐ | ☐ | ☐ |
| 3 | Error de usuario no autorizado (NoAuthnContext) | Usuario válido pero sin permisos en IdP | IdP devuelve NoAuthnContext | Sistema detecta StatusCode indicando falta de autorización. Muestra: "No tiene permisos para acceder al Portal. Contacte al administrador de su organización." Registra en auditoría con userName (si disponible). NO es error del Portal, es configuración del IdP | ☐ | ☐ | ☐ |
| 4 | Error de validación de firma (HU-AD004C #3) | Firma SAML inválida | Validación falla | Sistema detecta firma inválida. Muestra mensaje genérico: "Error de autenticación. No se pudo verificar la identidad. Contacte a soporte con código: {error_id}". Registra en auditoría (CRÍTICO) con detalles completos: certificado usado, issuer, razón de fallo. Genera error_id único para correlación. NO permite reintentar (problema de configuración) | ☐ | ☐ | ☐ |
| 5 | Error de assertion expirada (HU-AD004C #4) | Timestamps fuera de ventana | Validación falla | Sistema detecta expiración. Muestra: "La sesión de autenticación expiró. Intente nuevamente." con botón "Iniciar Sesión Nuevamente". Registra timestamps en auditoría (WARNING). Usuario puede reintentar inmediatamente | ☐ | ☐ | ☐ |
| 6 | Error de certificado X.509 expirado (HU-AD004C #11) | Certificado configurado ha expirado | Validación detecta expiración | Sistema detecta cert expirado. Muestra: "El certificado de autenticación ha expirado. El sistema no puede procesar autenticaciones hasta que se renueve. Contacte al administrador." Registra en auditoría (CRÍTICO) con fecha de expiración. Genera alerta para administradores del Portal. NO recuperable sin renovar certificado | ☐ | ☐ | ☐ |
| 7 | Error de usuario no sincronizado (HU-AD004A #10) | Usuario autenticado en IdP pero no en Portal | Búsqueda de usuario falla | Sistema busca usuario por NameID, no encuentra. Muestra: "Usuario no encontrado en el Portal. Su cuenta debe ser sincronizada. Contacte al administrador con su email: {userName}". Registra en auditoría (WARNING) con userName. Administrador debe ejecutar sincronización SCIM | ☐ | ☐ | ☐ |
| 8 | Error de usuario inactivo | Usuario existe pero active=false o deleted_at presente | Validación de usuario falla | Sistema encuentra usuario pero inactivo. Muestra: "Su cuenta está inactiva. Contacte al administrador para reactivarla." Registra en auditoría (INFO) con userName. Usuario no puede autenticarse hasta reactivación en AD + sincronización | ☐ | ☐ | ☐ |
| 9 | Error técnico de conexión a BD | BD no disponible durante validación | Sistema falla al consultar | Sistema detecta error de conexión (timeout, BD caída). Muestra: "Error temporal del sistema. Intente nuevamente en unos momentos." Registra en auditoría (ERROR) con stack trace completo. Ofrece botón "Reintentar". Monitoreo alerta a equipo técnico | ☐ | ☐ | ☐ |
| 10 | Pantalla de error SAML genérica | Error no categorizado específicamente | Cualquier error no manejado explícitamente | Sistema muestra pantalla de error consistente con: Logo Portal, Ícono de error, Título "Error de Autenticación", Mensaje específico (según tipo de error), Código de error único para soporte, Opciones: "Intentar Nuevamente" o "Contactar Soporte" (según error), Link a documentación de SSO. Diseño consistente con guía de estilos | ☐ | ☐ | ☐ |
| 11 | Error de configuración de IdP URL inválida | IdP URL configurada no responde | Redirect a IdP falla | Sistema intenta redirect pero URL inválida/no accesible. Muestra: "Error de configuración de autenticación. La URL del Identity Provider no está disponible. Contacte a soporte con código: {error_id}". Registra en auditoría (ERROR). Administrador debe validar URL en HU-AD001A | ☐ | ☐ | ☐ |
| 12 | Timeout esperando respuesta de IdP | Usuario no completa login en IdP en tiempo razonable | Timeout (5 min) | Sistema detecta que no recibió SAMLResponse en 5 minutos (sesión temporal expira). Usuario regresa manualmente al Portal. Muestra: "La sesión de autenticación expiró por inactividad. Intente nuevamente." Limpia sesión temporal. Usuario puede reiniciar flujo | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 13 | Estructura obligatoria de auditoría | Eventos de errores SAML | Sistema registra | Registro contiene: ID Evento (UUID - usado como error_id), Tipo Evento (INTEGRACION_AD_SAML_ERROR_*), Fecha/Hora (UTC ISO 8601), Usuario (NameID si disponible o NULL), Cliente (tenant_id), IP Local (NULL), IP Pública (IP usuario), Resultado "FALLIDO", Descripción, Severidad (INFO/WARNING/ERROR/CRITICAL), Datos Adicionales (detalles técnicos completos) | ☐ | ☐ | ☐ |
| 14 | Auditoría de credenciales incorrectas | Usuario falla autenticación en IdP | AuthnFailed recibido | Registro: Tipo "INTEGRACION_AD_SAML_ERROR_CREDENCIALES", Resultado "FALLIDO", Severidad "INFO", Descripción "Credenciales incorrectas para usuario {nameId}", Datos Adicionales: {"tenant_id": "UUID", "nameId": "juan@empresa.com", "statusCode": "AuthnFailed", "statusMessage": "Invalid credentials"} | ☐ | ☐ | ☐ |
| 15 | Auditoría de firma inválida | Validación de firma falla | Error crítico | Registro: Tipo "INTEGRACION_AD_SAML_ERROR_FIRMA", Resultado "FALLIDO", Severidad "CRITICAL", Descripción "Firma SAML inválida detectada", Datos Adicionales: {"tenant_id": "UUID", "error_id": "UUID", "issuer": "IdP-entity", "razon": "Certificate mismatch", "certificado_thumbprint": "ABC123..."} | ☐ | ☐ | ☐ |
| 16 | Auditoría de certificado expirado | Validación detecta cert expirado | Error de configuración | Registro: Tipo "INTEGRACION_AD_SAML_ERROR_CERTIFICADO_EXPIRADO", Resultado "FALLIDO", Severidad "CRITICAL", Descripción "Certificado X.509 expirado para tenant {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "error_id": "UUID", "expirado_desde": "2026-01-15", "accion_requerida": "Renovar certificado en configuración"} | ☐ | ☐ | ☐ |
| 17 | Auditoría de usuario no sincronizado | Usuario autenticado pero no existe | Error de sincronización | Registro: Tipo "INTEGRACION_AD_SAML_ERROR_USUARIO_NO_SINCRONIZADO", Resultado "FALLIDO", Severidad "WARNING", Descripción "Usuario {nameId} no sincronizado en Portal", Datos Adicionales: {"tenant_id": "UUID", "nameId": "nuevo@empresa.com", "accion_requerida": "Ejecutar sincronización SCIM"} | ☐ | ☐ | ☐ |
| 18 | Auditoría de error técnico | BD, timeout, o error interno | Fallo del sistema | Registro: Tipo "INTEGRACION_AD_SAML_ERROR_TECNICO", Resultado "FALLIDO", Severidad "ERROR", Descripción "Error técnico durante autenticación SAML", Datos Adicionales: {"tenant_id": "UUID", "error_id": "UUID", "error_tipo": "DatabaseTimeout", "stack_trace": "..."} | ☐ | ☐ | ☐ |
| 19 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, tenant, evento, severidad. Exportable CSV/Excel. Alertas automáticas para CRITICAL events | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU-AD004D-ErrorSAML.html** - Pantalla de error genérica

**Elementos:**
- Logo Portal (esquina superior)
- Ícono de error (rojo, tamaño grande)
- Título: "Error de Autenticación" (H3)
- **Mensaje principal** (variable según error):
  - Texto claro y accionable
  - Ejemplos en escenarios 1-12
- **Código de error** (para errores de configuración):
  - "Código de error: {error_id}" (monospace, pequeño, copiable)
  - Permite a usuario reportar a soporte
- **Opciones de acción**:
  - Botón "Intentar Nuevamente" (contained, si error recuperable)
  - Botón "Contactar Soporte" (outlined, abre mailto o chat)
  - Link "Volver al Inicio" (text)
- **Información adicional** (colapsable):
  - "¿Qué salió mal?"
  - Explicación breve del error (NO técnica)
  - Pasos sugeridos
- Link a documentación: "Guía de Solución de Problemas de SSO"

**Tipos de Pantalla:**

**Tipo 1: Error Recuperable** (credenciales, expiración, timeout)
- Botón primario: "Intentar Nuevamente" → Redirige a login
- Tono: Neutral, no alarmante
- Ejemplo: "La sesión de autenticación expiró. Intente nuevamente."

**Tipo 2: Error de Configuración** (firma, certificado, IdP URL)
- Botón primario: "Contactar Soporte"
- Código de error visible y copiable
- Tono: Informativo, indica problema del sistema
- Ejemplo: "Error de configuración. Contacte a soporte con código: ABC-123"

**Tipo 3: Error de Autorización** (usuario no autorizado, inactivo)
- Botón: "Contactar Administrador"
- Tono: Informativo, indica acción del usuario
- Ejemplo: "Su cuenta está inactiva. Contacte al administrador."

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Mensajes de error revelan información sensible** | Mensajes muy detallados podrían ayudar a atacantes | Alto | Baja | Mensajes genéricos al usuario, detalles solo en auditoría, revisión de seguridad de mensajes, NO exponer stack traces | Mitigado |
| **Usuarios bloqueados sin visibilidad** | Errores de configuración impiden login masivo sin alerta clara | Alto | Media | Alertas automáticas para CRITICAL errors, dashboard de errores SAML para administradores, monitoreo de tasa de fallos | Mitigado |
| **Códigos de error predecibles** | Códigos secuenciales podrían revelar volumen de errores | Bajo | Baja | Usar UUIDs como error_id, no secuenciales | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance

- Auto-recuperación de errores (reintentos automáticos) → Usuario decide si reintentar
- Notificaciones push a administradores → Emails/alertas son HU futura
- Chatbot para diagnóstico de errores → Documentación estática solamente
- Diferentes mensajes por idioma → Solo español inicialmente

### Dependencias

**Depende de:**
- **HU-AD004A, B, C**: Todos los componentes SAML generan errores manejados aquí
- **Sistema de Auditoría**: Logs detallados

**Relacionada con:**
- **HU-AD007A-D - Audit & Logs**: Consulta de errores desde dashboard

### Valor Independiente

- ✅ Mejora UX en casos de error
- ✅ Facilita debugging para administradores
- ✅ Previene exposición de información sensible
- ✅ Reduce tickets de soporte con mensajes claros

### Especificación Técnica

**Categorización de Errores:**

| Categoría | Severidad | Acción Usuario | Ejemplo |
|-----------|-----------|----------------|---------|
| **Usuario** | INFO | Reintentar | Credenciales incorrectas |
| **Temporal** | WARNING | Reintentar en momentos | Timeout, BD temporal |
| **Configuración** | ERROR/CRITICAL | Contactar soporte | Firma inválida, cert expirado |
| **Autorización** | INFO | Contactar admin organización | No autorizado, cuenta inactiva |

**Generación de error_id:**
```javascript
const errorId = uuidv4(); // ej: "f47ac10b-58cc-4372-a567-0e02b2c3d479"

// Auditoría incluye error_id
auditLog({
  id: errorId, // Mismo que error_id mostrado al usuario
  tipo: 'INTEGRACION_AD_SAML_ERROR_FIRMA',
  // ... resto de campos
});

// Mensaje al usuario
return {
  message: 'Error de autenticación. Contacte a soporte.',
  errorId: errorId, // Mostrar al usuario para correlación
  recoverable: false
};
```

**StatusCodes SAML Comunes:**

| StatusCode | Significado | Manejo |
|------------|-------------|--------|
| `urn:oasis:names:tc:SAML:2.0:status:Success` | Exitoso | Continuar autenticación |
| `urn:oasis:names:tc:SAML:2.0:status:Responder` | Error genérico IdP | Mensaje "Error del IdP. Intente nuevamente" |
| `urn:oasis:names:tc:SAML:2.0:status:AuthnFailed` | Credenciales incorrectas | "Credenciales incorrectas" |
| `urn:oasis:names:tc:SAML:2.0:status:NoAuthnContext` | Usuario no autorizado | "No tiene permisos" |
| `urn:oasis:names:tc:SAML:2.0:status:RequestDenied` | Solicitud denegada | "Solicitud denegada por IdP" |

**Ejemplo de Response con Error:**
```xml
<samlp:Response>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Responder">
      <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:AuthnFailed"/>
    </samlp:StatusCode>
    <samlp:StatusMessage>Invalid username or password</samlp:StatusMessage>
  </samlp:Status>
</samlp:Response>
```

### Mensajes de Error - Guía de Redacción

**Principios:**
1. **Claro**: Usuario entiende qué pasó sin jerga técnica
2. **Accionable**: Usuario sabe qué hacer a continuación
3. **Seguro**: NO revela información interna del sistema
4. **Empático**: Tono amigable, no culpar al usuario

**Ejemplos Buenos vs Malos:**

❌ Malo: "SAML signature verification failed with error code 0x8004"
✅ Bueno: "Error de autenticación. Contacte a soporte."

❌ Malo: "User object not found in database table `users` for tenant UUID abc-123"
✅ Bueno: "Usuario no encontrado. Su cuenta debe ser sincronizada."

❌ Malo: "Certificate expired on 2026-01-15. Renew certificate in IdP configuration."
✅ Bueno: "El certificado de autenticación ha expirado. Contacte al administrador."

### Referencias

- **SAML 2.0 StatusCodes**: http://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf (Section 3.2.2.2)
- **OWASP Error Handling**: https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html
- **Estándar Auditoría**: Tipos `INTEGRACION_AD_SAML_ERROR_*`
- **Guía de Estilos**: Material UI, Error States, Messaging Guidelines

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - SAML Authentication
