# Registro de Cambios del Contexto

## 2026-01-22 - Actualización Mayor: Enfoque de Desarrollo Incremental

### 🎯 Objetivo de la Actualización
Cambiar el enfoque de las Historias de Usuario desde **funcionalidades completas** hacia **incrementos entregables pequeños** que priorizan:
- Facilidad de desarrollo
- Simplicidad
- Entrega continua (CI/CD)
- Atomicidad real

---

### 📝 Documentos Actualizados

#### 1. **Claude_instructions.md**
**Sección modificada:** "⚛️ ATOMIZACIÓN DE HISTORIAS DE USUARIO (OBLIGATORIO)" (líneas 100-158)

**Cambios principales:**
- ✅ Nuevo principio fundamental: HUs lo más pequeñas posible manteniendo valor entregable
- ✅ Cambio de criterio de atomización: De ">15 escenarios" a ">8 escenarios"
- ✅ Filosofía de Desarrollo Incremental: HUs orientadas a incrementos, NO a funcionalidades
- ✅ Nueva regla de oro: "Si puede dividirse en partes más pequeñas que aporten valor, debe dividirse"
- ✅ Introducción del concepto de "Vertical Slices" (rebanadas verticales)
- ✅ Estrategias de atomización más agresivas:
  - Prioridad al camino feliz
  - Una operación por HU (separar CRUD)
  - Separar validaciones complejas
  - Incrementar progresivamente la complejidad
- ✅ Nuevos anti-patrones documentados
- ✅ Ejemplo extendido de atomización (HU005 - Gestión de Usuarios con 8 HUs atomizadas)

**Antes:**
```
¿Cuándo atomizar? → Cuando tiene >15 escenarios
Enfoque: Funcionalidades completas atomizadas solo si son muy grandes
```

**Después:**
```
¿Cuándo atomizar? → Cuando tiene >8 escenarios, múltiples pantallas,
                     múltiples operaciones, etc.
Enfoque: Incrementos entregables pequeños (1-2 días desarrollo)
Filosofía: Vertical Slices que atraviesan todas las capas pero con alcance mínimo
```

---

#### 2. **HU plantilla.md**
**Secciones agregadas:**

**Nueva sección: "Dependencias"**
- Para indicar qué HUs deben completarse antes
- Para listar qué HUs dependen de esta
- Para mencionar integraciones o sistemas externos necesarios

**Nueva sección: "Fuera de Alcance"**
- Para clarificar explícitamente qué NO incluye la HU atómica
- Para indicar qué aspectos se cubrirán en otras HUs
- Para justificar por qué ciertos elementos están fuera de alcance

**Justificación:**
Estas secciones son CRÍTICAS para HUs atómicas porque:
- Definen claramente el perímetro del incremento
- Evitan ambigüedades sobre qué debe implementarse
- Facilitan la planificación de dependencias
- Permiten priorización flexible

---

#### 3. **Guia_Vertical_Slices_Desarrollo_Incremental.md** (NUEVO)
**Documento completamente nuevo** con:

**Contenido:**
1. **Filosofía de Incrementos vs Funcionalidades**
   - Comparación enfoque tradicional vs incremental
   - Explicación del concepto de Vertical Slice

2. **5 Principios de Atomización Incremental**
   - Prioridad al Camino Feliz
   - Una Operación por HU
   - Separar Validaciones Complejas
   - Incrementar Progresivamente la Complejidad
   - Separar Interfaz de Lógica de Negocio Crítica

3. **5 Estrategias de Descomposición**
   - Por Camino del Usuario
   - Por Nivel de Detalle
   - Por Rol o Permiso
   - Por Contexto de Datos

4. **2 Ejemplos Prácticos Completos**
   - Administración de Resoluciones de Facturación (1 HU → 6 HUs atómicas)
   - Integración con Active Directory (1 HU → 5 HUs atómicas)

5. **Plantilla para Pensar en Vertical Slices**
   - 6 preguntas guía para descomponer funcionalidades

6. **Checklist de Calidad de HU Atómica**
   - 9 criterios para validar correcta atomización

7. **Anti-Patrones Documentados**
   - HU por "Módulo Completo"
   - HU por "Capa Horizontal"
   - HU con Demasiados "Y"
   - HU "Técnica"

8. **Ejemplos de Transformación**
   - Antes y después de aplicar atomización

**Propósito:**
Este documento es la **guía práctica** para aplicar el nuevo enfoque. Complementa las instrucciones de Claude_instructions.md con ejemplos concretos y casos de uso reales del proyecto.

---

### 📊 Comparación del Enfoque Antiguo vs Nuevo

| Aspecto | Enfoque Antiguo | Enfoque Nuevo |
|---------|----------------|---------------|
| **Orientación** | Funcionalidades completas | Incrementos entregables |
| **Criterio de atomización** | >15 escenarios | >8 escenarios |
| **Tamaño típico de HU** | 2-3 semanas desarrollo | 1-2 días desarrollo |
| **Número de escenarios** | 10-20+ escenarios | 3-8 escenarios |
| **Enfoque** | Entregar módulo completo | Entregar slice vertical pequeño |
| **Priorización** | Difícil (todo o nada) | Flexible (HUs independientes) |
| **Feedback** | Tardío (al final) | Temprano (cada semana) |
| **Testing** | Complejo (muchos escenarios) | Simple (pocos escenarios) |
| **Riesgo** | Alto (mucho código junto) | Bajo (poco código por vez) |

---

### 🎯 Impacto en HUs Existentes

**HUs existentes NO han sido modificadas** (según instrucciones del usuario).

**HUs que se beneficiarían de re-atomización** (ejemplos):
- HU004 - Administración de Empresas (actualmente atomizada en A/B/C/D/E, podría refinarse más)
- HU005 - Visualización y Búsqueda de Usuarios (probablemente necesita atomización)
- HU006 - Creación de Usuarios y Asignación de Permisos (probablemente necesita atomización)
- HU009/HU0010 - Gestión de Resoluciones (podrían atomizarse más)
- HU-AD* - Serie de Active Directory (algunas HUs podrían consolidarse o re-dividirse)

**Próximos pasos:**
Usuario indicará específicamente qué HUs deben reestructurarse aplicando el nuevo enfoque.

---

### ✅ Documentos NO Modificados (no requerían cambios)

- **Glosario.md** - Términos de negocio (sin cambios necesarios)
- **Inventario de Roles y Permisos.md** - Catálogo de roles (sin cambios necesarios)
- **Estándar de Auditoría Funcional.md** - Lineamientos de auditoría (sin cambios necesarios)
- **GUIA_DE_ESTILOS.md** - Estilos visuales Material UI (sin cambios necesarios)
- **Tablas Maestras/** - Datos de referencia (sin cambios necesarios)

Estos documentos son de referencia y no están relacionados con el enfoque de atomización.

---

### 📚 Documentos de Contexto Actualizados

```
/Contexto/
├── Claude_instructions.md                             ✅ ACTUALIZADO
├── HU plantilla.md                                    ✅ ACTUALIZADO
├── Guia_Vertical_Slices_Desarrollo_Incremental.md    🆕 NUEVO
├── CHANGELOG_Contexto.md                              🆕 NUEVO (este archivo)
│
├── Glosario.md                                        ⚪ Sin cambios
├── Inventario de Roles y Permisos.md                 ⚪ Sin cambios
├── Estándar de Auditoría Funcional.md                ⚪ Sin cambios
├── GUIA_DE_ESTILOS.md                                ⚪ Sin cambios
├── APL03 - Estándar de Seguridad...csv               ⚪ Sin cambios
├── Guia de estilos/                                   ⚪ Sin cambios
└── tablas Maestras/                                   ⚪ Sin cambios
```

---

### 🚀 Cómo Usar el Nuevo Enfoque

**Para generar nuevas HUs:**
1. Leer **Claude_instructions.md** (enfoque obligatorio)
2. Consultar **Guia_Vertical_Slices_Desarrollo_Incremental.md** (ejemplos prácticos)
3. Usar la plantilla actualizada **HU plantilla.md** (con secciones de Dependencias y Fuera de Alcance)
4. Aplicar checklist de calidad de HU atómica

**Para reestructurar HUs existentes:**
1. Identificar la HU a reestructurar
2. Aplicar preguntas de la "Plantilla para Pensar en Vertical Slices"
3. Descomponer en HUs más pequeñas (1-2 días desarrollo cada una)
4. Definir dependencias claras entre HUs atomizadas
5. Especificar "Fuera de Alcance" en cada HU

---

### 📖 Referencias Rápidas

| Necesitas... | Consulta... |
|-------------|------------|
| Entender el enfoque obligatorio | **Claude_instructions.md** - Sección "Atomización" |
| Ver ejemplos prácticos de atomización | **Guia_Vertical_Slices_Desarrollo_Incremental.md** - Sección "Ejemplos Prácticos" |
| Saber cómo descomponer una funcionalidad | **Guia_Vertical_Slices_Desarrollo_Incremental.md** - Sección "Plantilla para Pensar en Vertical Slices" |
| Validar si una HU está bien atomizada | **Guia_Vertical_Slices_Desarrollo_Incremental.md** - Sección "Checklist de Calidad" |
| Crear una nueva HU | **HU plantilla.md** (con nuevas secciones) |

---

### 🎓 Principios Clave a Recordar

1. **Regla de Oro:** Si puede dividirse en partes más pequeñas que aporten valor, debe dividirse.

2. **Vertical Slices:** Atravesar todas las capas (UI, backend, datos) pero con alcance mínimo.

3. **Camino Feliz Primero:** La primera HU de una funcionalidad debe ser solo el camino feliz básico.

4. **Una Operación por HU:** Separar Listado, Creación, Edición, Eliminación en HUs diferentes.

5. **1-2 Días de Desarrollo:** Si una HU toma más tiempo, probablemente necesita atomizarse más.

6. **3-8 Escenarios:** Rango ideal de escenarios funcionales por HU (sin contar auditoría).

---

**Fin del Registro de Cambios**
