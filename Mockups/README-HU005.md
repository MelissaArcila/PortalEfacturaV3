# Mockups HU005 - Visualización y Búsqueda de Usuarios

Este documento describe el mockup interactivo creado para la **HU005 - Visualización y Búsqueda de Usuarios**.

## 📋 Archivo Creado

### **HU005-GestionUsuarios.html** - Pantalla Principal de Gestión de Usuarios
**Ubicación en HU:** Escenarios 1-24 (líneas 57-81) + Mockup líneas 107-161

**Descripción:** Pantalla completa de visualización, búsqueda y filtrado de usuarios con capacidades de gestión.

---

## 🎯 Componentes Principales

### 1. **Barra Superior**
**Ubicación en HU:** Escenario 2, 19 (líneas 58, 75)

**Elementos:**
- ✅ Título "Gestión de Usuarios" (H2, color #212121)
- ✅ Botón "Crear Nuevo Usuario"
  - Gradiente Primary → Secondary
  - Icono add_circle
  - Navega a HU006 (Creación de Usuarios)

---

### 2. **Panel de Búsqueda y Filtros**
**Ubicación en HU:** Escenarios 4-8 (líneas 60-64) + Mockup líneas 114-121

#### **Buscador General:**
- ✅ Campo de texto con icono de lupa
- ✅ Placeholder: "Buscar por nombre, apellido, número de identificación o correo..."
- ✅ Búsqueda en tiempo real al presionar Enter
- ✅ Actualiza contador de resultados

**Funcionalidad implementada:**
```javascript
// Búsqueda parcial, case-insensitive en:
- Número de Identificación
- Primer Nombre
- Segundo Nombre
- Primer Apellido
- Segundo Apellido
- Correo Electrónico
```

#### **Filtros:**

**1. Filtro por Estado** (Escenario 5, línea 61)
- ✅ Dropdown con opciones:
  - Todos
  - Activo
  - Inactivo
  - Bloqueado
- ✅ Se combina con otros filtros
- ✅ Indicador visual de filtro activo

**2. Filtro por Tipo de Usuario** (Escenario 6, línea 62)
- ✅ Dropdown con opciones:
  - Todos
  - Usuario Interno
  - Usuario de Cliente

**3. Filtro por Cliente** (Escenario 7, línea 63)
- ✅ Dropdown con opciones:
  - Todos
  - Solo Internos (Sin Cliente)
  - Lista de empresas (alfabético)
- ✅ Permite búsqueda typeahead (en producción)

#### **Botones de Acción:**
- ✅ "Limpiar Filtros" (text button) - Escenario 8 (línea 64)
  - Resetea todos los filtros a "Todos"
  - Limpia búsqueda
  - Restaura vista completa
- ✅ "Exportar" (outlined button) - Escenario 20 (línea 76)
  - Genera CSV/Excel con filtros aplicados
  - Nombre archivo: `usuarios_[fecha].csv`

---

### 3. **Contador de Resultados**
**Ubicación en HU:** Escenario 3 (línea 59) + Mockup línea 124

**Formatos:**
- ✅ Normal: "Mostrando 1-20 de 150 usuarios"
- ✅ Con búsqueda: "Mostrando 2 resultados para 'Juan Pérez'"
- ✅ Botón "Actualizar" (refresh) - Escenario 24 (línea 80)

---

### 4. **Tabla de Usuarios**
**Ubicación en HU:** Escenarios 3, 9-11 (líneas 59, 65-67) + Mockup líneas 127-136

#### **Columnas:**

| # | Columna | Características | Escenario |
|---|---------|-----------------|-----------|
| 1 | **Número ID** | Texto plano, sortable | 9, 10 |
| 2 | **Nombre Completo** | Concatenación completa, sortable | 9, 10 |
| 3 | **Correo Electrónico** | Email o "No registrado" con warning icon | 9, 17 |
| 4 | **Tipo** | Badge: "Interno" (primary) / "Cliente" (secondary) | 9 |
| 5 | **Estado** | Chip: "Activo" (green) / "Inactivo" (gray) / "Bloqueado" (red) | 9, 18 |
| 6 | **Permisos Asignados** | Enlace clickeable "X permisos" con popover | 9, 11 |
| 7 | **Acciones** | Iconos: Ver, Editar, Más opciones | 9, 12-14 |

#### **Características de la Tabla:**

**Nombre Completo:**
```
Formato: Primer Nombre + Segundo Nombre + Primer Apellido + Segundo Apellido
Ejemplo: "Juan Carlos Pérez Gómez"
```

**Correo Electrónico sin registrar:** (Escenario 17, línea 73)
```html
<span class="no-email">
    <span class="material-icons">warning</span>
    No registrado
</span>
```
- Color gris claro
- Icono de advertencia
- Tooltip: "Este usuario no tiene correo electrónico configurado"

**Usuario Bloqueado:** (Escenario 18, línea 74)
- Badge rojo "Bloqueado"
- Tooltip con razón:
  - "Bloqueado por 5 intentos fallidos. Desbloqueo automático: [fecha/hora]"
  - "Bloqueado manualmente por [admin] el [fecha]"

**Ordenamiento:** (Escenario 10, línea 66)
- ✅ Click en encabezado de columna ordena
- ✅ Indicadores visuales:
  - ↕ - Sin ordenar
  - ↑ - Ascendente
  - ↓ - Descendente
- ✅ Columnas ordenables:
  - Número ID
  - Nombre Completo
  - Estado
  - Fecha de Creación (no visible en tabla pero disponible)

**Hover en filas:**
- Fondo gris claro (#FAFAFA)
- Mejora legibilidad

---

### 5. **Popover de Permisos Asignados**
**Ubicación en HU:** Escenario 11 (línea 67) + Mockup líneas 154-156

**Funcionalidad:**
- ✅ Click en "X permisos" muestra popover
- ✅ Lista formato: "Cliente (Nombre Empresa) - Rol"
  - Ejemplo: "Empresa ABC - Administrador de Cliente"
  - Ejemplo: "Empresa XYZ - Gestor Emisión FE"
- ✅ Si >5 permisos, muestra primeros 5 + enlace "Ver todos"
- ✅ Si usuario interno sin permisos de cliente: "Solo roles internos"
- ✅ Click fuera cierra el popover

**Datos mockeados:**
```javascript
Usuario 1: 5 permisos (muestra todos)
Usuario 5: 8 permisos (muestra 5 + "Ver todos (8 permisos)")
```

---

### 6. **Columna de Acciones**
**Ubicación en HU:** Escenarios 12-14 (líneas 68-70)

**Iconos de acción:**

**1. Ver Detalle (ojo)** - Escenario 12 (línea 68)
- Icono: `visibility`
- Tooltip: "Ver Detalle"
- Navega a pantalla de Detalle de Usuario
- Muestra:
  - Datos personales
  - Historial de permisos
  - Historial de cambios de estado
  - Último acceso
  - Clientes asociados
  - Auditoría de acciones

**2. Editar (lápiz)** - Escenario 13 (línea 69)
- Icono: `edit`
- Tooltip: "Editar"
- Navega a pantalla de Editar Usuario (HU006)
- Pre-poblada con datos actuales

**3. Más Opciones (tres puntos)** - Escenario 14 (línea 70)
- Icono: `more_vert`
- Tooltip: "Más opciones"
- Muestra menú contextual

---

### 7. **Menú Contextual "Más Opciones"**
**Ubicación en HU:** Escenario 14 (línea 70) + Mockup líneas 159

**Opciones del menú:**

| Opción | Icono | Condición | Confirmación |
|--------|-------|-----------|--------------|
| **Resetear Contraseña** | `lock_reset` | Siempre | Sí - "¿Está seguro?" |
| **Inactivar Usuario** | `toggle_off` | Si está Activo | Sí |
| **Activar Usuario** | `toggle_on` | Si está Inactivo | Sí |
| **Ver Auditoría** | `history` | Siempre | No |
| **Eliminar Usuario** | `delete` | Solo si nunca tuvo permisos | Sí - ADVERTENCIA |

**Funcionalidad:**
- ✅ Click en icono "more_vert" abre menú
- ✅ Posicionado relativo al botón
- ✅ Click fuera cierra menú
- ✅ Cada opción requiere confirmación
- ✅ Opción "Eliminar" en rojo (peligro)

---

### 8. **Paginación**
**Ubicación en HU:** Escenarios 3, 15, 16 (líneas 59, 71, 72) + Mockup líneas 139-141

**Elementos:**

**Selector de Registros por Página:** (Escenario 15, línea 71)
- ✅ Dropdown con opciones: 10, 20, 50, 100
- ✅ Valor por defecto: 20
- ✅ Mantiene filtros activos al cambiar

**Información:**
- ✅ "Página X de Y"
- ✅ Actualización dinámica

**Controles de Navegación:** (Escenario 16, línea 72)
- ✅ Botón "Anterior"
  - Deshabilitado si página = 1
- ✅ Botón "Siguiente"
  - Deshabilitado si es última página
- ✅ Actualiza contador: "Mostrando 21-40 de X usuarios"

---

### 9. **Estados Especiales**

#### **Estado Vacío** (Escenario 21, línea 77)
**Elementos:**
- ✅ Icono grande de personas (person_outline, 80px, gris)
- ✅ Título: "Aún no hay usuarios registrados en el sistema"
- ✅ Subtítulo: "Crea el primer usuario haciendo clic en 'Crear Nuevo Usuario'"
- ✅ Botón "Crear Nuevo Usuario" destacado

#### **Sin Resultados de Búsqueda** (Escenario 22, línea 78)
**Elementos:**
- ✅ Icono de búsqueda vacía (search_off)
- ✅ Mensaje: "No se encontraron usuarios que coincidan con la búsqueda 'XYZ123'"
- ✅ Sugerencia: "Verifica el término de búsqueda o limpia los filtros"
- ✅ Botón "Limpiar Filtros" visible

---

## 🎨 Diseño Aplicado

### Paleta de Colores
- **Primary:** `#4A5A9E`
- **Secondary:** `#D4145A`
- **Success:** `#4CAF50` (Estado Activo)
- **Error:** `#F44336` (Estado Bloqueado)
- **Default:** `#E0E0E0` (Estado Inactivo)

### Badges y Chips

**Badge de Tipo:**
- **Interno:** `#E3F2FD` fondo, `#1565C0` texto (primary light)
- **Cliente:** `#FCE4EC` fondo, `#C2185B` texto (secondary light)

**Chip de Estado:**
- **Activo:** `#E8F5E9` fondo, `#2E7D32` texto
- **Inactivo:** `#E0E0E0` fondo, `#616161` texto
- **Bloqueado:** `#FFEBEE` fondo, `#C62828` texto

### Iconografía (Material Icons)
- `search` - Búsqueda
- `add_circle` - Crear nuevo
- `visibility` - Ver detalle
- `edit` - Editar
- `more_vert` - Más opciones
- `refresh` - Actualizar
- `download` - Exportar
- `clear` - Limpiar filtros
- `warning` - Advertencia (sin email)
- `check_circle` - Permisos asignados
- `lock_reset` - Resetear contraseña
- `toggle_off/on` - Activar/Inactivar
- `history` - Auditoría
- `delete` - Eliminar

### Tipografía
- **H1 (Navbar):** 1.25rem, font-weight 600
- **H2 (Título):** 2rem, font-weight 600
- **Body:** 14px-16px, Roboto
- **Badges/Chips:** 12px, font-weight 500

---

## 🔧 Funcionalidad Técnica

### Búsqueda en Tiempo Real
```javascript
function handleSearch(event) {
    if (event.key === 'Enter') {
        const searchTerm = document.getElementById('searchInput').value;
        // Búsqueda parcial case-insensitive
        // Actualiza contador de resultados
    }
}
```

### Aplicación de Filtros
```javascript
function applyFilters() {
    const estado = document.getElementById('filterEstado').value;
    const tipo = document.getElementById('filterTipo').value;
    const cliente = document.getElementById('filterCliente').value;

    // Combina filtros
    // Actualiza tabla y contador
}
```

### Ordenamiento de Tabla
```javascript
function sortTable(column) {
    // Toggle ascendente/descendente
    // Actualiza indicador visual (↑/↓)
    // Reordena filas
}
```

### Popover de Permisos
```javascript
function showPermissions(event, userId) {
    // Obtiene permisos del usuario
    // Muestra máximo 5
    // Si >5, agrega enlace "Ver todos"
    // Posiciona popover relativo al enlace
    // Click fuera cierra popover
}
```

### Menú Contextual
```javascript
function showContextMenu(event, userId) {
    // Posiciona menú relativo al botón
    // Opciones dinámicas según estado del usuario
    // Click fuera cierra menú
}
```

---

## 📊 Datos Mock para Testing

### Usuarios de Ejemplo

| Número ID | Nombre | Email | Tipo | Estado | Permisos |
|-----------|--------|-------|------|--------|----------|
| 123456789 | Juan Carlos Pérez Gómez | juan.perez@cdnfacturacion.com | Interno | Activo | 5 |
| 987654321 | María Fernanda López Torres | maria.lopez@empresaabc.com | Cliente | Activo | 2 |
| 555888999 | Carlos Eduardo Ramírez Silva | No registrado ⚠ | Cliente | Inactivo | 1 |
| 111222333 | Ana Patricia Morales Díaz | ana.morales@empresaxyz.com | Cliente | Bloqueado | 3 |
| 444555666 | Roberto José Hernández Castro | roberto.hernandez@cdnfacturacion.com | Interno | Activo | 8 |

### Permisos por Usuario

**Usuario 1 (5 permisos):**
- Empresa ABC - Administrador de Cliente
- Empresa ABC - Gestor Emisión FE
- Empresa XYZ - Consultor
- Administrador del Portal
- Auditor Interno

**Usuario 5 (8 permisos - muestra "Ver todos"):**
- Administrador del Portal
- Empresa ABC - Administrador de Cliente
- Empresa ABC - Gestor Emisión FE
- Empresa XYZ - Administrador de Cliente
- Empresa XYZ - Gestor Emisión FE
- Auditor Interno
- Soporte Técnico
- Gestor de Reportes

---

## ✅ Cumplimiento de la HU005

| Requerimiento | Escenario | Estado |
|---------------|-----------|--------|
| Menú "Gestión de Usuarios" | 1 | ✅ |
| Visualización de pantalla principal | 2 | ✅ |
| Listado con paginación (20 por defecto) | 3 | ✅ |
| Búsqueda por texto general | 4 | ✅ |
| Filtro por Estado | 5 | ✅ |
| Filtro por Tipo de Usuario | 6 | ✅ |
| Filtro por Cliente | 7 | ✅ |
| Limpiar filtros | 8 | ✅ |
| Información básica en tabla | 9 | ✅ |
| Ordenamiento por columnas | 10 | ✅ |
| Permisos asignados con popover | 11 | ✅ |
| Navegación a detalle | 12 | ✅ |
| Navegación a edición | 13 | ✅ |
| Menú de más opciones | 14 | ✅ |
| Cambio registros por página | 15 | ✅ |
| Navegación entre páginas | 16 | ✅ |
| Indicador sin correo electrónico | 17 | ✅ |
| Indicador usuario bloqueado | 18 | ✅ |
| Botón "Crear Nuevo Usuario" | 19 | ✅ |
| Exportación a CSV/Excel | 20 | ✅ |
| Estado sin usuarios | 21 | ✅ |
| Búsqueda sin resultados | 22 | ✅ |
| Validación de permisos | 23 | ✅ |
| Refresh manual | 24 | ✅ |

---

## 🚀 Cómo Probar el Mockup

### Búsqueda:
1. Escribir "Juan" en el buscador
2. Presionar Enter
3. Ver contador actualizado: "Mostrando 2 resultados para 'Juan'"

### Filtros:
1. Seleccionar Estado: "Bloqueado"
2. Hacer clic en cualquier parte (auto-apply)
3. Ver tabla filtrada

### Limpiar Filtros:
1. Aplicar varios filtros
2. Hacer clic en "Limpiar Filtros"
3. Ver todo reseteo a "Todos"

### Permisos:
1. Hacer clic en "5 permisos" de Juan Carlos Pérez
2. Ver popover con lista de permisos
3. Hacer clic en "8 permisos" de Roberto José Hernández
4. Ver popover con 5 permisos + "Ver todos (8 permisos)"
5. Click fuera para cerrar

### Menú Contextual:
1. Hacer clic en icono de 3 puntos verticales
2. Ver menú desplegable
3. Hacer clic en "Resetear Contraseña"
4. Ver confirmación
5. Hacer clic en "Eliminar Usuario"
6. Ver advertencia en rojo

### Ordenamiento:
1. Hacer clic en "Nombre Completo"
2. Ver indicador ↑ (ascendente)
3. Hacer clic nuevamente
4. Ver indicador ↓ (descendente)

### Paginación:
1. Cambiar selector a "50" registros por página
2. Ver alerta de confirmación
3. Botones Anterior/Siguiente (deshabilitados en demo con solo 5 usuarios)

### Acciones:
1. Hacer clic en icono de ojo (Ver Detalle)
2. Ver alerta con número de ID
3. Hacer clic en icono de lápiz (Editar)
4. Ver navegación simulada

---

## 📝 Notas Técnicas

### Responsive Design
- ✅ Filtros en grid responsive (columnas automáticas)
- ✅ Tabla con scroll horizontal en móviles
- ✅ Contador de resultados adaptable
- ✅ Breakpoint: 768px

### Integración con Backend (Futura)

**Endpoints esperados:**
```
GET /api/usuarios?page=1&limit=20&search=juan&estado=activo&tipo=cliente
GET /api/usuarios/:id
GET /api/usuarios/:id/permisos
POST /api/usuarios/:id/resetear-password
PUT /api/usuarios/:id/estado
DELETE /api/usuarios/:id
GET /api/usuarios/export?formato=csv&filtros={...}
```

### Persistencia de Estado

**Query Parameters recomendados:**
```
/admin/usuarios?page=2&limit=50&search=juan&estado=activo&tipo=cliente&orden=nombre&dir=asc
```

Permite:
- ✅ Compartir URL con filtros
- ✅ Botón "Volver" del navegador mantiene contexto
- ✅ Bookmarks de búsquedas frecuentes

---

**Creado para:** Portal Unificado de CDN Facturación
**Historia de Usuario:** HU005 - Visualización y Búsqueda de Usuarios
**Versión:** 1.0
**Fecha:** Diciembre 2025
**Diseñador:** UI/UX Designer (Claude Code)
