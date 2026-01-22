# Propuesta de Atomización: HU005 - Visualización y Búsqueda de Usuarios

**Fecha:** 2026-01-22
**Estado:** PROPUESTA PARA REVISIÓN
**HU Original:** HU005 - Visualización y Búsqueda de Usuarios

---

## 📊 Análisis de la HU005 Original

### Situación Actual
- **Total de escenarios:** 32 (23 funcionales + 9 auditoría)
- **Criterio de atomización:** >8 escenarios → ✅ REQUIERE ATOMIZACIÓN
- **Complejidad:** MUY ALTA (múltiples operaciones, 3 mockups diferentes)
- **Tiempo estimado:** 2-3 semanas de desarrollo (demasiado largo)
- **Operaciones mezcladas:** Listado, búsqueda, filtros, paginación, ordenamiento, navegación, detalle, acciones

### Problemas Identificados
1. ❌ **Demasiado grande:** Imposible entregar incrementalmente
2. ❌ **Mezcla múltiples operaciones:** Listado + Búsqueda + Filtros + Detalle + Acciones
3. ❌ **Testing complejo:** 32 escenarios dificultan pruebas exhaustivas
4. ❌ **Feedback tardío:** No se puede demostrar progreso hasta completar todo
5. ❌ **Énfasis en visualización elaborada:** Popover de permisos, menú contextual complejo, múltiples estados visuales

### Indicaciones del Usuario
> "Esta sección no es crítica, es informativa. Debemos priorizar **funcionalidad** sobre **visualización**."

**Implicaciones:**
- ✅ Simplificar visualizaciones complejas (popover de permisos, menú contextual)
- ✅ Enfocarse en que funcione, no en que sea visualmente elaborado
- ✅ Eliminar componentes UI no esenciales
- ✅ Priorizar datos correctos sobre presentación perfecta

---

## 🎯 Estrategia de Atomización Aplicada

Siguiendo el nuevo enfoque de **Vertical Slices** y **desarrollo incremental**:

### Principios Aplicados
1. ✅ **Prioridad al camino feliz:** Listado básico primero
2. ✅ **Una operación por HU:** Separar Listado, Búsqueda, Filtros
3. ✅ **Simplicidad:** Eliminar visualizaciones no esenciales
4. ✅ **Incrementos pequeños:** 1-2 días de desarrollo por HU
5. ✅ **3-8 escenarios por HU:** Rango ideal

### Resultado
**1 HU grande (32 escenarios)** → **5 HUs atómicas (4-7 escenarios cada una)**

---

## 📦 HUs Derivadas Propuestas

### **HU005A - Listado Básico de Usuarios**
**Prioridad:** 🔴 ALTA (Debe implementarse primero)
**Tiempo estimado:** 1-2 días

**Alcance:**
- Acceso al módulo de Gestión de Usuarios (menú)
- Visualización de tabla básica con usuarios activos
- Paginación simple (20 registros por página)
- Columnas esenciales: Número ID, Nombre Completo, Correo, Tipo, Estado
- Botón "Crear Nuevo Usuario" (navega a HU006)
- Validación de permisos (solo Administrador del Portal)
- Estado vacío (sin usuarios en el sistema)

**Escenarios incluidos:**
1. Acceso al módulo de Gestión de Usuarios (esc. 1)
2. Visualización de pantalla principal básica (esc. 2 simplificado)
3. Listado inicial con paginación (esc. 3)
4. Visualización de información básica en tabla (esc. 9 simplificado)
5. Botón "Crear Nuevo Usuario" (esc. 19)
6. Sistema sin usuarios registrados (esc. 20)
7. Validación de permiso de acceso (esc. 22)

**Total:** 7 escenarios funcionales + 2 auditoría

**Auditoría:**
- Auditoría de acceso a pantalla (esc. 26)
- Auditoría de acceso no autorizado (esc. 30)

**Simplificaciones aplicadas:**
- ❌ NO incluir columna "Permisos Asignados" con popover (no es esencial)
- ❌ NO incluir columna "Acciones" con múltiples iconos (se agregará en HU posterior si es necesario)
- ✅ Solo mostrar información básica: ID, Nombre, Correo, Tipo, Estado
- ✅ Paginación simple: solo "Anterior/Siguiente", sin selector de registros por página
- ✅ Ordenamiento predeterminado: fecha creación descendente (sin ordenamiento manual aún)

**Fuera de Alcance:**
- Búsqueda de usuarios (se cubre en HU005B)
- Filtros (se cubre en HU005C)
- Ordenamiento por columnas (se cubre en HU005D)
- Cambio de registros por página (se cubre en HU005D)
- Navegación a detalle o edición (se cubre en HU005E)
- Acciones sobre usuarios (se evaluará si es necesario)

**Mockup:**
- Versión simplificada de `HU005-GestionUsuarios.html`
- Sin buscador, sin filtros, sin columnas complejas
- Tabla básica con 5 columnas + paginación simple

---

### **HU005B - Búsqueda de Usuarios por Texto**
**Prioridad:** 🟡 MEDIA
**Tiempo estimado:** 1 día
**Dependencia:** HU005A

**Alcance:**
- Agregar campo de búsqueda general en la pantalla
- Búsqueda por: Número ID, Nombre, Apellido, Correo
- Búsqueda parcial, case-insensitive
- Actualización de tabla con resultados
- Contador de resultados
- Manejo de búsqueda sin resultados
- Botón para limpiar búsqueda

**Escenarios incluidos:**
1. Búsqueda de usuarios por texto general (esc. 4)
2. Búsqueda sin resultados (esc. 21)
3. Limpiar búsqueda (esc. 8 - parcial)
4. Mantener paginación con búsqueda activa

**Total:** 4 escenarios funcionales + 1 auditoría

**Auditoría:**
- Auditoría de búsqueda de usuarios (esc. 27)

**Simplificaciones aplicadas:**
- ✅ Búsqueda simple por texto, sin búsqueda avanzada
- ✅ Botón explícito "Buscar" o Enter para ejecutar (no búsqueda en tiempo real)
- ✅ Contador simple: "X resultados encontrados"

**Fuera de Alcance:**
- Filtros por Estado/Tipo/Cliente (HU005C)
- Búsqueda con debounce en tiempo real (puede agregarse como mejora posterior)
- Resaltado de términos de búsqueda en resultados

**Mockup:**
- Agregar barra de búsqueda a mockup de HU005A
- Estado de "sin resultados"

---

### **HU005C - Filtros Manuales de Usuarios**
**Prioridad:** 🟡 MEDIA
**Tiempo estimado:** 1-2 días
**Dependencia:** HU005A

**Alcance:**
- Agregar dropdowns de filtros: Estado, Tipo de Usuario, Cliente
- Botón "Aplicar Filtros" (aplicación manual, NO automática)
- Botón "Limpiar Filtros"
- Combinación de múltiples filtros
- Indicadores visuales de filtros activos
- Mantener filtros al cambiar de página

**Escenarios incluidos:**
1. Selección de filtro por Estado (esc. 5)
2. Selección de filtro por Tipo de Usuario (esc. 6)
3. Selección de filtro por Cliente (esc. 7)
4. Aplicar filtros seleccionados (esc. 7a)
5. Limpiar filtros y búsqueda (esc. 8 completo)
6. Combinación de filtros con búsqueda (integración con HU005B)

**Total:** 6 escenarios funcionales + 1 auditoría

**Auditoría:**
- Auditoría de aplicación de filtros (esc. 28)

**Simplificaciones aplicadas:**
- ✅ Aplicación manual con botón (no aplicación automática en tiempo real)
- ✅ Filtros simples: dropdowns estándar
- ✅ Indicador visual básico de filtros activos (badge o texto)

**Fuera de Alcance:**
- Filtros avanzados adicionales (por rango de fechas, por permisos específicos, etc.)
- Guardado de combinaciones de filtros favoritas
- Persistencia de filtros entre sesiones

**Mockup:**
- Agregar panel de filtros a mockup de HU005A
- Botones "Aplicar Filtros" y "Limpiar Filtros"

---

### **HU005D - Ordenamiento y Paginación Avanzada**
**Prioridad:** 🟢 BAJA (Mejora de UX, no crítica)
**Tiempo estimado:** 1 día
**Dependencia:** HU005A

**Alcance:**
- Ordenamiento por columnas (click en encabezado)
- Indicador visual de ordenamiento (flechas)
- Ordenamiento ascendente/descendente
- Selector de registros por página (10, 20, 50, 100)
- Navegación directa a página específica
- Botón "Actualizar" (refresh manual)

**Escenarios incluidos:**
1. Ordenamiento por columnas (esc. 10)
2. Cambio de cantidad de registros por página (esc. 15)
3. Navegación entre páginas mejorada (esc. 16 extendido)
4. Refresh manual de la lista (esc. 23)

**Total:** 4 escenarios funcionales + 0 auditoría adicional

**Simplificaciones aplicadas:**
- ✅ Ordenamiento solo por columnas seleccionadas (ID, Nombre, Estado)
- ✅ Sin ordenamiento por múltiples columnas simultáneamente
- ✅ Sin auto-refresh, solo manual

**Fuera de Alcance:**
- Auto-refresh periódico (cada 30 segundos)
- Notificaciones en tiempo real de cambios
- Persistencia de preferencias de ordenamiento

**Mockup:**
- Agregar flechas de ordenamiento en encabezados de tabla
- Selector de registros por página en paginación
- Botón "Actualizar" en barra superior

**NOTA:** Esta HU puede posponerse si no es prioritaria para el negocio.

---

### **HU005E - Visualización Simplificada de Detalle de Usuario**
**Prioridad:** 🟡 MEDIA
**Tiempo estimado:** 1-2 días
**Dependencia:** HU005A

**Alcance (SIMPLIFICADO):**
- Icono "Ver Detalle" en tabla de usuarios
- Navegación a pantalla de detalle
- **Sección 1: Datos Personales** (card)
  - Número ID, Nombre completo, Correo, Tipo, Estado
  - Fecha de creación, Creado por
- **Sección 2: Información de Sesión** (card)
  - Último acceso, IP del último acceso
  - Total de accesos, Intentos fallidos actuales
- **Sección 3: Permisos Actuales** (card simplificada)
  - Lista simple de clientes asociados y roles
  - Sin historial de cambios (se puede agregar después si es necesario)
- Botón "Editar Usuario" (navega a HU006/HU008)
- Botón "Volver" (regresa a listado)

**Escenarios incluidos:**
1. Navegación a detalle de usuario (esc. 12)
2. Visualización de datos personales
3. Visualización de información de sesión
4. Visualización de permisos actuales (simplificado)
5. Navegación a edición (esc. 13)

**Total:** 5 escenarios funcionales + 1 auditoría

**Auditoría:**
- Auditoría de visualización de detalle (esc. 29)

**Simplificaciones aplicadas (según indicación del usuario):**
- ❌ **NO incluir Timeline de Historial de Permisos** (demasiado complejo, no esencial)
- ❌ **NO incluir Timeline de Historial de Cambios de Estado** (no esencial)
- ❌ **NO incluir tabla de Auditoría de Acciones del Usuario** (puede consultarse en módulo de auditoría)
- ✅ Solo mostrar información **actual y relevante**
- ✅ Diseño simple con cards Material UI básicas
- ✅ Enfoque: **información funcional**, no visualización elaborada

**Fuera de Alcance:**
- Historiales complejos con timelines (se pueden agregar en Release 2 si se requieren)
- Gráficos de actividad del usuario
- Estadísticas avanzadas
- Edición inline desde detalle

**Mockup:**
- Versión **muy simplificada** de `HU005-DetalleUsuario.html`
- Solo 3 cards: Datos Personales, Sesión, Permisos Actuales
- Sin timelines, sin tablas de auditoría, sin gráficos

---

## 🚫 Elementos ELIMINADOS por Simplicidad

Basado en la indicación de priorizar funcionalidad sobre visualización:

### ❌ NO se incluirán (al menos en primera versión):

1. **Popover de permisos en tabla** (esc. 11 original)
   - **Razón:** Complejidad visual no esencial, el detalle ya muestra permisos
   - **Alternativa:** Click en usuario → ver detalle completo

2. **Menú contextual "Más Opciones"** (esc. 14 original)
   - **Razón:** Complejidad UI innecesaria
   - **Alternativa:** Iconos directos simples o acciones desde detalle

3. **Modales de Cambio de Estado con razón obligatoria** (mockup HU005-CambiarEstadoUsuario.html)
   - **Razón:** Se puede manejar desde edición de usuario (HU008)
   - **Estado:** Se evaluará si es necesario en HU008 o como HU separada

4. **Indicador visual complejo de usuario bloqueado** (esc. 17 original)
   - **Razón:** El badge "Bloqueado" es suficiente
   - **Simplificación:** Solo badge rojo, sin tooltip elaborado

5. **Historial de cambios en detalle** (mockup original)
   - **Razón:** No es crítico para operación diaria
   - **Alternativa:** Consultar módulo de auditoría si se necesita

6. **Tabla de auditoría en detalle** (mockup original)
   - **Razón:** Duplica funcionalidad del módulo de auditoría
   - **Alternativa:** Enlace al módulo de auditoría filtrado por usuario

---

## 📅 Secuencia de Implementación Recomendada

### Sprint 1 (Semana 1)
1. **HU005A** - Listado Básico de Usuarios (1-2 días)
   - ✅ Demo: Ver tabla de usuarios paginada
   - ✅ Valor: Administrador puede visualizar usuarios del sistema

### Sprint 2 (Semana 2)
2. **HU005B** - Búsqueda de Usuarios (1 día)
   - ✅ Demo: Buscar un usuario específico
   - ✅ Valor: Localizar usuarios rápidamente
3. **HU005C** - Filtros Manuales (1-2 días)
   - ✅ Demo: Filtrar usuarios por estado/tipo
   - ✅ Valor: Analizar subconjuntos de usuarios

### Sprint 3 (Semana 3)
4. **HU005E** - Detalle Simplificado (1-2 días)
   - ✅ Demo: Ver información completa de un usuario
   - ✅ Valor: Consultar datos y permisos de un usuario
5. **HU005D** - Ordenamiento y Paginación Avanzada (1 día) *[OPCIONAL]*
   - ✅ Demo: Ordenar tabla, cambiar registros por página
   - ✅ Valor: Mejorar experiencia de navegación

**Total:** 3 semanas con entregas semanales vs 2-3 semanas sin entregas

---

## 📊 Comparación: Antes vs Después

| Aspecto | HU005 Original | HUs Atomizadas |
|---------|---------------|----------------|
| **Total HUs** | 1 | 5 |
| **Escenarios por HU** | 32 | 4-7 promedio |
| **Tiempo de desarrollo** | 2-3 semanas | 1-2 días por HU |
| **Complejidad testing** | MUY ALTA | BAJA-MEDIA |
| **Entregas demostrables** | 1 (al final) | 5 (semanales) |
| **Feedback del negocio** | Tardío (semana 3) | Temprano (semana 1) |
| **Riesgo** | Alto (todo o nada) | Bajo (incremental) |
| **Priorización flexible** | No | Sí |
| **Enfoque visual** | Elaborado (popovers, menús, timelines) | Simplificado (funcional) |

---

## ✅ Ventajas de Esta Atomización

1. **Entregas semanales:** Demo funcional cada semana
2. **Feedback temprano:** Validar con negocio desde semana 1
3. **Testing más simple:** 4-7 escenarios por HU = pruebas exhaustivas
4. **Desarrollo más rápido:** Enfoque en una operación a la vez
5. **Priorización flexible:** Si filtros no son urgentes, se pueden posponer
6. **Simplicidad:** Funcionalidad sobre visualización (según indicación)
7. **Menor riesgo:** Problemas detectados tempranamente
8. **CI/CD habilitado:** Deploys semanales de incrementos reales

---

## 🎯 Recomendación Final

**Implementar las 5 HUs atomizadas en el orden propuesto:**

**CRÍTICAS (implementar primero):**
- ✅ **HU005A** - Listado Básico (OBLIGATORIA)
- ✅ **HU005B** - Búsqueda (MUY ÚTIL)

**IMPORTANTES (implementar después):**
- ✅ **HU005C** - Filtros (ÚTIL)
- ✅ **HU005E** - Detalle Simplificado (IMPORTANTE)

**OPCIONALES (evaluar necesidad):**
- ⚪ **HU005D** - Ordenamiento Avanzado (MEJORA UX)

**Elementos ELIMINADOS permanentemente** (o para Release 2):
- ❌ Popover de permisos en tabla
- ❌ Menú contextual complejo
- ❌ Historiales con timelines elaborados
- ❌ Tabla de auditoría en detalle

---

## 🚀 Próximos Pasos

1. **Revisar esta propuesta** con el equipo y Product Owner
2. **Aprobar el enfoque simplificado** (funcionalidad > visualización)
3. **Crear las 5 HUs atomizadas** a partir de HU005
4. **Mover HU005 original** a carpeta "Obsoletas"
5. **Actualizar mockups** para reflejar la simplicidad requerida
6. **Comenzar desarrollo** con HU005A

---

**Fin de la Propuesta de Atomización HU005**
