# Inventario de Roles y Permisos

## Propósito

Este documento define y mantiene el inventario centralizado de todos los roles y permisos del Portal Unificado. Cada rol representa un conjunto de capacidades y restricciones funcionales que determinan qué acciones puede realizar un usuario en el sistema.

Este inventario debe actualizarse cada vez que:
- Se crea una nueva Historia de Usuario que introduce roles o permisos
- Se identifican nuevas capacidades funcionales
- Se modifican o eliminan permisos existentes

---

## Principios de Roles y Permisos

1. **Mínimo Privilegio**: Cada rol debe tener únicamente los permisos necesarios para cumplir sus funciones.
2. **Segregación de Responsabilidades**: Los roles críticos deben estar separados para evitar conflictos de interés.
3. **Contexto de Cliente**: Los permisos se aplican dentro del contexto del cliente seleccionado durante la sesión.
4. **Claridad Funcional**: Los roles y permisos se definen a nivel funcional, no técnico.

---

## Roles Definidos

### 1. Administrador del Sistema

**Descripción:**
Usuario con privilegios elevados responsable de la administración general del Portal Unificado. Tiene capacidades para gestionar usuarios, clientes, configuraciones globales y acceso a información de auditoría para supervisión de seguridad y cumplimiento.

**Alcance:**
Transversal a todos los clientes. Puede operar sobre múltiples clientes y configuraciones globales del sistema.

**Responsabilidades:**
- Administración de usuarios del sistema
- Administración de clientes (activación/desactivación)
- Supervisión de seguridad y auditoría
- Configuración de parámetros globales del sistema
- Atención de incidentes de seguridad

**Permisos asignados:**

| Permiso | Descripción | Módulo/Área | HU Origen | Estado |
|---------|-------------|-------------|-----------|--------|
| **CONSULTAR_AUDITORIA** | Consultar registros de auditoría del sistema con filtros avanzados (usuario, tipo de evento, fecha, IP, cliente, severidad) | Auditoría | HU001 | Activo |
| **EXPORTAR_AUDITORIA** | Exportar resultados de consultas de auditoría para análisis externo | Auditoría | HU001 | Activo |
| **GESTIONAR_USUARIOS_SISTEMA** | Crear, modificar, activar y desactivar cuentas de usuarios del sistema | Gestión de Usuarios | Pendiente | Pendiente |
| **GESTIONAR_CLIENTES** | Activar, desactivar y consultar información de clientes | Gestión de Clientes | Pendiente | Pendiente |
| **DESBLOQUEAR_CUENTAS** | Desbloquear manualmente cuentas de usuarios bloqueadas por intentos fallidos | Seguridad | Pendiente | Pendiente |
| **CONFIGURAR_PARAMETROS_GLOBALES** | Modificar configuraciones globales del sistema | Configuración | Pendiente | Pendiente |
| **CONSULTAR_INFORMACION_TODOS_CLIENTES** | Consultar información de cualquier cliente del sistema | Transversal | Pendiente | Pendiente |

**Restricciones:**
- No debe poder modificar o eliminar registros de auditoría
- Las acciones realizadas por el Administrador del Sistema también deben ser auditadas

---

### 2. Administrador de Cliente

**Descripción:**
Usuario responsable de la administración de un cliente específico. Tiene capacidades para gestionar usuarios de su cliente, configurar parámetros operativos del cliente y consultar información relacionada exclusivamente con su cliente.

**Alcance:**
Limitado al contexto del cliente al cual pertenece. Solo puede operar sobre la información y usuarios de su propio cliente.

**Responsabilidades:**
- Administración de usuarios del cliente
- Configuración de parámetros del cliente
- Consulta de información operativa del cliente
- Gestión de resoluciones de facturación del cliente
- Soporte de primer nivel a usuarios del cliente

**Permisos asignados:**

| Permiso | Descripción | Módulo/Área | HU Origen | Estado |
|---------|-------------|-------------|-----------|--------|
| **GESTIONAR_USUARIOS_CLIENTE** | Crear, modificar, activar y desactivar usuarios asociados a su cliente | Gestión de Usuarios | Pendiente | Pendiente |
| **CONFIGURAR_CLIENTE** | Modificar configuraciones y parámetros operativos de su cliente | Configuración | Pendiente | Pendiente |
| **CONSULTAR_DOCUMENTOS_CLIENTE** | Consultar todos los documentos electrónicos de su cliente | Documentos | Pendiente | Pendiente |
| **DESCARGAR_DOCUMENTOS_CLIENTE** | Descargar archivos (XML, PDF, AttachedDocument) de documentos de su cliente | Documentos | Pendiente | Pendiente |
| **GESTIONAR_RESOLUCIONES** | Crear, modificar y consultar resoluciones de facturación de su cliente | Facturación | Pendiente | Pendiente |
| **CONSULTAR_REPORTES_CLIENTE** | Generar y consultar reportes operativos de su cliente | Reportes | Pendiente | Pendiente |
| **REENVIAR_CORREOS** | Reenviar correos electrónicos asociados a documentos de su cliente | Documentos | Pendiente | Pendiente |

**Restricciones:**
- Solo puede acceder a información del cliente al cual pertenece
- No puede consultar auditoría del sistema
- No puede gestionar otros clientes
- No puede modificar parámetros globales del sistema

---

## Roles Pendientes de Definición

Los siguientes roles han sido identificados pero requieren definición completa en futuras Historias de Usuario:

| Rol | Descripción Preliminar | Prioridad | Notas |
|-----|------------------------|-----------|-------|
| **Usuario Operativo** | Usuario que realiza operaciones diarias de emisión y consulta de documentos | Alta | Mencionado en Glosario |
| **Usuario de Consulta** | Usuario con permisos de solo lectura para consultar información | Media | Mencionado en Glosario |
| **Usuario Contable/Financiero** | Usuario enfocado en reportes contables y financieros | Baja | Por definir según necesidades |
| **Soporte Técnico** | Usuario del equipo de soporte con capacidades de diagnóstico | Media | Por definir según necesidades |

---

## Permisos del Sistema

### Catálogo de Permisos

Todos los permisos definidos en el sistema, organizados por módulo:

#### Módulo: Auditoría
| Permiso | Descripción | Roles Asignados | HU Origen |
|---------|-------------|-----------------|-----------|
| CONSULTAR_AUDITORIA | Consultar registros de auditoría con filtros avanzados | Administrador del Sistema | HU001 |
| CONSULTAR_AUDITORIA_PROPIA | Consultar el historial de acciones propias del usuario | Todos los usuarios | Pendiente |
| EXPORTAR_AUDITORIA | Exportar resultados de auditoría | Administrador del Sistema | HU001 |

#### Módulo: Gestión de Usuarios
| Permiso | Descripción | Roles Asignados | HU Origen |
|---------|-------------|-----------------|-----------|
| GESTIONAR_USUARIOS_SISTEMA | Gestión completa de usuarios del sistema | Administrador del Sistema | Pendiente |
| GESTIONAR_USUARIOS_CLIENTE | Gestión de usuarios dentro de un cliente | Administrador de Cliente | Pendiente |

#### Módulo: Gestión de Clientes
| Permiso | Descripción | Roles Asignados | HU Origen |
|---------|-------------|-----------------|-----------|
| GESTIONAR_CLIENTES | Activar/desactivar clientes | Administrador del Sistema | Pendiente |
| CONFIGURAR_CLIENTE | Configurar parámetros de un cliente | Administrador de Cliente | Pendiente |
| CONSULTAR_INFORMACION_TODOS_CLIENTES | Consultar información de cualquier cliente | Administrador del Sistema | Pendiente |

#### Módulo: Seguridad
| Permiso | Descripción | Roles Asignados | HU Origen |
|---------|-------------|-----------------|-----------|
| AUTENTICARSE | Acceder al sistema mediante credenciales | Todos los usuarios | HU001 |
| SELECCIONAR_CLIENTE | Seleccionar contexto de cliente | Todos los usuarios | HU001 |
| DESBLOQUEAR_CUENTAS | Desbloqueo manual de cuentas | Administrador del Sistema | Pendiente |
| CAMBIAR_CONTRASENA_PROPIA | Cambiar su propia contraseña | Todos los usuarios | Pendiente |

#### Módulo: Documentos Electrónicos
| Permiso | Descripción | Roles Asignados | HU Origen |
|---------|-------------|-----------------|-----------|
| CONSULTAR_DOCUMENTOS_CLIENTE | Consultar documentos del cliente | Administrador de Cliente | Pendiente |
| DESCARGAR_DOCUMENTOS_CLIENTE | Descargar archivos de documentos | Administrador de Cliente | Pendiente |
| EMITIR_DOCUMENTOS | Emitir documentos electrónicos | Pendiente | Pendiente |
| REENVIAR_CORREOS | Reenviar correos de documentos | Administrador de Cliente | Pendiente |

#### Módulo: Configuración
| Permiso | Descripción | Roles Asignados | HU Origen |
|---------|-------------|-----------------|-----------|
| CONFIGURAR_PARAMETROS_GLOBALES | Configurar parámetros globales | Administrador del Sistema | Pendiente |
| CONFIGURAR_CLIENTE | Configurar parámetros de cliente | Administrador de Cliente | Pendiente |
| GESTIONAR_RESOLUCIONES | Gestionar resoluciones de facturación | Administrador de Cliente | Pendiente |

#### Módulo: Reportes
| Permiso | Descripción | Roles Asignados | HU Origen |
|---------|-------------|-----------------|-----------|
| CONSULTAR_REPORTES_CLIENTE | Consultar reportes del cliente | Administrador de Cliente | Pendiente |
| EXPORTAR_REPORTES | Exportar reportes | Pendiente | Pendiente |

---

## Nomenclatura de Permisos

Los permisos siguen esta convención de nomenclatura:

**Formato:** `<ACCION>_<ENTIDAD>_<ALCANCE_OPCIONAL>`

**Ejemplos:**
- `CONSULTAR_AUDITORIA`
- `GESTIONAR_USUARIOS_SISTEMA`
- `GESTIONAR_USUARIOS_CLIENTE`
- `DESCARGAR_DOCUMENTOS_CLIENTE`
- `CONFIGURAR_PARAMETROS_GLOBALES`

**Acciones comunes:**
- `CONSULTAR`: Visualizar información
- `GESTIONAR`: Crear, modificar, activar, desactivar
- `DESCARGAR`: Obtener archivos
- `EXPORTAR`: Exportar datos
- `EMITIR`: Crear nuevos documentos electrónicos
- `CONFIGURAR`: Modificar parámetros y configuraciones
- `REENVIAR`: Volver a enviar información

---

## Matriz de Roles y Permisos

| Permiso / Rol | Admin Sistema | Admin Cliente | Usuario Operativo | Usuario Consulta |
|---------------|---------------|---------------|-------------------|------------------|
| **Autenticación y Seguridad** |
| AUTENTICARSE | ✅ | ✅ | ⏳ | ⏳ |
| SELECCIONAR_CLIENTE | ✅ | ✅ | ⏳ | ⏳ |
| DESBLOQUEAR_CUENTAS | ✅ | ❌ | ❌ | ❌ |
| CAMBIAR_CONTRASENA_PROPIA | ⏳ | ⏳ | ⏳ | ⏳ |
| **Auditoría** |
| CONSULTAR_AUDITORIA | ✅ | ❌ | ❌ | ❌ |
| CONSULTAR_AUDITORIA_PROPIA | ⏳ | ⏳ | ⏳ | ⏳ |
| EXPORTAR_AUDITORIA | ✅ | ❌ | ❌ | ❌ |
| **Gestión de Usuarios** |
| GESTIONAR_USUARIOS_SISTEMA | ⏳ | ❌ | ❌ | ❌ |
| GESTIONAR_USUARIOS_CLIENTE | ❌ | ⏳ | ❌ | ❌ |
| **Gestión de Clientes** |
| GESTIONAR_CLIENTES | ⏳ | ❌ | ❌ | ❌ |
| CONFIGURAR_CLIENTE | ❌ | ⏳ | ❌ | ❌ |
| CONSULTAR_INFORMACION_TODOS_CLIENTES | ⏳ | ❌ | ❌ | ❌ |
| **Documentos** |
| CONSULTAR_DOCUMENTOS_CLIENTE | ✅ | ⏳ | ⏳ | ⏳ |
| DESCARGAR_DOCUMENTOS_CLIENTE | ✅ | ⏳ | ⏳ | ⏳ |
| EMITIR_DOCUMENTOS | ❌ | ⏳ | ⏳ | ❌ |
| REENVIAR_CORREOS | ✅ | ⏳ | ⏳ | ❌ |
| **Configuración** |
| CONFIGURAR_PARAMETROS_GLOBALES | ⏳ | ❌ | ❌ | ❌ |
| GESTIONAR_RESOLUCIONES | ❌ | ⏳ | ❌ | ❌ |
| **Reportes** |
| CONSULTAR_REPORTES_CLIENTE | ✅ | ⏳ | ⏳ | ⏳ |
| EXPORTAR_REPORTES | ✅ | ⏳ | ⏳ | ❌ |

**Leyenda:**
- ✅ Permiso activo y definido
- ⏳ Permiso pendiente de definición en HU
- ❌ Permiso no asignado a este rol

---

## Uso en Historias de Usuario

### Al crear una nueva HU:

1. **Identificar roles involucrados**: Revisar este inventario y seleccionar el rol más adecuado para los actores de la HU.

2. **Si el rol no existe**: Preguntar al usuario si se debe crear un nuevo rol o usar uno existente.

3. **Identificar permisos requeridos**: Determinar qué capacidades funcionales necesita el usuario para cumplir el objetivo de la HU.

4. **Actualizar el inventario**: Agregar nuevos permisos identificados y actualizar las asignaciones de roles.

5. **Referenciar en la HU**: En el enunciado general de la historia, usar el rol exacto del inventario (ej: "Como **Administrador de Cliente**").

### Plantilla para actualización:

Cuando se identifiquen nuevos permisos en una HU, agregarlos con esta estructura:

```markdown
| Permiso | Descripción | Roles Asignados | HU Origen |
|---------|-------------|-----------------|-----------|
| NOMBRE_PERMISO | Descripción clara del permiso | Roles que lo tienen | HUXXX |
```

---

## Auditoría de Permisos

Según el Estándar de Auditoría Funcional, las siguientes acciones relacionadas con permisos deben ser auditadas:

- Modificación de permisos de un usuario
- Modificación de permisos de un rol
- Asignación de roles a usuarios
- Creación o eliminación de roles
- Intentos de acceso no autorizado a funcionalidades

---

## Lineamientos de Seguridad Aplicables (APL03)

- **AP-0053**: El sistema no debe permitir a un actor aumentar privilegios para sí mismo
- **AP-0054**: Debe existir un módulo de configuración de autorizaciones
- **AP-0055**: Restringir acceso a objetos sensibles solo a usuarios autorizados
- **AP-0060**: Restringir acceso a funciones críticas solo a usuarios autorizados
- **AP-0063**: Los privilegios requeridos deben estar definidos

---

## Notas Finales

- Este inventario es un **documento vivo** que debe actualizarse con cada nueva HU.
- Los permisos con estado "Pendiente" se completarán cuando se creen las HUs correspondientes.
- La asignación de permisos a roles puede ajustarse según evolucionen los requisitos del negocio.
- Este documento define permisos **funcionales**. Las implementaciones técnicas (claims, tokens, policies) son responsabilidad de arquitectura.

---

**Última actualización:** 2025-12-17
**Versión:** 1.0
**Roles definidos:** 2
**Permisos activos:** 3
**Permisos pendientes:** 24
