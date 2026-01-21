# HU-AD008A - Manejo de Errores SCIM

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 2 |
| **Epic** | Integración con Active Directory mediante SCIM |
| **Estado** | BORRADOR |

---

## Contexto

Manejo estandarizado de errores en endpoint SCIM según especificación SCIM 2.0 (RFC 7644).

---

## Escenarios Básicos

| Nº | Criterio de aceptación (Título) | Resultado esperado |
|----|--------------------------------|---------------------|
| 1 | Error 400 - Bad Request | JSON malformado, campos faltantes → `{"schemas": ["urn:ietf:params:scim:api:messages:2.0:Error"], "status": "400", "scimType": "invalidSyntax", "detail": "Missing required attribute: userName"}` |
| 2 | Error 401 - Unauthorized | Bearer Token inválido/expirado → `{"status": "401", "detail": "Authentication failed"}` |
| 3 | Error 404 - Not Found | Usuario no existe → `{"status": "404", "detail": "User not found"}` |
| 4 | Error 409 - Conflict | userName duplicado → `{"status": "409", "scimType": "uniqueness", "detail": "userName already exists"}` |
| 5 | Error 429 - Too Many Requests | Rate limit excedido → `{"status": "429", "detail": "Too many requests"}` + header `Retry-After: 60` |
| 6 | Error 500 - Internal Server Error | Error inesperado → `{"status": "500", "detail": "Internal server error"}`. NO exponer stack traces. Registrar en logs técnicos |
| 7 | Retry automático del cliente SCIM | Servidor devuelve 503 Service Unavailable → Cliente SCIM debe respetar `Retry-After` header y reintentar |

**Notas:**
- Todos los errores devuelven JSON SCIM estándar (no HTML)
- Mensajes genéricos para seguridad (no revelar internals)
- Logs técnicos contienen detalles completos para debugging

---

**Versión:** 1.0
**Módulo:** Integración AD/SCIM - Error Handling
