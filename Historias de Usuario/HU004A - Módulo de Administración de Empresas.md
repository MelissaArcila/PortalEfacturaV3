# HU004A - Módulo de Administración de Empresas

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 1 |
| **Epic** | Gestión de Clientes y Administración |
| **Estado** | BORRADOR |
| **Propietario** | Por definir |
| **Arquitecto** | Por definir |
| **Desarrolladores** | Por definir |
| **Tester** | Por definir |
| **Incidentes relacionados** | N/A |
| **Arquitectura relacionada** | Portal Unificado - Módulo de Administración |

---

## Contexto

El Portal Unificado requiere un módulo centralizado para la administración de Empresas (Clientes) que permita a los Administradores del Portal visualizar, buscar y gestionar las empresas registradas en el sistema.

Este módulo actúa como punto de entrada para todas las operaciones relacionadas con empresas: creación (HU004B), edición de datos de identificación, información complementaria (HU004C), ubicación (HU004D), y registros mercantiles (HU004E).

Esta HU establece la **base del módulo de administración** y es independiente de las HUs de gestión de datos específicos de empresa.

---

**Notas:**
- Solo Usuarios con rol de Administrador del Portal pueden acceder al módulo
- Esta HU NO incluye la creación o edición de datos de empresa, solo la visualización y navegación
- La creación y edición de empresas se cubren en HU004B, HU004C, HU004D, HU004E

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador del Portal** | Quiero acceder a un módulo de Administración de Empresas | Para tener un punto centralizado donde gestionar todas las empresas del sistema |
| Como **Administrador del Portal** | Quiero ver un listado de todas las empresas registradas | Para tener visibilidad completa de los clientes en el sistema |
| Como **Administrador del Portal** | Quiero buscar empresas por NIT o nombre | Para encontrar rápidamente una empresa específica |
| Como **Administrador del Portal** | Quiero navegar a la creación de nueva empresa | Para registrar nuevos clientes en el sistema |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso al módulo con permisos correctos | Usuario con rol Administrador del Portal autenticado | Usuario navega al menú de Administración | Sistema muestra opción "Administración de Empresas" en el menú. Al hacer clic, navega al módulo y muestra pantalla de listado de empresas | ☐ | ☐ | ☐ |
| 2 | Acceso denegado sin permisos | Usuario SIN rol Administrador del Portal intenta acceder al módulo mediante URL directa | Usuario accede a URL del módulo | Sistema valida permisos, muestra mensaje "No tiene permisos para acceder a esta sección", redirige a pantalla de inicio, y registra intento en auditoría | ☐ | ☐ | ☐ |
| 3 | Visualización de listado de empresas | Administrador accede al módulo | Sistema carga listado | Sistema muestra tabla con columnas: Número de Identificación, Nombre (Razón Social o Nombres y Apellidos), Estado (badge Activo/Inactivo), Fecha de Creación, Acciones (Ver/Editar). Ordenado por fecha de creación descendente. Paginación de 20 registros por página | ☐ | ☐ | ☐ |
| 4 | Búsqueda de empresas | Administrador en listado de empresas | Usuario escribe en campo de búsqueda y presiona Enter o botón Buscar | Sistema filtra listado mostrando solo empresas cuyo Número de Identificación o Nombre contengan el texto ingresado (búsqueda parcial, case insensitive). Muestra mensaje "X empresas encontradas" o "No se encontraron empresas" si no hay resultados | ☐ | ☐ | ☐ |
| 5 | Navegación a creación de empresa | Administrador en listado | Administrador hace clic en botón "Crear Nueva Empresa" | Sistema navega a pantalla de creación de empresa (HU004B - Datos de Identificación) | ☐ | ☐ | ☐ |
| 6 | Navegación a vista de detalle de empresa | Administrador en listado | Administrador hace clic en botón "Ver" junto a una empresa | Sistema navega a pantalla de detalle que muestra toda la información de la empresa: Identificación, Información Complementaria, Ubicación, Registros Mercantiles. Opciones para editar cada sección disponibles | ☐ | ☐ | ☐ |
| 7 | Listado vacío - primera vez | Administrador accede al módulo en sistema nuevo sin empresas creadas | Sistema carga listado | Sistema muestra mensaje "No hay empresas registradas. Cree la primera empresa haciendo clic en el botón Crear Nueva Empresa" con ilustración/icono. Botón "Crear Nueva Empresa" prominente | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 8 | Estructura obligatoria de registro de auditoría | Eventos de acceso al módulo | Sistema registra evento | Registro contiene: ID Único del Evento (UUID), Tipo de Evento (ADMINISTRACION_EMPRESA_*), Fecha y Hora (UTC ISO 8601), Usuario, Cliente (NULL), IP Local, IP Pública, Resultado (EXITOSO/FALLIDO), Descripción, Nivel de Severidad (INFO/WARNING/ERROR), Datos Adicionales (JSON) | ☐ | ☐ | ☐ |
| 9 | Auditoría de acceso exitoso al módulo | Administrador autorizado accede al módulo | Sistema carga módulo | Registro: Tipo "ADMINISTRACION_EMPRESA_ACCESO", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {usuario} accedió al módulo de Administración de Empresas", Datos Adicionales: {"empresas_totales": 45} | ☐ | ☐ | ☐ |
| 10 | Auditoría de acceso denegado | Usuario sin permisos intenta acceder | Sistema deniega acceso | Registro: Tipo "ADMINISTRACION_EMPRESA_ACCESO_DENEGADO", Resultado "FALLIDO", Severidad "WARNING", Descripción "Usuario {usuario} sin permisos intentó acceder al módulo", Datos Adicionales: {"rol_usuario": "Gestor FE", "url_intentada": "/admin/empresas"} | ☐ | ☐ | ☐ |
| 11 | Auditoría de búsqueda de empresas | Administrador realiza búsqueda | Usuario busca | Registro: Tipo "ADMINISTRACION_EMPRESA_BUSQUEDA", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {usuario} buscó empresas con criterio {texto_busqueda}", Datos Adicionales: {"texto_busqueda": "900123", "resultados_encontrados": 3} | ☐ | ☐ | ☐ |
| 12 | Inmutabilidad y consulta de auditoría | Cualquier evento | Sistema genera registro | Registros NO pueden modificarse ni eliminarse. Solo lectura. Permite filtrar por: fechas, usuario, tipo de evento, resultado, severidad. Exportable a CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU004A-ListadoEmpresas.html** - Listado de empresas

**Elementos:**
- Título: "Administración de Empresas" (H3, color Primary Dark)
- Botón "Crear Nueva Empresa" (contained, gradiente Primary → Secondary, prominente, alineado derecha)
- Campo de búsqueda (TextField outlined, placeholder "Buscar por NIT o nombre...", icono de lupa, con botón "Buscar")
- Tabla de empresas:
  - Columnas: Número de Identificación, Nombre, Estado, Fecha de Creación, Acciones
  - Estado: badge verde "Activo" o gris "Inactivo"
  - Acciones: botón "Ver" (outlined), botón "Editar" (text)
- Paginación (20 por página): "Mostrando 1-20 de 45 empresas", botones Anterior/Siguiente
- Estado vacío: Ilustración + mensaje + botón CTA

**Estados a implementar:**
- Listado con empresas (5-10 empresas de ejemplo)
- Listado vacío (primera vez)
- Listado con resultados de búsqueda
- Listado sin resultados de búsqueda

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Acceso no autorizado por validación solo en frontend** | Si la validación de permisos solo se implementa en frontend, usuarios sin permisos podrían acceder mediante URL directa o manipulación | Alto | Baja | Validar permisos en backend en TODOS los endpoints. Frontend oculta opciones + Backend rechaza con HTTP 403. Registrar intentos en auditoría | Mitigado |
| **Rendimiento con muchas empresas** | Si el sistema tiene miles de empresas, el listado podría tardar en cargar o saturar la interfaz | Medio | Media | Implementar paginación (20 por página), índices en base de datos, búsqueda del lado servidor, lazy loading | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance (Out of Scope)

**Esta HU NO incluye:**
- Creación de empresas → **HU004B - Datos de Identificación**
- Edición de datos de identificación → **HU004B**
- Edición de información complementaria → **HU004C**
- Edición de ubicación → **HU004D**
- Gestión de registros mercantiles → **HU004E**
- Inactivación/eliminación de empresas → HU futura
- Exportación de listado a Excel/CSV → HU futura
- Filtros avanzados (por estado, fecha, tipo) → HU futura

### Dependencias

**Esta HU depende de:**
- **HU001 - Autenticación**: Usuario Administrador debe estar autenticado
- **Sistema de Permisos**: Validación de rol "Administrador del Portal"

**Esta HU es prerequisito para:**
- **HU004B - Datos de Identificación de Empresa**: Navegación desde botón "Crear Nueva Empresa"
- **HU004C, D, E**: Navegación desde vista de detalle para editar secciones

### Valor de Negocio Independiente

Esta HU puede entregarse de forma independiente:
- ✅ Permite visualizar empresas existentes (si las hay en BD)
- ✅ Permite buscar empresas
- ✅ Valida permisos de acceso
- ✅ Establece base para navegación a creación/edición
- ✅ Puede funcionar con empresas creadas manualmente en BD para testing

### Referencias

- **Estándar de Auditoría Funcional**: Tipos `ADMINISTRACION_EMPRESA_*`
- **Estándar de Seguridad APL03**: Validación de permisos en todas las capas
- **Glosario**: Empresa, Cliente, Administrador del Portal
- **Guía de Estilos**: Material UI, paleta Primary/Secondary, componentes Table/TextField/Button/Badge

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**HU Original:** HU004 - Creación de Empresas (atomizada en HU004A, B, C, D, E)
