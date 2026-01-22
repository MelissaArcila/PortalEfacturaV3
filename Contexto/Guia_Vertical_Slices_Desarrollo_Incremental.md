# Guía de Vertical Slices y Desarrollo Incremental

**Versión:** 1.0
**Fecha:** 2026-01-22
**Estado:** ACTIVO

---

## 📋 Propósito de esta Guía

Esta guía establece los principios y prácticas para descomponer funcionalidades en **incrementos entregables pequeños** (Vertical Slices), priorizando **facilidad de desarrollo**, **simplicidad** y **entrega continua**.

---

## 🎯 Filosofía: Incrementos, No Funcionalidades

### ❌ Enfoque Tradicional (Orientado a Funcionalidades)
- Construir una funcionalidad completa antes de entregar
- Ejemplo: "HU004 - Módulo de Administración de Empresas" con 20+ escenarios
- Resultado: Desarrollo largo (semanas), testing complejo, feedback tardío

### ✅ Enfoque Incremental (Orientado a Valor)
- Construir el incremento más pequeño que agregue valor verificable
- Ejemplo: "HU004A - Listado básico de empresas (solo lectura)" con 3-5 escenarios
- Resultado: Desarrollo rápido (1-2 días), testing simple, feedback temprano

---

## 🧩 ¿Qué es un Vertical Slice?

Un **Vertical Slice** (rebanada vertical) es un incremento que:
- ✅ Atraviesa **todas las capas necesarias** del sistema (UI, lógica de negocio, datos)
- ✅ Entrega **una capacidad completa** pero con alcance mínimo
- ✅ Es **independiente** y puede desplegarse por sí solo
- ✅ Aporta **valor demostrable** al usuario o negocio

### Metáfora Visual
Imagina un pastel de capas (UI, Backend, Base de Datos):
- ❌ **Horizontal:** Construir toda la UI, luego todo el Backend, luego toda la BD
- ✅ **Vertical:** Cortar una rebanada delgada que incluya un poco de cada capa

---

## 📏 Principios de Atomización Incremental

### 1. Prioridad al Camino Feliz
**Regla:** La primera HU de una funcionalidad debe ser SOLO el camino feliz básico.

**Ejemplo: Gestión de Usuarios**
- **Primera HU:** HU005A - "Listado básico de usuarios activos"
  - Escenarios: Visualizar tabla, paginación básica, ordenar por nombre
  - Sin filtros, sin búsqueda, sin manejo de errores complejos
- **HUs posteriores:** Agregar búsqueda, filtros, errores, etc.

### 2. Una Operación por HU
**Regla:** Separar CRUD en HUs independientes.

**Ejemplo: Administración de Empresas**
- HU004A - Listado (Read)
- HU004B - Crear empresa: datos básicos (Create - parte 1)
- HU004C - Crear empresa: datos complementarios (Create - parte 2)
- HU004D - Editar empresa (Update)
- HU004E - Inactivar empresa (Delete/Soft Delete)

### 3. Separar Validaciones Complejas
**Regla:** Si una validación tiene lógica de negocio compleja, puede ser una HU separada.

**Ejemplo: Creación de Usuario**
- HU006A - Crear usuario: formulario básico (validaciones simples: requeridos, formato email)
- HU006B - Crear usuario: validación de unicidad contra AD (validación compleja de integración)
- HU006C - Crear usuario: asignación de permisos (lógica adicional)

### 4. Incrementar Progresivamente la Complejidad
**Regla:** Construir de lo simple a lo complejo, no todo a la vez.

**Ejemplo: Búsqueda de Documentos**
- HU Básica: Búsqueda por un solo campo (ej: número de documento)
- HU Intermedia: Búsqueda por múltiples campos
- HU Avanzada: Búsqueda con filtros combinados y rangos de fechas

### 5. Separar Interfaz de Lógica de Negocio Crítica
**Regla:** Si la lógica de negocio es compleja, puede tener su propia HU de backend antes de la UI.

**Ejemplo: Cálculo de Impuestos**
- HU010A - Lógica de cálculo de impuestos (backend funcional, testeable vía API)
- HU010B - Interfaz de visualización de impuestos calculados (UI que consume la lógica)

---

## 🚀 Estrategias de Descomposición

### Estrategia 1: Por Camino del Usuario
Dividir según el flujo que sigue un usuario paso a paso.

**Ejemplo: Recuperación de Contraseña**
- HU002A - Usuario solicita recuperación (pantalla inicial, envío email)
- HU002B - Usuario hace clic en enlace (validación token, pantalla de restablecimiento)
- HU002C - Usuario establece nueva contraseña (cambio efectivo, confirmación)

### Estrategia 2: Por Nivel de Detalle
Construir primero la versión básica, luego agregar detalles.

**Ejemplo: Listado de Facturas**
- HU011A - Listado básico (tabla con columnas esenciales, paginación)
- HU011B - Agregar columnas adicionales configurables
- HU011C - Agregar exportación a Excel/PDF

### Estrategia 3: Por Rol o Permiso
Dividir funcionalidad según diferentes roles que la usan.

**Ejemplo: Dashboard**
- HU012A - Dashboard para Administrador de Cliente (métricas de su cliente)
- HU012B - Dashboard para Administrador de Portal (métricas globales)

### Estrategia 4: Por Contexto de Datos
Dividir según el alcance de los datos que maneja.

**Ejemplo: Reportes**
- HU013A - Reporte de documentos emitidos: un solo cliente
- HU013B - Reporte de documentos emitidos: múltiples clientes (para admin portal)

---

## 📊 Ejemplos Prácticos de Atomización

### Ejemplo 1: Administración de Resoluciones de Facturación

**Funcionalidad Completa (NO HACER ESTO):**
```
HU009 - Gestión de Resoluciones de Facturación
- 25+ escenarios que incluyen:
  - Listado con filtros complejos
  - Creación con múltiples validaciones DIAN
  - Edición de rangos
  - Activación/Inactivación
  - Consulta de estado ante DIAN
  - Auditoría completa
```

**Enfoque Incremental (HACER ESTO):**
```
HU009A - Listado básico de resoluciones activas
  - Visualizar tabla con columnas esenciales
  - Paginación simple
  - Ordenar por fecha de creación
  (3-4 escenarios, 1 día desarrollo)

HU009B - Crear resolución: datos básicos obligatorios
  - Formulario con campos mínimos requeridos por DIAN
  - Validaciones de formato
  - Guardar en estado "Borrador"
  (5-6 escenarios, 1 día desarrollo)

HU009C - Crear resolución: validación ante DIAN
  - Validar resolución contra servicio DIAN
  - Actualizar estado según respuesta DIAN
  - Manejo de errores de validación
  (6-8 escenarios, 2 días desarrollo)

HU009D - Activar/Inactivar resolución
  - Cambiar estado de resolución
  - Validaciones de negocio (no inactivar si hay facturas asociadas)
  - Auditoría del cambio
  (4-5 escenarios, 1 día desarrollo)

HU009E - Consultar estado de resolución ante DIAN
  - Botón de consulta manual
  - Actualizar información desde DIAN
  - Visualizar histórico de consultas
  (4 escenarios, 1 día desarrollo)

HU009F - Filtros avanzados de resoluciones
  - Filtrar por estado, prefijo, rango de fechas
  - Combinar múltiples filtros
  - Limpiar filtros
  (5-6 escenarios, 1 día desarrollo)
```

**Beneficios:**
- Cada HU se completa en 1-2 días
- Se puede hacer demo cada semana con algo funcionando
- Testing más simple y enfocado
- Priorización flexible (ej: si HU009E no es crítica, puede posponerse)

---

### Ejemplo 2: Integración con Active Directory

**Funcionalidad Completa (NO HACER ESTO):**
```
HU-AD001 - Integración SCIM con Active Directory
- 30+ escenarios cubriendo:
  - Configuración completa
  - Sincronización bidireccional
  - Mapeo de roles
  - Manejo de errores
  - Auditoría
  - Monitoreo
```

**Enfoque Incremental (HACER ESTO):**
```
HU-AD001A - Configurar endpoint SCIM básico
  - Recibir peticiones SCIM de AD
  - Autenticar peticiones con token
  - Retornar respuesta válida (aunque vacía)
  (4 escenarios, 1 día desarrollo)

HU-AD001B - Sincronizar usuarios desde AD (solo lectura)
  - Leer usuarios desde AD vía SCIM
  - Guardar en tabla temporal
  - Visualizar usuarios sincronizados
  (5 escenarios, 1-2 días desarrollo)

HU-AD001C - Mapeo básico de roles AD a roles del Portal
  - Configurar equivalencias (tabla de mapeo)
  - Asignar rol del Portal según grupo AD
  - Aplicar al crear usuario
  (6 escenarios, 1 día desarrollo)

HU-AD001D - Sincronización automática programada
  - Ejecutar sincronización cada N horas
  - Detectar usuarios nuevos/modificados/eliminados
  - Actualizar estado en el Portal
  (7 escenarios, 2 días desarrollo)

HU-AD001E - Manejo de errores de sincronización
  - Detectar errores de conexión con AD
  - Reintentar automáticamente
  - Notificar administrador si falla
  - Log de errores
  (8 escenarios, 1-2 días desarrollo)
```

---

## 🎨 Plantilla para Pensar en Vertical Slices

Cuando recibas una funcionalidad para descomponer, responde estas preguntas:

### 1. ¿Cuál es el camino feliz más simple?
- ¿Qué es lo MÍNIMO que un usuario debe poder hacer?
- ¿Puedo construir solo eso primero?

### 2. ¿Puedo separar operaciones?
- ¿Hay CRUD involucrado? → Separar en HUs diferentes
- ¿Hay listado + detalle? → Separar en HUs diferentes

### 3. ¿Puedo separar validaciones?
- ¿Hay validaciones simples (requeridos, formato)? → Primera HU
- ¿Hay validaciones complejas (contra servicios externos, lógica de negocio)? → HUs separadas

### 4. ¿Puedo construir sin filtros/búsqueda primero?
- Listado básico sin filtros → Primera HU
- Agregar búsqueda → Segunda HU
- Agregar filtros avanzados → Tercera HU

### 5. ¿Puedo separar por rol?
- ¿Diferentes roles ven/hacen cosas diferentes? → HUs separadas por rol

### 6. ¿Puedo posponer el manejo de errores complejos?
- Camino feliz + errores básicos → Primera HU
- Errores complejos y edge cases → HUs posteriores

---

## ✅ Checklist de Calidad de una HU Atómica

Una HU está bien atomizada si cumple:

- [ ] Tiene **≤8 escenarios funcionales** (sin contar auditoría)
- [ ] Se puede **desarrollar en 1-2 días** máximo
- [ ] Es **demostrable** al negocio de forma independiente
- [ ] Agrega **valor verificable** aunque sea mínimo
- [ ] Tiene **una sola responsabilidad** clara
- [ ] **No mezcla** múltiples operaciones (ej: crear + editar)
- [ ] El título describe **claramente el incremento específico**
- [ ] Las **dependencias** con otras HUs están claras
- [ ] El **"Fuera de Alcance"** está bien definido

---

## 🚨 Anti-Patrones a Evitar

### ❌ Anti-Patrón 1: HU por "Módulo Completo"
```
HU005 - Gestión de Usuarios
- Listado, búsqueda, filtros, creación, edición, eliminación, permisos, auditoría
- 25+ escenarios
- 2-3 semanas de desarrollo
```

**Por qué es malo:** Imposible de entregar incrementalmente, testing complejo, feedback tardío.

### ❌ Anti-Patrón 2: HU por "Capa Horizontal"
```
HU004A - Modelo de datos de Empresas
HU004B - API REST de Empresas
HU004C - Interfaz de Administración de Empresas
```

**Por qué es malo:** Ninguna HU es demostrable por sí sola, no aporta valor hasta completar las 3.

### ❌ Anti-Patrón 3: HU con Demasiados "Y"
```
HU006 - Crear usuario y asignar permisos y enviar correo de bienvenida y generar contraseña temporal
```

**Por qué es malo:** Múltiples responsabilidades, difícil de estimar y testear.

### ❌ Anti-Patrón 4: HU "Técnica"
```
HU-TECH001 - Implementar JWT para autenticación
```

**Por qué es malo:** Es una decisión técnica, no una historia de usuario funcional.

---

## 📈 Beneficios del Enfoque Incremental

### Para el Negocio
- ✅ **Feedback temprano:** Ver progreso cada semana
- ✅ **Flexibilidad:** Cambiar prioridades fácilmente
- ✅ **Menor riesgo:** Detectar problemas antes
- ✅ **ROI temprano:** Usar funcionalidad parcial mientras se completa

### Para el Equipo de Desarrollo
- ✅ **Menor complejidad:** Enfocarse en una cosa a la vez
- ✅ **Mejor estimación:** HUs pequeñas son más predecibles
- ✅ **Menos bugs:** Código más simple, menos errores
- ✅ **CI/CD habilitado:** Deploys frecuentes de incrementos

### Para QA
- ✅ **Testing más exhaustivo:** Menos escenarios = más cobertura
- ✅ **Detección temprana:** Bugs encontrados en días, no semanas
- ✅ **Regresión más simple:** Menos código que validar

---

## 🎓 Ejemplos de Transformación

### Transformación 1: De Funcionalidad a Incrementos

**Antes:**
```
HU - Gestión de Resoluciones de Facturación
```

**Después:**
```
HU009A - Listado básico de resoluciones
HU009B - Crear resolución: datos básicos
HU009C - Crear resolución: validar con DIAN
HU009D - Activar/Inactivar resolución
HU009E - Consultar estado ante DIAN
HU009F - Editar resolución: ajustar rangos
HU009G - Filtros avanzados de resoluciones
```

### Transformación 2: De Monolito a Slices

**Antes:**
```
HU - Autenticación y Gestión de Sesión
```

**Después:**
```
HU001A - Autenticación básica (usuario/contraseña, un solo cliente)
HU001B - Selección de cliente (múltiples clientes)
HU001C - Bloqueo por intentos fallidos
HU001D - Desbloqueo automático
HU001E - Gestión de sesión con JWT
HU001F - Cierre de sesión
```

---

## 🔗 Referencias Adicionales

- **Claude_instructions.md** - Sección de Atomización (líneas 100+)
- **HU plantilla.md** - Plantilla estándar para todas las HUs
- **Glosario.md** - Términos de negocio
- **Inventario de Roles y Permisos.md** - Catálogo de roles y permisos

---

## 📝 Notas Finales

**Recuerda:** El objetivo NO es escribir menos HUs, sino escribir HUs más pequeñas y entregables.

**Mantra:** "¿Puedo dividir esto en algo más pequeño que aún aporte valor?" → Si la respuesta es SÍ, divídelo.

**Resultado esperado:** Entregas semanales de incrementos funcionales pequeños, no entregas mensuales de funcionalidades completas.

---

**Fin de la Guía de Vertical Slices y Desarrollo Incremental**
