# HU-AD002C - Operación GET /Users (Consultar Usuarios)

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
| **Arquitectura relacionada** | Portal Unificado - API SCIM, Gestión de Usuarios |

---

## Contexto

Esta HU implementa las operaciones **GET /Users** del endpoint SCIM 2.0 que permiten al servidor Active Directory del cliente **consultar usuarios** sincronizados en el Portal Unificado.

Soporta dos modos:
1. **GET /Users/{id}** - Consultar un usuario específico por su UUID
2. **GET /Users?filter=...** - Listar usuarios con filtros básicos

Estas operaciones son necesarias para que el servidor SCIM del cliente pueda:
- Verificar si un usuario ya existe antes de crearlo
- Consultar el estado actual de usuarios sincronizados
- Validar que la sincronización fue exitosa

---

**Notas:**
- Solo devuelve usuarios del tenant especificado en la URL (aislamiento)
- Solo devuelve usuarios con `gestionado_por_ad=true`
- Soporta filtros básicos: `userName eq "..."`, `externalId eq "..."`
- Paginación: max 200 resultados por petición
- No soporta ordenamiento (sort) ni filtros complejos

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Servidor SCIM del Cliente AD** | Quiero consultar un usuario por su UUID | Para verificar el estado actual de un usuario sincronizado |
| Como **Servidor SCIM del Cliente AD** | Quiero buscar un usuario por userName o externalId | Para verificar si el usuario ya existe antes de crearlo |
| Como **Servidor SCIM del Cliente AD** | Quiero listar todos los usuarios sincronizados | Para validar que la sincronización masiva fue exitosa |
| Como **Portal Unificado** | Quiero devolver solo usuarios del tenant autenticado | Para mantener aislamiento de datos entre clientes |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | GET de usuario específico por UUID | Cliente conoce UUID del usuario | Envía GET `https://portal.efac.com/scim/v2/{tenant_id}/Users/{id}` | Sistema valida: (1) {id} es UUID válido, (2) Usuario existe, (3) Usuario pertenece al tenant, (4) Usuario tiene `gestionado_por_ad=true`. Si válido: devuelve HTTP 200 con JSON SCIM del usuario (id, externalId, userName, name, emails, active, groups, meta). Si no existe: HTTP 404 `{"status": "404", "detail": "User not found"}` | ☐ | ☐ | ☐ |
| 2 | GET de usuario de otro tenant | Cliente intenta acceder a usuario de otro tenant | Sistema valida tenant_id | Sistema verifica que usuario pertenece al tenant de la URL. Si NO pertenece: HTTP 404 (no revela que el usuario existe en otro tenant por seguridad) | ☐ | ☐ | ☐ |
| 3 | GET de usuario gestionado manualmente (no AD) | Cliente solicita usuario con `gestionado_por_ad=false` | Sistema valida flag | Sistema devuelve HTTP 404 (usuarios manuales no están expuestos en endpoint SCIM) | ☐ | ☐ | ☐ |
| 4 | Listado de todos los usuarios sin filtros | Cliente envía GET `https://portal.efac.com/scim/v2/{tenant_id}/Users` | Sistema lista usuarios | Sistema devuelve HTTP 200 con JSON SCIM ListResponse: `{"schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"], "totalResults": N, "startIndex": 1, "itemsPerPage": N, "Resources": [{usuario1}, {usuario2}, ...]}`. Solo incluye usuarios con `gestionado_por_ad=true` del tenant | ☐ | ☐ | ☐ |
| 5 | Paginación - Más de 200 usuarios | Tenant tiene más de 200 usuarios | Sistema pagina resultados | Sistema devuelve primeros 200 usuarios, incluye atributo `totalResults` con total real. Cliente puede solicitar siguiente página con `?startIndex=201&count=200`. Respuesta incluye `startIndex` y `itemsPerPage` actuales | ☐ | ☐ | ☐ |
| 6 | Filtro por userName exacto | Cliente busca usuario específico | Envía GET `https://portal.efac.com/scim/v2/{tenant_id}/Users?filter=userName eq "juan.perez@empresa.com"` | Sistema busca usuario con ese userName (case-insensitive por estándar email). Si existe: devuelve ListResponse con 1 recurso. Si no existe: devuelve `totalResults: 0, Resources: []` | ☐ | ☐ | ☐ |
| 7 | Filtro por externalId exacto | Cliente busca por GUID del AD | Envía GET `.../Users?filter=externalId eq "a1b2c3d4-..."` | Sistema busca usuario con ese externalId. Si existe: devuelve ListResponse con 1 recurso. Si no existe: devuelve totalResults: 0 | ☐ | ☐ | ☐ |
| 8 | Filtro inválido o no soportado | Cliente envía filtro complejo no soportado | Envía `.../Users?filter=name.givenName co "Juan"` (contains) | Sistema devuelve HTTP 400 `{"status": "400", "scimType": "invalidFilter", "detail": "Filter not supported. Only 'eq' operator on userName and externalId"}` | ☐ | ☐ | ☐ |
| 9 | Respuesta incluye atributo groups | Usuario tiene roles asignados | Sistema construye respuesta | Sistema incluye atributo `groups` en respuesta con array de roles del usuario: `"groups": [{"value": "Administrador del Portal", "display": "Administrador del Portal"}, ...]` (mapea roles internos a formato SCIM groups) | ☐ | ☐ | ☐ |
| 10 | Respuesta incluye meta con lastModified | Usuario fue modificado | Sistema incluye timestamps | Sistema incluye en `meta`: `{"resourceType": "User", "created": "2026-01-15T10:00:00Z", "lastModified": "2026-01-20T14:30:00Z", "location": "https://..."}`. Cliente puede usar lastModified para detectar cambios | ☐ | ☐ | ☐ |
| 11 | Listado vacío - Sin usuarios sincronizados | Tenant sin usuarios de AD | Sistema lista | Sistema devuelve HTTP 200 `{"schemas": [...], "totalResults": 0, "startIndex": 1, "itemsPerPage": 0, "Resources": []}` | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 12 | Estructura obligatoria de auditoría | Consultas GET a endpoint | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_CONSULTA_*), Fecha/Hora (UTC ISO 8601), Usuario (NULL), Cliente (tenant_id), IP Local (NULL), IP Pública (IP servidor AD), Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 13 | Auditoría de consulta exitosa por UUID | Cliente consulta usuario específico | GET /Users/{id} exitoso | Registro: Tipo "INTEGRACION_AD_CONSULTA_USUARIO", Resultado "EXITOSO", Severidad "INFO", Descripción "Consulta SCIM de usuario {userName}", Datos Adicionales: {"tenant_id": "UUID", "user_id": "UUID", "userName": "juan.perez@empresa.com"} | ☐ | ☐ | ☐ |
| 14 | Auditoría de listado de usuarios | Cliente lista todos los usuarios | GET /Users exitoso | Registro: Tipo "INTEGRACION_AD_CONSULTA_LISTADO", Resultado "EXITOSO", Severidad "INFO", Descripción "Listado SCIM de usuarios para tenant {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "totalResults": 45, "filtro_aplicado": null} | ☐ | ☐ | ☐ |
| 15 | Auditoría de filtro aplicado | Cliente usa filtro en consulta | GET con filter exitoso | Registro: Tipo "INTEGRACION_AD_CONSULTA_FILTRADA", Resultado "EXITOSO", Severidad "INFO", Descripción "Consulta SCIM con filtro aplicado", Datos Adicionales: {"tenant_id": "UUID", "filtro": "userName eq \"juan.perez@empresa.com\"", "resultados": 1} | ☐ | ☐ | ☐ |
| 16 | Auditoría de usuario no encontrado | Cliente consulta UUID inexistente | GET devuelve 404 | Registro: Tipo "INTEGRACION_AD_CONSULTA_NO_ENCONTRADO", Resultado "FALLIDO", Severidad "INFO", Descripción "Usuario solicitado no encontrado", Datos Adicionales: {"tenant_id": "UUID", "user_id_solicitado": "UUID"} | ☐ | ☐ | ☐ |
| 17 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, tenant, evento, resultado. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: N/A - Operación API sin interfaz gráfica**

**Documentación API:**

#### **Endpoints**
```
GET https://portal.efac.com/scim/v2/{tenant_id}/Users/{id}
GET https://portal.efac.com/scim/v2/{tenant_id}/Users
GET https://portal.efac.com/scim/v2/{tenant_id}/Users?filter={expression}
GET https://portal.efac.com/scim/v2/{tenant_id}/Users?startIndex=N&count=M
```

#### **Headers**
```http
Authorization: Bearer {token}
```

#### **Ejemplo 1: GET Usuario por UUID**

**Request:**
```
GET https://portal.efac.com/scim/v2/abc123/Users/f1e2d3c4-b5a6-7890-cdef-1234567890ab
Authorization: Bearer xyz789token
```

**Response: HTTP 200**
```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
  "id": "f1e2d3c4-b5a6-7890-cdef-1234567890ab",
  "externalId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "userName": "juan.perez@empresa.com",
  "name": {
    "givenName": "Juan",
    "familyName": "Pérez"
  },
  "emails": [
    {
      "value": "juan.perez@empresa.com",
      "type": "work",
      "primary": true
    }
  ],
  "active": true,
  "groups": [
    {
      "value": "Administrador del Portal",
      "display": "Administrador del Portal"
    }
  ],
  "meta": {
    "resourceType": "User",
    "created": "2026-01-15T10:00:00Z",
    "lastModified": "2026-01-20T14:30:00Z",
    "location": "https://portal.efac.com/scim/v2/abc123/Users/f1e2d3c4-b5a6-7890-cdef-1234567890ab"
  }
}
```

#### **Ejemplo 2: Listado con Filtro**

**Request:**
```
GET https://portal.efac.com/scim/v2/abc123/Users?filter=userName eq "juan.perez@empresa.com"
Authorization: Bearer xyz789token
```

**Response: HTTP 200**
```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
  "totalResults": 1,
  "startIndex": 1,
  "itemsPerPage": 1,
  "Resources": [
    {
      "id": "f1e2d3c4-b5a6-7890-cdef-1234567890ab",
      "externalId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "userName": "juan.perez@empresa.com",
      "name": {...},
      "active": true,
      "groups": [...],
      "meta": {...}
    }
  ]
}
```

#### **Ejemplo 3: Listado con Paginación**

**Request:**
```
GET https://portal.efac.com/scim/v2/abc123/Users?startIndex=201&count=100
```

**Response: HTTP 200**
```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
  "totalResults": 350,
  "startIndex": 201,
  "itemsPerPage": 100,
  "Resources": [...]
}
```

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Exposición de usuarios de otros tenants** | Bug en validación podría exponer usuarios de otros clientes | Crítico | Baja | Validación estricta de tenant_id en todas las consultas, testing exhaustivo de aislamiento | Mitigado |
| **Performance con listados grandes** | Tenants con miles de usuarios podrían causar lentitud | Medio | Media | Paginación obligatoria (max 200), índices en BD, limit en consultas | Mitigado |
| **Filtros complejos causan errores** | Clientes SCIM avanzados podrían enviar filtros no soportados | Bajo | Media | Validar filtros soportados, devolver HTTP 400 claro con mensaje descriptivo | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance

- Filtros complejos (operadores: `co`, `sw`, `pr`, `gt`, `lt`, `and`, `or`) → Solo `eq` soportado
- Ordenamiento (sort) → No soportado según ServiceProviderConfig
- Selección de atributos (attributes, excludedAttributes) → Devuelve todos los atributos
- ETags para control de concurrencia → No soportado

### Dependencias

**Depende de:**
- **HU-AD002A - Endpoint SCIM Base**: Usa autenticación y estructura
- **HU-AD002B - POST /Users**: Usuarios deben existir para consultarlos

**Relacionada con:**
- **HU-AD003A - Sincronización Inicial**: Usa GET para verificar usuarios existentes antes de crearlos
- **HU-AD002D - PUT/PATCH**: Cliente puede consultar usuario antes de actualizarlo

### Valor Independiente

- ✅ Permite verificar usuarios sincronizados
- ✅ Esencial para validación de sincronizaciones
- ✅ Puede probarse independientemente con usuarios creados manualmente
- ✅ Complementa POST para flujo completo de creación

### Especificación Técnica

**Filtros Soportados:**

| Filtro | Ejemplo | Descripción |
|--------|---------|-------------|
| userName eq | `userName eq "user@empresa.com"` | Búsqueda exacta por userName (case-insensitive) |
| externalId eq | `externalId eq "a1b2-..."` | Búsqueda exacta por GUID del AD |

**Parámetros de Paginación:**

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| startIndex | integer | 1 | Índice del primer resultado (base 1, no 0) |
| count | integer | 200 | Número de resultados por página (max 200) |

**Índices de BD Requeridos:**
```sql
CREATE INDEX idx_users_tenant_username ON users(tenant_id, userName);
CREATE INDEX idx_users_tenant_externalid ON users(tenant_id, external_id);
CREATE INDEX idx_users_gestionado_ad ON users(gestionado_por_ad) WHERE gestionado_por_ad = true;
```

### Referencias

- **RFC 7644 Section 3.4 - Retrieving Resources**: https://datatracker.ietf.org/doc/html/rfc7644#section-3.4
- **RFC 7644 Section 3.4.2 - Filtering**: https://datatracker.ietf.org/doc/html/rfc7644#section-3.4.2
- **RFC 7644 Section 3.4.2.4 - Pagination**: https://datatracker.ietf.org/doc/html/rfc7644#section-3.4.2.4
- **Estándar Auditoría**: Tipos `INTEGRACION_AD_CONSULTA_*`
- **Glosario**: ListResponse, startIndex, itemsPerPage, filter

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - Endpoint SCIM
