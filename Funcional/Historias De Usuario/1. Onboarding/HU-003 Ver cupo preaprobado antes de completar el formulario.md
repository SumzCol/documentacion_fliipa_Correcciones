#### HU-003: Ver cupo preaprobado antes de completar el formulario
| Campo | Detalle |
|:---:|:---:|
| **Actor** | Cliente empresarial |
| **Historia** | Como cliente empresarial, quiero consultar mi cupo preaprobado antes de completar el formulario, para decidir si continúo con el proceso de solicitud. |
| **Prioridad** | Alta |
| **Criterios de aceptación** | Una vez identificada la preaprobación del cliente, el sistema muestra el valor del cupo preaprobado o sugerido antes de exigirle completar todos los pasos del onboarding. El valor mostrado corresponde al cupo disponible para el proceso de solicitud según la información de preaprobación cargada y procesada por la plataforma. |
| **Relaciones** | Casos de uso: [CU-002](../../Casos de Uso/1. Onboarding/CU-002 Consultar y ver cupo preaprobado.md). Requerimiento: [RF-005](../../Requerimientos/Requerimientos Funcionales.md). Historias relacionadas : [HU-002](../1. Onboarding/HU-002 Consultar si tengo cupo preaprobado con mi n�mero de documento.md), [HU-004](../1. Onboarding/HU-004 Confirmar identidad.md). |
| **Referencias** | [Procesos — 01 Onboarding Digital](../../../Operaciones/Procesos/01 Onboarding Digital.md); `b2b/fliipa-back/src/controllers/checkouts/create-checkout.ts` |
| **Autor** | María Fernanda Herazo |
| **Fecha** | 18/08/2026 |
| **Versión** | V.1.7 |
| **Comentarios** | **Confirmado en plataforma:** la preaprobación y el cupo están operativos en el motor de riesgo. Tras la carga del archivo, las reglas duras evaluables son `PCT_EFECTIVO` y `CUPO_REAL` (configurables en settings de producto); el valor de cupo preaprobado / asignado corresponde a **cupo real** (`CUPO_REAL`), no a `CUPO_CALCULADO`. |


