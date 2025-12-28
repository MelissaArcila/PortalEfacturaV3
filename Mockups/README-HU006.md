# HU006 - Creación de Usuarios y Asignación de Permisos
## Documentación de Mockups Interactivos

### Información General

| Campo | Valor |
|-------|-------|
| **Historia de Usuario** | HU006 - Creación de Usuarios y Asignación de Permisos |
| **Fecha de Creación** | 2024 |
| **Archivos Generados** | 5 mockups HTML interactivos |
| **Tecnologías** | HTML5, CSS3, JavaScript (Vanilla) |
| **Diseño Base** | Material UI + Guía de Estilos CDN Facturación |

---

## Archivos Generados

### 1. HU006-CrearUsuario.html
**Propósito**: Formulario principal de creación de usuario con asignación de permisos.

**Características Principales**:
- Formulario estructurado en 3 secciones: Tipo de Usuario, Datos Personales, Permisos
- Validación en tiempo real de campos obligatorios
- Validación de unicidad del Número de Identificación con API mock
- Selector dinámico de empresa y roles según productos contratados
- Tabla de permisos asignados con funcionalidad de agregar/eliminar
- Filtros en tabla de permisos por empresa y rol
- Botón "Crear Usuario" habilitado solo cuando se cumplen todos los requisitos
- Demo interactivo con 5 estados

**Secciones del Formulario**:

#### Sección 1: Tipo de Usuario
- Radio buttons con diseño Material UI
- Opciones: "Usuario de Cliente" y "Usuario Interno"
- Texto explicativo para cada tipo
- Condiciona el comportamiento de la sección de permisos

#### Sección 2: Datos Personales
- **Número de Identificación** (*obligatorio):
  - Solo acepta números (máx 15 dígitos)
  - Validación de unicidad en blur/enter
  - Muestra checkmark verde si es único
  - Muestra error rojo con nombre del usuario existente si está duplicado
  - Mock data: `12345678` y `87654321` ya existen

- **Primer Apellido** (*obligatorio):
  - Solo letras, espacios, guiones y apóstrofes
  - Máximo 50 caracteres

- **Segundo Apellido** (opcional):
  - Mismas validaciones que Primer Apellido

- **Primer Nombre** (*obligatorio):
  - Solo letras, espacios, guiones y apóstrofes
  - Máximo 50 caracteres

- **Segundo Nombre** (opcional):
  - Mismas validaciones que Primer Nombre

- **Correo Electrónico** (opcional):
  - Validación de formato email estándar
  - Muestra checkmark verde si formato válido

- **Estado**:
  - Badge verde "Activo" (no editable)
  - Texto informativo: "El usuario se creará en estado Activo por defecto"

#### Sección 3: Asignación de Permisos
- **Selección de Empresa**:
  - Dropdown con lista de empresas activas
  - Si Usuario Interno: incluye opción "Sin Cliente (Rol Interno)"
  - Si Usuario de Cliente: solo muestra empresas
  - Empresas mock incluidas:
    - Empresa ABC S.A.S. (Productos: FE, POS, RADIAN)
    - Industrias XYZ Ltda. (Productos: FE, RADIAN)
    - Comercializadora DEF (Productos: Solo FE)
    - Servicios GHI Corp. (Productos: FE, POS, Nómina, RADIAN)
    - Tecnología JKL S.A. (Sin productos)

- **Selección de Rol**:
  - Dropdown dinámico basado en empresa seleccionada
  - Si empresa sin productos: solo "Administrador de Cliente"
  - Si empresa con productos: "Administrador de Cliente" + roles de gestor según productos
  - Si "Sin Cliente (Rol Interno)": muestra roles internos
  - Roles de Cliente disponibles:
    - Administrador de Cliente (siempre disponible)
    - Gestor Emisión FE (producto 1)
    - Gestor Emisión POS (producto 2)
    - Gestor Nómina Electrónica (producto 3)
    - Gestor RADIAN (producto 7)
  - Roles Internos disponibles:
    - Administrador de Portal
    - Analista Interno
    - Soporte Técnico
    - Auditor Interno
    - Desarrollador
    - Consultor Funcional

- **Botón "Agregar Permiso"**:
  - Valida que empresa y rol estén seleccionados
  - Previene permisos duplicados (misma empresa + mismo rol)
  - Agrega fila a tabla de permisos
  - Limpia selectores después de agregar

- **Tabla "Permisos Asignados"**:
  - Columnas: Empresa, Rol, Acción
  - Contador de permisos en header
  - Estado vacío muestra icono y mensaje
  - Filtros por empresa y rol (visibles cuando hay permisos)
  - Paginación (si >10 permisos - no implementada en mockup)
  - Botón "Eliminar" por fila con confirmación

**Validaciones del Formulario**:
- Tipo de usuario seleccionado
- Número de identificación único y válido (checkmark verde)
- Primer apellido ingresado
- Primer nombre ingresado
- Al menos 1 permiso asignado
- Botón "Crear Usuario" deshabilitado hasta cumplir todos los requisitos

**Estados del Demo**:
1. **Normal**: Formulario limpio
2. **ID Duplicado**: Muestra error de número de identificación existente (12345678)
3. **Formulario Lleno**: Datos personales completados
4. **Con Permisos**: Formulario lleno + 3 permisos agregados
5. **Listo para Crear**: Todo completo y validado, botón habilitado

**Botones de Acción**:
- **Cancelar**: Muestra confirmación antes de salir
- **Crear Usuario**: Navega a resumen (deshabilitado hasta cumplir requisitos)

**Manejo de Datos**:
- Guarda datos en `sessionStorage` al ir a resumen
- Preserva datos en caso de error

---

### 2. HU006-ResumenUsuario.html
**Propósito**: Pantalla de revisión antes de confirmar la creación del usuario.

**Características Principales**:
- Recupera datos de `sessionStorage`
- Muestra resumen completo del usuario a crear
- Permite volver a editar manteniendo datos
- Confirmación final antes de crear

**Secciones**:

#### Tipo de Usuario
- Badge visual según tipo:
  - Usuario de Cliente (azul)
  - Usuario Interno (morado)
  - Usuario Interno con permisos de Cliente (naranja - si tiene permisos mixtos)

#### Datos Personales
- Lista de datos con formato label: value
- Campos mostrados:
  - Número de Identificación
  - Nombre Completo (construido con todos los nombres/apellidos)
  - Primer Apellido
  - Segundo Apellido (o "No especificado")
  - Primer Nombre
  - Segundo Nombre (o "No especificado")
  - Correo Electrónico (o "No especificado")
  - Estado (badge verde "Activo")

#### Permisos Asignados
- Resumen visual con contador e icono
- Tabla completa con todas las combinaciones Empresa + Rol
- Mensaje descriptivo del acceso que tendrá el usuario

**Botones de Acción**:
- **Volver y Editar**: Regresa al formulario manteniendo TODOS los datos
- **Confirmar Creación**: Procede a crear el usuario (navega a pantalla de éxito)

**Lógica**:
```javascript
// Detecta tipo mixto
const hasInternalPermissions = permissions.some(p => p.companyName === 'Interno');
const hasClientPermissions = permissions.some(p => p.companyName !== 'Interno');

if (userType === 'internal' && hasClientPermissions) {
    // Muestra badge "Usuario Interno con permisos de Cliente"
}
```

---

### 3. HU006-CreacionExitosa.html
**Propósito**: Confirmación visual de creación exitosa del usuario.

**Características Principales**:
- Animación de entrada (scaleIn)
- Icono de éxito con animación de pulso
- Header verde con gradiente
- Resumen del usuario creado
- Opciones de navegación post-creación

**Elementos Visuales**:

#### Header
- Fondo gradiente verde (éxito)
- Icono circular blanco con checkmark verde (animado)
- Título: "¡Usuario Creado Exitosamente!"
- Mensaje personalizado con nombre completo, ID y cantidad de permisos

#### Contenido

**Resumen del Usuario**:
- Fondo gris claro con borde verde
- Datos principales:
  - Nombre Completo
  - Número de Identificación
  - Correo Electrónico
  - Tipo de Usuario
  - Estado

**Resumen de Permisos**:
- Badge verde con contador
- Lista de hasta 5 permisos
- Si >5: muestra "y X permisos más..."

**Información Adicional**:
- Box azul informativo
- Indica que falta asignar contraseña
- Guía de próximos pasos

**Botones de Acción**:
1. **Ver Usuario** (primario con gradiente):
   - En implementación real: navega a detalle del usuario
   - Mock: muestra alert y va a gestión

2. **Crear Otro Usuario** (outlined):
   - Limpia `sessionStorage`
   - Regresa al formulario de creación limpio

3. **Volver a Gestión de Usuarios** (texto):
   - Limpia `sessionStorage`
   - Navega a HU005-GestionUsuarios.html

**Animaciones**:
```css
@keyframes scaleIn {
    from { transform: scale(0.9); opacity: 0; }
    to { transform: scale(1); opacity: 1; }
}

@keyframes pulse {
    0%, 100% { transform: scale(1); }
    50% { transform: scale(1.05); }
}
```

---

### 4. HU006-ErrorCreacion.html
**Propósito**: Manejo de errores durante el proceso de creación con preservación de datos.

**Características Principales**:
- Animación de shake al cargar
- Header rojo con gradiente
- Demo interactivo con 4 tipos de error
- Detalles contextuales según tipo de error
- Preservación de datos ingresados

**Estados del Demo**:

1. **Error Genérico**:
   - Icono: `error`
   - Mensaje: "Ocurrió un error al crear el usuario"
   - Detalles: Instrucciones generales de resolución
   - Qué puede hacer: verificar datos, reintentar, contactar soporte

2. **Timeout**:
   - Icono: `schedule`
   - Mensaje: "La solicitud excedió el tiempo de espera"
   - Causas posibles:
     - Problemas de conectividad
     - Servidor con alta carga
     - Conexión inestable
   - Recomendación: esperar y reintentar

3. **Validación Backend**:
   - Icono: `rule`
   - Mensaje: "Los datos enviados no cumplen con los requisitos"
   - Problemas posibles:
     - ID duplicado (validación de integridad)
     - Roles no válidos para empresas
     - Empresa inactiva
     - Conflictos de permisos
   - Recomendación: revisar ID y permisos

4. **Error de Base de Datos**:
   - Icono: `storage`
   - Mensaje: "Error de conexión con la base de datos"
   - Causas:
     - Problemas de conectividad BD
     - Mantenimiento programado
     - Error técnico del servidor
   - Recomendación: contactar soporte con timestamp

**Elementos Visuales**:

#### Header
- Fondo gradiente rojo
- Icono circular con warning (animado)
- Mensaje de error dinámico

#### Detalles del Error
- Box naranja con borde izquierdo
- Título del tipo de error
- Descripción contextual
- Lista de causas posibles
- Recomendaciones específicas

#### Información de Preservación
- Box azul informativo
- "Sus datos han sido preservados"
- Mensaje tranquilizador

**Botones de Acción**:
- **Aceptar** (primario): Regresa al formulario manteniendo datos
- **Contactar Soporte Técnico** (outlined): Abre canal de soporte

**Animación de Entrada**:
```css
@keyframes shake {
    0%, 100% { transform: translateX(0); }
    25% { transform: translateX(-10px); }
    75% { transform: translateX(10px); }
}
```

---

### 5. HU006-DialogoConfirmacionEliminarPermiso.html
**Propósito**: Diálogo modal de confirmación al eliminar un permiso de la tabla.

**Características Principales**:
- Modal centrado con overlay oscuro
- Animaciones de entrada (fadeIn + slideUp)
- Información detallada del permiso a eliminar
- Validación especial para último permiso
- Demo interactivo con 3 escenarios

**Estados del Demo**:

1. **Rol de Cliente**:
   - Muestra 3 permisos en tabla
   - Permite eliminar cualquiera
   - No hay restricciones

2. **Rol Interno**:
   - Muestra mix de permisos internos y de cliente
   - Permite eliminar cualquiera
   - Demuestra flexibilidad del sistema

3. **Último Permiso**:
   - Muestra solo 1 permiso en tabla
   - Al intentar eliminar, muestra warning
   - Botón "Confirmar" deshabilitado
   - Mensaje: "Este es el último permiso. El usuario debe tener al menos un permiso"

**Estructura del Modal**:

#### Header
- Icono de delete (rojo)
- Título: "¿Eliminar Permiso?"

#### Body
- Mensaje de confirmación con empresa y rol resaltados
- Box con información del permiso:
  - Empresa
  - Rol
- Warning (solo si es último permiso):
  - Icono naranja
  - Mensaje de restricción

#### Footer
- **Cancelar** (texto): Cierra modal sin acción
- **Confirmar** (rojo con gradiente): Ejecuta eliminación

**Funcionalidades**:
- Click fuera del modal: cierra
- Tecla ESC: cierra modal
- Click en overlay: cierra modal
- Click en contenido del modal: no cierra (event.stopPropagation)

**Validación de Último Permiso**:
```javascript
if (isLastPermission) {
    warningElement.style.display = 'flex';
    confirmBtn.disabled = true;
    confirmBtn.style.opacity = '0.5';
    confirmBtn.style.cursor = 'not-allowed';
}
```

**Animaciones**:
```css
@keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
}

@keyframes slideUp {
    from { transform: translateY(50px); opacity: 0; }
    to { transform: translateY(0); opacity: 1; }
}
```

---

## Patrones de Diseño Aplicados

### Colores
- **Primary**: `#4A5A9E` (Azul corporativo)
- **Secondary**: `#D4145A` (Magenta corporativo)
- **Success**: `#4caf50` (Verde Material)
- **Error**: `#D4145A` / `#ff6b6b` (Rojo)
- **Warning**: `#ff9800` (Naranja)
- **Info**: `#1976d2` (Azul claro)

### Tipografía
- **Familia**: Roboto (Google Fonts)
- **Pesos**: 300 (Light), 400 (Regular), 500 (Medium), 700 (Bold)
- **Tamaños**:
  - H1: 28px
  - H2: 20px
  - H3: 16-18px
  - Body: 14px
  - Small: 12-13px

### Espaciado
- **Base**: 8px (sistema de múltiplos de 8)
- **Padding secciones**: 24-32px
- **Gap entre elementos**: 8-16px
- **Margin bottom**: 16-32px

### Componentes Material UI

#### Radio Buttons
- Cards seleccionables con borde
- Hover: borde azul + fondo tenue
- Selected: borde azul + fondo azul claro
- Título + descripción en cada opción

#### Input Fields
- Borde gris (`#ccc`)
- Focus: borde azul + shadow azul tenue
- Error: borde rojo
- Success: borde verde + checkmark
- Hint text: gris claro (12px)

#### Buttons
- **Contained**: Gradiente primary → secondary
- **Outlined**: Borde sólido, background blanco
- **Text**: Sin borde, background transparente
- Hover: elevación con shadow
- Disabled: gris + cursor not-allowed

#### Badges
- Border-radius: 16-20px
- Padding: 6-8px horizontal
- Icono + texto
- Colores según estado

#### Tables
- Header: fondo gris claro
- Hover en fila: fondo gris muy tenue
- Bordes sutiles
- Padding generoso (12-16px)

#### Modal
- Overlay: rgba(0,0,0,0.5)
- Dialog: shadow pronunciada
- Border-radius: 8px
- Animaciones de entrada

### Iconografía
- **Familia**: Material Icons (Google)
- **Tamaños**: 16px (inline), 24px (normal), 32-48px (destacados), 64-80px (heroes)
- **Colores**: Según contexto (primary, error, success, etc.)

---

## Flujo de Navegación

```
HU005-GestionUsuarios.html (Gestión de Usuarios)
    ↓ [Clic en "Crear Nuevo Usuario"]
HU006-CrearUsuario.html (Formulario de Creación)
    ├─→ [Clic en "Cancelar" + Confirmar] → HU005-GestionUsuarios.html
    └─→ [Clic en "Crear Usuario" + Validación OK]
        ↓
    HU006-ResumenUsuario.html (Resumen)
        ├─→ [Clic en "Volver y Editar"] → HU006-CrearUsuario.html (con datos)
        └─→ [Clic en "Confirmar Creación"]
            ├─→ [Éxito] → HU006-CreacionExitosa.html
            │       ├─→ [Ver Usuario] → (Detalle Usuario - no implementado)
            │       ├─→ [Crear Otro] → HU006-CrearUsuario.html (limpio)
            │       └─→ [Volver a Gestión] → HU005-GestionUsuarios.html
            └─→ [Error] → HU006-ErrorCreacion.html
                    └─→ [Aceptar] → HU006-CrearUsuario.html (con datos preservados)
```

---

## Validaciones Implementadas

### Frontend (Tiempo Real)

#### Número de Identificación
- **Input**: Solo numéricos (regex: `/\D/g`)
- **Longitud**: Máximo 15 caracteres
- **Unicidad**: Validación contra array mock en `blur`/`enter`
- **Feedback**:
  - Único: checkmark verde + mensaje de éxito
  - Duplicado: borde rojo + mensaje con nombre del usuario existente

#### Nombres y Apellidos
- **Input**: Solo letras, espacios, guiones, apóstrofes
- **Regex**: `/[^a-zA-ZáéíóúÁÉÍÓÚñÑ\s\-']/g`
- **Longitud**: Máximo 50 caracteres
- **Obligatorios**: Primer Nombre, Primer Apellido
- **Opcionales**: Segundo Nombre, Segundo Apellido

#### Correo Electrónico
- **Formato**: Regex estándar email
- **Regex**: `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`
- **Obligatorio**: No (opcional)
- **Feedback**: Checkmark verde si válido, borde rojo si inválido

#### Permisos
- **Mínimo**: 1 permiso requerido
- **Duplicados**: Previene misma combinación Empresa + Rol
- **Empresa + Rol**: Ambos obligatorios para agregar
- **Último permiso**: No se puede eliminar (validación en modal)

### Validación de Formulario Completo
```javascript
function validateForm() {
    const isValid =
        currentUserType &&                           // Tipo seleccionado
        idNumber.value.trim() &&                     // ID ingresado
        !idNumber.classList.contains('error') &&     // ID sin error
        idNumber.classList.contains('success') &&    // ID único (checkmark)
        firstLastName.value.trim() &&                // Primer apellido
        firstName.value.trim() &&                    // Primer nombre
        permissions.length > 0;                      // Al menos 1 permiso

    createBtn.disabled = !isValid;
}
```

---

## Lógica de Negocio Implementada

### 1. Roles Dinámicos según Productos Contratados

```javascript
function handleCompanyChange() {
    const selectedOption = companySelect.options[companySelect.selectedIndex];
    const products = JSON.parse(selectedOption.dataset.products || '[]');

    // Siempre agregar Administrador de Cliente
    const adminRole = mockRoles.client.find(r => r.productId === null);
    roleSelect.appendChild(createOption(adminRole));

    // Agregar roles de gestores solo si empresa tiene productos contratados
    mockRoles.client
        .filter(r => r.productId && products.includes(r.productId))
        .forEach(role => {
            roleSelect.appendChild(createOption(role));
        });

    // Mensaje si no hay productos
    if (products.length === 0 && roleSelect.options.length === 2) {
        addInfoOption('Esta empresa no tiene productos contratados');
    }
}
```

**Mapeo de Productos a Roles**:
- Producto 1 (Emisión FE) → Gestor Emisión FE
- Producto 2 (Emisión POS) → Gestor Emisión POS
- Producto 3 (Nómina Electrónica) → Gestor Nómina Electrónica
- Producto 7 (RADIAN) → Gestor RADIAN
- Sin producto específico → Administrador de Cliente (siempre disponible)

### 2. Usuario Interno vs Cliente

```javascript
if (currentUserType === 'internal') {
    // Opción "Sin Cliente (Rol Interno)" disponible
    // Puede tener roles internos
    // Puede tener roles de cliente (para soporte)
} else if (currentUserType === 'client') {
    // Solo empresas de cliente
    // Solo roles de cliente
    // NO puede tener roles internos
}
```

### 3. Prevención de Permisos Duplicados

```javascript
function addPermission() {
    const isDuplicate = permissions.some(p =>
        p.companyId == companyId && p.roleId == roleId
    );

    if (isDuplicate) {
        showError(`Este permiso ya fue agregado. El usuario ya tiene el rol ${roleName} en ${companyName}`);
        return;
    }

    permissions.push({ companyId, companyName, roleId, roleName });
}
```

### 4. Determinación de Tipo Mixto en Resumen

```javascript
function getUserTypeBadge() {
    const hasInternalPermissions = permissions.some(p => p.companyName === 'Interno');
    const hasClientPermissions = permissions.some(p => p.companyName !== 'Interno');

    if (userType === 'internal' && hasClientPermissions) {
        return 'Usuario Interno con permisos de Cliente';
    } else if (userType === 'internal') {
        return 'Usuario Interno';
    } else {
        return 'Usuario de Cliente';
    }
}
```

---

## Manejo de Datos entre Pantallas

### SessionStorage
Los datos se persisten entre pantallas usando `sessionStorage`:

```javascript
// Guardar datos (HU006-CrearUsuario.html)
const formData = {
    userType: currentUserType,
    idNumber: document.getElementById('idNumber').value,
    firstLastName: document.getElementById('firstLastName').value,
    secondLastName: document.getElementById('secondLastName').value,
    firstName: document.getElementById('firstName').value,
    secondName: document.getElementById('secondName').value,
    email: document.getElementById('email').value,
    permissions: permissions
};
sessionStorage.setItem('newUserData', JSON.stringify(formData));

// Recuperar datos (HU006-ResumenUsuario.html / HU006-CreacionExitosa.html)
const userData = JSON.parse(sessionStorage.getItem('newUserData') || '{}');

// Limpiar datos (después de crear o cancelar)
sessionStorage.removeItem('newUserData');
```

### Flujo de Datos

1. **Formulario → Resumen**:
   - Guardar en sessionStorage
   - Navegar a resumen
   - Cargar desde sessionStorage

2. **Resumen → Volver a Editar**:
   - Navegar de vuelta
   - Datos permanecen en sessionStorage
   - Formulario los carga automáticamente (feature pendiente en mockup)

3. **Resumen → Confirmar → Éxito**:
   - Navegar a éxito
   - Cargar datos para mostrar resumen
   - NO limpiar sessionStorage aún (para botón "Ver Usuario")

4. **Éxito → Siguiente Acción**:
   - Limpiar sessionStorage
   - Navegar según botón elegido

5. **Error → Volver**:
   - Mantener sessionStorage
   - Cargar datos al volver al formulario

---

## Datos Mock Utilizados

### Empresas
```javascript
const mockCompanies = [
    { id: 1, name: 'Empresa ABC S.A.S.', products: [1, 2, 7] },      // FE, POS, RADIAN
    { id: 2, name: 'Industrias XYZ Ltda.', products: [1, 7] },       // FE, RADIAN
    { id: 3, name: 'Comercializadora DEF', products: [1] },          // Solo FE
    { id: 4, name: 'Servicios GHI Corp.', products: [1, 2, 3, 7] },  // Todos
    { id: 5, name: 'Tecnología JKL S.A.', products: [] }             // Sin productos
];
```

### Roles Internos
```javascript
const internalRoles = [
    { id: 1, name: 'Administrador de Portal' },
    { id: 2, name: 'Analista Interno' },
    { id: 3, name: 'Soporte Técnico' },
    { id: 4, name: 'Auditor Interno' },
    { id: 5, name: 'Desarrollador' },
    { id: 6, name: 'Consultor Funcional' }
];
```

### Roles de Cliente
```javascript
const clientRoles = [
    { id: 10, name: 'Administrador de Cliente', productId: null },  // Siempre disponible
    { id: 11, name: 'Gestor Emisión FE', productId: 1 },
    { id: 12, name: 'Gestor Emisión POS', productId: 2 },
    { id: 13, name: 'Gestor Nómina Electrónica', productId: 3 },
    { id: 14, name: 'Gestor RADIAN', productId: 7 }
];
```

### Usuarios Existentes (para validación de ID)
```javascript
const existingUsers = [
    { id: '12345678', name: 'Juan Carlos Pérez López' },
    { id: '87654321', name: 'María Fernanda Gómez Ruiz' }
];
```

---

## Responsive Design

### Breakpoints
- **Desktop**: > 768px (diseño completo)
- **Mobile**: ≤ 768px (adaptaciones)

### Adaptaciones Mobile

#### HU006-CrearUsuario.html
```css
@media (max-width: 768px) {
    .permissions-inputs {
        grid-template-columns: 1fr;  /* Stack vertical */
    }

    .form-grid {
        grid-template-columns: 1fr;  /* Una columna */
    }

    .radio-group {
        flex-direction: column;  /* Radio buttons verticales */
    }

    table {
        font-size: 12px;  /* Texto más pequeño */
    }
}
```

#### HU006-DialogoConfirmacionEliminarPermiso.html
```css
@media (max-width: 768px) {
    .modal-dialog {
        width: 95%;
        margin: 20px;
    }

    .modal-actions {
        flex-direction: column-reverse;  /* Botones verticales */
    }

    .btn {
        width: 100%;
        justify-content: center;
    }
}
```

---

## Escenarios de Prueba

### Escenario 1: Crear Usuario de Cliente Básico
1. Abrir HU006-CrearUsuario.html
2. Seleccionar "Usuario de Cliente"
3. Ingresar ID: `11223344`
4. Ingresar nombre: Carlos, apellido: Rodríguez
5. Seleccionar empresa: "Empresa ABC S.A.S."
6. Seleccionar rol: "Gestor Emisión FE"
7. Clic en "Agregar Permiso"
8. Clic en "Crear Usuario"
9. Verificar resumen en HU006-ResumenUsuario.html
10. Clic en "Confirmar Creación"
11. Verificar éxito en HU006-CreacionExitosa.html

### Escenario 2: ID Duplicado
1. Usar demo "ID Duplicado" o ingresar `12345678`
2. Hacer blur del campo
3. Verificar error rojo: "Este número de identificación ya está registrado. Usuario existente: Juan Carlos Pérez López"
4. Verificar que botón "Crear Usuario" está deshabilitado

### Escenario 3: Usuario Interno con Múltiples Permisos
1. Seleccionar "Usuario Interno"
2. Completar datos personales
3. Agregar permiso: "Sin Cliente (Rol Interno)" → "Soporte Técnico"
4. Agregar permiso: "Empresa ABC S.A.S." → "Gestor Emisión FE"
5. Agregar permiso: "Empresa ABC S.A.S." → "Gestor RADIAN"
6. Verificar 3 permisos en tabla
7. En resumen, verificar badge "Usuario Interno con permisos de Cliente"

### Escenario 4: Eliminar Permiso
1. Agregar 3 permisos
2. Clic en "Eliminar" en segundo permiso
3. Verificar modal de confirmación
4. Confirmar eliminación
5. Verificar que tabla muestra 2 permisos

### Escenario 5: Intento de Eliminar Último Permiso
1. Usar demo "Último Permiso" en HU006-DialogoConfirmacionEliminarPermiso.html
2. Clic en "Eliminar" del único permiso
3. Verificar warning: "Este es el último permiso..."
4. Verificar que botón "Confirmar" está deshabilitado

### Escenario 6: Empresa sin Productos
1. Seleccionar empresa: "Tecnología JKL S.A."
2. Verificar que solo muestra "Administrador de Cliente"
3. Verificar mensaje: "Esta empresa no tiene productos contratados"

### Escenario 7: Prevención de Duplicados
1. Agregar permiso: "Empresa ABC" → "Gestor FE"
2. Intentar agregar mismo permiso nuevamente
3. Verificar error: "Este permiso ya fue agregado..."

### Escenario 8: Volver y Editar
1. Completar formulario y llegar a resumen
2. Clic en "Volver y Editar"
3. Verificar que todos los datos permanecen
4. Modificar algún dato
5. Crear nuevamente

### Escenario 9: Error de Creación
1. Abrir HU006-ErrorCreacion.html
2. Probar cada tipo de error con botones demo
3. Verificar detalles específicos de cada error
4. Clic en "Aceptar"
5. Verificar regreso a formulario

### Escenario 10: Cancelación
1. Completar parcialmente el formulario
2. Clic en "Cancelar"
3. Confirmar en diálogo
4. Verificar navegación a gestión de usuarios

---

## Mejoras Futuras (Fuera de Alcance del Mockup)

### Funcionalidades
- [ ] Autoguardado en localStorage cada 30 segundos
- [ ] Recuperación de datos después de cierre accidental del navegador
- [ ] Búsqueda typeahead en selectores de empresa y rol
- [ ] Importación masiva de usuarios desde CSV
- [ ] Generación automática de contraseña temporal
- [ ] Envío de email de bienvenida al crear usuario
- [ ] Validación de formato de email contra dominios corporativos
- [ ] Sugerencias de roles basadas en otros usuarios de la misma empresa
- [ ] Plantillas de permisos predefinidas (ej: "Gestor Completo FE + RADIAN")
- [ ] Historial de cambios en permisos con timeline visual

### UX
- [ ] Progress bar indicando % de completitud del formulario
- [ ] Tooltips explicativos en cada campo
- [ ] Ayuda contextual inline
- [ ] Wizard con pasos 1-2-3 en lugar de secciones
- [ ] Indicador visual de campos con errores en el scroll
- [ ] Resaltado de campos obligatorios faltantes al intentar crear
- [ ] Confirmación de email con código de verificación
- [ ] Vista previa de permisos agrupados por empresa

### Técnicas
- [ ] Debouncing en validación de ID para reducir llamadas a API
- [ ] Infinite scroll en tabla de permisos si > 50
- [ ] Virtual scrolling para listas largas de empresas
- [ ] Service Worker para funcionamiento offline
- [ ] Lazy loading de roles al expandir empresa
- [ ] Compresión de datos en sessionStorage
- [ ] Encriptación de datos sensibles en storage
- [ ] Web Components para reutilización

### Accesibilidad
- [ ] ARIA labels completos
- [ ] Navegación completa por teclado
- [ ] Skip links
- [ ] Anuncios de screen reader para validaciones
- [ ] Focus trap en modal
- [ ] Contraste AAA en todos los textos
- [ ] Soporte para modo de alto contraste
- [ ] Indicadores visuales adicionales (no solo color)

---

## Integración con Backend

### Endpoints Esperados

#### POST /api/users/check-id-availability
**Request**:
```json
{
    "idNumber": "12345678"
}
```

**Response (Disponible)**:
```json
{
    "available": true
}
```

**Response (No disponible)**:
```json
{
    "available": false,
    "existingUser": {
        "id": "uuid-123",
        "fullName": "Juan Carlos Pérez López"
    }
}
```

#### GET /api/companies?status=active
**Response**:
```json
{
    "companies": [
        {
            "id": "uuid-1",
            "name": "Empresa ABC S.A.S.",
            "status": "ACTIVE",
            "products": [
                { "id": 1, "name": "Emisión FE" },
                { "id": 2, "name": "Emisión POS" },
                { "id": 7, "name": "RADIAN" }
            ]
        }
    ]
}
```

#### GET /api/roles?type=INTERNAL|CLIENT
**Response**:
```json
{
    "roles": [
        {
            "id": "uuid-role-1",
            "name": "Administrador de Cliente",
            "type": "CLIENT",
            "productId": null
        },
        {
            "id": "uuid-role-2",
            "name": "Gestor Emisión FE",
            "type": "CLIENT",
            "productId": 1
        }
    ]
}
```

#### POST /api/users
**Request**:
```json
{
    "userType": "CLIENT",
    "idNumber": "11223344",
    "firstName": "Carlos",
    "secondName": "Alberto",
    "firstLastName": "Rodríguez",
    "secondLastName": "Martínez",
    "email": "carlos.rodriguez@empresa.com",
    "status": "ACTIVE",
    "permissions": [
        {
            "companyId": "uuid-company-1",
            "roleId": "uuid-role-11"
        },
        {
            "companyId": "uuid-company-1",
            "roleId": "uuid-role-14"
        }
    ]
}
```

**Response (Éxito)**:
```json
{
    "success": true,
    "userId": "uuid-new-user",
    "message": "Usuario creado exitosamente",
    "permissionsAssigned": 2
}
```

**Response (Error)**:
```json
{
    "success": false,
    "errorType": "VALIDATION_ERROR",
    "message": "El número de identificación ya está registrado",
    "details": {
        "field": "idNumber",
        "existingUserId": "uuid-123"
    }
}
```

### Eventos de Auditoría Esperados

Cada acción debe generar registro de auditoría según el estándar definido en HU006.

**Tipos de Eventos**:
- `ADMINISTRACION_USUARIO_CREACION_EXITOSA`
- `ADMINISTRACION_USUARIO_PERMISO_ASIGNADO` (uno por cada permiso)
- `ADMINISTRACION_USUARIO_VALIDACION_ID_DUPLICADO`
- `ADMINISTRACION_USUARIO_CREACION_ERROR`
- `ADMINISTRACION_USUARIO_CREACION_CANCELADA`
- `ADMINISTRACION_USUARIO_ACCESO_DENEGADO`

**Estructura de Registro**:
```json
{
    "eventId": "uuid",
    "eventType": "ADMINISTRACION_USUARIO_CREACION_EXITOSA",
    "timestamp": "2024-01-15T10:30:45.123Z",
    "userId": "uuid-admin",
    "clientId": null,
    "clientName": null,
    "ipLocal": "192.168.1.100",
    "ipPublic": "203.0.113.50",
    "result": "EXITOSO",
    "severity": "INFO",
    "description": "Administrador admin@cdn.com creó usuario Carlos Alberto Rodríguez Martínez con ID 11223344",
    "additionalData": {
        "createdUserId": "uuid-new-user",
        "idNumber": "11223344",
        "fullName": "Carlos Alberto Rodríguez Martínez",
        "userType": "Usuario de Cliente",
        "status": "Activo",
        "email": "carlos.rodriguez@empresa.com",
        "permissionsAssignedCount": 2
    }
}
```

---

## Conclusión

Los mockups de HU006 implementan un flujo completo de creación de usuarios con asignación de permisos, siguiendo las mejores prácticas de UX y los estándares de Material UI. La solución es:

- **Funcional**: Todas las validaciones y lógica de negocio implementadas
- **Interactiva**: 5 estados demo para probar diferentes escenarios
- **Responsive**: Adaptable a móviles y tablets
- **Accesible**: Uso de HTML semántico y ARIA labels básicos
- **Auditable**: Preparada para integración con sistema de auditoría
- **Escalable**: Código modular y bien documentado

Los mockups sirven como especificación visual y funcional para el equipo de desarrollo backend y frontend, asegurando alineación entre diseño, funcionalidad y requisitos de negocio.
