# HU-AD009A - Gestión de Bearer Tokens SCIM

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 2 |
| **Epic** | Integración con Active Directory mediante SCIM |
| **Estado** | BORRADOR |

---

## Contexto

Gestión segura del ciclo de vida de Bearer Tokens SCIM usados para autenticación de clientes AD.

---

## Escenarios Básicos

| Nº | Criterio de aceptación (Título) | Resultado esperado |
|----|--------------------------------|---------------------|
| 1 | Generación segura de tokens | Token = UUID v4 (128 bits) + timestamp. Hash SHA-256 almacenado en BD (nunca plaintext). Token visible solo una vez al generarse |
| 2 | Regeneración de token | Admin puede regenerar token. Token anterior se invalida inmediatamente. Cliente debe actualizar configuración en AD. Registra en auditoría |
| 3 | Revocación de token | Admin puede revocar token (marca como `revoked=true`). Peticiones SCIM con ese token reciben 401. Útil si token comprometido |
| 4 | Rotación automática periódica | Opcionalmente, tokens rotan cada 90 días. Sistema notifica admin 15 días antes para coordinar actualización con cliente |
| 5 | Auditoría de uso de tokens | Cada uso del token se registra: timestamp, IP, operación. Permite detectar uso anómalo (IP desconocida, horarios inusuales) |
| 6 | Tokens con expiración opcional | Admin puede configurar fecha de expiración para token (ej: token temporal para testing). Después de expiración: 401 |

**Notas:**
- Solo hash SHA-256 se almacena en BD (no token plaintext)
- Tokens NO tienen permisos granulares (acceso completo al tenant)
- Cliente responsable de almacenar token de forma segura

---

**Versión:** 1.0
**Módulo:** Integración AD/SCIM - Credentials Management
