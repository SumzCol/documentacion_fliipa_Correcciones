### CU-007: Firmar contrato mediante firma digital

![Diagrama de caso de uso CU-007](imagenes/diagrama_CU-007.svg)

| Campo | Detalle |
|---|---|
| **Actores** | Cliente empresarial |
| **Descripción** | El cliente revisa y firma su contrato mediante código OTP, para activar su cupo de crédito sin necesidad de papeleo físico. Este es el mecanismo de firma vigente; no hay proveedor externo de firma digital. |
| **Precondiciones** | La solicitud del cliente fue aprobada y está lista para la generación del contrato. |
| **Flujo principal** | 1. El sistema genera el contrato del cliente.<br>2. El cliente revisa el contenido del contrato.<br>3. El cliente firma el contrato mediante el código OTP enviado para este fin.<br>4. El sistema genera el PDF firmado.<br>5. El sistema envía el PDF firmado al correo del cliente con la plantilla Sendgrid *B2B - Firma de contrato* (saludo con el nombre del cliente).<br>6. El sistema registra el timestamp, OTP y copia del contrato exacto que firmo en ese momento para referencias futuras. |
| **Flujos alternativos / excepciones** | A1. El cliente no completa la firma: el contrato queda pendiente de firma y el cupo no se activa. |
| **Postcondiciones** | El cliente cuenta con el contrato firmado y su cupo queda activado. |
| **Reglas de negocio** | La firma se realiza únicamente mediante código OTP; ese es el mecanismo vigente y el que se debe mantener. |
| **Historias de usuario relacionadas** | [HU-009](../../Historias De Usuario/2. KYC/HU-009 Firmar contrato mediante firma digital con proveedor externo.md) (Firmar contrato mediante OTP) |
| **Estado en plataforma** | Implementado (`sign-contract.ts`, `send-contract.controller.ts`, `send-signature-otp.ts`, `contract-pdf-email.service.ts`, catálogo `contract` en communications). El OTP **es** el mecanismo de firma; no hay proveedor externo. El correo post-firma usa Sendgrid (plantilla *B2B - Firma de contrato*) y sigue adjuntando el PDF. |
| **Referencias** | Fuente: ficha [HU-009](../../Historias De Usuario/2. KYC/HU-009 Firmar contrato mediante firma digital con proveedor externo.md) (Firmar contrato mediante OTP)— *Historias de Usuario — Fliipa*, carpeta "2. KYC" (repositorio `documentacion_fliipa`, María Fernanda Herazo). |
