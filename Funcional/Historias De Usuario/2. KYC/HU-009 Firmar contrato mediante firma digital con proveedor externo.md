
#### HU-009: Firmar contrato mediante OTP 

| Campo | Detalle |
|:---:|:---:|
| **Actor** | Cliente empresarial |
| **Historia** | Como cliente empresarial, quiero firmar mi contrato mediante un código OTP, para activar mi cupo sin papeleo físico. |
| **Prioridad** | Alta |
| **Criterios de aceptación** | El cliente revisa el contrato y lo firma a través del portal Usuario por medio de un OTP enviado por los canales definido para este flujo. El sistema genera el PDF firmado y lo envía por correo al cliente con la plantilla Sendgrid *B2B - Firma de contrato* (nombre del cliente + PDF adjunto). |
| **Relaciones** | Casos de uso: [CU-007](../../Casos de Uso/2. KYC/CU-007 Firmar contrato mediante firma digital.md). Requerimientos: [RF-013](../../Requerimientos/Requerimientos Funcionales.md), [RF-014](../../Requerimientos/Requerimientos Funcionales.md). |
| **Referencias** | `sign-contract.ts`, `send-signature-otp.ts`, `send-contract/send-contract.controller.ts` |
 **Autor** | María Fernanda Herazo |
| **Fecha** | 09/09/2026 |
| **Versión** | V.1.8 |
| **Comentarios** | **Corrección v1.8**: la firma del contrato **sí se realiza con OTP**. Se revierte la v1.6 (proveedor externo): ese cambio no está en código ni es el mecanismo vigente. `send-signature-otp.ts` y `sign-contract.ts` son el flujo actual. El nombre de archivo de esta ficha se mantiene por no romper enlaces. |
