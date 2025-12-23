# Guía de Estilos - Portal Unificado CDN Facturación

Esta guía de estilos define los estándares visuales y de diseño para el Portal Unificado de CDN Facturación, utilizando Material UI como framework de componentes.

## Tabla de Contenidos

1. [Paleta de Colores](#paleta-de-colores)
2. [Tipografía](#tipografía)
3. [Espaciado y Grid](#espaciado-y-grid)
4. [Componentes Material UI](#componentes-material-ui)
5. [Iconografía](#iconografía)
6. [Elevación y Sombras](#elevación-y-sombras)
7. [Animaciones y Transiciones](#animaciones-y-transiciones)
8. [Patrones de Diseño](#patrones-de-diseño)
9. [Accesibilidad](#accesibilidad)

---

## Paleta de Colores

### Colores Primarios

Color principal del sistema basado en el logo CDN:

```javascript
const primary = {
  main: '#4A5A9E',          // Primary Main - Azul del logo
  light: '#6B7BC4',         // Primary Light
  dark: '#364378',          // Primary Dark
  contrastText: '#FFFFFF'
}
```

**Primary Main: #4A5A9E**
- RGB: (74, 90, 158)
- Uso: Color principal de marca, botones primarios, elementos destacados

**Primary Light: #6B7BC4**
- RGB: (107, 123, 196)
- Uso: Estados hover, variantes claras

**Primary Dark: #364378**
- RGB: (54, 67, 120)
- Uso: Estados active/pressed, énfasis fuerte

### Colores Secundarios

Color secundario para acentos y elementos complementarios:

```javascript
const secondary = {
  main: '#D4145A',          // Secondary Main - Magenta
  light: '#E94581',         // Secondary Light
  dark: '#A00F44',          // Secondary Dark
  contrastText: '#FFFFFF'
}
```

**Secondary Main: #D4145A**
- RGB: (212, 20, 90)
- Uso: Acentos, elementos secundarios, CTAs alternativos

**Secondary Light: #E94581**
- RGB: (233, 69, 129)
- Uso: Estados hover secundarios

**Secondary Dark: #A00F44**
- RGB: (160, 15, 68)
- Uso: Estados active secundarios

### Colores de Estado

Colores semánticos para estados del sistema:

```javascript
const statusColors = {
  success: {
    main: '#4CAF50',        // Verde - Éxito
    contrastText: '#FFFFFF'
  },
  error: {
    main: '#F44336',        // Rojo - Error
    contrastText: '#FFFFFF'
  },
  warning: {
    main: '#FF9800',        // Naranja - Advertencia
    contrastText: '#000000'
  },
  info: {
    main: '#2196F3',        // Azul - Información
    contrastText: '#FFFFFF'
  }
}
```

**Success: #4CAF50**
- Uso: Mensajes de éxito, confirmaciones, estados completados

**Error: #F44336**
- Uso: Mensajes de error, validaciones fallidas, acciones destructivas

**Warning: #FF9800**
- Uso: Advertencias, precauciones, acciones que requieren atención

**Info: #2196F3**
- Uso: Información general, mensajes informativos, ayuda

### Gradientes

Gradiente principal para elementos destacados:

```css
/* Gradiente Principal: Azul a Magenta */
background: linear-gradient(135deg, #4A5A9E 0%, #D4145A 100%);
```

Aplicación en CSS:
```css
.gradient-primary {
  background: linear-gradient(135deg, #4A5A9E 0%, #D4145A 100%);
}
```

Aplicación en Material UI:
```javascript
sx={{
  background: 'linear-gradient(135deg, #4A5A9E 0%, #D4145A 100%)',
}}
```

### Colores Neutrales

Escala de grises para textos, fondos y bordes:

```javascript
const grey = {
  50: '#FAFAFA',
  100: '#F5F5F5',
  200: '#EEEEEE',
  300: '#E0E0E0',
  400: '#BDBDBD',
  500: '#9E9E9E',
  600: '#757575',
  700: '#616161',
  800: '#424242',
  900: '#212121'
}

const textColors = {
  primary: 'rgba(0, 0, 0, 0.87)',       // Texto principal
  secondary: 'rgba(0, 0, 0, 0.6)',      // Texto secundario
  disabled: 'rgba(0, 0, 0, 0.38)'       // Texto deshabilitado
}
```

---

## Tipografía

### Familia de Fuentes

Fuente principal: **Roboto** (Material UI default)

```css
font-family: 'Roboto', -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif;
```

Para contextos donde se requiera la fuente corporativa Gotham, usar fallback a Roboto:

```css
font-family: 'Gotham', 'Roboto', -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif;
```

### Escala Tipográfica

Basada en Material UI con ajustes para el portal:

| Variante | Tamaño | Peso | Uso |
|----------|--------|------|-----|
| **H1** | 2.5rem (40px) | 600 (Semi-bold) | Títulos principales de página |
| **H2** | 2rem (32px) | 600 (Semi-bold) | Secciones principales |
| **H3** | 1.75rem (28px) | 600 (Semi-bold) | Subsecciones |
| **H4** | 1.5rem (24px) | 600 (Semi-bold) | Títulos de tarjetas |
| **H5** | 1.25rem (20px) | 600 (Semi-bold) | Subtítulos |
| **H6** | 1.125rem (18px) | 600 (Semi-bold) | Encabezados menores |
| **Body1** | 1rem (16px) | 400 (Regular) | Texto de cuerpo principal |
| **Body2** | 0.875rem (14px) | 400 (Regular) | Texto secundario |
| **Caption** | 0.75rem (12px) | 400 (Regular) | Etiquetas, ayuda |
| **Button** | 0.875rem (14px) | 600 (Semi-bold) | Texto de botones |

### Configuración de Tipografía Material UI

```javascript
typography: {
  fontFamily: 'Roboto, Arial, sans-serif',
  h1: {
    fontSize: '2.5rem',     // 40px
    fontWeight: 600,
    lineHeight: 1.2
  },
  h2: {
    fontSize: '2rem',       // 32px
    fontWeight: 600,
    lineHeight: 1.3
  },
  h3: {
    fontSize: '1.75rem',    // 28px
    fontWeight: 600,
    lineHeight: 1.3
  },
  h4: {
    fontSize: '1.5rem',     // 24px
    fontWeight: 600,
    lineHeight: 1.4
  },
  body1: {
    fontSize: '1rem',       // 16px
    fontWeight: 400,
    lineHeight: 1.5
  },
  body2: {
    fontSize: '0.875rem',   // 14px
    fontWeight: 400,
    lineHeight: 1.5
  },
  button: {
    fontSize: '0.875rem',
    fontWeight: 600,
    textTransform: 'uppercase',
    letterSpacing: '0.02857em'
  }
}
```

---

## Espaciado y Grid

### Sistema de Espaciado

Material UI utiliza un factor de espaciado base de **8px**:

```javascript
const spacing = (factor) => factor * 8; // 8px base

// Ejemplos:
spacing(1)  // 8px
spacing(2)  // 16px
spacing(3)  // 24px
spacing(4)  // 32px
spacing(6)  // 48px
spacing(8)  // 64px
```

### Grid System

Basado en un sistema de 12 columnas:

```jsx
<Grid container spacing={3}>
  <Grid item xs={12} sm={6} md={4}>
    {/* Contenido */}
  </Grid>
</Grid>
```

**Breakpoints:**
- xs: 0px
- sm: 600px
- md: 960px
- lg: 1280px
- xl: 1920px

---

## Componentes Material UI

### Botones

#### Variantes de Botones

**Primary Contained** - Botón principal:
```jsx
<Button variant="contained" color="primary">
  Ingresar
</Button>
```

**Secondary Contained** - Botón secundario:
```jsx
<Button variant="contained" color="secondary">
  Acción Secundaria
</Button>
```

**Primary Outlined** - Botón con borde:
```jsx
<Button variant="outlined" color="primary">
  Cancelar
</Button>
```

**Text Button** - Botón de texto:
```jsx
<Button variant="text" color="primary">
  Ver más
</Button>
```

**Botón con Gradiente** - Para acciones especiales:
```jsx
<Button
  variant="contained"
  sx={{
    background: 'linear-gradient(135deg, #4A5A9E 0%, #D4145A 100%)',
    '&:hover': {
      background: 'linear-gradient(135deg, #364378 0%, #A00F44 100%)',
    },
  }}
>
  Acción Especial
</Button>
```

#### Estilos de Botones

```css
/* Botón Primario */
.btn-primary {
  background-color: #4A5A9E;
  color: #FFFFFF;
  padding: 10px 24px;
  border-radius: 8px;
  font-weight: 600;
  text-transform: uppercase;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.btn-primary:hover {
  background-color: #364378;
  box-shadow: 0 6px 16px rgba(74, 90, 158, 0.3);
}

/* Botón Secundario */
.btn-secondary {
  background-color: #D4145A;
  color: #FFFFFF;
}

.btn-secondary:hover {
  background-color: #A00F44;
}

/* Botón con Gradiente */
.btn-gradient {
  background: linear-gradient(135deg, #4A5A9E 0%, #D4145A 100%);
  color: #FFFFFF;
}

.btn-gradient:hover {
  background: linear-gradient(135deg, #364378 0%, #A00F44 100%);
  transform: translateY(-2px);
}
```

### Campos de Formulario (TextField)

```jsx
<TextField
  label="Nombre de usuario"
  variant="outlined"
  fullWidth
  placeholder="Ingrese su nombre de usuario"
/>
```

**Estilos personalizados:**
```jsx
<TextField
  sx={{
    '& .MuiOutlinedInput-root': {
      '&:hover fieldset': {
        borderColor: 'primary.main',
      },
      '&.Mui-focused fieldset': {
        borderColor: 'primary.main',
      },
    },
  }}
/>
```

### Alertas

```jsx
// Éxito
<Alert severity="success">
  La operación se completó correctamente.
</Alert>

// Error
<Alert severity="error">
  Credenciales incorrectas.
</Alert>

// Advertencia
<Alert severity="warning">
  Acceso no disponible. Contacte al administrador.
</Alert>

// Información
<Alert severity="info">
  Seleccione el cliente con el cual desea trabajar.
</Alert>
```

**Estilos CSS para Alertas:**
```css
/* Alerta de Éxito */
.alert-success {
  background-color: #E8F5E9;
  border-left: 4px solid #4CAF50;
  color: #2E7D32;
}

/* Alerta de Error */
.alert-error {
  background-color: #FFEBEE;
  border-left: 4px solid #F44336;
  color: #C62828;
}

/* Alerta de Advertencia */
.alert-warning {
  background-color: #FFF3E0;
  border-left: 4px solid #FF9800;
  color: #E65100;
}

/* Alerta de Información */
.alert-info {
  background-color: #E3F2FD;
  border-left: 4px solid #2196F3;
  color: #1565C0;
}
```

### Tarjetas (Cards)

```jsx
<Card>
  <CardContent>
    <Typography variant="h5" component="div">
      Facturación Electrónica
    </Typography>
    <Typography variant="body2" color="text.secondary">
      Gestione sus facturas electrónicas de manera eficiente.
    </Typography>
  </CardContent>
  <CardActions>
    <Button size="small" variant="text" color="primary">
      Ver más
    </Button>
    <Button size="small" variant="contained" color="primary">
      Ingresar
    </Button>
  </CardActions>
</Card>
```

**Estilos CSS para Cards:**
```css
.card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: box-shadow 0.3s;
}

.card:hover {
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
}
```

### Chips / Etiquetas

```jsx
// Activo
<Chip
  icon={<CheckCircleIcon />}
  label="Activo"
  color="success"
/>

// Inactivo
<Chip
  icon={<CancelIcon />}
  label="Inactivo"
  color="default"
/>

// Bloqueado
<Chip
  icon={<BlockIcon />}
  label="Bloqueado"
  color="error"
/>
```

**Estilos CSS para Chips:**
```css
/* Chip Activo */
.chip-success {
  background-color: #E8F5E9;
  color: #2E7D32;
}

/* Chip Inactivo */
.chip-default {
  background-color: #E0E0E0;
  color: #616161;
}

/* Chip Bloqueado */
.chip-error {
  background-color: #FFEBEE;
  color: #C62828;
}
```

---

## Iconografía

### Material Icons

Utilizamos **Material Icons** para mantener consistencia con Material UI.

```html
<link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons">
```

### Iconos Comunes

| Icono | Nombre | Uso |
|-------|--------|-----|
| login | `login` | Iniciar sesión |
| logout | `logout` | Cerrar sesión |
| search | `search` | Búsqueda |
| person | `person` | Usuario/Perfil |
| business | `business` | Cliente/Empresa |
| dashboard | `dashboard` | Panel principal |
| receipt | `receipt` | Factura |
| description | `description` | Documento |
| settings | `settings` | Configuración |
| check_circle | `check_circle` | Éxito/Confirmación |
| error | `error` | Error |
| warning | `warning` | Advertencia |

### Uso de Iconos

```jsx
import PersonIcon from '@mui/icons-material/Person';
import BusinessIcon from '@mui/icons-material/Business';

<PersonIcon color="primary" />
<BusinessIcon fontSize="large" />
```

**Tamaños:**
- small: 20px
- medium: 24px (default)
- large: 35px

**CSS:**
```css
.icon-primary {
  color: #4A5A9E;
  font-size: 36px;
}
```

---

## Elevación y Sombras

### Niveles de Elevación Material UI

```javascript
const shadows = {
  none: 'none',
  sm: '0 2px 4px rgba(0, 0, 0, 0.1)',
  md: '0 2px 8px rgba(0, 0, 0, 0.1)',
  lg: '0 4px 12px rgba(0, 0, 0, 0.15)',
  xl: '0 10px 20px rgba(0, 0, 0, 0.25)'
}
```

### Aplicación en Material UI

```jsx
<Paper elevation={2}>
  {/* Contenido */}
</Paper>

// O con sx:
<Box sx={{ boxShadow: 2 }}>
  {/* Contenido */}
</Box>
```

---

## Animaciones y Transiciones

### Curvas de Transición

```css
/* Estándar */
transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);

/* Rápida */
transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);

/* Lenta */
transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
```

### Animaciones Comunes

```css
/* Fade In */
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* Slide Down */
@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Spin (para loading) */
@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
```

---

## Patrones de Diseño

### Layout Principal

```jsx
<Box sx={{ display: 'flex', minHeight: '100vh' }}>
  {/* Sidebar/Navigation */}
  <Drawer variant="permanent">
    {/* Menú */}
  </Drawer>

  {/* Contenido Principal */}
  <Box component="main" sx={{ flexGrow: 1, p: 3 }}>
    {/* Contenido de la página */}
  </Box>
</Box>
```

### Formularios

```jsx
<Box component="form" noValidate>
  <TextField
    label="Campo requerido"
    required
    fullWidth
    margin="normal"
  />

  <Button
    type="submit"
    variant="contained"
    color="primary"
    fullWidth
    sx={{ mt: 3 }}
  >
    Enviar
  </Button>
</Box>
```

---

## Accesibilidad

### Principios Clave

1. **Contraste de Color**
   - Texto principal: mínimo 4.5:1
   - Texto grande: mínimo 3:1
   - Componentes UI: mínimo 3:1

2. **Navegación por Teclado**
   - Todos los elementos interactivos deben ser accesibles por teclado
   - Orden de tabulación lógico
   - Indicadores de foco visibles

3. **Etiquetas y Descripciones**
   ```jsx
   <TextField
     label="Email"
     aria-label="Ingrese su correo electrónico"
     aria-describedby="email-helper"
   />
   ```

4. **Roles ARIA**
   ```jsx
   <div role="alert" aria-live="polite">
     {errorMessage}
   </div>
   ```

---

## Implementación del Tema Material UI

### Archivo theme.js

```javascript
import { createTheme } from '@mui/material/styles';

const theme = createTheme({
  palette: {
    primary: {
      main: '#4A5A9E',
      light: '#6B7BC4',
      dark: '#364378',
      contrastText: '#FFFFFF'
    },
    secondary: {
      main: '#D4145A',
      light: '#E94581',
      dark: '#A00F44',
      contrastText: '#FFFFFF'
    },
    success: {
      main: '#4CAF50'
    },
    error: {
      main: '#F44336'
    },
    warning: {
      main: '#FF9800'
    },
    info: {
      main: '#2196F3'
    },
    background: {
      default: '#FAFAFA',
      paper: '#FFFFFF'
    },
    text: {
      primary: 'rgba(0, 0, 0, 0.87)',
      secondary: 'rgba(0, 0, 0, 0.6)',
      disabled: 'rgba(0, 0, 0, 0.38)'
    }
  },
  typography: {
    fontFamily: 'Roboto, Arial, sans-serif',
    h1: {
      fontSize: '2.5rem',
      fontWeight: 600
    },
    h2: {
      fontSize: '2rem',
      fontWeight: 600
    },
    h3: {
      fontSize: '1.75rem',
      fontWeight: 600
    },
    h4: {
      fontSize: '1.5rem',
      fontWeight: 600
    },
    body1: {
      fontSize: '1rem',
      fontWeight: 400
    },
    body2: {
      fontSize: '0.875rem',
      fontWeight: 400
    },
    button: {
      fontSize: '0.875rem',
      fontWeight: 600,
      textTransform: 'uppercase'
    }
  },
  shape: {
    borderRadius: 8
  },
  shadows: [
    'none',
    '0 2px 4px rgba(0, 0, 0, 0.1)',
    '0 2px 8px rgba(0, 0, 0, 0.1)',
    '0 4px 12px rgba(0, 0, 0, 0.15)',
    '0 10px 20px rgba(0, 0, 0, 0.25)',
    // ... resto de sombras Material UI
  ]
});

export default theme;
```

### Uso en la Aplicación

```javascript
import { ThemeProvider } from '@mui/material/styles';
import CssBaseline from '@mui/material/CssBaseline';
import theme from './theme';

function App() {
  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      {/* Tu aplicación */}
    </ThemeProvider>
  );
}

export default App;
```

---

## Recursos Adicionales

- [Material UI Documentation](https://mui.com/)
- [Material Icons](https://fonts.google.com/icons)
- [Material Design Guidelines](https://material.io/design)
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)

---

**Versión:** 2.0.0
**Última actualización:** 2025-12-18
**Equipo:** Diseño UI/UX - Portal Unificado CDN Facturación
