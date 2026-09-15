# 6. Proveedores externos

[← Volver a Actores](README.md)

| Proveedor | Rol |
|---|---|
| Experian | Consulta score e historial crediticio y datos de contacto. |
| Druo | Pasarela de pago y débito automático. |
| Olimpia | Validación biométrica y verificación de identidad (KYC). |
| Zenvia | Gestión de notificaciones y comunicaciones con clientes vía WhatsApp y SMS. Las plantillas de WhatsApp (outbound IDs) se resuelven en el catálogo de `communications` (`whatsapp-templates.ts`), no como secreto de entorno. |
| Sendgrid | Envío de comunicaciones por email. Las plantillas dinámicas se resuelven en el catálogo de `communications` (`email-templates.ts`). |
| Colpatria | Fondeador del crédito vía fiducia (concentra el origen y el retorno del dinero) **y** responsable de definir las políticas y reglas de originación de crédito que Fliipa implementa en su motor de riesgo. |
| Nebula | Proveedor de llamadas *(sin fuente confirmada en el material revisado)*. |

## Fuentes consultadas

- Journeys Colpatria B2B, junio 2026 — *Journeys Fran finales-1.pdf*
- Integraciones técnicas (Olimpia, Zenvia) — `tecnico/Integraciones/Olimpia.md`, `tecnico/Integraciones/Zenvia.md`
- Alcance del Producto (`producto/alcance.md`)
 - Notas de la reunión "Producto: Weekly Sync" (10 jul 2026) y su transcripción asociada — rol de Colpatria en el motor de riesgo.
