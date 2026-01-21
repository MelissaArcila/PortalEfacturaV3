# HU-AD005A - Gestión de Sesiones JWT

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
| **Incidencias relacionados** | N/A |
| **Arquitectura relacionada** | Portal Unificado - Autenticación, Session Management, JWT |

---

## Contexto

Esta HU implementa el sistema de **gestión de sesiones JWT** para usuarios autenticados vía SAML 2.0 (AD), incluyendo generación, validación, renovación, y almacenamiento de tokens.

Tras autenticación SAML exitosa (HU-AD004A), el Portal genera un **JWT (JSON Web Token)** que:
- Contiene información del usuario (user_id, tenant_id, roles)
- Tiene duración configurable por tenant (default 4 horas según HU-AD001A)
- Se almacena en BD para permitir invalidación proactiva (HU-AD006)
- Se envía al cliente como cookie HttpOnly segura

---

**Notas:**
- JWT es stateful (se almacena en BD) para permitir invalidación
- Duración de sesión se configura por tenant en HU-AD001A
- Sesiones de usuarios AD tienen flag `origen_saml=true`
- Refresh tokens no se implementan (usuario re-autentica vía SAML si expira)

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Portal Unificado** | Quiero generar JWT tras autenticación SAML exitosa | Para mantener sesión del usuario sin requerir autenticación en cada petición |
| Como **Portal Unificado** | Quiero almacenar JWTs en BD | Para poder invalidar sesiones proactivamente cuando cambien permisos del usuario |
| Como **Usuario** | Quiero que mi sesión dure 4 horas sin re-autenticar | Para trabajar sin interrupciones frecuentes |
| Como **Portal Unificado** | Quiero validar JWT en cada petición protegida | Para garantizar que usuario está autenticado y sesión no ha expirado o sido invalidada |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Generación de JWT tras autenticación SAML | Validación SAML exitosa, usuario existe y activo | Sistema crea sesión | Sistema genera JWT con payload: `{user_id: UUID, tenant_id: UUID, userName: "juan@empresa.com", roles: ["Administrador del Portal"], iat: timestamp, exp: iat + duración_config}`. Firma JWT con secret del Portal (HS256). Genera UUID para session_id. Almacena en tabla `sessions`: session_id, user_id, tenant_id, jwt_token, origen_saml=true, created_at, expires_at, ip_usuario, user_agent. Establece cookie HttpOnly: nombre `session_token`, valor JWT, httpOnly=true, secure=true (HTTPS only), sameSite=Strict, expires=expires_at. Devuelve JWT | ☐ | ☐ | ☐ |
| 2 | Duración de sesión configurable por tenant | Sistema genera JWT | Obtiene configuración | Sistema consulta `session_duration_hours` de `tenant_ad_configuration` (HU-AD001A). Si configurado: usa ese valor. Si NULL: usa default 4 horas. Calcula `exp = iat + (duration * 3600)`. JWT expira según configuración específica del tenant | ☐ | ☐ | ☐ |
| 3 | Validación de JWT en peticiones protegidas | Usuario hace petición a endpoint protegido | Middleware valida JWT | Sistema: (1) Lee cookie `session_token`, (2) Verifica firma JWT con secret del Portal, (3) Decodifica payload, (4) Valida `exp > now` (no expirado), (5) Consulta BD: `SELECT * FROM sessions WHERE jwt_token = {token} AND invalidated_at IS NULL`, (6) Si sesión existe y NO invalidada: permite petición, (7) Si sesión invalidada: HTTP 401 "Session invalidated", (8) Si JWT expirado: HTTP 401 "Session expired", (9) Si firma inválida: HTTP 401 "Invalid token" | ☐ | ☐ | ☐ |
| 4 | Extracción de contexto de usuario desde JWT | JWT válido | Sistema extrae datos | Middleware extrae `user_id`, `tenant_id`, `roles` del payload JWT validado. Inyecta en contexto de la petición: `req.user = {id, tenantId, userName, roles}`. Resto de la aplicación accede a usuario autenticado desde `req.user`. NO requiere consulta adicional a BD para cada petición (performance) | ☐ | ☐ | ☐ |
| 5 | Sesión expirada - Re-autenticación automática | JWT exp < now | Usuario intenta acceder | Sistema detecta JWT expirado en validación, devuelve HTTP 401. Frontend detecta 401, redirige automáticamente a login SSO (HU-AD004B escenario 7). Si usuario tiene sesión activa en IdP: silent re-authentication (obtiene nuevo JWT sin ingresar credenciales). Usuario experimenta renovación transparente | ☐ | ☐ | ☐ |
| 6 | Logout voluntario del usuario | Usuario hace clic en "Cerrar Sesión" | Sistema invalida sesión | Sistema: (1) Obtiene JWT de cookie, (2) Marca sesión en BD: `UPDATE sessions SET invalidated_at = NOW(), logout_type = 'VOLUNTARIO' WHERE jwt_token = {token}`, (3) Elimina cookie del navegador (expiration=0), (4) Registra logout en auditoría, (5) Redirige a pantalla "Sesión cerrada exitosamente" con botón "Iniciar Sesión". Sesión ya no válida para futuras peticiones | ☐ | ☐ | ☐ |
| 7 | Límite de sesiones concurrentes por usuario | Usuario inicia sesión en segundo dispositivo | Sistema cuenta sesiones | Sistema consulta sesiones activas del usuario: `SELECT COUNT(*) FROM sessions WHERE user_id = {id} AND invalidated_at IS NULL AND expires_at > NOW()`. Si count >= límite (ej: 5 sesiones): invalida sesión más antigua automáticamente, crea nueva. Si < límite: crea nueva sesión. Usuario puede tener máximo N sesiones simultáneas | ☐ | ☐ | ☐ |
| 8 | Consulta de sesiones activas del usuario | Usuario accede a "Mis Sesiones Activas" | Sistema lista sesiones | Sistema consulta: `SELECT session_id, created_at, last_activity, ip_usuario, user_agent, origen_saml FROM sessions WHERE user_id = {id} AND invalidated_at IS NULL AND expires_at > NOW()`. Muestra tabla con: Fecha Inicio, Última Actividad, Dispositivo (user_agent parseado), IP, Ubicación (estimada por IP), botón "Cerrar Sesión" individual. Usuario puede cerrar sesión en otros dispositivos | ☐ | ☐ | ☐ |
| 9 | Cerrar sesión específica (dispositivo remoto) | Usuario hace clic en "Cerrar Sesión" de otra sesión | Sistema invalida sesión remota | Sistema marca session_id específico como invalidado: `UPDATE sessions SET invalidated_at = NOW(), logout_type = 'REMOTO'`. Próxima petición desde ese dispositivo recibe 401. Usuario en ese dispositivo debe re-autenticarse | ☐ | ☐ | ☐ |
| 10 | Actualización de last_activity | Usuario hace petición con JWT válido | Middleware actualiza timestamp | Sistema actualiza: `UPDATE sessions SET last_activity = NOW() WHERE jwt_token = {token}` (cada 5 minutos para no saturar BD). Permite monitorear inactividad y sesiones zombie | ☐ | ☐ | ☐ |
| 11 | Cleanup de sesiones expiradas | Job programado (diario, 2am) | Sistema limpia BD | Sistema ejecuta: `DELETE FROM sessions WHERE expires_at < NOW() - INTERVAL '7 days'` (borra sesiones expiradas hace >7 días, mantiene histórico reciente). Reduce tamaño de tabla. Registra cantidad de sesiones eliminadas en log técnico | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 12 | Estructura obligatoria de auditoría | Eventos de sesión | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_SESION_*), Fecha/Hora (UTC ISO 8601), Usuario (user_id), Cliente (tenant_id), IP Local (NULL), IP Pública (IP usuario), Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 13 | Auditoría de sesión creada | JWT generado tras SAML | Sesión iniciada | Registro: Tipo "INTEGRACION_AD_SESION_CREADA", Resultado "EXITOSO", Severidad "INFO", Descripción "Sesión creada para usuario {userName} vía SAML", Datos Adicionales: {"session_id": "UUID", "user_id": "UUID", "tenant_id": "UUID", "duracion_horas": 4, "ip_usuario": "203.0.113.5", "user_agent": "Mozilla/5.0..."} | ☐ | ☐ | ☐ |
| 14 | Auditoría de logout voluntario | Usuario cierra sesión | Sesión cerrada | Registro: Tipo "INTEGRACION_AD_SESION_LOGOUT", Resultado "EXITOSO", Severidad "INFO", Descripción "Usuario {userName} cerró sesión voluntariamente", Datos Adicionales: {"session_id": "UUID", "duracion_sesion_minutos": 125} | ☐ | ☐ | ☐ |
| 15 | Auditoría de sesión expirada | JWT exp alcanzado | Acceso denegado | Registro: Tipo "INTEGRACION_AD_SESION_EXPIRADA", Resultado "FALLIDO", Severidad "INFO", Descripción "Intento de acceso con sesión expirada", Datos Adicionales: {"session_id": "UUID", "user_id": "UUID", "exp_timestamp": "2026-01-20T14:00:00Z"} | ☐ | ☐ | ☐ |
| 16 | Auditoría de sesión invalidada | Sesión cerrada remotamente | Acceso denegado | Registro: Tipo "INTEGRACION_AD_SESION_INVALIDADA", Resultado "FALLIDO", Severidad "INFO", Descripción "Intento de acceso con sesión invalidada", Datos Adicionales: {"session_id": "UUID", "invalidated_at": "2026-01-20T13:00:00Z", "logout_type": "REMOTO"} | ☐ | ☐ | ☐ |
| 17 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, usuario, tenant, evento. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU-AD005A-SesionesActivas.html** - Vista de sesiones

**Elementos:**
- Título: "Mis Sesiones Activas" (H3)
- Subtítulo: "Dispositivos con sesión iniciada en su cuenta"
- **Tarjeta por sesión:**
  - Etiqueta "Sesión Actual" (si es la sesión del navegador actual)
  - Dispositivo: "Chrome en Windows 10" (parseado de user_agent)
  - IP: "203.0.113.5"
  - Ubicación: "Medellín, Colombia" (estimada)
  - Inicio: "20 Ene 2026, 10:00 AM"
  - Última actividad: "Hace 5 minutos"
  - Botón "Cerrar Sesión" (outlined, deshabilitado si es sesión actual, confirmación si no)
- Info box: "Si no reconoce alguna de estas sesiones, ciérrela inmediatamente y cambie su contraseña corporativa"
- Botón "Cerrar Todas las Demás Sesiones" (text, confirmación)

**Estados:**
- 1 sesión (solo actual)
- 3-4 sesiones (múltiples dispositivos)
- Sesión cerrada (confirmación)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **JWT secret comprometido** | Si secret se filtra, atacante puede generar JWTs válidos | Crítico | Baja | Secret fuerte (256 bits random), rotación periódica (HU futura), almacenar en variables de entorno seguras, NO en código | Mitigado |
| **Sesiones no invalidadas tras cambio de roles** | Usuario conserva permisos antiguos en JWT hasta expiración | Alto | Media | Implementar invalidación proactiva (HU-AD006C), TTL corto de JWT (4h), validar roles críticos en backend además de JWT | Mitigado |
| **Tabla sessions crece indefinidamente** | Millones de sesiones podrían saturar BD | Medio | Alta | Cleanup automático diario (escenario 11), índices optimizados, particionamiento de tabla si >10M registros | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance

- Refresh tokens → Usuario re-autentica vía SAML al expirar
- JWT firmado con RSA (asimétrico) → HS256 (simétrico) es suficiente
- Revocación de JWT sin almacenar en BD (stateless JWT) → Stateful por diseño para invalidación
- Geolocalización precisa de IP → Solo estimación básica

### Dependencias

**Depende de:**
- **HU-AD004A - SAML Authentication**: Genera sesión tras autenticación
- **HU-AD001A - Configuración**: Duración de sesión configurable

**Es prerequisito para:**
- **HU-AD006A, B, C - Invalidación Proactiva**: Usa tabla sessions

### Valor Independiente

- ✅ Permite sesiones persistentes tras SAML
- ✅ Soporta múltiples dispositivos
- ✅ Usuario puede gestionar sus sesiones
- ✅ Base para invalidación proactiva

### Especificación Técnica

**Modelo de Datos:**
```sql
CREATE TABLE sessions (
  session_id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  tenant_id UUID REFERENCES tenants(id),
  jwt_token TEXT NOT NULL,
  origen_saml BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW(),
  expires_at TIMESTAMP NOT NULL,
  last_activity TIMESTAMP DEFAULT NOW(),
  invalidated_at TIMESTAMP NULL,
  logout_type VARCHAR(50) NULL, -- VOLUNTARIO, REMOTO, PROACTIVO, EXPIRADO
  ip_usuario VARCHAR(45),
  user_agent TEXT
);

CREATE INDEX idx_sessions_user_active ON sessions(user_id) WHERE invalidated_at IS NULL AND expires_at > NOW();
CREATE INDEX idx_sessions_jwt ON sessions(jwt_token) WHERE invalidated_at IS NULL;
CREATE INDEX idx_sessions_expires ON sessions(expires_at);
```

**Estructura JWT:**
```json
{
  "user_id": "f1e2d3c4-b5a6-7890-cdef-1234567890ab",
  "tenant_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "userName": "juan.perez@empresa.com",
  "roles": ["Administrador del Portal", "Contador"],
  "iat": 1705747200,
  "exp": 1705761600
}
```

**Firma JWT:**
```javascript
const jwt = require('jsonwebtoken');

// Generar
const token = jwt.sign(payload, process.env.JWT_SECRET, {
  algorithm: 'HS256'
});

// Validar
try {
  const decoded = jwt.verify(token, process.env.JWT_SECRET);
  // decoded contiene payload
} catch (err) {
  // Firma inválida o expirado
}
```

**Cookie Segura:**
```javascript
res.cookie('session_token', jwt, {
  httpOnly: true,    // No accesible desde JavaScript
  secure: true,      // Solo HTTPS
  sameSite: 'Strict', // Protección CSRF
  maxAge: durationMs, // Expira con JWT
  path: '/'
});
```

### Referencias

- **RFC 7519 - JSON Web Token (JWT)**: https://datatracker.ietf.org/doc/html/rfc7519
- **OWASP JWT Cheat Sheet**: https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html
- **Estándar Auditoría**: Tipos `INTEGRACION_AD_SESION_*`
- **Glosario**: JWT, Session, HttpOnly Cookie, Stateful JWT

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - Session Management
