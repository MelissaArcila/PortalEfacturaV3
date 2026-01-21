# HU-AD002D - Operaciones PUT, PATCH, DELETE /Users

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

Esta HU implementa las operaciones de **actualización y eliminación** de usuarios en el endpoint SCIM 2.0:
- **PUT /Users/{id}** - Reemplazo completo del usuario
- **PATCH /Users/{id}** - Actualización parcial de atributos específicos
- **DELETE /Users/{id}** - Eliminación lógica (soft delete) del usuario

Estas operaciones permiten al servidor Active Directory mantener sincronizados los datos cuando:
- Un usuario cambia de nombre, email, o grupos en AD → PUT/PATCH
- Un usuario es deshabilitado en AD → PATCH con `active: false`
- Un usuario es eliminado de AD → DELETE (soft delete en Portal)

---

**Notas:**
- Solo usuarios con `gestionado_por_ad=true` pueden ser actualizados/eliminados por SCIM
- DELETE es **soft delete**: marca usuario como eliminado pero NO borra físicamente
- PUT reemplaza **todos** los atributos (requiere enviar datos completos)
- PATCH actualiza **solo** los atributos especificados
- Cambios en `groups` actualizan roles del usuario automáticamente

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Servidor SCIM del Cliente AD** | Quiero actualizar datos de un usuario cuando cambien en AD | Para mantener sincronizados nombre, email, grupos, y estado activo |
| Como **Servidor SCIM del Cliente AD** | Quiero eliminar usuarios que fueron borrados del AD | Para prevenir acceso de usuarios que ya no existen en AD |
| Como **Portal Unificado** | Quiero realizar soft delete en vez de eliminación física | Para mantener histórico de auditoría y preservar relaciones de datos |
| Como **Portal Unificado** | Quiero actualizar roles automáticamente cuando cambien grupos | Para reflejar cambios de permisos desde AD en tiempo real |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|-----------------|
| 1 | PUT - Reemplazo completo de usuario | Usuario existe en Portal | Servidor envía PUT `https://portal.efac.com/scim/v2/{tenant_id}/Users/{id}` con JSON completo | Sistema valida: (1) Usuario existe, (2) Pertenece al tenant, (3) `gestionado_por_ad=true`, (4) JSON incluye todos los atributos requeridos. Si válido: reemplaza TODOS los atributos del usuario, actualiza `lastModified`, recalcula roles según `groups`, devuelve HTTP 200 con usuario actualizado. Si inválido: HTTP 404 o 400 | ☐ | ☐ | ☐ |
| 2 | PUT - Validación de campos obligatorios | Servidor envía PUT sin userName | Sistema valida payload | Sistema verifica presencia de atributos obligatorios (userName, active, externalId). Si falta alguno: HTTP 400 `{"status": "400", "detail": "Missing required attribute for PUT operation"}` | ☐ | ☐ | ☐ |
| 3 | PUT - Actualización de grupos/roles | PUT cambia atributo groups | Sistema procesa cambios | Sistema: (1) Valida nuevos grupos contra catálogo de roles (HU-AD001B), (2) Elimina roles anteriores del usuario, (3) Asigna nuevos roles basados en grupos válidos, (4) Registra cambio en auditoría. Usuario ahora tiene solo los roles de la nueva lista de grupos | ☐ | ☐ | ☐ |
| 4 | PATCH - Actualización parcial de atributos | Usuario necesita cambio en email solamente | Servidor envía PATCH con `{"Operations": [{"op": "replace", "path": "emails[type eq \"work\"].value", "value": "nuevo@empresa.com"}]}` | Sistema actualiza SOLO el atributo especificado, mantiene otros sin cambios, actualiza `lastModified`, devuelve HTTP 200 con usuario completo actualizado | ☐ | ☐ | ☐ |
| 5 | PATCH - Cambio de estado activo | Usuario deshabilitado en AD | Servidor envía PATCH `{"Operations": [{"op": "replace", "path": "active", "value": false}]}` | Sistema actualiza `active=false`, invalida sesiones activas del usuario (si las hay), actualiza `lastModified`, devuelve HTTP 200. Usuario no podrá autenticarse hasta que se reactive | ☐ | ☐ | ☐ |
| 6 | PATCH - Agregar grupo (rol) | Usuario obtiene nuevo grupo en AD | Servidor envía PATCH `{"Operations": [{"op": "add", "path": "groups", "value": [{"value": "Contador", "display": "Contador"}]}]}` | Sistema: (1) Valida "Contador" contra catálogo, (2) AGREGA rol sin eliminar roles existentes, (3) Devuelve HTTP 200. Usuario ahora tiene rol adicional | ☐ | ☐ | ☐ |
| 7 | PATCH - Remover grupo (rol) | Usuario removido de grupo en AD | Servidor envía PATCH `{"Operations": [{"op": "remove", "path": "groups[value eq \"Contador\"]"}]}` | Sistema: (1) Identifica y remueve rol "Contador", (2) Mantiene otros roles, (3) Si era el último rol, usuario queda sin permisos, (4) Devuelve HTTP 200 | ☐ | ☐ | ☐ |
| 8 | PATCH - Operación inválida | Servidor envía operación no soportada | Envía PATCH con `op: "move"` (no soportado) | Sistema devuelve HTTP 400 `{"status": "400", "scimType": "invalidSyntax", "detail": "Operation 'move' not supported. Supported: add, remove, replace"}` | ☐ | ☐ | ☐ |
| 9 | DELETE - Soft delete de usuario | Usuario eliminado de AD | Servidor envía DELETE `https://portal.efac.com/scim/v2/{tenant_id}/Users/{id}` | Sistema valida: (1) Usuario existe, (2) Pertenece al tenant, (3) `gestionado_por_ad=true`. Si válido: marca usuario como eliminado (`deleted_at=timestamp`, `active=false`), invalida sesiones activas, NO borra físicamente el registro, devuelve HTTP 204 No Content. Usuario ya no aparece en GET /Users pero existe en BD | ☐ | ☐ | ☐ |
| 10 | DELETE - Usuario ya eliminado | Servidor intenta eliminar usuario ya marcado deleted | Sistema verifica estado | Sistema devuelve HTTP 404 `{"status": "404", "detail": "User not found"}` (usuarios eliminados no existen para SCIM). Operación idempotente | ☐ | ☐ | ☐ |
| 11 | Validación - Usuario NO gestionado por AD | Servidor intenta actualizar/eliminar usuario manual | Sistema valida flag | Sistema verifica `gestionado_por_ad=false`, devuelve HTTP 404 (protege usuarios creados manualmente en el Portal) | ☐ | ☐ | ☐ |
| 12 | Validación - Usuario de otro tenant | Servidor intenta modificar usuario de otro tenant | Sistema valida aislamiento | Sistema verifica tenant_id del usuario coincide con URL. Si NO coincide: HTTP 404 (no revela existencia) | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 13 | Estructura obligatoria de auditoría | Eventos de actualización/eliminación SCIM | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_USUARIO_*), Fecha/Hora (UTC ISO 8601), Usuario (NULL), Cliente (tenant_id), IP Local (NULL), IP Pública (IP servidor AD), Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 14 | Auditoría de PUT exitoso | Usuario actualizado completamente | PUT exitoso | Registro: Tipo "INTEGRACION_AD_USUARIO_ACTUALIZADO_PUT", Resultado "EXITOSO", Severidad "INFO", Descripción "Usuario {userName} actualizado (PUT) desde AD", Datos Adicionales: {"tenant_id": "UUID", "user_id": "UUID", "userName": "juan.perez@empresa.com", "cambios": {"grupos_anteriores": ["Admin"], "grupos_nuevos": ["Admin", "Contador"]}} | ☐ | ☐ | ☐ |
| 15 | Auditoría de PATCH exitoso | Usuario modificado parcialmente | PATCH exitoso | Registro: Tipo "INTEGRACION_AD_USUARIO_ACTUALIZADO_PATCH", Resultado "EXITOSO", Severidad "INFO", Descripción "Usuario {userName} modificado (PATCH) desde AD", Datos Adicionales: {"tenant_id": "UUID", "user_id": "UUID", "operaciones": [{"op": "replace", "path": "active", "value": false}]} | ☐ | ☐ | ☐ |
| 16 | Auditoría de cambio de roles | Grupos actualizados afectan roles | Sistema recalcula roles | Registro: Tipo "INTEGRACION_AD_USUARIO_ROLES_ACTUALIZADOS", Resultado "EXITOSO", Severidad "INFO", Descripción "Roles actualizados para usuario {userName}", Datos Adicionales: {"user_id": "UUID", "roles_anteriores": ["Administrador del Portal"], "roles_nuevos": ["Administrador del Portal", "Contador"]} | ☐ | ☐ | ☐ |
| 17 | Auditoría de DELETE exitoso | Usuario eliminado (soft delete) | DELETE exitoso | Registro: Tipo "INTEGRACION_AD_USUARIO_ELIMINADO", Resultado "EXITOSO", Severidad "WARNING", Descripción "Usuario {userName} eliminado (soft delete) desde AD", Datos Adicionales: {"tenant_id": "UUID", "user_id": "UUID", "userName": "juan.perez@empresa.com", "deleted_at": "2026-01-20T15:00:00Z"} | ☐ | ☐ | ☐ |
| 18 | Auditoría de operación rechazada | Usuario no gestionado por AD o no existe | Operación rechazada | Registro: Tipo "INTEGRACION_AD_OPERACION_RECHAZADA", Resultado "FALLIDO", Severidad "WARNING", Descripción "Intento de modificar usuario no gestionado por AD o inexistente", Datos Adicionales: {"tenant_id": "UUID", "user_id_solicitado": "UUID", "operacion": "PUT/PATCH/DELETE"} | ☐ | ☐ | ☐ |
| 19 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, tenant, evento, resultado, userName. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: N/A - Operación API sin interfaz gráfica**

**Documentación API:**

#### **Endpoints**
```
PUT    https://portal.efac.com/scim/v2/{tenant_id}/Users/{id}
PATCH  https://portal.efac.com/scim/v2/{tenant_id}/Users/{id}
DELETE https://portal.efac.com/scim/v2/{tenant_id}/Users/{id}
```

#### **Ejemplo 1: PUT - Reemplazo Completo**

**Request:**
```http
PUT https://portal.efac.com/scim/v2/abc123/Users/f1e2d3c4-...
Authorization: Bearer xyz789token
Content-Type: application/scim+json

{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
  "id": "f1e2d3c4-b5a6-7890-cdef-1234567890ab",
  "externalId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "userName": "juan.perez@empresa.com",
  "name": {
    "givenName": "Juan Carlos",
    "familyName": "Pérez García"
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

**Response: HTTP 200**
```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
  "id": "f1e2d3c4-b5a6-7890-cdef-1234567890ab",
  "externalId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "userName": "juan.perez@empresa.com",
  "name": {
    "givenName": "Juan Carlos",
    "familyName": "Pérez García"
  },
  "emails": [...],
  "active": true,
  "groups": [...],
  "meta": {
    "resourceType": "User",
    "created": "2026-01-15T10:00:00Z",
    "lastModified": "2026-01-20T15:30:00Z",
    "location": "https://..."
  }
}
```

#### **Ejemplo 2: PATCH - Cambiar Estado Activo**

**Request:**
```http
PATCH https://portal.efac.com/scim/v2/abc123/Users/f1e2d3c4-...
Authorization: Bearer xyz789token
Content-Type: application/scim+json

{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
  "Operations": [
    {
      "op": "replace",
      "path": "active",
      "value": false
    }
  ]
}
```

**Response: HTTP 200** (usuario completo con active=false actualizado)

#### **Ejemplo 3: PATCH - Agregar Grupo**

**Request:**
```http
PATCH https://portal.efac.com/scim/v2/abc123/Users/f1e2d3c4-...

{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
  "Operations": [
    {
      "op": "add",
      "path": "groups",
      "value": [
        {
          "value": "Gestor de Facturación Electrónica",
          "display": "Gestor de Facturación Electrónica"
        }
      ]
    }
  ]
}
```

#### **Ejemplo 4: DELETE - Soft Delete**

**Request:**
```http
DELETE https://portal.efac.com/scim/v2/abc123/Users/f1e2d3c4-...
Authorization: Bearer xyz789token
```

**Response: HTTP 204 No Content** (sin body)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **PUT sobrescribe datos no sincronizados** | Si usuario tiene datos adicionales en Portal no en AD, PUT los elimina | Medio | Media | Validar que solo atributos SCIM se actualizan, preservar datos específicos del Portal (ej: preferencias UI) | Mitigado |
| **DELETE físico causa pérdida de auditoría** | Eliminación física borraría histórico del usuario | Alto | Baja | Implementar SOLO soft delete, mantener registros con deleted_at, excluir de listados pero preservar en BD | Mitigado |
| **Cambio de grupos elimina permisos críticos** | Error en AD podría remover administradores accidentalmente | Alto | Media | Auditoría detallada de cambios de roles, alertas cuando administradores pierden permisos, validar grupos antes de aplicar | Mitigado |
| **PATCH con path complejo falla** | Paths JSON complejos pueden causar errores de parsing | Bajo | Media | Soportar solo paths simples comunes, validar sintaxis, devolver errores claros | Aceptado |

---

## Notas Adicionales

### Fuera de Alcance

- Restauración de usuarios eliminados (undelete) → HU futura o manual por administrador
- PATCH con operaciones complejas (`move`, `copy`) → Solo `add`, `remove`, `replace`
- Validación de conflictos de concurrencia (ETags) → No soportado
- Bulk operations (actualizar múltiples usuarios) → No soportado

### Dependencias

**Depende de:**
- **HU-AD002A - Endpoint SCIM Base**: Usa autenticación
- **HU-AD002B - POST /Users**: Usuarios deben existir
- **HU-AD001B - Catálogo de Roles**: Valida grupos en actualizaciones

**Relacionada con:**
- **HU-AD006C - Invalidación Proactiva**: DELETE y PATCH active=false invalidan sesiones
- **HU-AD003B - Sincronización Delta**: Usa PUT/PATCH para cambios incrementales

### Valor Independiente

- ✅ Completa CRUD de usuarios SCIM
- ✅ Permite sincronización bidireccional de cambios
- ✅ Esencial para mantener datos actualizados
- ✅ Soporta deshabilitación temporal de usuarios

### Especificación Técnica

**Operaciones PATCH Soportadas:**

| Operación | Path Ejemplo | Descripción |
|-----------|--------------|-------------|
| replace | `"active"` | Reemplaza valor de atributo simple |
| replace | `"name.givenName"` | Reemplaza subatr ibuto |
| add | `"groups"` | Agrega elemento a array |
| remove | `"groups[value eq \"X\"]"` | Remueve elemento específico de array |

**Soft Delete - Modelo de Datos:**
```sql
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMP NULL;
CREATE INDEX idx_users_deleted ON users(deleted_at) WHERE deleted_at IS NOT NULL;

-- Vista para usuarios activos (no eliminados)
CREATE VIEW users_active AS
SELECT * FROM users WHERE deleted_at IS NULL;
```

**Invalidación de Sesiones:**
- DELETE → Invalida todas las sesiones del usuario
- PATCH active=false → Invalida todas las sesiones del usuario
- Otros PATCH → No invalida sesiones (cambios se reflejan en próximo token refresh)

### Referencias

- **RFC 7644 Section 3.5 - Modifying Resources**: https://datatracker.ietf.org/doc/html/rfc7644#section-3.5
- **RFC 7644 Section 3.5.2 - PATCH**: https://datatracker.ietf.org/doc/html/rfc7644#section-3.5.2
- **RFC 7644 Section 3.6 - Deleting Resources**: https://datatracker.ietf.org/doc/html/rfc7644#section-3.6
- **RFC 6902 - JSON Patch**: https://datatracker.ietf.org/doc/html/rfc6902 (referencia para operaciones)
- **Estándar Auditoría**: Tipos `INTEGRACION_AD_USUARIO_*`

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - Endpoint SCIM
