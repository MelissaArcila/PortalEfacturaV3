# HU-AD002B - Operación POST /Users (Crear Usuario)

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

Esta HU implementa la operación **POST /Users** del endpoint SCIM 2.0 que permite al servidor Active Directory del cliente **crear usuarios** en el Portal Unificado mediante sincronización SCIM.

Cuando un usuario se crea en el Active Directory del cliente y pertenece a los grupos configurados para sincronización, el servidor SCIM envía una petición POST con los datos del usuario al Portal.

El Portal valida los datos, crea el usuario, asigna roles basándose en los grupos (usando catálogo HU-AD001B), y marca el usuario como gestionado por AD.

---

**Notas:**
- Operación **idempotente**: si el usuario ya existe (mismo externalId), devuelve HTTP 409 Conflict
- userName debe ser **único** en el tenant
- Sincronización **unidireccional**: AD → Portal (Portal no puede editar usuarios gestionados por AD)
- Usuarios creados por SCIM tienen flag `gestionado_por_ad=true`
- Roles se asignan automáticamente según grupos recibidos (validados contra catálogo)

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Servidor SCIM del Cliente AD** | Quiero enviar petición POST al endpoint /Users con datos del usuario | Para crear automáticamente usuarios del AD en el Portal |
| Como **Portal Unificado** | Quiero validar datos del usuario antes de crearlo | Para garantizar integridad de datos y evitar duplicados |
| Como **Portal Unificado** | Quiero asignar roles automáticamente según grupos recibidos | Para que usuarios tengan permisos correctos basados en su configuración AD |
| Como **Portal Unificado** | Quiero marcar usuarios como gestionados por AD | Para prevenir ediciones manuales que serían sobrescritas en próxima sincronización |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Estructura de petición POST | Servidor SCIM AD crea usuario | Envía POST `https://portal.efac.com/scim/v2/{tenant_id}/Users` | Sistema recibe JSON SCIM con atributos obligatorios: `userName` (email corporativo), `name` (givenName, familyName), `active` (true/false), `externalId` (GUID del AD), `emails` (array), opcional: `groups` (array de nombres de grupos AD). Header `Authorization: Bearer {token}`, `Content-Type: application/scim+json` | ☐ | ☐ | ☐ |
| 2 | Validación de campos obligatorios | Sistema recibe POST | Sistema valida payload | Sistema verifica: (1) `userName` presente y no vacío, (2) `active` presente (booleano), (3) `externalId` presente (GUID). Si falta alguno: HTTP 400 `{"status": "400", "scimType": "invalidValue", "detail": "Missing required attribute: {atributo}"}` | ☐ | ☐ | ☐ |
| 3 | Validación de userName único | Sistema valida userName | Sistema consulta BD | Sistema verifica que `userName` no exista en el tenant. Si existe: HTTP 409 `{"status": "409", "scimType": "uniqueness", "detail": "userName already exists"}`. Si es único: continúa creación | ☐ | ☐ | ☐ |
| 4 | Validación de externalId único | Sistema valida externalId | Sistema consulta BD | Sistema verifica que `externalId` (GUID del AD) no exista en el tenant. Si existe: HTTP 409 `{"status": "409", "scimType": "uniqueness", "detail": "User with this externalId already exists"}` (indica que usuario ya fue sincronizado) | ☐ | ☐ | ☐ |
| 5 | Creación exitosa de usuario sin grupos | Servidor SCIM envía usuario sin atributo `groups` | Sistema procesa creación | Sistema crea usuario en BD: genera UUID interno, almacena userName, name, emails, active, externalId, tenant_id, marca `gestionado_por_ad=true`, `roles=[]` (vacío). Devuelve HTTP 201 con JSON SCIM del usuario creado incluyendo `id` (UUID asignado), `meta` (resourceType, created, lastModified, location). Registra en auditoría | ☐ | ☐ | ☐ |
| 6 | Creación con grupos válidos - Asignación de roles | Servidor envía usuario con `groups: ["Administrador del Portal", "Contador"]` | Sistema procesa grupos | Sistema: (1) Valida cada grupo contra catálogo de roles (HU-AD001B, case-sensitive), (2) Si grupo existe en catálogo: asigna rol al usuario, (3) Si grupo NO existe: registra warning en auditoría, ignora grupo. Usuario creado con roles asignados: `roles: ["Administrador del Portal", "Contador"]`. Devuelve HTTP 201 | ☐ | ☐ | ☐ |
| 7 | Creación con grupos inválidos | Servidor envía `groups: ["administrador", "Grupo Inexistente"]` | Sistema valida grupos | Sistema valida contra catálogo: ambos nombres NO coinciden (case-sensitive). Sistema crea usuario pero NO asigna ningún rol (`roles=[]`), registra warning en auditoría "Grupos no reconocidos: 'administrador', 'Grupo Inexistente'", devuelve HTTP 201 con usuario sin roles | ☐ | ☐ | ☐ |
| 8 | Normalización de atributo active | Servidor envía `active: false` | Sistema procesa usuario | Sistema crea usuario pero con estado `active=false` (inactivo). Usuario existe en BD pero no puede autenticarse. Útil para sincronizar usuarios deshabilitados en AD. Devuelve HTTP 201 | ☐ | ☐ | ☐ |
| 9 | Respuesta SCIM estándar con meta | Usuario creado exitosamente | Sistema devuelve respuesta | Sistema devuelve HTTP 201 con JSON SCIM: `{"schemas": [...], "id": "UUID", "externalId": "GUID-AD", "userName": "user@empresa.com", "name": {...}, "emails": [...], "active": true, "groups": [...], "meta": {"resourceType": "User", "created": "2026-01-20T10:30:00Z", "lastModified": "2026-01-20T10:30:00Z", "location": "https://portal.efac.com/scim/v2/{tenant_id}/Users/{id}"}}` | ☐ | ☐ | ☐ |
| 10 | Error por JSON malformado | Servidor envía JSON inválido | Sistema parsea payload | Sistema detecta error de sintaxis JSON, devuelve HTTP 400 `{"status": "400", "detail": "Invalid JSON syntax"}`. No crea usuario | ☐ | ☐ | ☐ |
| 11 | Error por schema SCIM incorrecto | Servidor envía JSON sin schema SCIM o schema inválido | Sistema valida schema | Sistema verifica presencia de `schemas: ["urn:ietf:params:scim:schemas:core:2.0:User"]`. Si falta o es incorrecto: HTTP 400 `{"status": "400", "detail": "Invalid or missing SCIM schema"}` | ☐ | ☐ | ☐ |
| 12 | Idempotencia - Petición duplicada | Servidor envía POST duplicado (mismo externalId) | Sistema detecta duplicado | Sistema devuelve HTTP 409 Conflict (no crea duplicado). Cliente debe usar PUT/PATCH para actualizar usuario existente | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 13 | Estructura obligatoria de auditoría | Eventos de creación de usuarios SCIM | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_USUARIO_*), Fecha/Hora (UTC ISO 8601), Usuario (NULL - petición de servidor), Cliente (tenant_id), IP Local (NULL), IP Pública (IP del servidor AD), Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 14 | Auditoría de usuario creado exitosamente | Sistema crea usuario | Creación exitosa | Registro: Tipo "INTEGRACION_AD_USUARIO_CREADO", Resultado "EXITOSO", Severidad "INFO", Descripción "Usuario {userName} creado desde AD para tenant {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "user_id": "UUID", "userName": "juan.perez@empresa.com", "externalId": "GUID-AD", "roles_asignados": ["Administrador del Portal"], "active": true} | ☐ | ☐ | ☐ |
| 15 | Auditoría de usuario creado sin roles | Usuario creado sin grupos válidos | Sistema crea usuario vacío | Registro: Tipo "INTEGRACION_AD_USUARIO_CREADO_SIN_ROLES", Resultado "EXITOSO", Severidad "WARNING", Descripción "Usuario {userName} creado sin roles (grupos AD no reconocidos)", Datos Adicionales: {"tenant_id": "UUID", "user_id": "UUID", "userName": "juan.perez@empresa.com", "grupos_recibidos": ["administrador", "Grupo X"], "grupos_no_reconocidos": ["administrador", "Grupo X"]} | ☐ | ☐ | ☐ |
| 16 | Auditoría de userName duplicado | Sistema detecta userName existente | Creación rechazada | Registro: Tipo "INTEGRACION_AD_USUARIO_DUPLICADO", Resultado "FALLIDO", Severidad "WARNING", Descripción "Intento de crear usuario con userName duplicado", Datos Adicionales: {"tenant_id": "UUID", "userName": "existente@empresa.com", "user_id_existente": "UUID"} | ☐ | ☐ | ☐ |
| 17 | Auditoría de validación fallida | Faltan campos obligatorios o formato inválido | Creación rechazada | Registro: Tipo "INTEGRACION_AD_USUARIO_VALIDACION_FALLIDA", Resultado "FALLIDO", Severidad "INFO", Descripción "Petición POST /Users inválida", Datos Adicionales: {"tenant_id": "UUID", "error": "Missing required attribute: userName"} | ☐ | ☐ | ☐ |
| 18 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, tenant, evento, resultado, userName. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: N/A - Operación API sin interfaz gráfica**

**Documentación API:**

#### **Endpoint**
```
POST https://portal.efac.com/scim/v2/{tenant_id}/Users
```

#### **Headers**
```http
Authorization: Bearer {token}
Content-Type: application/scim+json
```

#### **Ejemplo de Payload (Request)**
```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
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
    },
    {
      "value": "Contador",
      "display": "Contador"
    }
  ]
}
```

#### **Ejemplo de Respuesta (Response) - HTTP 201**
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
    },
    {
      "value": "Contador",
      "display": "Contador"
    }
  ],
  "meta": {
    "resourceType": "User",
    "created": "2026-01-20T10:30:00Z",
    "lastModified": "2026-01-20T10:30:00Z",
    "location": "https://portal.efac.com/scim/v2/abc123/Users/f1e2d3c4-b5a6-7890-cdef-1234567890ab"
  }
}
```

#### **Ejemplo de Error - HTTP 409 (Duplicado)**
```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:Error"],
  "status": "409",
  "scimType": "uniqueness",
  "detail": "userName already exists"
}
```

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Usuarios creados sin roles** | Si todos los grupos AD son inválidos, usuarios quedan sin permisos | Medio | Media | Registrar warning en auditoría, dashboard para administradores mostrando usuarios sin roles, documentación clara del catálogo para clientes | Mitigado |
| **userName duplicado causa pérdida de sincronización** | Si userName ya existe, creación falla y AD no reintenta | Medio | Baja | Devolver HTTP 409 claro, registrar en auditoría, documentar que cliente debe usar externalId único y userName único | Mitigado |
| **Atributo active malinterpretado** | Cliente envía active=false sin intención, usuarios quedan bloqueados | Bajo | Baja | Documentar claramente significado de active, validar tipo booleano | Aceptado |

---

## Notas Adicionales

### Fuera de Alcance

- Actualización de usuarios existentes → **HU-AD002D - PUT/PATCH /Users**
- Consulta de usuarios → **HU-AD002C - GET /Users**
- Eliminación de usuarios → **HU-AD002D - DELETE /Users**
- Validación de formato de email → Validación básica solamente
- Creación masiva (bulk) → No soportado según ServiceProviderConfig

### Dependencias

**Depende de:**
- **HU-AD002A - Endpoint SCIM Base**: Usa autenticación y estructura base
- **HU-AD001B - Catálogo de Roles**: Valida grupos contra catálogo
- **Gestión de Usuarios del Portal**: BD de usuarios debe existir

**Relacionada con:**
- **HU-AD003A - Sincronización Inicial**: Usa POST masivamente para crear usuarios
- **HU-AD002D - PUT/PATCH**: Para actualizar usuarios creados por POST

### Valor Independiente

- ✅ Permite creación de usuarios desde AD
- ✅ Asigna roles automáticamente
- ✅ Puede probarse con herramientas SCIM o Postman
- ✅ Entrega valor antes de implementar UPDATE/DELETE
- ✅ Suficiente para sincronización inicial de usuarios

### Especificación Técnica

**Atributos SCIM Soportados:**

| Atributo | Tipo | Obligatorio | Descripción |
|----------|------|-------------|-------------|
| schemas | array[string] | Sí | Debe incluir "urn:ietf:params:scim:schemas:core:2.0:User" |
| externalId | string (UUID) | Sí | GUID del usuario en Active Directory |
| userName | string (email) | Sí | Email corporativo, debe ser único en tenant |
| name.givenName | string | No | Nombre(s) |
| name.familyName | string | No | Apellido(s) |
| emails | array[object] | No | Emails del usuario |
| active | boolean | Sí | true=activo, false=inactivo |
| groups | array[object] | No | Grupos/roles del AD |

**Modelo de Datos (Adición a tabla users):**

```sql
ALTER TABLE users ADD COLUMN gestionado_por_ad BOOLEAN DEFAULT false;
ALTER TABLE users ADD COLUMN external_id UUID; -- GUID del AD
ALTER TABLE users ADD COLUMN tenant_id UUID REFERENCES tenants(id);
CREATE UNIQUE INDEX idx_users_tenant_username ON users(tenant_id, userName);
CREATE UNIQUE INDEX idx_users_tenant_externalid ON users(tenant_id, external_id);
```

### Referencias

- **RFC 7644 Section 3.3 - Creating Resources**: https://datatracker.ietf.org/doc/html/rfc7644#section-3.3
- **RFC 7643 Section 4 - User Resource Schema**: https://datatracker.ietf.org/doc/html/rfc7643#section-4
- **Estándar Auditoría**: Tipos `INTEGRACION_AD_USUARIO_*`
- **Glosario**: externalId, userName, active, groups, idempotente

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - Endpoint SCIM
