### CU-011: Pagar cuota por PSE

![Diagrama de caso de uso CU-011](imagenes/diagrama_CU-011.svg)

| Campo | Detalle |
|---|---|
| **Actores** | Cliente empresarial |
| **Descripción** | El cliente paga el saldo adeudado de un desembolso de forma voluntaria mediante PSE, usando un link de checkout de Druo generado desde el portal de redención. |
| **Precondiciones** | El cliente tiene un desembolso activo con saldo adeudado y, cuando aplique, una cuenta Druo usable para el flujo de checkout. |
| **Flujo principal** | 1. El cliente solicita el link de pago desde el portal de redención (desembolso concreto).<br>2. b2b reserva una fila en `druo_payment_links`, crea el checkout en Druo (`primary_reference` = clientId, `secondary_reference` = disbursementId) y devuelve la URL.<br>3. El cliente completa el pago PSE en la página hospedada de Druo.<br>4. Druo envía `transaction.successful` a `services/webhooks`, que lo reenvía a b2b (`POST /internal/druo/transaction-events`).<br>5. b2b concilia el link por references, marca el uuid de la transacción (idempotencia) y registra el pago en core (`payment_references`, canal `druo_checkout`, estado `success`). |
| **Flujos alternativos / excepciones** | A1. El pago no se completa (`transaction.failed` / `transaction.canceled`): se cierra el link local para permitir un link nuevo; el crédito no se abona.<br>A2. Fallo al registrar en core tras el claim local: alerta operativa; una redelivery del webhook reintenta el registro mientras `core_payment_reference_id` esté vacío.<br>A3. Evento sin references utilizables o sin fila local: no se auto-abona (evita doble crédito); ops concilia a mano. |
| **Postcondiciones** | El abono queda en `payment_references` del desembolso y el saldo / plan de pagos lo refleja. |
| **Reglas de negocio** | El pago por PSE es voluntario e iniciado por el cliente, distinto del débito automático (ver [CU-026](CU-026 Gestionar debito automatico de creditos vencidos.md)). Ambos usan Druo. El monto del link es el **saldo total adeudado hoy**, no una cuota parcial. |
| **Historias de usuario relacionadas** | [HU-013](../../Historias De Usuario/4. Cobranza/HU-013 Prepagar la cuota por PSE.md) (Prepagar la cuota por PSE) |
| **Estado en plataforma** | Implementado en `credits-platform` (mint + webhook de conciliación + registro en core). Débito automático (HU-014 / CU-026) sigue fuera de este caso. |
| **Referencias** | Fuente: ficha [HU-013](../../Historias De Usuario/4. Cobranza/HU-013 Prepagar la cuota por PSE.md). Código: `backends/b2b` (payment link + `apply-druo-transaction-event`), `services/webhooks` (`druo.webhook`), `backends/core` (`POST /payments`). |

> **Nota de versión (2026-08-27):** Este caso de uso se separó del CU-011 original (v1.0 del catálogo), que combinaba el pago voluntario por PSE y el débito automático de créditos vencidos en una sola ficha. El débito automático ahora vive en [CU-026](CU-026 Gestionar debito automatico de creditos vencidos.md).

> **Nota de versión (2026-09-28):** Se actualiza el estado de "no implementado" a implementado para el pago voluntario vía checkout Druo / webhook `transaction.successful`. Queda pendiente de producto alinear el wording "cuota" vs "saldo total del día".
