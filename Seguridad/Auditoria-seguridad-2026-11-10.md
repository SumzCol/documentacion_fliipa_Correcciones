- **Checkout, a medias.** Completar el checkout exige `accepted: true` y no deja borrar una aceptación ya guardada. El texto aceptado no se compara con el legal oficial.
## Hallazgos ordenados
Estado de todos: **abierto** al 20 nov 2026.
### Críticos
#### C1. Ninguna ruta de b2b comprueba identidad
- **Dónde.** `backends/b2b/src/routes.ts`. Firebase Hosting publica `api.fliipa.com` y reescribe `/api/**` a b2b.
- **Qué pasa.** El login firma un JWT de 15 minutos con documento, tipo y `clientId`. Ahí termina el control. `validateSession` verifica firma y expiración, y no está montado en ninguna ruta. Además lee la cookie `fliipa_auth_token`; Hosting solo reenvía `__session`. `getClientIdFromSession` existe y nadie lo llama.
- **Impacto.** La ficha del cliente (contacto, Experian, KYC, fotos, documentos bancarios, débitos, correo) se lee y se escribe con un identificador, sin titular.
- **Cierre.** Bearer con `jwt.verify` en rutas de cliente. El `clientId` del token tiene que ser el `:id`. Lista corta y explícita de rutas públicas.
#### C2. Borrado de producto y alta de prueba sin secreto
- **Dónde.** `backends/b2b/src/controllers/clients/users-management-delete.ts`. Rutas `POST /internal/users-management/delete/preview`, `.../execute` y `.../test-users/create`.
- **Qué pasa.** Preview, ejecución y alta de cliente de prueba no leen `X-Internal-Secret`. La ejecución acepta `hardDelete`.
- **Impacto.** Borrado de datos de producto y creación de clientes de prueba desde el mismo host público que el resto de b2b.
- **Cierre.** Secreto en todo `/internal`. `hardDelete` solo desde admin-back, con auditoría.
#### C3. PIN débil y cambio sin sesión
- **Dónde.** `backends/b2b/src/utils/hash.ts`, `backends/b2b/src/controllers/clients/update-client-password.ts`. También lo usa el login y la recuperación de PIN.
- **Qué pasa.** `encodeHash` no es un KDF: reduce el PIN a un entero de 32 bits con una sal única escrita en el repo. `PUT /clients/:id/password` guarda el valor nuevo en Base64 y no pide el PIN actual ni una sesión.
- **Impacto.** El secreto de acceso del cliente no resiste y se puede reemplazar sin ser el titular.
- **Cierre.** argon2 o bcrypt, sal por usuario. El cambio pide el PIN actual o el flujo de recuperación. Fuera el Base64.
### Altos
#### A1. La sesión de redemption no está firmada
- **Dónde.** `apps/redemption/lib/session-envelope.ts`, `apps/redemption/lib/session.ts`.
- **Qué pasa.** `__session` es JSON en Base64. `getSession` lee el usuario del sobre. Decodifica el JWT para copiar datos de impersonalización y no llama a `jwt.verify`. Si el token viene roto, el usuario del sobre sigue valiendo. Las actions de crédito, cupón y firma usan ese `clientId`.
- **Impacto.** Quien arme la cookie elige el cliente con el que redemption habla con b2b. b2b, a su vez, no vuelve a mirar el token (C1).
- **Cierre.** Firmar `__session` o verificar el JWT en cada `getSession`.
#### A2. GET y PATCH de cliente no atan la petición al titular
- **Dónde.** `backends/b2b/src/controllers/clients/update-client.ts` y el GET de cliente. El payload acepta `termsAndConditions` si trae `accepted` y un `copy` de texto.
- **Qué pasa.** No se compara el llamador con el `:id`. El login firma el JWT aunque TyC sea falso.
- **Impacto.** Sirve para leer la ficha y para dejar asentada una aceptación de términos que el titular no dio.
- **Cierre.** Misma atadura de C1. El registro de TyC solo con el titular autenticado.
#### A3. Import de compras y reportes internos sin secreto
- **Dónde.** `backends/b2b/src/controllers/internal/coupon-purchases.ts`. Reportes `GET /internal/reports/funnel-traffic` y `GET /internal/reports/milestones`.
- **Qué pasa.** Alta, lote, cierre e import manual de compras de cupón, y los dos reportes, no comprueban `X-Internal-Secret`. Otras rutas internas (Druo, product settings, campañas, KYC interno) sí lo hacen.
- **Impacto.** Escritura de compras y lectura de embudo e hitos desde internet.
- **Cierre.** El mismo secreto de C2, en estas rutas.
#### A4. La misma API abierta alcanza operación de staff y de dinero
No es otro fallo distinto de C1. Es el alcance real, más ancho que la ficha.
Sin JWT ni secreto también quedan estas superficies:
- Cupones: alta, edición, borrado, import y export.
- Listado y export de clientes.
- Pago manual, conexión y débito Druo por `clientId`.
- Reevaluación KYC, refresh de Experian, URLs de documentos bancarios y fotos de checkout.
- Lista negra, comité y notas de cobranza.
- OTP del cliente y desbloqueo de login.
- Correo de decisión de crédito y correo con plantilla.
- Restore de demo.
- `POST /client-version/reload-exhausted`: alerta de Slack con mención y el contexto que mande quien llama.
**Cierre.** Entra en C1 y C2. Rutas de staff solo con secreto. Rutas del titular solo con el JWT de ese cliente.
### Medios
#### M1. TyC de redemption solo redirige, y el checkout no compara el texto
- **Dónde.** `apps/redemption/proxy.ts`, `backends/b2b/src/utils/validate-checkout-completion.ts`, `backends/b2b/src/types/terms-and-conditions.ts`.
- **Qué pasa.** El proxy manda a términos si la cookie dice que no se aceptaron. Ninguna server action llama a `getSessionTermsAccepted`. El checkout exige `accepted: true` y que `copy` sea texto. No guarda ni compara versión o hash del legal publicado.
- **Impacto.** Se puede avanzar el portal sin un registro de términos atado a la action, y se puede completar checkout con un texto que no es el oficial.
- **Cierre.** TyC en cada action, no solo en el proxy. Guardar versión o hash del texto publicado y rechazar cualquier otro al aceptar y al completar.
#### M2. CORS refleja el origen y el JSON admite 50 MB
- **Dónde.** `backends/b2b/src/index.ts`.
- **Qué pasa.** `origin: true` con credenciales. El parser JSON acepta cuerpos de 50 MB. No hay rate limit general en login, OTP ni escrituras. El tope de teléfono en checkout es un caso aparte.
- **Impacto.** Cualquier origen puede llamar la API con credenciales del browser. Un cuerpo grande presiona memoria. Login, OTP y escrituras no tienen freno de volumen.
- **Cierre.** Lista de orígenes. Límite de JSON acorde a cada ruta. Rate limit en login, OTP y escrituras.
#### M3. Webhook de SendGrid sin firma, con log completo
- **Dónde.** `services/webhooks/src/controllers/webhooks/sendgrid-events.webhook.ts`. `POST /api/v1/webhooks/sendgrid`.
- **Qué pasa.** No hay verificación de firma. El handler escribe en log método, URL, query, headers y body, y responde recibido. Hoy no aplica una acción de negocio.
- **Impacto.** Cualquier POST entra al log. Un evento real de SendGrid deja datos de correo en Cloud Logging.
- **Cierre.** Verificar la firma de SendGrid antes de loguear o procesar. No persistir el request crudo.
#### M4. Webhook de Zenvia sin firma
- **Dónde.** `services/webhooks/src/controllers/webhooks/zenvia-events.webhook.ts`. `POST /api/v1/webhooks/zenvia`.
- **Qué pasa.** No hay verificación de firma. Un evento de estado de mensaje se procesa. Druo, en el mismo servicio, sí verifica.
- **Impacto.** Se puede marcar el estado de un mensaje sin ser Zenvia.
- **Cierre.** Misma barra que Druo: firma o rechazo.
#### M5. Valores fijos si faltan secretos de b2b
- **Dónde.** `backends/b2b/src/config/index.ts`.
- **Qué pasa.** Si no vienen por entorno, `JWT_SECRET` e `IMPERSONATE_INTERNAL_SECRET` caen a un valor fijo del código. El arranque en producción solo se niega a seguir cuando falta `INTERNAL_S2S_SECRET`. Ese chequeo no cubre estos dos.
- **Impacto.** Si el entorno de STG o PROD no tiene la variable, las rutas que sí piden secreto aceptan el valor del repo, y los JWT se firman con la clave del repo. No se confirmó Secret Manager.
- **Cierre.** Confirmar las dos variables en STG y PROD. Quitar el fallback. Negarse a arrancar en producción si faltan, igual que el secreto S2S.
### Bajos
#### B1. Slack lleva documento y texto de TyC
- **Dónde.** `backends/b2b/src/services/login.ts` (aviso de login sin TyC) y `backends/b2b/src/services/notify-checkout-funnel-alerts.ts`.
- **Qué pasa.** El aviso de login sin TyC manda documento y el objeto de términos. Las alertas de checkout incluyen documento. El enriquecimiento de legado B2C suma más contexto de documento.
- **Impacto.** Datos del cliente en un canal de operaciones, más allá de lo que hace falta para actuar.
- **Cierre.** Slack sin documento ni copy. Identificador interno y enlace al admin.
## Orden de cierre
No escribir código hasta aprobar el corte.
| Orden | Corte | Qué cierra | Hallazgos |
|---|---|---|---|
| 1 | Primero | Bearer con `jwt.verify` en rutas de cliente. `clientId` del token igual al `:id`. Lista corta de rutas públicas. | C1, A2, A4 |
| 2 | Primero | Todo `/internal` y lo de staff exige `X-Internal-Secret`. `hardDelete` solo desde admin-back, con auditoría. Fuera el valor fijo. | C2, A3, M5 |
| 3 | Primero | argon2 o bcrypt, sal por usuario. El cambio pide el PIN actual o recuperación. Fuera el Base64. | C3 |
| 3b | Primero | Firma en SendGrid y Zenvia. Dejar de loguear el request crudo. | M3, M4 |
| 4 | Siguiente | Firmar `__session` o verificar el JWT en cada `getSession`. TyC en cada action. | A1, M1 |
| 5 | Siguiente | Versión o hash del texto legal publicado. Rechazar cualquier otro al aceptar y al completar. | M1 |
| 6 | Siguiente | CORS con lista de orígenes. JSON más chico. Rate limit en login, OTP y escrituras. Slack sin documento ni copy. | M2, B1 |
## Decisión que pide el informe
Aprobar el primer corte: 1, 2, 3 y la firma de SendGrid y Zenvia, y quitar los fallbacks de secreto. Dejar 4, 5 y el resto del borde para el siguiente corte.
## Qué no dice este informe
- No afirma que STG o PROD tengan, o no tengan, `JWT_SECRET` e `IMPERSONATE_INTERNAL_SECRET` en Secret Manager.
- No mide si alguien ya usó estas rutas.
- No propone un cambio de código. El corte de arriba es la decisión pendiente.