#### HU-043 — Evaluar las reglas KYC y guardar el resultado de cada una
| Campo | Detalle |
|:---:|:---:|
| **Actor** | Sistema (Fliipa) |
| **Historia** | Como Fliipa, quiero evaluar las cuatro reglas duras de KYC y guardar el resultado de cada una (con contraste onboarding vs consultado), para soporte y trazabilidad. |
| **Prioridad** | Alta |
| **Criterios de aceptación** | Se evalúan siempre: representante/identidad; Colpatria; Reconocer (contacto); cuenta bancaria vs centrales. Identidad: CC/CE compara tipo y número de documento contra Reconocer del titular. NIT compara primero el representante de la cámara embebida en HPJ; si no hay, usa el Reconocer de la cédula del representante ya consultado al inicio. Advance no aporta representante. Si no hay documento en esas fuentes, no hay persona para comparar. Cada regla deja outcome y, si aplica, causal y comparación lado a lado. Fallas de "no cumple" se distinguen de "servicio de centrales caído". |
| **Relaciones** | [HU-038](../5. Operaci�n admin/HU-038 Consultar informaci�n de centrales.md), [HU-039](../5. Operaci�n admin/HU-039  Ver coincidencias entre Fliipa y Reconocer.md) (consulta y match **manual** en admin); [HU-042](../2. KYC/HU-042  Iniciar la evaluaci�n KYC al quedar la solicitud en REQUESTED.md); [HU-044](../2. KYC/HU-044 Decidir approved - pre-rejected.md). |
| **Autor** | María Fernanda Herazo |
| **Fecha** | 18/08/2026 |
| **Versión** | V.1.8 |
| **Estado en plataforma** | **En desarrollo.** Las 4 reglas ya corren en el motor con comparación; se está completando persistir la corrida/expediente por cliente y usarla en el flujo automático. |
| **Comentarios** | La regla de cuenta compara onboarding vs centrales; no es validar el PDF ni Olimpia. El match visual de Reconocer en admin (HU-039) no es esta regla automática. |
