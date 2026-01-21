# HU-AD004C - Validación de Aserciones SAML

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 2 |
| **Epic** | Integración con Active Directory mediante SAML |
| **Estado** | BORRADOR |
| **Propietario** | Por definir |
| **Arquitecto** | Por definir |
| **Desarrolladores** | Por definir |
| **Tester** | Por definir |
| **Incidentes relacionados** | N/A |
| **Arquitectura relacionada** | Portal Unificado - Autenticación, SAML 2.0, Seguridad |

---

## Contexto

Esta HU implementa las **validaciones de seguridad críticas** que el Portal debe realizar sobre las aserciones SAML recibidas del Identity Provider (IdP) del cliente antes de crear una sesión autenticada.

Las validaciones son **obligatorias** y **no negociables** para prevenir ataques de suplantación, replay attacks, y otras vulnerabilidades de seguridad SAML.

Una aserción SAML es válida SOLO si pasa **todas** las validaciones. Si alguna falla, la autenticación se rechaza completamente.

---

**Notas:**
- Validaciones basadas en especificación SAML 2.0 y mejores prácticas de seguridad
- Orden de validación importa (validar firma antes de parsear contenido)
- Logs detallados de validaciones fallidas para debugging sin exponer información sensible al usuario
- Validaciones son idénticas para todos los tenants (sin excepciones)

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Portal Unificado** | Quiero validar la firma XML de la aserción SAML | Para garantizar que proviene del IdP legítimo y no ha sido alterada |
| Como **Portal Unificado** | Quiero validar timestamps de la aserción | Para prevenir replay attacks con aserciones antiguas |
| Como **Portal Unificado** | Quiero validar el Audience de la aserción | Para asegurar que la aserción fue generada para este Portal específicamente |
| Como **Administrador de Seguridad** | Quiero logs detallados de validaciones fallidas | Para investigar intentos de ataque o problemas de configuración |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Validación #1: Decodificación Base64 | Portal recibe SAMLResponse en POST a ACS | Sistema procesa respuesta | Sistema decodifica parámetro `SAMLResponse` de Base64 a string XML. Si decodificación falla: rechaza con HTTP 400 "Invalid SAMLResponse encoding", registra error en auditoría (ERROR), NO continúa validaciones. Si exitoso: continúa a validación #2 | ☐ | ☐ | ☐ |
| 2 | Validación #2: Parseo de XML | XML decodificado | Sistema parsea XML | Sistema parsea XML SAML Response usando parser seguro (previene XXE attacks). Valida que es XML válido y contiene estructura SAML correcta (Response > Assertion). Si parseo falla o estructura incorrecta: rechaza "Malformed SAML Response", registra error. Si exitoso: continúa a validación #3 | ☐ | ☐ | ☐ |
| 3 | Validación #3: Firma XML Digital | XML parseado | Sistema valida firma | Sistema: (1) Localiza elemento `<Signature>` en XML (debe estar en Response o Assertion), (2) Obtiene certificado X.509 configurado para el tenant (HU-AD001A), (3) Valida firma XML usando certificado público, (4) Verifica que firma cubre toda la Assertion. Si firma inválida o faltante: rechaza "Invalid SAML signature", registra en auditoría (CRÍTICO). Si exitoso: continúa a validación #4 | ☐ | ☐ | ☐ |
| 4 | Validación #4: Timestamps (NotBefore, NotOnOrAfter) | Assertion parseada y firmada válida | Sistema valida tiempo | Sistema extrae atributos `NotBefore` y `NotOnOrAfter` de `<Conditions>`. Obtiene timestamp actual (UTC). Valida: `NotBefore - 5min <= now <= NotOnOrAfter + 5min` (tolerancia ±5 min para desincronización de relojes). Si fuera de ventana: rechaza "SAML assertion expired or not yet valid", registra timestamps en auditoría. Si válido: continúa a validación #5 | ☐ | ☐ | ☐ |
| 5 | Validación #5: Audience Restriction | Conditions de assertion | Sistema valida audience | Sistema extrae `<AudienceRestriction>` de Conditions. Valida que contiene Entity ID del Portal: `https://portal.efac.com/saml/{tenant_id}`. Si Audience NO coincide: rechaza "SAML assertion not intended for this service provider", registra audience esperado vs recibido. Si coincide: continúa a validación #6 | ☐ | ☐ | ☐ |
| 6 | Validación #6: InResponseTo (si aplica) | Assertion con InResponseTo | Sistema valida correlación | Si Response contiene atributo `InResponseTo`: Sistema busca RequestID en sesión temporal (almacenado en HU-AD004B). Si NO encuentra o no coincide: rechaza "Invalid InResponseTo, possible replay attack", registra. Si coincide o no está presente (IdP-initiated): continúa a validación #7 | ☐ | ☐ | ☐ |
| 7 | Validación #7: Subject Confirmation | Subject de assertion | Sistema valida confirmación | Sistema valida `<SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">`. Valida que `<SubjectConfirmationData>`: (1) Recipient = ACS URL del Portal, (2) NotOnOrAfter no ha expirado (±5 min tolerancia). Si inválido: rechaza "Invalid Subject Confirmation". Si válido: continúa a validación #8 | ☐ | ☐ | ☐ |
| 8 | Validación #8: Extracción de NameID | Subject confirmado | Sistema extrae identidad | Sistema extrae `<NameID>` de Subject. Valida: (1) NameID presente y no vacío, (2) Format es email o persistent (soportados). Extrae valor de NameID (ej: "juan.perez@empresa.com"). Si NameID inválido o faltante: rechaza "Missing or invalid NameID". Si válido: NameID se usa como userName para buscar usuario | ☐ | ☐ | ☐ |
| 9 | Validación #9: Unicidad de Assertion ID | Assertion ID extraído | Sistema previene replay | Sistema extrae atributo `ID` de Assertion. Consulta cache/BD de IDs procesados recientemente (últimas 24h). Si ID ya existe: rechaza "SAML assertion ID already processed, possible replay attack", registra (CRÍTICO). Si único: almacena ID en cache con TTL 24h, marca como procesado, continúa autenticación | ☐ | ☐ | ☐ |
| 10 | Validación completa exitosa | Todas las 9 validaciones pasaron | Sistema procesa usuario | Sistema: (1) Marca validación como exitosa, (2) Extrae NameID como userName, (3) Busca usuario en BD (HU-AD004A escenario 6), (4) Si usuario existe y activo: genera JWT y crea sesión, (5) Si usuario no existe/inactivo: rechaza con mensaje apropiado. Registra login exitoso en auditoría | ☐ | ☐ | ☐ |
| 11 | Validación con certificado expirado | Certificado X.509 del IdP ha expirado | Sistema valida firma | Sistema valida firma con certificado, detecta que certificado está expirado (fecha actual > validTo del certificado). Rechaza "IdP certificate expired", registra en auditoría (CRÍTICO), muestra mensaje al usuario "Error de configuración: certificado expirado. Contacte a soporte". Administrador debe renovar certificado en HU-AD001A | ☐ | ☐ | ☐ |
| 12 | Validación con múltiples aserciones | Response contiene >1 Assertion | Sistema procesa | Sistema detecta múltiples Assertions en Response. Valida y procesa SOLO la primera Assertion, ignora las demás (comportamiento estándar). Registra warning si hay >1 assertion | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 13 | Estructura obligatoria de auditoría | Eventos de validación SAML | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_SAML_VALIDACION_*), Fecha/Hora (UTC ISO 8601), Usuario (NameID o NULL), Cliente (tenant_id), IP Local (NULL), IP Pública (IP usuario), Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 14 | Auditoría de validación exitosa completa | Todas las validaciones pasaron | Assertion válida | Registro: Tipo "INTEGRACION_AD_SAML_VALIDACION_EXITOSA", Resultado "EXITOSO", Severidad "INFO", Descripción "Assertion SAML validada exitosamente para usuario {nameId}", Datos Adicionales: {"tenant_id": "UUID", "nameId": "juan@empresa.com", "assertion_id": "assertion-uuid", "issuer": "IdP-entity-id"} | ☐ | ☐ | ☐ |
| 15 | Auditoría de firma inválida | Validación #3 falla | Autenticación rechazada | Registro: Tipo "INTEGRACION_AD_SAML_FIRMA_INVALIDA", Resultado "FALLIDO", Severidad "CRITICAL", Descripción "Firma SAML inválida detectada para tenant {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "ip_usuario": "203.0.113.5", "razon": "Signature verification failed", "issuer": "IdP-entity-id"} | ☐ | ☐ | ☐ |
| 16 | Auditoría de assertion expirada | Validación #4 falla | Autenticación rechazada | Registro: Tipo "INTEGRACION_AD_SAML_ASERCION_EXPIRADA", Resultado "FALLIDO", Severidad "WARNING", Descripción "Assertion SAML expirada", Datos Adicionales: {"tenant_id": "UUID", "nameId": "juan@empresa.com", "notBefore": "2026-01-20T10:00:00Z", "notOnOrAfter": "2026-01-20T10:05:00Z", "timestamp_validacion": "2026-01-20T10:10:00Z"} | ☐ | ☐ | ☐ |
| 17 | Auditoría de replay attack detectado | Validación #9 falla (ID duplicado) | Ataque prevenido | Registro: Tipo "INTEGRACION_AD_SAML_REPLAY_DETECTADO", Resultado "FALLIDO", Severidad "CRITICAL", Descripción "Posible replay attack: Assertion ID ya procesado", Datos Adicionales: {"tenant_id": "UUID", "assertion_id": "assertion-uuid", "ip_usuario": "203.0.113.5", "primer_uso": "2026-01-20T10:00:00Z", "intento_duplicado": "2026-01-20T10:05:00Z"} | ☐ | ☐ | ☐ |
| 18 | Auditoría de certificado expirado | Validación #3 detecta cert expirado | Configuración inválida | Registro: Tipo "INTEGRACION_AD_SAML_CERTIFICADO_EXPIRADO", Resultado "FALLIDO", Severidad "CRITICAL", Descripción "Certificado X.509 del IdP expirado para tenant {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "certificado_expirado_desde": "2026-01-15T00:00:00Z", "accion_requerida": "Renovar certificado en configuración AD"} | ☐ | ☐ | ☐ |
| 19 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, tenant, evento, resultado, severidad. Exportable CSV/Excel. Alertas automáticas si CRITICAL events | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: N/A - Validaciones backend sin interfaz gráfica directa**

**Mensajes de Error al Usuario (según validación fallida):**

| Validación Fallida | Mensaje al Usuario | Acción Sugerida |
|--------------------|-------------------|----------------|
| #1, #2 (Decodificación/Parseo) | "Error al procesar respuesta de autenticación. Intente nuevamente." | Reintentar login |
| #3 (Firma) | "Error de autenticación: no se pudo verificar la identidad. Contacte a soporte." | Contactar administrador |
| #4 (Timestamps) | "La sesión de autenticación ha expirado. Intente nuevamente." | Reintentar login |
| #5 (Audience) | "Error de configuración de autenticación. Contacte a soporte." | Contactar administrador |
| #6 (InResponseTo) | "Error de seguridad detectado. No se pudo completar autenticación." | Contactar administrador |
| #7, #8 (Subject/NameID) | "Error al obtener información de usuario. Contacte a soporte." | Contactar administrador |
| #9 (Replay) | "Esta sesión de autenticación ya fue utilizada. Inicie sesión nuevamente." | Reintentar login |
| #11 (Certificado expirado) | "El certificado de autenticación ha expirado. Contacte al administrador del sistema." | Administrador renueva cert |

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Validación de firma implementada incorrectamente** | Bug en validación podría permitir aserciones falsificadas | Crítico | Baja | Usar biblioteca SAML probada (passport-saml, python3-saml), testing exhaustivo con aserciones manipuladas, revisión de seguridad de código, penetration testing | Mitigado |
| **Tolerancia de tiempo muy amplia** | ±5 min podría permitir replay en ventana extendida | Medio | Baja | Combinado con validación #9 (unicidad de ID), monitoreo de intentos en ventana corta, posibilidad de reducir a ±2 min en configuración | Mitigado |
| **Cache de IDs procesados saturado** | Muchos logins podrían saturar cache/BD de IDs | Bajo | Media | TTL automático de 24h, usar Redis para performance, límite de 100k IDs almacenados (suficiente para 4k logins/día), cleanup automático | Mitigado |
| **XXE attack en parseo XML** | Parser XML vulnerable podría exponer archivos del servidor | Crítico | Baja | Usar parser XML seguro con external entities deshabilitadas, validación estricta de DTD, testing con payloads XXE | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance

- Validación de atributos SAML adicionales (grupos, roles) → Se obtienen de BD Portal (sincronizados por SCIM), NO de assertion
- Encriptación de aserciones SAML → Solo firma digital requerida
- Validación de Response (solo Assertion) → Puede validar ambos si IdP lo requiere (HU futura)
- Configuración de tolerancia de tiempo por tenant → Global ±5 min para todos

### Dependencias

**Depende de:**
- **HU-AD001A - Configuración Cliente**: Certificado X.509 debe estar configurado
- **HU-AD004B - Flujo Login SSO**: Genera RequestID para validación #6

**Es prerequisito para:**
- **HU-AD004D - Manejo Errores SAML**: Usa mensajes de error definidos aquí
- **HU-AD005A - Session Management**: Sesión se crea solo si validación exitosa

### Valor Independiente

- ✅ Previene ataques de suplantación y replay
- ✅ Cumple estándar de seguridad SAML 2.0
- ✅ Puede probarse con aserciones manipuladas (testing)
- ✅ Provee logs detallados para debugging

### Especificación Técnica

**Orden de Validaciones (crítico):**
```
1. Decodificar Base64
2. Parsear XML (seguro)
3. Validar Firma Digital ← DEBE ser antes de confiar en contenido
4. Validar Timestamps
5. Validar Audience
6. Validar InResponseTo
7. Validar Subject Confirmation
8. Extraer NameID
9. Validar Unicidad de ID
```

**Librerías Recomendadas:**
- **Node.js**: `passport-saml`, `xml-crypto`, `xmldom` (con XXE protection)
- **Python**: `python3-saml`, `lxml` (con defusedxml)
- **.NET**: `System.IdentityModel.Tokens.Saml2`

**Prevención XXE (XML External Entity):**
```javascript
// Configuración segura de parser XML
const xmlParser = new DOMParser({
  locator: {},
  errorHandler: {
    warning: () => {},
    error: (e) => { throw e; },
    fatalError: (e) => { throw e; }
  }
});

// Deshabilitar external entities
const options = {
  disallow_doctype: true,
  disallow_external_entities: true
};
```

**Cache de Assertion IDs (Redis):**
```redis
SETEX assertion_id:{assertion-uuid} 86400 "processed"
# TTL = 24 horas = 86400 segundos
# Valor simple "processed" (solo necesitamos existencia)

# Consulta antes de procesar
EXISTS assertion_id:{assertion-uuid}
# Si devuelve 1 → Ya procesado (replay)
# Si devuelve 0 → Nuevo, procesar
```

**Validación de Certificado X.509:**
```javascript
const cert = getTenantCertificate(tenant_id); // De HU-AD001A
const certValid = cert.validFrom <= now && now <= cert.validTo;

if (!certValid) {
  throw new Error('IdP certificate expired');
}

// Validar firma XML
const signatureValid = verifyXmlSignature(
  assertionXml,
  cert.publicKey,
  'RSA-SHA256'
);
```

### Ejemplo de Assertion SAML Válida (simplificado)

```xml
<saml:Assertion ID="assertion-uuid-123" IssueInstant="2026-01-20T10:00:00Z">
  <saml:Issuer>https://adfs.empresa.com</saml:Issuer>
  <ds:Signature>...</ds:Signature>

  <saml:Subject>
    <saml:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress">
      juan.perez@empresa.com
    </saml:NameID>
    <saml:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
      <saml:SubjectConfirmationData
        NotOnOrAfter="2026-01-20T10:05:00Z"
        Recipient="https://portal.efac.com/saml/tenant-uuid/acs"/>
    </saml:SubjectConfirmation>
  </saml:Subject>

  <saml:Conditions NotBefore="2026-01-20T09:59:00Z" NotOnOrAfter="2026-01-20T10:05:00Z">
    <saml:AudienceRestriction>
      <saml:Audience>https://portal.efac.com/saml/tenant-uuid</saml:Audience>
    </saml:AudienceRestriction>
  </saml:Conditions>
</saml:Assertion>
```

### Referencias

- **SAML 2.0 Core**: Sections 2.3 (Assertions), 2.5 (Subject Confirmation)
- **SAML Security**: http://docs.oasis-open.org/security/saml/v2.0/saml-sec-consider-2.0-os.pdf
- **OWASP SAML Security Cheat Sheet**: https://cheatsheetseries.owasp.org/cheatsheets/SAML_Security_Cheat_Sheet.html
- **Estándar Auditoría**: Tipos `INTEGRACION_AD_SAML_VALIDACION_*`
- **Glosario**: Assertion, Firma XML, NameID, Audience, Replay Attack, XXE

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - SAML Authentication
