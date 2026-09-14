# Requerimientos No Funcionales

| Documento | Requerimientos No Funcionales |
|-----------|--------------------------------|
| **Proyecto** | Fliipa |
| **Versión** | 1.0 |
| **Estado** | En revisión |
| **Responsable** | Producto, negocio y QA |
| **Última actualización** | 2026-07-10 |

---

## Control de versiones

| Versión | Fecha | Autor | Descripción |
|---------|-------|-------|-------------|
| 0.1 | 2026-07-06 | Maria Fernanda Herazo | Borrador vacío (pendiente de completar). |
| 1.0 | 2026-07-10 | María Fernanda Herazo | Primera versión completa: requerimientos de seguridad, auditoría y trazabilidad, disponibilidad/monitoreo y cumplimiento, organizados en tablas por categoría, construidos a partir de las reglas de negocio y una revisión directa del código fuente de `credits-platform-main`. |
| 1.1 | 2026-08-13 | María Fernanda Herazo | Se corrige RNF-013: ya no describe un bloqueo por fuerza bruta sobre un código de verificación (OTP) para la firma, dado que ese flujo se eliminó y la firma ahora es digital con un proveedor externo (consistente con RF-013 y HU-009). RNF-001 y RNF-007 se mantienen sin cambios: son hallazgos vigentes sobre el OTP de identidad en el checkout, no sobre la firma. |
| 1.2 | 2026-09-09 | Equipo técnico | Se revierte RNF-013: la firma del contrato **sí usa OTP**. El bloqueo por intentos fallidos (`SIGNATURE_LOCKOUT_ENABLED`, 3 intentos, 24 h) vuelve a aplicar. No hay proveedor externo de firma. |

---

## Objetivo

Definir las condiciones de calidad, seguridad, disponibilidad y cumplimiento que el sistema Fliipa debe satisfacer, más allá de sus funcionalidades. Incluye hallazgos de seguridad detectados durante la revisión del código de `credits-platform-main`, que representan brechas entre el comportamiento esperado y el comportamiento actual del sistema.

## Alcance

Cubre seguridad, auditoría y trazabilidad, disponibilidad y monitoreo, y cumplimiento. No incluye requisitos funcionales (ver [Requerimientos Funcionales](./README.md)) ni la arquitectura técnica detallada (ver [Seguridad](../../Infraestructura/01 Arquitectura y Entornos.md) y [Arquitectura](../../Infraestructura/01 Arquitectura y Entornos.md), ambos aún pendientes de completar).

## Documentos relacionados

- [Funcional](../../README.md)
- [Flipa - Biblioteca de Conocimiento](../../README.md)
- [Mapa Del Conocimiento](../../README.md)
- [Onboarding](../../README.md)
- [Convenciones](../../README.md)
- [Negocio](../../README.md)
- [Tecnico](../../Infraestructura/01 Arquitectura y Entornos.md)
- [Qa](../../README.md)
- [Documento Funcional](../../README.md)
- [Casos De Uso](../../README.md)
- [Historias Usuario](../Historias De Usuario/README.md)
- [Requerimientos Funcionales](./README.md)

## Contenido

Cada requerimiento usa el identificador `RNF-XXX`. La columna "Severidad" solo aplica a hallazgos de seguridad (Crítica/Alta/Media); para los demás requerimientos indica su prioridad de cumplimiento.

### Seguridad

| ID | Requerimiento | Severidad | Estado actual / Hallazgo | Fuente |
|----|----------------|-----------|---------------------------|--------|
| RNF-001 | La validación de OTP debe aceptar únicamente el código realmente generado para el checkout, sin códigos comodín ni excepciones hardcodeadas. | Crítica | No conforme: existe un código comodín (`"490831"`, truncado al largo esperado) que valida como correcto cualquier OTP sin importar el código real generado. | `backends/b2b/src/controllers/otp/validate-otp.ts` |
| RNF-002 | El PIN del cliente debe protegerse con un mecanismo de hash criptográfico seguro y consistente en todos los flujos que lo modifican. | Alta | No conforme: un flujo hashea el PIN con codificación base64 simple (comentario propio: "for demonstration purposes only"), mientras el flujo de checkout usa `encodeHash`, una función distinta. | `backends/b2b/src/controllers/clients/update-client-password.ts`, `checkouts/update-client-password-from-checkout.ts` |
| RNF-003 | La autenticación del panel administrativo debe estar activa en todo ambiente distinto a desarrollo local, sin posibilidad de bypass por variables de entorno mal configuradas. | Crítica | No conforme: `REQUIRE_AUTH=false` desactiva la autenticación por completo y asigna automáticamente el rol `sys_admin`. Se recomienda que se ignore si `NODE_ENV=production`. | `backends/admin/src/middleware/auth.middleware.ts`, `auth.controller.ts` |
| RNF-004 | La política de CORS del panel administrativo debe restringir explícitamente los orígenes permitidos en todo ambiente no local. | Alta | No conforme por defecto: si `CORS_ALLOWED_ORIGINS` no está configurado, se permite cualquier origen con credenciales (`origin: true`), documentado en el propio código como "solo apto para desarrollo local". | `backends/admin/src/config/cors.ts` |
| RNF-005 | Solo se deben crear o modificar cuentas del panel administrativo con dominios de correo autorizados. | Media | Conforme: se exige que el correo termine en `@sumz.co`. | `backends/admin/src/services/user.service.ts` |
| RNF-006 | Las contraseñas del panel administrativo deben almacenarse con un algoritmo de hash robusto y las sesiones deben expirar en un plazo razonable. | Media | Conforme: bcrypt (`SALT_ROUNDS = 10`) y JWT con expiración de 7 días. | `backends/admin/src/services/auth.service.ts` |
| RNF-007 | Los canales de notificación declarados como disponibles (WhatsApp, correo, SMS) deben reportar el resultado real del envío. | Alta | No conforme para SMS: responde `success: true` sin enviar el mensaje (`console.warn("SMS channel not implemented yet")`), lo que puede ocultar fallas de entrega. | `backends/b2b/src/controllers/otp/send-otp.ts` |
| RNF-008 | Los datos sensibles del cliente (documento, PIN, información bancaria) no deben incluirse en las respuestas de los endpoints de consulta salvo que sea estrictamente necesario. | Media | Conforme parcialmente: `getClient` y `updateClient` excluyen explícitamente `hashedPin` de la respuesta. | `backends/b2b/src/controllers/clients/*.ts` |

### Auditoría y trazabilidad

| ID | Requerimiento | Prioridad | Estado actual / Hallazgo | Fuente |
|----|----------------|-----------|---------------------------|--------|
| RNF-009 | Toda modificación sobre entidades de negocio del core (líneas de crédito, desembolsos, settings) debe quedar registrada con el usuario que la ejecutó, el valor anterior y el nuevo valor. | Alta | Conforme: mecanismo `runAuditedTransaction` y log `audit_log`, consultable desde el panel admin. | `backends/admin/src/controllers/clients.controller.ts` (`getClientAuditedOperations`) |
| RNF-010 | Toda interacción de cobranza con el cliente (llamada, WhatsApp, correo, visita) debe quedar registrada, incluyendo canal, resultado y evidencia cuando aplique. | Alta | Conforme: regla documentada en Reglas Negocio e implementada mediante el módulo de notas de cobranza. | [Reglas Negocio](../../README.md); `backends/b2b/src/controllers/clients/collection-notes.ts` |

### Disponibilidad y monitoreo

| ID | Requerimiento | Prioridad | Estado actual / Hallazgo | Fuente |
|----|----------------|-----------|---------------------------|--------|
| RNF-011 | El sistema debe permitir monitorear en tiempo real la disponibilidad y latencia del core bancario y de la base de datos. | Alta | Conforme. | `backends/admin/src/services/system-core-health.service.ts`, `cloud-sql-status.service.ts` |
| RNF-012 | El sistema debe monitorear la disponibilidad de los proveedores externos críticos para el negocio (riesgo, biometría, recaudo, notificaciones, core bancario). | Alta | No conforme: el monitoreo de terceros implementado cubre GitHub, npm y GCP, pero no Experian, Druo, el proveedor de biometría, Zenvia/Sendgrid ni el core bancario. | `backends/admin/src/controllers/system-third-party-status.controller.ts` |
| RNF-013 | El OTP de firma del contrato debe bloquearse temporalmente tras varios intentos fallidos, para proteger contra fuerza bruta. | Media | Conforme: `otpAttempts` (default 3) y `blockedUntil` (24 h), activable mediante `SIGNATURE_LOCKOUT_ENABLED`. | `backends/b2b/src/services/sign-contract.ts`, `backends/b2b/src/config/constants.ts` |
| RNF-014 | El acceso al portal del cliente debe bloquearse temporalmente tras varios intentos fallidos de PIN. | Media | Conforme: `loginAttempts` (default 3) y `lockedAt`, activable mediante `LOGIN_LOCKOUT_ENABLED`. | `backends/b2b/src/db/models/Client.ts`, `backends/b2b/src/config/index.ts` |

### Cumplimiento y datos

| ID | Requerimiento | Prioridad | Estado actual / Hallazgo | Fuente |
|----|----------------|-----------|---------------------------|--------|
| RNF-015 | Los parámetros financieros del crédito (tasa de interés, comisiones, seguro) deben estar documentados de forma consistente entre el código y la documentación de negocio. | Alta | No conforme: `insuranceRate` está comentado como tasa decimal (ej. "0.03") pero su valor real es `20000` (monto fijo en COP, no una tasa). Pendiente de validar con negocio y finanzas. | `backends/b2b/src/config/constants.ts` |
| RNF-016 | Los rangos de validación de un mismo campo de negocio (por ejemplo, día de corte del crédito) deben ser consistentes en todos los módulos que lo validan. | Media | No conforme: el panel admin acepta `cutoffDay` entre 0 y 31, mientras redemption exige 1 a 31. | `backends/admin/src/controllers/credit-lines.controller.ts`, `apps/redemption/lib/is-complete-redemption-credit-settings.ts` |
| RNF-017 | Los reportes a centrales de riesgo y las acciones de escalamiento jurídico deben ejecutarse según un único esquema de plazos de mora aprobado por negocio y jurídico. | Alta | Pendiente de resolver: negocio documenta dos esquemas de plazos distintos (buckets vs. journey Colpatria B2B) sin que el código disponible permita determinar cuál está implementado. | [Procesos](../../Operaciones/Procesos/README.md), [Reglas Negocio](../../README.md) |

## Fuentes consultadas

- [Reglas Negocio](../../README.md)
- [Procesos](../../Operaciones/Procesos/README.md)
- Inventario funcional del código fuente `credits-platform-main`: `backends/b2b/src/controllers/otp`, `backends/b2b/src/controllers/clients`, `backends/b2b/src/config/constants.ts`, `backends/b2b/src/config/index.ts`, `backends/admin/src/middleware/auth.middleware.ts`, `backends/admin/src/config/cors.ts`, `backends/admin/src/services/*`, `backends/admin/src/controllers/system-*.controller.ts`, realizado como parte de esta actualización.
