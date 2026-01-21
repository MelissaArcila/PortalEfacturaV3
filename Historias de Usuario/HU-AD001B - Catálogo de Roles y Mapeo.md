# HU-AD001B - Catálogo de Roles y Mapeo

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
| **Arquitectura relacionada** | Portal Unificado - Gestión de Roles, Integración AD/SCIM |

---

## Contexto

Esta HU establece el **Catálogo de Roles del Portal** que los Clientes deben replicar en su Active Directory para la integración mediante SCIM 2.0.

El mapeo de roles utiliza la estrategia de **Replicación Directa**: el Cliente es responsable de crear en su Active Directory los grupos/roles con **nombres idénticos** (case-sensitive) a los del catálogo del Portal.

No existe mapeo personalizado ni traducción de nombres. La sincronización SCIM asigna roles automáticamente cuando el nombre del grupo en AD coincide exactamente con un rol del catálogo.

---

**Notas:**
- Los nombres de roles son **case-sensitive** (Administrador ≠ administrador)
- El Cliente debe replicar los nombres EXACTAMENTE como aparecen en el catálogo
- Roles no reconocidos en AD se ignoran durante sincronización
- El catálogo es gestionado por el Portal (el Cliente no puede agregar roles personalizados)
- Esta HU muestra el catálogo y valida nombres durante sincronización SCIM

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador del Portal** | Quiero visualizar el catálogo completo de roles disponibles | Para conocer qué roles existen en el sistema y poder comunicarlos al cliente |
| Como **Administrador del Portal** | Quiero que el sistema valide que los roles recibidos desde AD existan en el catálogo | Para evitar asignación de roles inexistentes o mal escritos |
| Como **Cliente (Administrador AD)** | Quiero consultar el catálogo de roles con sus nombres exactos | Para configurar correctamente los grupos en mi Active Directory |
| Como **Sistema** | Quiero aplicar mapeo por replicación directa (case-sensitive) | Para asignar automáticamente roles cuando los nombres coincidan exactamente |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Visualización del catálogo de roles | Administrador del Portal accede a configuración AD de un cliente | Usuario hace clic en "Ver Catálogo de Roles" | Sistema muestra modal/pantalla "Catálogo de Roles del Portal" con tabla: Nombre del Rol (case-sensitive), Descripción, Permisos Asociados. Incluye advertencia: "⚠️ Los nombres son case-sensitive. El cliente debe crear grupos en AD con estos nombres EXACTOS" | ☐ | ☐ | ☐ |
| 2 | Contenido del catálogo de roles | Sistema muestra catálogo | Catálogo se carga | Sistema muestra roles predefinidos del Portal (ejemplos): "Administrador del Portal" (gestión completa del sistema), "Gestor de Facturación Electrónica" (emisión y gestión de FE), "Contador" (consulta de reportes y auditoría), "Consultor" (solo lectura), etc. Ordenados alfabéticamente | ☐ | ☐ | ☐ |
| 3 | Exportación del catálogo para el cliente | Administrador en vista de catálogo | Usuario hace clic en "Exportar Catálogo" | Sistema genera archivo CSV/Excel con columnas: Nombre del Rol (exacto), Descripción, Permisos. Muestra instrucciones: "Entregue este archivo al administrador de Active Directory del cliente para configurar los grupos" | ☐ | ☐ | ☐ |
| 4 | Indicador de roles configurados en AD | Cliente con integración AD activa y sincronización realizada | Sistema muestra catálogo | Sistema agrega columna "Estado en AD" que indica: checkmark verde "Configurado en AD" si el rol existe en grupos del AD del cliente, ícono gris "No configurado" si no existe. Permite al administrador ver qué roles el cliente ya configuró | ☐ | ☐ | ☐ |
| 5 | Validación de rol durante sincronización SCIM | Endpoint SCIM recibe usuario con roles desde AD | Sistema procesa petición SCIM | Sistema valida cada rol recibido contra catálogo (comparación case-sensitive). Si rol existe en catálogo: asigna al usuario. Si rol NO existe: registra warning en auditoría "Rol '{nombre_rol}' no reconocido, será ignorado", NO asigna el rol, continúa procesando otros roles válidos | ☐ | ☐ | ☐ |
| 6 | Múltiples roles válidos asignados | Usuario AD tiene múltiples grupos que coinciden con catálogo | Sistema procesa roles | Sistema asigna TODOS los roles válidos al usuario. Ejemplo: usuario en grupos "Administrador del Portal" y "Gestor de Facturación Electrónica" → recibe ambos roles en el Portal | ☐ | ☐ | ☐ |
| 7 | Rol con nombre similar pero no exacto | Usuario AD tiene grupo "administrador del portal" (minúsculas) | Sistema valida rol | Sistema NO reconoce el rol (case-sensitive), registra warning "Rol 'administrador del portal' no reconocido. ¿Quiso decir 'Administrador del Portal'?", NO asigna rol, muestra en auditoría | ☐ | ☐ | ☐ |
| 8 | Usuario AD sin roles reconocidos | Usuario AD sin grupos que coincidan con catálogo | Sistema procesa usuario | Sistema sincroniza el usuario pero NO asigna ningún rol, registra warning "Usuario '{username}' sincronizado sin roles (ningún grupo AD coincide con catálogo)", usuario queda sin permisos en el Portal hasta que se corrija en AD | ☐ | ☐ | ☐ |
| 9 | Actualización de roles del catálogo (administración) | Administrador del Portal en módulo de gestión de roles | Usuario agrega/edita/elimina rol del sistema | Sistema actualiza catálogo de roles disponibles, marca cambio en auditoría, requiere que clientes con AD actualicen sus grupos si hay cambios en nombres. Nota: Esta operación es administración del Portal, no específica de AD | ☐ | ☐ | ☐ |
| 10 | Consulta del catálogo desde documentación pública | Cliente busca catálogo antes de configurar AD | Cliente accede a URL pública/documentación | Sistema proporciona página pública/documentación con catálogo actualizado de roles, sus descripciones, y instrucciones para configuración en AD. Accesible sin autenticación | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 11 | Estructura obligatoria de auditoría | Eventos de mapeo de roles | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_ROL_*), Fecha/Hora (UTC ISO 8601), Usuario, Cliente (tenant_id), IP Local, IP Pública, Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 12 | Auditoría de catálogo exportado | Administrador exporta catálogo | Sistema genera archivo | Registro: Tipo "INTEGRACION_AD_CATALOGO_EXPORTADO", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {usuario} exportó catálogo de roles para cliente {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "roles_count": 5, "formato": "CSV"} | ☐ | ☐ | ☐ |
| 13 | Auditoría de rol no reconocido durante sincronización | Sistema recibe rol inválido desde AD | Sistema valida rol | Registro: Tipo "INTEGRACION_AD_ROL_NO_RECONOCIDO", Resultado "FALLIDO", Severidad "WARNING", Descripción "Rol '{nombre_rol}' del AD no reconocido en catálogo para cliente {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "rol_recibido": "administrador del portal", "usuario_afectado": "juan.perez", "sugerencia": "Administrador del Portal"} | ☐ | ☐ | ☐ |
| 14 | Auditoría de roles asignados correctamente | Sistema asigna roles válidos a usuario | Sistema completa asignación | Registro: Tipo "INTEGRACION_AD_ROLES_ASIGNADOS", Resultado "EXITOSO", Severidad "INFO", Descripción "Usuario {username} sincronizado con {n} roles desde AD", Datos Adicionales: {"tenant_id": "UUID", "username": "juan.perez", "roles_asignados": ["Administrador del Portal", "Gestor de Facturación Electrónica"]} | ☐ | ☐ | ☐ |
| 15 | Auditoría de usuario sin roles | Usuario sincronizado sin roles válidos | Sistema registra | Registro: Tipo "INTEGRACION_AD_USUARIO_SIN_ROLES", Resultado "EXITOSO", Severidad "WARNING", Descripción "Usuario {username} sincronizado sin roles (ningún grupo AD coincide con catálogo)", Datos Adicionales: {"tenant_id": "UUID", "username": "juan.perez", "grupos_ad_recibidos": ["Grupo Contabilidad", "Usuarios Internos"]} | ☐ | ☐ | ☐ |
| 16 | Auditoría de actualización de catálogo | Administrador modifica catálogo de roles | Sistema actualiza | Registro: Tipo "INTEGRACION_AD_CATALOGO_ACTUALIZADO", Resultado "EXITOSO", Severidad "INFO", Descripción "Catálogo de roles actualizado", Datos Adicionales: {"cambios": {"rol_agregado": "Auditor Externo", "rol_eliminado": null}} | ☐ | ☐ | ☐ |
| 17 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, usuario, cliente, evento, resultado. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU-AD001B-CatalogoRoles.html** - Visualización del catálogo

**Elementos:**
- Título: "Catálogo de Roles del Portal" (H3)
- Advertencia destacada: "⚠️ Los nombres de roles son case-sensitive. El cliente debe crear grupos en Active Directory con estos nombres EXACTOS"
- Tabla de roles:
  - Columnas: Nombre del Rol (monospace font para claridad), Descripción, Permisos Asociados, Estado en AD (si aplica)
  - Ejemplo de filas:
    - "Administrador del Portal" | "Gestión completa del sistema" | "Todos" | ✓ Configurado
    - "Gestor de Facturación Electrónica" | "Emisión y gestión de FE" | "FE.Crear, FE.Consultar..." | - No configurado
    - "Contador" | "Consulta de reportes y auditoría" | "Reportes.*, Audit.Read" | ✓ Configurado
    - "Consultor" | "Solo lectura" | "*.Read" | - No configurado
- Botones:
  - "Exportar Catálogo" (contained, ícono download) - genera CSV/Excel
  - "Cerrar" (outlined)
- Info box: "Entregue el catálogo exportado al administrador de Active Directory del cliente para que configure los grupos necesarios"

**Estados:**
- Vista desde configuración AD (con columna "Estado en AD")
- Vista desde documentación pública (sin columna "Estado en AD")
- Catálogo con 4-6 roles de ejemplo

#### **HU-AD001B-Instrucciones.html** - Documentación para cliente

**Elementos:**
- Título: "Guía de Configuración de Roles en Active Directory"
- Sección "Paso a Paso":
  1. Descargue el catálogo de roles (CSV adjunto)
  2. En su Active Directory, cree grupos de seguridad con los nombres EXACTOS del catálogo
  3. Importante: Los nombres son case-sensitive ("Administrador del Portal" ≠ "administrador del portal")
  4. Asigne usuarios a los grupos según los permisos que requieran
  5. La sincronización SCIM detectará automáticamente los grupos y asignará los roles correspondientes
- Tabla de ejemplo con roles
- Sección "Errores Comunes":
  - ❌ "administrador" → Debe ser "Administrador del Portal"
  - ❌ "gestor FE" → Debe ser "Gestor de Facturación Electrónica"
  - ✅ "Contador" → Correcto

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Clientes configuran roles con nombres incorrectos** | Variaciones en nombres (mayúsculas, espacios, tildes) causarán que roles no se asignen | Alto | Alta | Documentación clara con ejemplos, advertencias visibles, exportación de catálogo, sugerencias en auditoría cuando hay nombres similares, validación SCIM con feedback detallado | Mitigado |
| **Cambios en catálogo rompen configuraciones existentes** | Si se renombra un rol, clientes con AD configurado perderán asignaciones | Alto | Baja | Evitar renombrar roles (crear nuevos en su lugar), documentar política de versionado de roles, notificar a clientes con antelación si hay cambios | Mitigado |
| **Usuarios sin roles quedan bloqueados** | Si ningún grupo AD coincide, usuarios no tienen permisos | Medio | Media | Registrar warnings en auditoría, dashboard de usuarios sin roles para administradores, documentación clara para clientes | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance

- Mapeo personalizado de nombres (Cliente A: "Admin" → "Administrador del Portal") → Fuera de alcance permanentemente por diseño
- Roles personalizados por cliente → Sistema centralizado con catálogo único
- Traducción automática de nombres similares → Validación estricta case-sensitive
- Sincronización de permisos granulares desde AD → Solo roles predefinidos
- Auto-corrección de nombres similares → Requiere validación manual

### Dependencias

**Depende de:**
- **HU-AD001A - Configuración Cliente**: Integración AD debe estar activada
- **Sistema de Roles del Portal**: Catálogo de roles debe existir
- **HU-AD002C - Sincronización SCIM**: Validación de roles ocurre durante sincronización

**Relacionada con:**
- **HU-AD003A - Sincronización Inicial**: Usa catálogo para asignar roles masivamente
- **HU-AD006C - Invalidación por Cambio de Rol**: Detecta cambios en roles asignados

### Valor Independiente

- ✅ Permite visualizar y exportar catálogo de roles
- ✅ Establece reglas de mapeo (replicación directa)
- ✅ Puede entregarse antes de implementación completa de SCIM
- ✅ Documentación reutilizable para todos los clientes
- ✅ Validación de roles funciona de forma independiente

### Estrategia de Mapeo: Replicación Directa

**Justificación:**
- **Simplicidad**: No requiere tabla de mapeo personalizado por cliente
- **Mantenibilidad**: Un solo catálogo centralizado para todos los clientes
- **Consistencia**: Todos los clientes usan misma nomenclatura
- **Escalabilidad**: Agregar clientes no requiere configuración adicional de mapeo

**Responsabilidad del Cliente:**
- Crear grupos en AD con nombres exactos del catálogo
- Mantener sincronización con cambios en catálogo del Portal
- Validar periódicamente que nombres estén correctos

**Responsabilidad del Portal:**
- Mantener catálogo estable y documentado
- Comunicar cambios con antelación
- Proveer validación y feedback detallado en auditoría

### Ejemplo de Catálogo (Inicial)

| Nombre del Rol | Descripción | Permisos Asociados |
|----------------|-------------|-------------------|
| Administrador del Portal | Gestión completa del sistema, todos los módulos | *.* |
| Gestor de Facturación Electrónica | Emisión, consulta, anulación de FE | FE.Create, FE.Read, FE.Cancel, FE.Send |
| Contador | Consulta de reportes, auditoría, no puede emitir | Reports.*, Audit.Read, FE.Read |
| Consultor | Solo lectura en todos los módulos | *.Read |
| Soporte Técnico | Gestión de incidentes y configuración básica | Support.*, Config.Basic |

### Referencias

- **Estándar Auditoría**: Tipos `INTEGRACION_AD_ROL_*`
- **Glosario**: Rol, Catálogo de Roles, Mapeo por Replicación Directa, Case-Sensitive
- **Guía de Estilos**: Material UI, Primary/Secondary
- **SCIM 2.0 RFC 7643 - Resource Schema**: https://datatracker.ietf.org/doc/html/rfc7643#section-8.2

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - Configuración Base
