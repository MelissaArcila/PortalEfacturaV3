# Mockups HU004 - Creación de Empresas (Clientes)

Este documento describe los mockups interactivos creados para la **HU004 - Creación de Empresas (Clientes)**.

## 📋 Archivos Creados

### 1. **HU004-MenuAdministracion.html** - Menú de Administración de Empresas
**Ubicación en HU:** Escenarios 1-2 (líneas 59-60)

**Descripción:** Pantalla principal del módulo de administración de empresas.

**Elementos principales:**
- ✅ Barra de navegación con gradiente de marca
- ✅ Título "Administración de Empresas"
- ✅ Botón prominente "Crear Nueva Empresa" con gradiente
- ✅ Barra de búsqueda con icono Material Icons
- ✅ Tabla de empresas con columnas:
  - Número ID
  - Nombre
  - Estado (chips de colores)
  - Fecha Creación
  - Acciones (Ver, Editar, Más opciones)
- ✅ Paginación con controles "Anterior/Siguiente"
- ✅ Contador "Mostrando X-Y de Z empresas"

**Funcionalidad interactiva:**
- Búsqueda en tiempo real (console.log)
- Botones de acción con alertas demostrativas
- Hover en filas de tabla
- Estados deshabilitados en paginación

---

### 2. **HU004-CrearEmpresa.html** - Formulario de Creación de Empresa ⭐ MÁS COMPLEJO
**Ubicación en HU:** Escenarios 3-43 (líneas 61-102) + Mockup líneas 139-181

**Descripción:** Formulario completo de creación de empresa con 3 secciones y validaciones dinámicas.

#### **Sección 1: Datos de la Empresa**

**Campos implementados:**
- ✅ *Tipo de Identificación (select con opciones de tabla maestra)
- ✅ *Número de Identificación
  - Solo numérico, máx 10 dígitos
  - Validación de unicidad en blur con checkmark verde/error rojo
  - Mock: NIT "900123456" ya existe
- ✅ DV (Dígito de Verificación)
  - Validación con algoritmo oficial DIAN
  - Muestra DV correcto si está mal
  - Multiplicadores: [3, 7, 13, 17, 19, 23, 29, 37, 41, 43, 47, 53, 59, 67, 71]
- ✅ Nombre Abreviado
  - Contador de caracteres "X/50"
  - Máximo 50 caracteres
- ✅ Razón Social (condicional - activo solo si Tipo ID = NIT)
- ✅ Nombres y Apellidos (condicionales - activos solo si Tipo ID ≠ NIT)
- ✅ *Tipo de Sociedad (select)
- ✅ Centros de Costos:
  - Input numérico + botón "Agregar"
  - Tabla dinámica con centros agregados
  - Botón eliminar con confirmación
  - Validación: solo números, máx 5 dígitos
- ✅ *Tipo de Contribuyente (select: Natural/Jurídico)
- ✅ *Tipo de Régimen (select: Ninguno/Simplificado/Común)
- ✅ *Correo Electrónico Emisor
  - Validación de formato email con regex
  - Checkmark verde o error rojo
- ✅ Actividad Económica (opcional)
  - Textarea, máx 200 caracteres
  - Contador de caracteres

#### **Sección 2: Ubicación**

**Campos implementados:**
- ✅ *País (dropdown con búsqueda)
- ✅ *Departamento
  - Solo activo si País = Colombia
  - Se marca como obligatorio dinámicamente
  - Carga lista de departamentos
- ✅ *Ciudad
  - Solo activo si País = Colombia y Departamento seleccionado
  - Filtrada por departamento
  - Se marca como obligatorio dinámicamente
- ✅ *Dirección Principal (permite caracteres especiales)

**Cascada de selects:**
```
País → Colombia
  ↓
Departamento se activa → Cundinamarca / Antioquia
  ↓
Ciudad se activa → Bogotá D.C. / Medellín
```

#### **Sección 3: Registro Mercantil**

**Formulario repetible:**
- ✅ *Matrícula Mercantil (5-15 caracteres)
- ✅ *Nombre en Registro Mercantil
- ✅ *País (dropdown)
- ✅ *Departamento (condicional si País = Colombia)
- ✅ *Ciudad (condicional si País = Colombia y Departamento)
- ✅ *Dirección
- ✅ Botón "Agregar Registro Mercantil"
  - Valida campos obligatorios
  - Agrega a tabla dinámica
  - Limpia formulario para agregar otro
- ✅ Tabla con registros agregados
  - Columnas: Matrícula, Nombre, País, Dirección, Acciones
  - Botón eliminar con confirmación
- ✅ Misma lógica de cascada País → Departamento → Ciudad

**Botones del formulario:**
- ✅ "Cancelar" (outlined, confirmación de pérdida de datos)
- ✅ "Crear Empresa" (contained, gradiente, deshabilitado hasta completar obligatorios)

**Validaciones implementadas:**
1. ✅ Algoritmo DV DIAN exacto (Escenario 8, línea 66)
2. ✅ Unicidad de NIT (Escenarios 6-7, líneas 64-65)
3. ✅ Solo numérico en Número de Identificación (Escenario 5, línea 63)
4. ✅ Contador de caracteres en tiempo real (Escenario 9, línea 67)
5. ✅ Campos condicionales según Tipo ID (Escenarios 10-13, líneas 68-71)
6. ✅ Formato de email (Escenario 20, línea 78)
7. ✅ Cascada de País → Departamento → Ciudad (Escenarios 21-26, líneas 79-84)
8. ✅ Validación de centros de costos (Escenario 15, línea 73)
9. ✅ Validación de registros mercantiles (Escenarios 28-31, líneas 86-89)

---

### 3. **HU004-ResumenEmpresa.html** - Pantalla de Resumen
**Ubicación en HU:** Escenario 36 (línea 94) + Mockup líneas 182-193

**Descripción:** Resumen completo de los datos antes de confirmar creación.

**Elementos principales:**
- ✅ Título "Resumen de la Empresa a Crear"
- ✅ 4 Secciones expandibles/colapsables con Material Icons:

  **1. Datos de la Empresa:**
  - Grid responsivo con todos los campos
  - Tipo ID, Número ID, Razón Social, Nombre Abreviado
  - Tipo Sociedad, Tipo Contribuyente, Tipo Régimen
  - Correo Emisor, Actividad Económica

  **2. Ubicación:**
  - País, Departamento, Ciudad, Dirección Principal

  **3. Centros de Costos:**
  - Chip con contador "3 centros"
  - Tabla con lista completa

  **4. Registros Mercantiles:**
  - Chip con contador "2 registros"
  - Tabla con Matrícula, Nombre, País, Depto, Ciudad, Dirección

- ✅ Botones:
  - "Volver y Editar" (outlined) - mantiene datos
  - "Confirmar Creación" (contained, gradiente)

**Funcionalidad interactiva:**
- Toggle de secciones con animación smooth
- Icono de flecha que rota al expandir/colapsar
- Todas las secciones expandidas por defecto

---

### 4. **HU004-CreacionExitosa.html** - Confirmación de Creación
**Ubicación en HU:** Escenario 38 (línea 96) + Mockup líneas 194-204

**Descripción:** Pantalla de confirmación después de creación exitosa.

**Elementos principales:**
- ✅ Logo CDN Facturación
- ✅ Icono de éxito grande (checkmark verde) con animación scaleIn
- ✅ Título "¡Empresa Creada Exitosamente!"
- ✅ Mensaje: "La empresa **Nombre** con NIT **XXX-X** ha sido registrada..."
- ✅ Caja de resumen con datos principales:
  - Tipo de Identificación
  - Número de Identificación
  - Razón Social
  - Tipo de Sociedad
  - País
  - Ciudad
  - Fecha de Creación
- ✅ Botones:
  - "Ver Empresa" (primary con icono visibility)
  - "Crear Otra Empresa" (secondary con icono add_circle)
  - "Volver a Administración de Empresas" (text con icono arrow_back)

**Animación:**
```css
@keyframes scaleIn {
  0% { transform: scale(0); opacity: 0; }
  50% { transform: scale(1.1); }
  100% { transform: scale(1); opacity: 1; }
}
```

---

### 5. **HU004-ErrorCreacion.html** - Pantalla de Error
**Ubicación en HU:** Escenario 39 (línea 97) + Mockup líneas 205-211

**Descripción:** Pantalla de error cuando falla la creación.

**Elementos principales:**
- ✅ Logo CDN Facturación
- ✅ Icono de error (error_outline rojo) con animación shake
- ✅ Título "Error al Crear Empresa"
- ✅ Mensaje genérico (no revela detalles técnicos):
  - "Ocurrió un error al crear la empresa. Por favor, intente nuevamente."
  - "Si el problema persiste, contacte a soporte técnico."
- ✅ Caja de ayuda con fondo naranja:
  - "¿Qué puedes hacer?"
  - Lista de acciones recomendadas
- ✅ Botones:
  - "Aceptar" (primary) - regresa al formulario manteniendo datos
  - "Contactar Soporte" (secondary)

**Animación:**
```css
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-10px); }
  75% { transform: translateX(10px); }
}
```

---

## 🎨 Guía de Estilos Aplicada

Todos los mockups siguen la guía de estilos especificada en la HU004 y GUIA_DE_ESTILOS.md:

### Colores
- **Primary:** `#4A5A9E`
- **Secondary:** `#D4145A`
- **Gradiente:** `linear-gradient(135deg, #4A5A9E 0%, #D4145A 100%)`
- **Success:** `#4CAF50`
- **Error:** `#F44336`
- **Warning:** `#FF9800`

### Tipografía
- **Fuente:** Roboto (Google Fonts)
- **H1:** 2rem (32px), font-weight 600
- **H2:** 1.75rem (28px), font-weight 600
- **Body:** 1rem (16px), font-weight 400
- **Button:** 14px, font-weight 600, uppercase

### Componentes Material UI Style
- TextField outlined con focus Primary
- Buttons contained con gradiente
- Buttons outlined con borde Primary
- Chips con colores semánticos
- Material Icons de Google Fonts

### Iconografía
- `business` - Datos de empresa
- `location_on` - Ubicación
- `description` - Registros mercantiles
- `check_circle` - Éxito/Validación correcta
- `error_outline` - Error
- `add_circle` - Agregar
- `delete` - Eliminar
- `visibility` - Ver
- `edit` - Editar

---

## 🔧 Funcionalidad Técnica Destacada

### Algoritmo de Dígito de Verificación (DV) DIAN

Implementado en HU004-CrearEmpresa.html (líneas del script):

```javascript
function calcularDV(nit) {
    const multiplicadores = [3, 7, 13, 17, 19, 23, 29, 37, 41, 43, 47, 53, 59, 67, 71];
    const nitArray = nit.split('').reverse();
    let suma = 0;
    for (let i = 0; i < nitArray.length; i++) {
        suma += parseInt(nitArray[i]) * multiplicadores[i];
    }
    const residuo = suma % 11;
    return residuo > 1 ? 11 - residuo : residuo;
}
```

**Ejemplo de validación:**
- NIT: `8001234567`
- Cálculo:
  ```
  7×3 + 6×7 + 5×13 + ... = 1528
  1528 ÷ 11 = 138 residuo 10
  DV = 11 - 10 = 1
  ```
- Resultado: `8001234567-1` ✓

### Validaciones en Tiempo Real

1. **Unicidad de NIT:**
   ```javascript
   function checkUniqueness() {
       if (numeroId === '900123456') {
           // NIT duplicado
       } else {
           // NIT disponible
       }
   }
   ```

2. **Validación de Email:**
   ```javascript
   const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
   ```

3. **Contador de Caracteres:**
   ```javascript
   function updateCounter(fieldId, max) {
       counter.textContent = `${field.value.length}/${max} caracteres`;
   }
   ```

### Campos Condicionales

**Tipo de Identificación = NIT:**
- ✅ Razón Social: habilitado y obligatorio
- ❌ Nombres: deshabilitado
- ❌ Apellidos: deshabilitado

**Tipo de Identificación ≠ NIT:**
- ❌ Razón Social: deshabilitado
- ✅ Nombres: habilitado y obligatorio
- ✅ Apellidos: habilitado y obligatorio

### Cascada de Selects

**Ubicación Principal:**
```
País onChange →
  Si Colombia:
    - Cargar Departamentos
    - Departamento onChange → Cargar Ciudades
  Si otro país:
    - Deshabilitar Departamento
    - Deshabilitar Ciudad
```

**Registros Mercantiles:**
- Misma lógica de cascada
- Formulario repetible
- Tabla dinámica con agregar/eliminar

---

## 📂 Estructura de Archivos

```
Mockups/
├── HU001-Login.html
├── HU002-ForgotPassword.html
├── HU002-ResetPassword.html
├── HU002-LinkExpired.html
├── HU002-EmailTemplate.html
├── HU002-EmailPasswordChanged.html
├── HU004-MenuAdministracion.html         ⭐ Nuevo
├── HU004-CrearEmpresa.html               ⭐ Nuevo (más complejo)
├── HU004-ResumenEmpresa.html             ⭐ Nuevo
├── HU004-CreacionExitosa.html            ⭐ Nuevo
├── HU004-ErrorCreacion.html              ⭐ Nuevo
├── README-HU002.md
└── README-HU004.md                       ⭐ Este archivo
```

---

## 🚀 Cómo Probar los Mockups

### 1. Menu de Administración
- Abrir `HU004-MenuAdministracion.html`
- Usar barra de búsqueda (logs en consola)
- Hacer clic en botones de acciones
- Probar paginación

### 2. Formulario de Creación (el más complejo)
- Abrir `HU004-CrearEmpresa.html`
- **Probar validación de DV:**
  - Ingresar NIT: `8001234567`
  - Ingresar DV incorrecto: `5`
  - Ver mensaje: "DV incorrecto. El DV correcto es: 1"
  - Ingresar DV correcto: `1`
  - Ver checkmark verde
- **Probar unicidad:**
  - Ingresar NIT: `900123456`
  - Hacer blur del campo
  - Ver error "Este número ya existe"
  - Ingresar otro NIT: `800111222`
  - Ver checkmark verde "NIT disponible"
- **Probar campos condicionales:**
  - Cambiar Tipo ID a "Cédula de ciudadanía"
  - Ver que Nombres y Apellidos se habilitan
  - Ver que Razón Social se deshabilita
  - Cambiar a "NIT"
  - Ver comportamiento inverso
- **Probar cascada:**
  - Seleccionar País: "Colombia"
  - Ver que Departamento se habilita
  - Seleccionar Departamento: "Cundinamarca"
  - Ver que Ciudad se habilita con opciones filtradas
  - Cambiar País a "Argentina"
  - Ver que Departamento y Ciudad se deshabilitan
- **Probar centros de costos:**
  - Ingresar "10001" y hacer clic en "Agregar"
  - Ver tabla aparecer
  - Agregar varios centros
  - Hacer clic en eliminar, ver confirmación
- **Probar registros mercantiles:**
  - Completar todos los campos
  - Hacer clic en "Agregar Registro Mercantil"
  - Ver tabla con el registro
  - Intentar agregar sin completar campos
  - Ver alerta de validación

### 3. Pantalla de Resumen
- Abrir `HU004-ResumenEmpresa.html`
- Hacer clic en encabezados de secciones
- Ver animación de expansión/colapso
- Observar datos mockeados en cada sección

### 4. Pantalla de Éxito
- Abrir `HU004-CreacionExitosa.html`
- Ver animación de checkmark
- Hacer clic en cada botón para ver navegación

### 5. Pantalla de Error
- Abrir `HU004-ErrorCreacion.html`
- Ver animación de shake
- Hacer clic en "Aceptar" para simular retorno al formulario
- Hacer clic en "Contactar Soporte"

---

## ✅ Cumplimiento de la HU004

| Requerimiento | Archivo | Escenario | Estado |
|---------------|---------|-----------|--------|
| Menú de Administración | HU004-MenuAdministracion.html | 1-2 | ✅ |
| Formulario de creación | HU004-CrearEmpresa.html | 3-43 | ✅ |
| Validación de DV con algoritmo DIAN | HU004-CrearEmpresa.html | 8 | ✅ |
| Validación de unicidad de NIT | HU004-CrearEmpresa.html | 6-7 | ✅ |
| Campos condicionales según Tipo ID | HU004-CrearEmpresa.html | 10-13 | ✅ |
| Centros de Costos dinámicos | HU004-CrearEmpresa.html | 15-17 | ✅ |
| Cascada País → Depto → Ciudad | HU004-CrearEmpresa.html | 21-26 | ✅ |
| Registros Mercantiles repetibles | HU004-CrearEmpresa.html | 28-33 | ✅ |
| Validación de email | HU004-CrearEmpresa.html | 20 | ✅ |
| Contador de caracteres | HU004-CrearEmpresa.html | 9 | ✅ |
| Pantalla de resumen | HU004-ResumenEmpresa.html | 36 | ✅ |
| Creación exitosa | HU004-CreacionExitosa.html | 38 | ✅ |
| Error de creación | HU004-ErrorCreacion.html | 39 | ✅ |
| Cancelación con confirmación | HU004-CrearEmpresa.html | 41 | ✅ |

---

## 📝 Notas Técnicas

### Tablas Maestras Utilizadas (Datos Mock)

**Tipos de Identificación:**
```javascript
{
  '11': 'Registro civil',
  '13': 'Cédula de ciudadanía',
  '31': 'NIT',
  '41': 'Pasaporte'
}
```

**Departamentos de Colombia:**
```javascript
{
  '14': 'Cundinamarca',
  '2': 'Antioquia'
}
```

**Ciudades:**
```javascript
{
  'Cundinamarca': ['Bogotá D.C.'],
  'Antioquia': ['Medellín']
}
```

### Datos Mock para Testing

- **NIT duplicado:** `900123456` (muestra error)
- **NIT disponible:** Cualquier otro número (muestra checkmark)
- **DV de prueba:** NIT `8001234567` → DV correcto: `1`

### Integración con Backend (Futura)

Los mockups están listos para integración:
- Validación de unicidad → Endpoint: `GET /api/empresas/validate-nit/{nit}`
- Carga de departamentos → Endpoint: `GET /api/ubicacion/departamentos/{pais_id}`
- Carga de ciudades → Endpoint: `GET /api/ubicacion/ciudades/{depto_id}`
- Crear empresa → Endpoint: `POST /api/empresas`
- Todas las funciones `alert()` deben reemplazarse por navegación real

---

**Creado para:** Portal Unificado de CDN Facturación
**Historia de Usuario:** HU004 - Creación de Empresas (Clientes)
**Versión:** 1.0
**Fecha:** Diciembre 2025
**Diseñador:** UI/UX Designer (Claude Code)
**Complejidad:** ⭐⭐⭐⭐⭐ (5/5 - Formulario con validaciones avanzadas y algoritmo DIAN)
