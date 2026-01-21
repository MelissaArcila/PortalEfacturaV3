# HU-AD002A - Implementación Endpoint SCIM Base

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
| **Arquitectura relacionada** | Portal Unificado - API SCIM, Integración AD |

---

## Contexto

Esta HU implementa el **Endpoint SCIM 2.0 base** que recibe peticiones de sincronización desde los servidores Active Directory de los clientes.

El endpoint sigue el estándar **SCIM 2.0 (RFC 7644)** y proporciona operaciones para creación, actualización, lectura y eliminación (soft delete) de usuarios desde Active Directory.

La URL del endpoint es **multitenant**: `https://portal.efac.com/scim/v2/{tenant_id}/Users`

Cada cliente (tenant) sincroniza únicamente sus usuarios mediante autenticación con Bearer Token generado en HU-AD001A.

---

**Notas:**
- Endpoint público (internet), protegido por autenticación Bearer Token
- Implementa estándar SCIM 2.0 (RFC 7644)
- Operaciones soportadas: GET, POST, PUT, PATCH, DELETE
- Solo recursos `/Users` (grupos se sincronizan como atributo de usuario)
- Idempotente: operaciones repetidas producen mismo resultado
- Errores devuelven formato SCIM estándar (JSON con status, detail)

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Servidor SCIM del Cliente AD** | Quiero enviar peticiones SCIM al Portal con autenticación Bearer Token | Para sincronizar usuarios desde Active Directory al Portal |
| Como **Portal Unificado** | Quiero validar autenticación Bearer Token antes de procesar peticiones SCIM | Para asegurar que solo clientes autorizados puedan sincronizar usuarios |
| Como **Portal Unificado** | Quiero implementar endpoint SCIM 2.0 estándar | Para garantizar compatibilidad con servidores SCIM comerciales |
| Como **Portal Unificado** | Quiero aislar peticiones por tenant | Para que cada cliente solo pueda gestionar sus propios usuarios |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Estructura de URL del endpoint | Cliente configura servidor SCIM AD | Cliente obtiene URL del endpoint | Sistema proporciona URL: `https://portal.efac.com/scim/v2/{tenant_id}/Users` donde {tenant_id} es el UUID único del cliente. Ejemplo: `https://portal.efac.com/scim/v2/a1b2c3d4.../Users` | ☐ | ☐ | ☐ |
| 2 | Autenticación con Bearer Token | Servidor SCIM envía petición al endpoint | Petición incluye header `Authorization: Bearer {token}` | Sistema valida: (1) Header Authorization presente, (2) Token coincide con hash almacenado para el tenant_id, (3) Tenant tiene integración AD activa. Si válido: procesa petición. Si inválido: devuelve HTTP 401 con JSON SCIM: `{"schemas": ["urn:ietf:params:scim:api:messages:2.0:Error"], "status": "401", "detail": "Authentication failed"}` | ☐ | ☐ | ☐ |
| 3 | Validación de tenant_id en URL | Petición llega con tenant_id en URL | Sistema valida tenant | Sistema verifica: (1) tenant_id es UUID válido, (2) Tenant existe en BD, (3) Integración AD está activa. Si inválido: HTTP 404 `{"status": "404", "detail": "Tenant not found or AD integration disabled"}`. Si válido: continúa procesamiento | ☐ | ☐ | ☐ |
| 4 | Aislamiento de datos por tenant | Servidor SCIM opera sobre endpoint | Sistema procesa peticiones | Sistema ASEGURA que todas las operaciones (crear, leer, actualizar, eliminar usuarios) se limitan únicamente a usuarios del tenant especificado en la URL. Validación adicional: Bearer Token debe pertenecer al mismo tenant_id de la URL | ☐ | ☐ | ☐ |
| 5 | Formato de respuesta SCIM estándar | Endpoint procesa petición exitosa | Sistema devuelve respuesta | Sistema devuelve JSON SCIM 2.0 con: Header `Content-Type: application/scim+json`, schemas apropiados (urn:ietf:params:scim:schemas:core:2.0:User), atributos del recurso (id, userName, name, emails, active, groups) | ☐ | ☐ | ☐ |
| 6 | ServiceProviderConfig - Descubrimiento de capacidades | Cliente consulta `/scim/v2/{tenant_id}/ServiceProviderConfig` | Cliente obtiene configuración | Sistema devuelve JSON SCIM con capacidades soportadas: `{"patch": {"supported": true}, "bulk": {"supported": false}, "filter": {"supported": true, "maxResults": 200}, "changePassword": {"supported": false}, "sort": {"supported": false}, "etag": {"supported": false}}` | ☐ | ☐ | ☐ |
| 7 | ResourceTypes - Tipos de recursos soportados | Cliente consulta `/scim/v2/{tenant_id}/ResourceTypes` | Cliente obtiene tipos | Sistema devuelve lista: `[{"schema": "urn:ietf:params:scim:schemas:core:2.0:ResourceType", "id": "User", "name": "User", "endpoint": "/Users", "description": "User Account"}]` (solo Users, no Groups como recurso independiente) | ☐ | ☐ | ☐ |
| 8 | Schemas - Esquemas soportados | Cliente consulta `/scim/v2/{tenant_id}/Schemas` | Cliente obtiene esquemas | Sistema devuelve esquema de User con atributos soportados: id, externalId, userName, name (givenName, familyName), emails, active, groups (multivalued). Indica atributos obligatorios (userName, active) | ☐ | ☐ | ☐ |
| 9 | Error por Content-Type incorrecto | Cliente envía petición sin `Content-Type: application/scim+json` | Sistema valida headers | Sistema devuelve HTTP 400 `{"status": "400", "detail": "Content-Type must be application/scim+json"}`. No procesa petición | ☐ | ☐ | ☐ |
| 10 | Error por método HTTP no soportado | Cliente envía petición con método no implementado (ej: OPTIONS) | Sistema valida método | Sistema devuelve HTTP 405 `{"status": "405", "detail": "Method not allowed"}` | ☐ | ☐ | ☐ |
| 11 | Rate limiting básico | Cliente envía > 100 peticiones por minuto | Sistema aplica límite | Sistema devuelve HTTP 429 `{"status": "429", "detail": "Too many requests. Rate limit: 100 req/min"}`, incluye header `Retry-After: 60`. Registra en auditoría | ☐ | ☐ | ☐ |
| 12 | Logs de peticiones SCIM | Cualquier petición llega al endpoint | Sistema registra | Sistema registra en log técnico (no auditoría funcional): timestamp, tenant_id, método HTTP, URL, status code, tiempo de respuesta, IP origen. Para debugging y monitoreo | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 13 | Estructura obligatoria de auditoría | Eventos de autenticación y seguridad del endpoint | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_SCIM_*), Fecha/Hora (UTC ISO 8601), Usuario (NULL - petición de servidor), Cliente (tenant_id), IP Local (NULL), IP Pública (IP del servidor AD), Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 14 | Auditoría de autenticación fallida | Bearer Token inválido o expirado | Sistema rechaza | Registro: Tipo "INTEGRACION_AD_SCIM_AUTH_FALLIDA", Resultado "FALLIDO", Severidad "WARNING", Descripción "Intento de autenticación SCIM fallido para tenant {tenant_id}", Datos Adicionales: {"tenant_id": "UUID", "ip_origen": "203.0.113.5", "razon": "Token inválido"} | ☐ | ☐ | ☐ |
| 15 | Auditoría de tenant inválido | tenant_id no existe o sin integración AD | Sistema rechaza | Registro: Tipo "INTEGRACION_AD_SCIM_TENANT_INVALIDO", Resultado "FALLIDO", Severidad "WARNING", Descripción "Petición SCIM a tenant inexistente o sin AD activo", Datos Adicionales: {"tenant_id": "UUID", "ip_origen": "203.0.113.5"} | ☐ | ☐ | ☐ |
| 16 | Auditoría de rate limit excedido | Cliente excede límite de peticiones | Sistema bloquea | Registro: Tipo "INTEGRACION_AD_SCIM_RATE_LIMIT", Resultado "FALLIDO", Severidad "WARNING", Descripción "Rate limit excedido para tenant {tenant_id}", Datos Adicionales: {"tenant_id": "UUID", "peticiones_por_minuto": 150, "limite": 100} | ☐ | ☐ | ☐ |
| 17 | Auditoría de error de formato | JSON malformado o Content-Type incorrecto | Sistema rechaza | Registro: Tipo "INTEGRACION_AD_SCIM_ERROR_FORMATO", Resultado "FALLIDO", Severidad "INFO", Descripción "Petición SCIM con formato inválido", Datos Adicionales: {"tenant_id": "UUID", "error": "Content-Type incorrecto", "content_type_recibido": "application/json"} | ☐ | ☐ | ☐ |
| 18 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, tenant, evento, resultado, IP origen. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: N/A - Endpoint API sin interfaz gráfica**

**Documentación API:**

#### **Endpoint Base**
```
Base URL: https://portal.efac.com/scim/v2/{tenant_id}
```

#### **Recursos Disponibles**
- `GET /ServiceProviderConfig` - Capacidades del servidor
- `GET /ResourceTypes` - Tipos de recursos soportados
- `GET /Schemas` - Esquemas disponibles
- Operaciones sobre `/Users` (ver HU-AD002B, C, D)

#### **Autenticación**
```http
Authorization: Bearer {token_generado_en_HU-AD001A}
Content-Type: application/scim+json
```

#### **Ejemplo de Error**
```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:Error"],
  "status": "401",
  "detail": "Authentication failed"
}
```

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Ataques de fuerza bruta en Bearer Token** | Atacante intenta adivinar tokens mediante peticiones masivas | Alto | Media | Rate limiting estricto (100 req/min), tokens seguros (UUID + SHA-256), registro de intentos fallidos en auditoría, bloqueo temporal tras N intentos | Mitigado |
| **Fuga de datos entre tenants** | Bug en validación de tenant_id podría exponer datos de otros clientes | Crítico | Baja | Validación doble: (1) tenant_id en URL, (2) token pertenece al mismo tenant. Testing exhaustivo de aislamiento. Auditoría de todas las operaciones | Mitigado |
| **Endpoint público vulnerable a DDoS** | Atacante satura endpoint con peticiones | Alto | Media | Rate limiting, WAF, CDN con protección DDoS, monitoreo de tráfico anómalo | Mitigado |
| **Incompatibilidad con servidores SCIM comerciales** | Implementación no estándar causa errores en clientes | Alto | Baja | Implementar estrictamente RFC 7644, testing con Azure AD SCIM, testing con servidores SCIM de referencia | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance

- Operaciones CRUD específicas sobre `/Users` → **HU-AD002B, C, D**
- Bulk operations (crear/actualizar múltiples usuarios en una petición) → Fuera de alcance (no soportado según ServiceProviderConfig)
- Filtros avanzados SCIM → Soporte básico solamente
- Gestión de `/Groups` como recurso independiente → Grupos se sincronizan como atributo de User
- ETags para control de concurrencia → No soportado

### Dependencias

**Depende de:**
- **HU-AD001A - Configuración Cliente**: Bearer Token debe estar generado
- **HU-AD001B - Catálogo de Roles**: Valida roles recibidos en atributo `groups`

**Es prerequisito para:**
- **HU-AD002B - Operación POST /Users**: Usa autenticación y estructura base
- **HU-AD002C - Operación GET /Users**: Usa autenticación y estructura base
- **HU-AD002D - Operaciones PUT, PATCH, DELETE /Users**: Usa autenticación y estructura base

### Valor Independiente

- ✅ Establece infraestructura base de endpoint SCIM
- ✅ Implementa autenticación y seguridad
- ✅ Proporciona descubrimiento de capacidades (ServiceProviderConfig)
- ✅ Puede probarse con herramientas SCIM estándar
- ✅ Entrega valor antes de implementar operaciones CRUD completas

### Especificación Técnica

**Métodos HTTP Soportados:**
- GET, POST, PUT, PATCH, DELETE (sobre /Users)
- GET (sobre /ServiceProviderConfig, /ResourceTypes, /Schemas)

**Headers Requeridos:**
```http
Authorization: Bearer {token}
Content-Type: application/scim+json
```

**Headers de Respuesta:**
```http
Content-Type: application/scim+json
```

**Rate Limiting:**
- 100 peticiones por minuto por tenant
- HTTP 429 + header `Retry-After` si se excede

**Códigos de Estado HTTP:**
- 200 OK - Operación exitosa
- 201 Created - Recurso creado
- 204 No Content - Recurso eliminado
- 400 Bad Request - JSON malformado o parámetros inválidos
- 401 Unauthorized - Bearer Token inválido
- 404 Not Found - Recurso o tenant no existe
- 405 Method Not Allowed - Método HTTP no soportado
- 409 Conflict - Recurso ya existe (userName duplicado)
- 429 Too Many Requests - Rate limit excedido
- 500 Internal Server Error - Error del servidor

### Referencias

- **RFC 7644 - SCIM Protocol**: https://datatracker.ietf.org/doc/html/rfc7644
- **RFC 7643 - SCIM Core Schema**: https://datatracker.ietf.org/doc/html/rfc7643
- **Estándar Auditoría**: Tipos `INTEGRACION_AD_SCIM_*`
- **Glosario**: SCIM, Endpoint, Bearer Token, Tenant, Idempotente
- **Azure AD SCIM Implementation**: Referencia para testing de compatibilidad

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - Endpoint SCIM
