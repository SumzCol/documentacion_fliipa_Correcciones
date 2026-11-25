### CU-001: Iniciar contacto y solicitud de crédito

![Diagrama de caso de uso CU-001](imagenes/diagrama_CU-001.svg)

| Campo | Detalle |
|---|---|
| **Actores** | Cliente empresarial (principal); Asesor comercial (secundario) |
| **Descripción** | Un cliente preaprobado es contactado por el asesor comercial por varios canales y recibe un enlace único para iniciar (o retomar) su proceso de solicitud de crédito, con acompañamiento comercial durante todo el proceso. |
| **Precondiciones** | El cliente está en la base de clientes preaprobados. El asesor cuenta con las plantillas de contacto para correo, WhatsApp y llamada. |
| **Flujo principal** | 1. El asesor comercial se pone en contacto con el cliente a través de diferentes canales de comunicación, como correo electrónico, WhatsApp y llamadas telefónicas.<br>2. El cliente recibe por WhatsApp, enviado por el sistema como parte del proceso comercial (no generado ni enviado manualmente por el asesor), un enlace único asociado a su proceso de solicitud.<br>3. El cliente accede al enlace.<br>4. El sistema valida el Documento de Identidad del cliente.<br>5. Si existe una solicitud en estado `REQUEST_STARTED`, el sistema la reutiliza; si no, crea una nueva.<br>6. El asesor hace seguimiento remoto al cliente durante el proceso, resolviendo dudas o inquietudes. |
| **Flujos alternativos / excepciones** | A1. El cliente ya está registrado previamente: esto no impide su ingreso al proceso.<br>A2. El cliente no responde por ningún canal: el asesor registra la tasa de respuesta por canal para el seguimiento comercial. |
| **Postcondiciones** | El cliente cuenta con una solicitud de crédito activa (nueva o reutilizada) y ha sido contactado por el canal comercial correspondiente. |
| **Reglas de negocio** | El contacto debe intentarse por los tres canales definidos (correo, WhatsApp, llamada). El seguimiento comercial no requiere visitas presenciales. |
| **Historias de usuario relacionadas** | [HU-001](../../Historias De Usuario/1. Onboarding/HU-001 Recibir enlace �nico de solicitud.md) (Recibir enlace único de solicitud)<br>[HU-019](../../Historias De Usuario/1. Onboarding/HU-019 Contacto simult�neo multicanal.md) (Contacto simultáneo multicanal)<br>[HU-020](../../Historias De Usuario/1. Onboarding/HU-020 Acompa�ar la originaci�n del cliente.md) (Acompañar la originación del cliente) |
| **Estado en plataforma** | Enlace único / invite comercial (`/invite/c/c3.…`) y reutilización de solicitud: implementado. Campañas masivas email/WhatsApp desde admin **Comunicaciones** (CSV + mapeo de variables: link completo o ID `c3.…` sin URL para botones Zenvia). Contacto telefónico y orquestación simultánea de los tres canales: no hay herramienta única en código; sigue siendo proceso comercial / CRM externo. |
| **Referencias** | Fuente: fichas [HU-001](../../Historias De Usuario/1. Onboarding/HU-001 Recibir enlace �nico de solicitud.md), [HU-019](../../Historias De Usuario/1. Onboarding/HU-019 Contacto simult�neo multicanal.md) y [HU-020](../../Historias De Usuario/1. Onboarding/HU-020 Acompa�ar la originaci�n del cliente.md) — *Historias de Usuario — Fliipa*, carpeta "1. Onboarding" (repositorio `documentacion_fliipa`, María Fernanda Herazo).. |


