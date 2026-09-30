#### HU-013: Prepagar la cuota por PSE

| Campo | Detalle |
|:---:|:---:|
| **Actor** | Cliente empresarial |
| **Historia** | Como cliente empresarial, quiero prepagar mi crédito antes de su fecha de vencimiento mediante PSE, para no incurrir en mora y mantener un buen comportamiento de crédito. |
| **Prioridad** | Media |
| **Criterios de aceptación** | 1. Desde el portal de redención, el cliente puede solicitar un link de pago Druo (PSE) asociado a un desembolso vigente.<br><br>2. El link cobra el **saldo total adeudado hoy** del desembolso (`balanceDue`), no una cuota parcial.<br><br>3. Tras completar el pago en la página hospedada de Druo, el sistema recibe el webhook `transaction.successful`, concilia el link local y registra el abono en el crédito (`payment_references` en estado `success`).<br><br>4. El plan de pagos / saldo del cliente refleja el abono una vez registrado en core.<br><br>5. Si el registro en core falla tras marcar el link como pagado, una redelivery del webhook reintenta el registro mientras no exista `core_payment_reference_id`. |
| **Relaciones** | Casos de uso: [CU-011](../../Casos de Uso/4. Cobranza/CU-011 pagar y gestionar cuotas del credito .md) Requerimiento:  [RF-022](../../Requerimientos/Requerimientos Funcionales.md). Historia relacionada: [HU-014](../4. Cobranza/HU-014 Debito automático de créditos vencidos.md). |
| **Referencias** | [Procesos](../../../Operaciones/Procesos/07 Modelo Cobranza.md); implementación en `credits-platform`: mint del link en b2b (`create-disbursement-payment-link`), ingreso webhook en `services/webhooks` → b2b `POST /internal/druo/transaction-events` → core `POST /payments`. |
| **Autor** | María Fernanda Herazo |
| **Fecha** | 28/09/2026 |
| **Versión** | V.1.10 |
| **Comentarios** | **Historia separada de la HU-009 original (v1.5)**, que combinaba prepago y débito automático en un solo paso; aunque ambos son pagos y ambos usan Druo, son procesos distintos que pertenecen a flujos diferentes. **Corrección v1.8 (Check-in de Producto, 20 ago 2026):** se elimina la referencia a "fecha de corte"; el crédito vence 30 días después del desembolso. **Corrección v1.9 (2026-08-27):** CU-011 (pago voluntario PSE) separado de CU-026 (débito automático). **Actualización v1.10 (2026-09-28):** el prepago voluntario por PSE vía link de checkout Druo quedó implementado en `credits-platform` (rama `feat/webhooks-payment-reception`). La conciliación usa `primary_reference` / `secondary_reference` del evento (el create de Checkout no acepta `metadata`). El cargo implementado es saldo total del día, no una cuota suelta — validar con producto si el copy de "cuota" de esta HU debe alinearse. |
