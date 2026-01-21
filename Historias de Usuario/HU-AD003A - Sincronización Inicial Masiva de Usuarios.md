# HU-AD003A - Sincronización Inicial Masiva de Usuarios

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

Esta HU implementa el proceso de **Sincronización Inicial** que ocurre cuando un Cliente activa por primera vez la integración con Active Directory.

Durante esta sincronización, el servidor SCIM del cliente envía **todos los usuarios** de los grupos configurados para sincronización al Portal mediante peticiones POST masivas al endpoint `/Users`.

El Portal debe procesar estas peticiones de forma eficiente, crear los usuarios, asignar roles, y proporcionar visibilidad del progreso al administrador.

---

**Notas:**
- Ocurre **una vez** después de activar integración AD (HU-AD001A)
- Puede involucrar cientos o miles de usuarios según tamaño del cliente
- Servidor SCIM envía peticiones POST secuenciales o en paralelo (según su configuración)
- Portal debe manejar carga sin degradación de servicio
- Administrador del Portal puede monitorear progreso en tiempo real

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Servidor SCIM del Cliente AD** | Quiero enviar todos los usuarios de grupos sincronizados al Portal | Para realizar la sincronización inicial y poblar el Portal con usuarios del AD |
| Como **Administrador del Portal** | Quiero monitorear el progreso de la sincronización inicial | Para validar que todos los usuarios se están creando correctamente |
| Como **Portal Unificado** | Quiero procesar peticiones POST masivas sin afectar el servicio | Para garantizar disponibilidad durante sincronización de clientes grandes |
| Como **Portal Unificado** | Quiero detectar errores durante sincronización inicial y reportarlos | Para que el administrador pueda corregir problemas de configuración |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|\n| 1 | Inicio de sincronización inicial | Administrador activa integración AD por primera vez | Sistema guarda configuración | Sistema marca tenant con flag `sync_inicial_pendiente=true`, registra timestamp de inicio de sincronización. Servidor SCIM del cliente inicia envío de peticiones POST /Users | ☐ | ☐ | ☐ |
| 2 | Procesamiento de POST masivos | Servidor SCIM envía múltiples POST /Users | Sistema recibe peticiones | Sistema procesa cada POST usando lógica de HU-AD002B (validación, creación, asignación de roles). Cada usuario creado exitosamente incrementa contador de sincronización. Rate limiting permite hasta 100 peticiones/minuto por tenant | ☐ | ☐ | ☐ |
| 3 | Dashboard de monitoreo para administrador | Administrador accede a vista de sincronización | Sistema muestra progreso | Sistema muestra pantalla "Sincronización Inicial - {Nombre Cliente}" con: Estado (En Progreso/Completado/Con Errores), Usuarios Sincronizados (contador actualizado en tiempo real), Usuarios con Errores (contador), Última Actualización (timestamp), botón "Ver Detalle de Errores", indicador de progreso. Actualización automática cada 5 segundos | ☐ | ☐ | ☐ |
| 4 | Detección de finalización de sincronización | Servidor SCIM completa envío de usuarios | Sistema detecta inactividad | Sistema detecta que no hay nuevas peticiones POST en 5 minutos, marca sincronización como completada (`sync_inicial_pendiente=false`, `sync_inicial_completada_at=timestamp`), genera reporte de sincronización, notifica a administrador | ☐ | ☐ | ☐ |
| 5 | Registro de errores durante sincronización | Peticiones POST fallan (userName duplicado, validación) | Sistema captura errores | Sistema registra cada error en tabla temporal de errores de sincronización: user_data_enviado (JSON), error_tipo, error_mensaje, timestamp. Incrementa contador de errores. NO detiene sincronización, continúa procesando otros usuarios | ☐ | ☐ | ☐ |
| 6 | Vista de detalle de errores | Administrador hace clic en "Ver Detalle de Errores" | Sistema muestra tabla | Sistema muestra tabla con: userName intentado, Tipo de Error (duplicado/validación/grupos inválidos), Mensaje de Error, Timestamp. Permite exportar CSV. Administrador puede identificar y corregir problemas en AD | ☐ | ☐ | ☐ |
| 7 | Protección de servicio - Rate limiting durante sincronización | Cliente grande envía >100 peticiones/min | Sistema aplica límite | Sistema devuelve HTTP 429 para peticiones que excedan límite, incluye `Retry-After: 60`. Servidor SCIM debe respetar retry y reintentar. Portal mantiene disponibilidad para otros tenants | ☐ | ☐ | ☐ |
| 8 | Resumen post-sincronización | Sincronización completa | Sistema genera reporte | Sistema genera reporte con: Total Usuarios Enviados (estimado por contador), Usuarios Creados Exitosamente, Usuarios con Errores, Usuarios sin Roles (grupos no reconocidos), Duración Total. Disponible para descarga por administrador | ☐ | ☐ | ☐ |
| 9 | Sincronización inicial con 0 usuarios | Cliente activa AD pero no tiene usuarios en grupos configurados | Servidor no envía POST | Sistema espera 10 minutos de inactividad, marca sincronización como completada con 0 usuarios, muestra mensaje "Sincronización completada: 0 usuarios sincronizados. Verifique configuración de grupos en AD" | ☐ | ☐ | ☐ |
| 10 | Re-sincronización manual | Administrador detecta errores y quiere reiniciar | Administrador hace clic en "Reiniciar Sincronización" | Sistema muestra confirmación "¿Reiniciar sincronización? Esto marcará el proceso como pendiente nuevamente", si confirma: restablece flag `sync_inicial_pendiente=true`, limpia contadores y errores anteriores, permite nueva sincronización | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 11 | Estructura obligatoria de auditoría | Eventos de sincronización inicial | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_SYNC_INICIAL_*), Fecha/Hora (UTC ISO 8601), Usuario (Administrador que activó o NULL), Cliente (tenant_id), IP Local, IP Pública, Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 12 | Auditoría de inicio de sincronización | Integración AD activada | Sistema inicia sync | Registro: Tipo "INTEGRACION_AD_SYNC_INICIAL_INICIADA", Resultado "EXITOSO", Severidad "INFO", Descripción "Sincronización inicial iniciada para tenant {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "iniciada_por": "admin@portal.com", "timestamp_inicio": "2026-01-20T10:00:00Z"} | ☐ | ☐ | ☐ |
| 13 | Auditoría de sincronización completada | Sistema detecta finalización | Sync completado | Registro: Tipo "INTEGRACION_AD_SYNC_INICIAL_COMPLETADA", Resultado "EXITOSO", Severidad "INFO", Descripción "Sincronización inicial completada para tenant {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "usuarios_creados": 245, "usuarios_con_errores": 5, "duracion_minutos": 45, "timestamp_fin": "2026-01-20T10:45:00Z"} | ☐ | ☐ | ☐ |
| 14 | Auditoría de errores durante sincronización | Múltiples POST fallan | Sistema registra resumen | Registro: Tipo "INTEGRACION_AD_SYNC_INICIAL_ERRORES", Resultado "PARCIAL", Severidad "WARNING", Descripción "{n} usuarios con errores durante sincronización inicial", Datos Adicionales: {"tenant_id": "UUID", "total_errores": 5, "tipos_error": {"userName_duplicado": 2, "grupos_invalidos": 3}} | ☐ | ☐ | ☐ |
| 15 | Auditoría de re-sincronización | Administrador reinicia sync | Sistema reinicia | Registro: Tipo "INTEGRACION_AD_SYNC_INICIAL_REINICIADA", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {usuario} reinició sincronización inicial", Datos Adicionales: {"tenant_id": "UUID", "reiniciada_por": "admin@portal.com"} | ☐ | ☐ | ☐ |
| 16 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, tenant, evento, resultado. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU-AD003A-MonitoreoSync.html** - Dashboard de sincronización

**Elementos:**
- Título: "Sincronización Inicial AD - {Nombre Cliente}" (H3)
- **Tarjeta de Estado:**
  - Indicador de estado: Badge "En Progreso" (amarillo pulsante), "Completado" (verde), "Con Errores" (naranja)
  - Barra de progreso (indeterminada mientras está en progreso)
  - Última actualización: "Hace 3 segundos"
- **Métricas:**
  - Usuarios Sincronizados: **245** (número grande, actualizado en tiempo real)
  - Usuarios con Errores: **5** (en rojo si >0)
  - Usuarios sin Roles: **12** (en amarillo, warning)
  - Duración: "45 minutos" (contador en tiempo real)
- **Acciones:**
  - Botón "Ver Detalle de Errores" (outlined, habilitado si errores >0)
  - Botón "Descargar Reporte" (outlined, habilitado al completar)
  - Botón "Reiniciar Sincronización" (text, con confirmación)
  - Botón "Actualizar" (ícono refresh)
- Info box: "La sincronización se actualiza automáticamente cada 5 segundos. El proceso finaliza automáticamente después de 5 minutos de inactividad."

#### **HU-AD003A-ErroresSync.html** - Tabla de errores

**Elementos:**
- Título: "Errores de Sincronización - {Nombre Cliente}"
- Tabla:
  - Columnas: userName Intentado, Tipo de Error, Mensaje, Timestamp
  - Ejemplo de filas:
    - "usuario1@empresa.com" | "Validación" | "Missing required attribute: active" | "2026-01-20 10:05:23"
    - "usuario2@empresa.com" | "Duplicado" | "userName already exists" | "2026-01-20 10:08:45"
    - "usuario3@empresa.com" | "Grupos Inválidos" | "Grupos no reconocidos: 'grupo1', 'grupo2'" | "2026-01-20 10:12:10"
- Botón "Exportar CSV" (outlined)
- Botón "Cerrar" (text)

**Estados:**
- Sincronización en progreso (contador aumentando, barra pulsante)
- Sincronización completada sin errores (verde, checkmark)
- Sincronización completada con errores (naranja, advertencia)
- Sin usuarios sincronizados (mensaje especial)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Sincronización masiva degrada servicio** | Miles de POST simultáneos podrían saturar sistema | Alto | Media | Rate limiting estricto (100 req/min), procesamiento asíncrono, queue de peticiones, monitoreo de performance, alertas si latencia aumenta | Mitigado |
| **Errores masivos pasan desapercibidos** | Muchos usuarios con errores podrían no ser notados | Medio | Media | Dashboard con métricas en tiempo real, alertas si errores >10%, notificación email a administrador al completar, reporte descargable | Mitigado |
| **Sincronización nunca finaliza** | Si servidor SCIM no completa envío, queda pendiente indefinidamente | Medio | Baja | Timeout de 24 horas, después marca como "Completada con advertencia", permite re-sincronización manual | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance

- Sincronización delta (cambios incrementales) → **HU-AD003B**
- Bulk operations (crear múltiples usuarios en 1 petición) → No soportado según ServiceProviderConfig
- Sincronización bidireccional (Portal → AD) → Fuera de alcance permanentemente
- Rollback automático si fallan muchos usuarios → Manual por administrador

### Dependencias

**Depende de:**
- **HU-AD001A - Configuración Cliente**: Integración debe estar activa
- **HU-AD002B - POST /Users**: Usa para crear cada usuario
- **HU-AD001B - Catálogo de Roles**: Valida grupos durante creación masiva

**Relacionada con:**
- **HU-AD003B - Sincronización Delta**: Sincronización posterior a la inicial

### Valor Independiente

- ✅ Permite poblar Portal con usuarios de AD
- ✅ Provee visibilidad de sincronización masiva
- ✅ Detecta errores de configuración temprano
- ✅ Puede entregarse antes de sincronización delta

### Especificación Técnica

**Modelo de Datos (adición a tenant_ad_configuration):**
```sql
ALTER TABLE tenant_ad_configuration ADD COLUMN sync_inicial_pendiente BOOLEAN DEFAULT true;
ALTER TABLE tenant_ad_configuration ADD COLUMN sync_inicial_completada_at TIMESTAMP NULL;
ALTER TABLE tenant_ad_configuration ADD COLUMN sync_inicial_usuarios_creados INT DEFAULT 0;
ALTER TABLE tenant_ad_configuration ADD COLUMN sync_inicial_usuarios_errores INT DEFAULT 0;

CREATE TABLE sync_inicial_errores (
  id UUID PRIMARY KEY,
  tenant_id UUID REFERENCES tenants(id),
  user_data_enviado JSONB,
  error_tipo VARCHAR(100),
  error_mensaje TEXT,
  timestamp TIMESTAMP DEFAULT NOW()
);
```

**Detección de Finalización:**
- Algoritmo: Monitorear última petición POST recibida
- Si `last_post_timestamp + 5 minutos < now()` → Marcar como completada
- Validar cada 30 segundos

**Performance:**
- Rate limiting: 100 POST/minuto por tenant
- Queue asíncrona para procesamiento de usuarios
- Índices en BD para búsquedas rápidas de userName/externalId
- Cache de catálogo de roles en memoria

### Referencias

- **HU-AD002B - POST /Users**: Lógica de creación reutilizada
- **Estándar Auditoría**: Tipos `INTEGRACION_AD_SYNC_INICIAL_*`
- **Glosario**: Sincronización Inicial, Rate Limiting, Soft Delete
- **Guía de Estilos**: Material UI, Dashboard, Badges, Progress Indicators

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - Sincronización Inicial
