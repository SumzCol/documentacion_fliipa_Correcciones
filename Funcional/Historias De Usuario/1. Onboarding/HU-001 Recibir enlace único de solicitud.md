#### HU-001: Recibir enlace único de solicitud
| Campo | Detalle |
|:---:|:---:|
| **Actor** | Cliente empresarial |
| **Historia** | Como cliente empresarial, quiero recibir un enlace único de solicitud por WhatsApp, para poder iniciar mi proceso de crédito de forma directa. |
| **Prioridad** | Alta |
| **Criterios de aceptación** | El cliente recibe por WhatsApp un enlace único asociado a su proceso de solicitud. Al acceder al enlace, el sistema valida el Documento de Identidad y, si ya existe una solicitud en estado `REQUEST_STARTED`, la reutiliza; si no existe una solicitud iniciada, crea una nueva. El hecho de que el cliente ya esté registrado no impide el ingreso al proceso. |
| **Relaciones** | Casos de uso: [CU-001](../../Casos de Uso/1. Onboarding/CU-001 Iniciar contacto y solicitud de Credito.md), [CU-002](../../Casos de Uso/1. Onboarding/CU-002 Consultar y ver cupo preaprobado.md). Historias relacionadas: [HU-002](../1. Onboarding/HU-002 Consultar si tengo cupo preaprobado con mi n�mero de documento.md), [HU-003](../1. Onboarding/HU-003 Ver cupo preaprobado antes de completar el formulario.md), [HU-019](../1. Onboarding/HU-019 Contacto simult�neo multicanal.md). |
| **Referencias** | [Procesos — 01 Onboarding Digital](../../../Operaciones/Procesos/01 Onboarding Digital.md); invite comercial `buildCampaignInviteLink` → `{ link, id }` (`backends/b2b`); landing `/invite/c/{c3}`; campañas admin Comunicaciones |
| **Autor** | María Fernanda Herazo |
| **Fecha** | 18/08/2026 |
| **Versión** | v.1.8 |
| **Comentarios** | **Confirmado en plataforma:** al entrar con el enlace, el sistema valida el Documento de Identidad y, si ya existe una solicitud empezada en estado `REQUEST_STARTED`, la reutiliza; si no existe, crea una nueva. Que el cliente ya esté registrado no implica rechazar el ingreso. **Ajuste nov 2026:** el invite se genera como URL `{base}/invite/c/{c3.…}` y, por separado, el ID `c3.…` (sin URL) para variables de plantillas WhatsApp/Zenvia con prefijo fijo; el admin puede copiar link o ID desde acciones del cliente Core. |

