# HU-AD003B - Sincronización Delta (Cambios Incrementales)

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
| **Arquitectura relacionada** | Portal Unificado - Integración AD, Gestión de Usuarios |

---

## Contexto

Esta HU implementa la **Sincronización Delta**, que ocurre después de la sincronización inicial (HU-AD003A) y mantiene actualizados los datos de usuarios cuando cambian en Active Directory.

A diferencia de la sincronización inicial que envía todos los usuarios, la sincronización delta envía **solo los cambios**:
- Nuevos usuarios creados en AD → POST /Users
- Usuarios modificados en AD → PUT o PATCH /Users
- Usuarios eliminados de AD → DELETE /Users

El servidor SCIM del cliente ejecuta sincronización delta periódicamente (configuración típica: cada 15-30 minutos).

---

**Notas:**
- Ocurre **periódicamente** después de completar sincronización inicial
- Servidor SCIM usa timestamps (`lastModified`) para detectar cambios desde última sincronización
- Más eficiente que sincronización completa
- Portal no almacena timestamp de última sincronización (es responsabilidad del servidor SCIM)

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Servidor SCIM del Cliente AD** | Quiero enviar solo usuarios modificados desde última sincronización | Para mantener datos actualizados sin enviar todos los usuarios repetidamente |
| Como **Servidor SCIM del Cliente AD** | Quiero usar GET con filtros para identificar usuarios que requieren actualización | Para comparar estado actual en Portal vs AD y enviar solo cambios |
| Como **Portal Unificado** | Quiero procesar cambios incrementales de forma eficiente | Para mantener sincronización sin impacto en performance |
| Como **Administrador del Portal** | Quiero visibilidad de cuándo ocurren sincronizaciones delta | Para monitorear actividad de integración AD |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|\n| 1 | Servidor SCIM ejecuta sincronización delta periódica | Sincronización inicial completada, configuración AD activa | Timer del servidor SCIM se dispara (ej: cada 30 min) | Servidor SCIM del cliente identifica cambios en AD desde última sincronización (usando timestamps internos del AD), genera lista de usuarios creados/modificados/eliminados, envía peticiones SCIM correspondientes (POST/PUT/PATCH/DELETE) al Portal | ☐ | ☐ | ☐ |
| 2 | Creación de nuevo usuario (POST) | Usuario nuevo agregado a grupo sincronizado en AD | Servidor SCIM detecta nuevo usuario | Servidor envía POST `/scim/v2/{tenant_id}/Users` con datos del nuevo usuario. Portal procesa usando HU-AD002B, crea usuario, asigna roles, devuelve HTTP 201. Registra en auditoría como creación delta | ☐ | ☐ | ☐ |
| 3 | Actualización de usuario existente (PUT) | Usuario modificado en AD (cambio de nombre, grupos, etc.) | Servidor SCIM detecta cambio | Servidor envía PUT `/scim/v2/{tenant_id}/Users/{id}` con datos completos actualizados. Portal procesa usando HU-AD002D, actualiza usuario, recalcula roles si grupos cambiaron, devuelve HTTP 200. Registra en auditoría como actualización delta | ☐ | ☐ | ☐ |
| 4 | Actualización parcial con PATCH | Usuario solo cambia estado activo en AD | Servidor SCIM detecta cambio específico | Servidor envía PATCH con operación `replace active=false`. Portal procesa usando HU-AD002D, actualiza solo atributo modificado, invalida sesiones si active=false, devuelve HTTP 200 | ☐ | ☐ | ☐ |
| 5 | Eliminación de usuario (DELETE) | Usuario removido de grupos sincronizados o eliminado de AD | Servidor SCIM detecta eliminación | Servidor envía DELETE `/scim/v2/{tenant_id}/Users/{id}`. Portal procesa usando HU-AD002D, realiza soft delete (marca deleted_at), invalida sesiones, devuelve HTTP 204. Usuario ya no puede autenticarse | ☐ | ☐ | ☐ |
| 6 | Sincronización delta sin cambios | No hay cambios en AD desde última sincronización | Servidor SCIM ejecuta chequeo | Servidor SCIM detecta que no hay cambios (no hay usuarios con lastModified > última_sync), NO envía peticiones al Portal. Proceso completa sin actividad en Portal | ☐ | ☐ | ☐ |
| 7 | Múltiples cambios en misma sincronización delta | 5 usuarios creados, 10 modificados, 2 eliminados en AD | Servidor SCIM procesa cambios | Servidor envía: 5 POST, 10 PUT/PATCH, 2 DELETE al Portal. Portal procesa cada petición independientemente usando lógica respectiva (HU-AD002B, D). Rate limiting permite hasta 100 req/min. Todas las operaciones registradas en auditoría | ☐ | ☐ | ☐ |
| 8 | Uso de GET para verificar estado actual | Servidor SCIM necesita comparar estado antes de actualizar | Servidor consulta usuario | Servidor envía GET `/scim/v2/{tenant_id}/Users/{id}` (HU-AD002C), compara atributo `lastModified` o datos actuales con su estado en AD, determina si necesita enviar PUT/PATCH. Reduce actualizaciones innecesarias | ☐ | ☐ | ☐ |
| 9 | Dashboard de monitoreo de sincronizaciones delta | Administrador accede a vista de integración AD | Sistema muestra actividad | Sistema muestra tabla "Historial de Sincronizaciones Delta" con: Fecha/Hora, Usuarios Creados, Usuarios Actualizados, Usuarios Eliminados, Errores, Estado (Exitoso/Con Errores). Últimas 20 sincronizaciones. Permite filtrar por fecha | ☐ | ☐ | ☐ |
| 10 | Error durante sincronización delta no detiene operaciones futuras | PUT de un usuario falla por validación | Portal devuelve HTTP 400 | Portal procesa error individualmente (registra en auditoría), devuelve HTTP 400 al servidor SCIM con detalle. Servidor SCIM registra error pero continúa con otros cambios. Próxima sincronización delta puede reintentar cambio fallido | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 11 | Estructura obligatoria de auditoría | Eventos de sincronización delta | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_SYNC_DELTA_*), Fecha/Hora (UTC ISO 8601), Usuario (NULL), Cliente (tenant_id), IP Local (NULL), IP Pública (IP servidor AD), Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 12 | Auditoría de usuario creado en delta | Nuevo usuario enviado vía POST durante delta | Sistema crea usuario | Registro: Tipo "INTEGRACION_AD_SYNC_DELTA_USUARIO_CREADO", Resultado "EXITOSO", Severidad "INFO", Descripción "Usuario {userName} creado en sincronización delta", Datos Adicionales: {"tenant_id": "UUID", "user_id": "UUID", "userName": "nuevo@empresa.com", "origen": "sync_delta"} | ☐ | ☐ | ☐ |
| 13 | Auditoría de usuario actualizado en delta | Usuario modificado vía PUT/PATCH durante delta | Sistema actualiza | Registro: Tipo "INTEGRACION_AD_SYNC_DELTA_USUARIO_ACTUALIZADO", Resultado "EXITOSO", Severidad "INFO", Descripción "Usuario {userName} actualizado en sincronización delta", Datos Adicionales: {"tenant_id": "UUID", "user_id": "UUID", "operacion": "PUT", "cambios_roles": true} | ☐ | ☐ | ☐ |
| 14 | Auditoría de usuario eliminado en delta | Usuario eliminado vía DELETE durante delta | Sistema soft delete | Registro: Tipo "INTEGRACION_AD_SYNC_DELTA_USUARIO_ELIMINADO", Resultado "EXITOSO", Severidad "WARNING", Descripción "Usuario {userName} eliminado en sincronización delta", Datos Adicionales: {"tenant_id": "UUID", "user_id": "UUID", "userName": "eliminado@empresa.com"} | ☐ | ☐ | ☐ |
| 15 | Auditoría de resumen de sincronización delta | Sincronización delta completa | Sistema consolida eventos | Registro: Tipo "INTEGRACION_AD_SYNC_DELTA_RESUMEN", Resultado "EXITOSO", Severidad "INFO", Descripción "Sincronización delta completada para tenant {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "timestamp": "2026-01-20T14:30:00Z", "usuarios_creados": 2, "usuarios_actualizados": 5, "usuarios_eliminados": 1, "errores": 0} | ☐ | ☐ | ☐ |
| 16 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, tenant, evento, resultado. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU-AD003B-HistorialDelta.html** - Historial de sincronizaciones

**Elementos:**
- Título: "Historial de Sincronizaciones Delta - {Nombre Cliente}" (H3)
- Filtros:
  - Fecha Desde (DatePicker)
  - Fecha Hasta (DatePicker)
  - Botón "Filtrar" (outlined)
- Tabla:
  - Columnas: Fecha/Hora, Usuarios Creados, Usuarios Actualizados, Usuarios Eliminados, Errores, Estado
  - Ejemplo de filas:
    - "2026-01-20 14:30" | 2 | 5 | 1 | 0 | Badge "Exitoso" (verde)
    - "2026-01-20 14:00" | 0 | 3 | 0 | 0 | Badge "Exitoso" (verde)
    - "2026-01-20 13:30" | 1 | 0 | 0 | 1 | Badge "Con Errores" (naranja)
  - Paginación: 20 registros por página
- Info box: "Las sincronizaciones delta se ejecutan automáticamente cada 30 minutos (configuración del servidor SCIM del cliente)"
- Botón "Exportar Historial" (outlined) - genera CSV

**Estados:**
- Con sincronizaciones recientes
- Sin sincronizaciones (tabla vacía con mensaje)
- Filtrado por fechas

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Sincronizaciones delta muy frecuentes saturan sistema** | Cliente configura sync cada 1 minuto causando carga excesiva | Medio | Baja | Documentar intervalo recomendado (15-30 min), rate limiting protege contra sobrecarga, monitorear frecuencia por tenant | Mitigado |
| **Cambios en AD no se sincronizan por error de configuración** | Si servidor SCIM no detecta cambios correctamente, datos quedan desactualizados | Alto | Media | Dashboard muestra última sincronización, alertas si no hay actividad en 24h, documentación para clientes sobre configuración correcta | Mitigado |
| **DELETE accidental elimina muchos usuarios** | Error en configuración AD podría marcar muchos usuarios para eliminación | Alto | Baja | Soft delete reversible, alertas si DELETE >10% de usuarios en una sincronización, opción de restauración manual | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance

- Configuración de intervalo de sincronización delta desde Portal → Es configuración del servidor SCIM del cliente
- Sincronización delta push (AD notifica Portal inmediatamente) → Solo pull (servidor SCIM consulta periódicamente)
- Rollback automático de sincronización delta → Manual por administrador si necesario
- Sincronización bidireccional (Portal → AD) → Fuera de alcance permanentemente

### Dependencias

**Depende de:**
- **HU-AD003A - Sincronización Inicial**: Debe completarse primero
- **HU-AD002B - POST /Users**: Para crear nuevos usuarios
- **HU-AD002C - GET /Users**: Para consultar estado actual
- **HU-AD002D - PUT/PATCH/DELETE /Users**: Para actualizar/eliminar

**Relacionada con:**
- **HU-AD006C - Invalidación Proactiva**: Cambios de roles en delta pueden invalidar sesiones

### Valor Independiente

- ✅ Mantiene usuarios sincronizados con AD
- ✅ Más eficiente que re-sincronización completa
- ✅ Esencial para operación continua de integración AD
- ✅ Provee visibilidad de cambios periódicos

### Especificación Técnica

**Flujo Típico de Sincronización Delta (lado Servidor SCIM):**

1. **Timer se dispara** (ej: cada 30 minutos)
2. **Consultar AD** para usuarios con `whenChanged > última_sincronización_exitosa`
3. **Clasificar cambios**:
   - Nuevos usuarios (no existen en Portal) → POST
   - Usuarios modificados (existen pero datos difieren) → PUT/PATCH
   - Usuarios eliminados (existen en Portal pero no en AD) → DELETE
4. **Enviar peticiones SCIM** al Portal
5. **Actualizar timestamp** `última_sincronización_exitosa` tras completar

**Modelo de Datos (opcional para tracking):**
```sql
CREATE TABLE sync_delta_historial (
  id UUID PRIMARY KEY,
  tenant_id UUID REFERENCES tenants(id),
  timestamp TIMESTAMP DEFAULT NOW(),
  usuarios_creados INT DEFAULT 0,
  usuarios_actualizados INT DEFAULT 0,
  usuarios_eliminados INT DEFAULT 0,
  errores INT DEFAULT 0,
  duracion_segundos INT
);

CREATE INDEX idx_sync_delta_tenant_timestamp ON sync_delta_historial(tenant_id, timestamp DESC);
```

**Detección de Anomalías:**
- Si DELETE > 10% de usuarios totales → Generar alerta crítica
- Si no hay sincronización delta en 24h → Alerta a administrador
- Si errores > 20% de operaciones → Alerta

### Diferencias con Sincronización Inicial

| Aspecto | Sincronización Inicial | Sincronización Delta |
|---------|----------------------|---------------------|
| **Frecuencia** | Una vez (al activar AD) | Periódica (cada 15-30 min) |
| **Volumen** | Todos los usuarios | Solo cambios |
| **Operaciones** | Solo POST (crear) | POST, PUT, PATCH, DELETE |
| **Performance** | Puede tardar horas para miles de usuarios | Segundos/minutos típicamente |
| **Monitoreo** | Dashboard tiempo real con progreso | Historial de ejecuciones |

### Referencias

- **HU-AD002B, C, D**: Operaciones SCIM reutilizadas
- **RFC 7644 - SCIM Protocol**: Sincronización incremental
- **Estándar Auditoría**: Tipos `INTEGRACION_AD_SYNC_DELTA_*`
- **Glosario**: Sincronización Delta, Pull Synchronization, Soft Delete

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - Sincronización Inicial
