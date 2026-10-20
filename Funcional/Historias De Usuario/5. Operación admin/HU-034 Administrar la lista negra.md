#### HU-034: Administrar la lista negra (blacklist)

| Campo | Detalle |
|:---:|:---:|
| **Actor** | Administrador del producto |
| **Historia** | Como administrador, quiero agregar o retirar un documento de la lista negra, exista o no un cliente |
| **Prioridad** | Media |
| **Criterios de aceptaci贸n** | El administrador agrega un documento a la lista negra sin exigir que la persona exista como cliente. El motivo es obligatorio (m铆nimo 10 caracteres) y queda registrado qui茅n lo hizo. Si hay un 煤nico cliente b2b con ese documento, el registro se vincula a ese cliente; si no hay, queda solo el documento. Desde la ficha de un cliente existente tambi茅n se puede agregar o retirar. |
| **Relaciones** | Casos de uso: [CU-017](../../Casos de Uso/6. Operaci髇 Admin/CU-017 Administrar y consultar clientes en blacklist.md). Requerimiento:[RF-027](../../Requerimientos/Requerimientos Funcionales.md). Historia relacionada: [HU-011](../3. Credito/HU-011 Ver mensaje de no disponibilidad de cr閐ito.md)  |
| **Referencias** | `backends/b2b/src/services/client-blacklist.service.ts`, `apps/admin/src/app/product/blacklist/page.tsx` |
 **Autor** | Mar铆a Fernanda Herazo |
| **Fecha** | 18/08/2026 |
| **Versi贸n** | V.1.8 |
| **Comentarios** | **Correcci贸n v1.6**: se agrega al criterio de aceptaci贸n el registro del motivo de ingreso a la blacklist, que hac铆a falta. El motivo y qui茅n lo agreg贸 quedaron en el modelo. **Correcci贸n 2026-10-20:** la clave de la lista negra es el n煤mero de documento; el cliente es opcional. Un documento listado no entra por la landing, no inicia onboarding y no entra al portal de redemption, aunque no exista en b2b, core ni rules-engine. La persona ve un mensaje gen茅rico para contactar a Fliipa. La alerta operativa indica que no continu贸 el onboarding porque el documento est谩 en la lista negra e incluye el motivo. |



