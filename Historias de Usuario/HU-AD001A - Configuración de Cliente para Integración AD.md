# HU-AD001A - Configuración de Cliente para Integración AD

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 2 |
| **Epic** | Integración con Active Directory mediante SCIM |
| **Estado** | BORRADOR |
| **Propietario** | Por definir |
| **Arquitecto** | Por definir |
| **Desarrolladores** | Por definir |
| **Tester** | Por definir |
| **Incidentes relacionados** | N/A |
| **Arquitectura relacionada** | Portal Unificado - Módulo de Administración, Integración AD/SCIM |

---

## Contexto

Esta HU permite a los **Administradores del Portal** habilitar y configurar la integración con Active Directory para un Cliente (Tenant) específico mediante SCIM 2.0 y SAML 2.0.

La configuración es **por tenant** y permite que cada Cliente se integre con su propio Active Directory de forma independiente, manteniendo la arquitectura multitenant del Portal Unificado.

Una vez configurado, el Cliente puede utilizar SAML 2.0 para autenticación (SSO) y SCIM 2.0 para sincronización de usuarios y roles desde su AD.

---

**Notas:**
- Solo Administradores del Portal pueden configurar la integración AD
- Cada Cliente (Tenant) tiene su propia configuración independiente
- La integración AD es **opcional** - se puede activar/desactivar por cliente
- SAML 2.0 es **obligatorio** para autenticación (no negociable)
- SCIM 2.0 es para sincronización unidireccional AD → Portal
- Bearer Token SCIM se genera automáticamente y debe entregarse al cliente de forma segura

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador del Portal** | Quiero activar la integración AD para un cliente específico | Para permitir que el cliente use su Active Directory para autenticación y gestión de usuarios |
| Como **Administrador del Portal** | Quiero configurar los parámetros de conexión SAML y SCIM | Para establecer la conexión segura entre el Portal y el Active Directory del cliente |
| Como **Administrador del Portal** | Quiero generar el Bearer Token SCIM para el cliente | Para que el cliente pueda autenticar las peticiones SCIM desde su AD |
| Como **Administrador del Portal** | Quiero desactivar la integración AD cuando sea necesario | Para suspender temporalmente la integración sin perder la configuración |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso a configuración de integración AD | Administrador del Portal en vista de detalle de Cliente | Usuario hace clic en "Configurar Integración AD" | Sistema muestra pantalla "Configuración Integración AD - {Nombre Cliente}" con: Toggle "Activar Integración AD" (desactivado inicialmente), sección "Configuración SAML 2.0" (colapsada), sección "Configuración SCIM 2.0" (colapsada), botón "Guardar Configuración" (deshabilitado) | ☐ | ☐ | ☐ |
| 2 | Activación de integración AD | Administrador activa toggle | Usuario activa "Activar Integración AD" | Sistema expande secciones de configuración SAML y SCIM. Muestra campos obligatorios: *URL del IdP (Identity Provider), *Certificado X.509 del IdP (textarea para pegar), *Duración de Sesión JWT (select: 1h, 2h, 4h - default 4h), URL Endpoint SCIM (generada automáticamente: https://portal.efac.com/scim/v2/{tenant_id}), Bearer Token SCIM (generado automáticamente, oculto inicialmente). Habilita botón "Guardar Configuración" | ☐ | ☐ | ☐ |
| 3 | Validación de URL del IdP | Administrador ingresa URL del IdP | Usuario completa campo y hace blur | Sistema valida: (1) Formato URL válido (https://), (2) Dominio accesible. Si válido: checkmark verde. Si inválido: error "Ingrese una URL HTTPS válida del Identity Provider" | ☐ | ☐ | ☐ |
| 4 | Validación de Certificado X.509 | Administrador pega certificado | Usuario pega certificado y hace blur | Sistema valida: (1) Formato PEM (BEGIN CERTIFICATE...END CERTIFICATE), (2) Certificado válido y no expirado. Si válido: checkmark verde y muestra fecha de expiración. Si inválido: error "Certificado inválido o formato incorrecto. Use formato PEM" | ☐ | ☐ | ☐ |
| 5 | Generación automática de Bearer Token SCIM | Administrador activa integración | Sistema genera configuración | Sistema genera Bearer Token seguro (UUID v4 + hash SHA-256), muestra campo con token oculto (***), botón "Copiar Token" (ícono), botón "Regenerar Token" (ícono). Muestra advertencia: "⚠️ Guarde este token de forma segura. No podrá verlo nuevamente" | ☐ | ☐ | ☐ |
| 6 | Copia de Bearer Token SCIM | Administrador hace clic en "Copiar Token" | Usuario copia token | Sistema copia token al portapapeles, muestra tooltip "Token copiado", registra evento de copia en auditoría | ☐ | ☐ | ☐ |
| 7 | Regeneración de Bearer Token | Administrador hace clic en "Regenerar Token" | Usuario confirma regeneración | Sistema muestra diálogo "¿Regenerar Token SCIM? El token anterior dejará de funcionar inmediatamente". Si confirma: genera nuevo token, invalida anterior, muestra nuevo token, registra en auditoría. Si cancela: cierra diálogo | ☐ | ☐ | ☐ |
| 8 | Guardado exitoso de configuración | Administrador completa todos los campos obligatorios | Usuario hace clic en "Guardar Configuración" | Sistema valida campos obligatorios, guarda configuración en BD (tenant_id, idp_url, certificate, session_duration, bearer_token_hash, estado_activo=true), genera URL endpoint SCIM, muestra mensaje "Configuración guardada exitosamente", muestra instrucciones para entregar al cliente: URL Endpoint SCIM y Bearer Token. Registra en auditoría | ☐ | ☐ | ☐ |
| 9 | Desactivación de integración AD | Administrador desactiva toggle en configuración existente | Usuario desactiva "Activar Integración AD" | Sistema muestra diálogo "¿Desactivar Integración AD? Los usuarios no podrán autenticarse con Active Directory hasta reactivarla". Si confirma: actualiza estado_activo=false en BD, mantiene configuración guardada, colapsa secciones, muestra mensaje "Integración AD desactivada", registra en auditoría. Si cancela: mantiene toggle activo | ☐ | ☐ | ☐ |
| 10 | Edición de configuración existente | Administrador accede a cliente con integración configurada | Sistema carga configuración | Sistema muestra formulario pre-poblado con: Toggle activo, URL IdP, Certificado (oculto parcialmente por seguridad), Duración sesión, URL Endpoint SCIM, Bearer Token (oculto, solo botones Copiar/Regenerar). Permite editar URL IdP, certificado, duración. Botón "Guardar Cambios" disponible | ☐ | ☐ | ☐ |
| 11 | Validación de campos obligatorios | Administrador intenta guardar sin completar obligatorios | Usuario hace clic en "Guardar" con campos faltantes | Sistema valida y muestra mensaje "Complete todos los campos obligatorios (*)", marca campos faltantes en rojo, hace scroll al primero. No permite guardar | ☐ | ☐ | ☐ |
| 12 | Error técnico durante guardado | Administrador confirma guardado pero ocurre error backend | Sistema intenta guardar pero falla | Sistema muestra mensaje "Ocurrió un error al guardar la configuración. Intente nuevamente o contacte a soporte", mantiene datos ingresados, registra error en auditoría (ERROR) | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|-----------------|
| 13 | Estructura obligatoria de auditoría | Eventos de configuración AD | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_CONFIGURACION_*), Fecha/Hora (UTC ISO 8601), Usuario, Cliente (tenant_id), IP Local, IP Pública, Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 14 | Auditoría de configuración creada | Administrador guarda configuración inicial | Sistema guarda | Registro: Tipo "INTEGRACION_AD_CONFIGURACION_CREADA", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {usuario} configuró integración AD para cliente {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "idp_url": "https://adfs.cliente.com", "session_duration": "4h", "estado_activo": true} | ☐ | ☐ | ☐ |
| 15 | Auditoría de token copiado | Administrador copia Bearer Token | Sistema copia | Registro: Tipo "INTEGRACION_AD_TOKEN_COPIADO", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {usuario} copió Bearer Token SCIM para cliente {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "token_prefix": "abc123..."} | ☐ | ☐ | ☐ |
| 16 | Auditoría de token regenerado | Administrador regenera token | Sistema regenera | Registro: Tipo "INTEGRACION_AD_TOKEN_REGENERADO", Resultado "EXITOSO", Severidad "WARNING", Descripción "Administrador {usuario} regeneró Bearer Token SCIM para cliente {nombre_cliente}. Token anterior invalidado", Datos Adicionales: {"tenant_id": "UUID", "token_anterior_prefix": "xyz789...", "token_nuevo_prefix": "abc123..."} | ☐ | ☐ | ☐ |
| 17 | Auditoría de integración desactivada | Administrador desactiva integración | Sistema desactiva | Registro: Tipo "INTEGRACION_AD_CONFIGURACION_DESACTIVADA", Resultado "EXITOSO", Severidad "WARNING", Descripción "Administrador {usuario} desactivó integración AD para cliente {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID"} | ☐ | ☐ | ☐ |
| 18 | Auditoría de configuración editada | Administrador edita configuración | Sistema guarda cambios | Registro: Tipo "INTEGRACION_AD_CONFIGURACION_EDITADA", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {usuario} editó configuración AD para cliente {nombre_cliente}", Datos Adicionales: {"tenant_id": "UUID", "cambios": {"idp_url_anterior": "https://old.com", "idp_url_nuevo": "https://new.com"}} | ☐ | ☐ | ☐ |
| 19 | Auditoría de error técnico | Error en backend | Sistema falla | Registro: Tipo "INTEGRACION_AD_CONFIGURACION_ERROR", Resultado "FALLIDO", Severidad "ERROR", Descripción "Error al configurar integración AD", Datos Adicionales: {"tenant_id": "UUID", "error_tipo": "timeout"} | ☐ | ☐ | ☐ |
| 20 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, usuario, cliente, evento, resultado. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU-AD001A-ConfiguracionAD.html** - Configuración de integración

**Elementos:**
- Título: "Configuración Integración AD - {Nombre Cliente}" (H3)
- Toggle: "Activar Integración AD" (Switch Material UI)
- **Sección SAML 2.0** (expandible cuando toggle activo):
  - *URL del IdP (TextField con validación URL HTTPS)
  - *Certificado X.509 (textarea, monospace font, placeholder: "-----BEGIN CERTIFICATE-----")
  - Fecha de expiración del certificado (read-only, calculada automáticamente)
  - *Duración de Sesión JWT (select: 1 hora, 2 horas, 4 horas - default 4h)
- **Sección SCIM 2.0** (expandible cuando toggle activo):
  - URL Endpoint SCIM (TextField read-only, valor: https://portal.efac.com/scim/v2/{tenant_id})
  - Bearer Token SCIM:
    - Campo oculto (******), botón "Copiar" (ícono), botón "Regenerar" (ícono)
    - Advertencia: "⚠️ Guarde este token de forma segura. No podrá verlo nuevamente"
- **Instrucciones para el Cliente** (info box):
  - "Proporcione al cliente: (1) URL Endpoint SCIM, (2) Bearer Token SCIM"
  - "El cliente debe configurar estos valores en su servidor SCIM AD"
- Botones:
  - "Cancelar" (outlined)
  - "Guardar Configuración" (contained gradiente, deshabilitado hasta cumplir obligatorios)

**Estados:**
- Toggle desactivado (secciones colapsadas)
- Toggle activado (secciones expandidas, campos obligatorios visibles)
- Certificado válido (checkmark verde + fecha expiración)
- Certificado inválido (error)
- Token generado (oculto con botones)
- Edición de configuración existente (campos pre-poblados)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Bearer Token expuesto** | Si el Bearer Token es interceptado, atacante podría enviar peticiones SCIM no autorizadas | Alto | Media | Generar tokens seguros (UUID + SHA-256), usar HTTPS obligatorio, mostrar token solo una vez, registrar todas las operaciones en auditoría, permitir regeneración inmediata | Mitigado |
| **Certificado X.509 expirado** | Si el certificado expira, autenticación SAML fallará | Alto | Media | Validar fecha de expiración al guardar, mostrar fecha de expiración en UI, enviar notificaciones 30 días antes de expiración (HU futura) | Mitigado |
| **URL IdP inaccesible** | Si la URL del IdP cambia o no está disponible, autenticación fallará | Alto | Baja | Validar accesibilidad al guardar, permitir edición, registrar errores de conexión en auditoría | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance

- Configuración de mapeo de roles → **HU-AD001B**
- Implementación del Endpoint SCIM → **HU-AD002A, B, C, D**
- Sincronización inicial de usuarios → **HU-AD003A, B**
- Implementación de autenticación SAML → **HU-AD004A, B, C, D**
- Notificaciones de certificados próximos a expirar → HU futura
- Prueba de conectividad con AD → HU futura

### Dependencias

**Depende de:**
- **Módulo de Administración**: Navegación desde vista de detalle de Cliente
- **Sistema de Tenants**: Cliente (Tenant) debe existir

**Es prerequisito para:**
- **HU-AD001B - Catálogo de Roles y Mapeo**: Requiere integración AD activada
- **HU-AD002A-D - Endpoint SCIM**: Usa Bearer Token generado aquí
- **HU-AD004A-D - Autenticación SAML**: Usa URL IdP y certificado configurados aquí

### Valor Independiente

- ✅ Permite configurar integración AD para clientes específicos
- ✅ Genera credenciales SCIM de forma segura
- ✅ Permite activar/desactivar integración sin perder configuración
- ✅ Puede entregarse antes que implementación de endpoints SCIM/SAML
- ✅ Permite testing de UI y validaciones de forma independiente

### Modelo de Datos (Resumen)

**Tabla: tenant_ad_configuration**
- tenant_id (UUID, FK, PK)
- estado_activo (boolean)
- idp_url (varchar 500)
- certificate_x509 (text)
- certificate_expiration_date (timestamp)
- session_duration_hours (int, default 4)
- scim_endpoint_url (varchar 500, generada)
- bearer_token_hash (varchar 256, SHA-256)
- created_at (timestamp)
- updated_at (timestamp)
- created_by (UUID, FK users)

### Referencias

- **Estándar Auditoría**: Tipos `INTEGRACION_AD_CONFIGURACION_*`
- **Glosario**: Active Directory, SCIM, SAML, IdP, Bearer Token, Tenant
- **Guía de Estilos**: Material UI, Primary/Secondary
- **SCIM 2.0 RFC 7644**: https://datatracker.ietf.org/doc/html/rfc7644
- **SAML 2.0 Spec**: http://docs.oasis-open.org/security/saml/v2.0/

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - Configuración Base
