# Mockups - Portal Unificado Efactura

Este directorio contiene los mockups interactivos en HTML para las diferentes historias de usuario del Portal Unificado.

## HU001 - Autenticación y Selección de Cliente

### Archivos disponibles

1. **HU001-Login.html** - Pantalla de autenticación
2. **HU001-ClientSelection.html** - Pantalla de selección de cliente

### Cómo visualizar los mockups

Simplemente abre los archivos HTML en tu navegador web preferido. Los mockups son completamente funcionales y no requieren servidor web ni dependencias adicionales.

### Características

- **Totalmente interactivos**: Incluyen funcionalidad JavaScript para simular el comportamiento real
- **Selector de estados**: Cada mockup incluye un panel en la esquina superior derecha que permite alternar entre diferentes estados de la interfaz
- **Diseño responsivo**: Se adaptan automáticamente a diferentes tamaños de pantalla
- **Sin dependencias**: No requieren frameworks externos, solo HTML, CSS y JavaScript vanilla

### Estados disponibles por mockup

#### HU001-Login.html
- **Normal**: Estado inicial sin errores
- **Error - Credenciales incorrectas**: Muestra el mensaje de error cuando las credenciales son inválidas
- **Error - Acceso no disponible**: Muestra el mensaje cuando el usuario no tiene clientes activos

#### HU001-ClientSelection.html
- **Normal**: Vista estándar con 8 clientes
- **Pocos clientes**: Vista con solo 3 clientes disponibles
- **Muchos clientes**: Vista con 15 clientes para probar el scroll
- **Error - Cliente inactivo**: Muestra el error al seleccionar un cliente inactivo

### Funcionalidades implementadas

#### Pantalla de Login
- Campos de entrada para usuario y contraseña
- Validación visual de errores
- Mensajes de error contextuales
- Diseño centrado y accesible

#### Pantalla de Selección de Cliente
- Lista desplegable con búsqueda en tiempo real
- Filtrado por NIT o nombre de cliente
- Contador dinámico de clientes disponibles
- Indicador visual del cliente seleccionado
- Botón de ingreso que se habilita solo cuando hay selección
- Información del usuario autenticado
- Opciones para continuar o cancelar

### Tecnologías utilizadas

- HTML5
- CSS3 (Flexbox, Grid, Gradientes, Transiciones)
- JavaScript ES6+ (Vanilla)

### Notas de diseño

Los mockups implementan:
- Paleta de colores moderna con gradientes (#667eea a #764ba2)
- Tipografía system fonts para mejor rendimiento
- Espaciado consistente y jerarquía visual clara
- Feedback visual en interacciones (hover, focus, active)
- Accesibilidad básica (labels, placeholder text, estados de focus)
- Animaciones y transiciones suaves para mejor UX

### Próximos pasos

Estos mockups sirven como referencia visual para:
1. Validación con stakeholders y usuarios
2. Definición de componentes reutilizables
3. Guía de implementación para el equipo de desarrollo
4. Documentación de patrones de diseño del sistema

### Actualizaciones

- **2025-12-18**: Creación inicial de mockups para HU001
