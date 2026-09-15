# 3. Firma de contrato y activación

*Versión corregida — incorpora los ajustes acordados en el check-in de Producto del 23 de julio de 2026 (ver "Fuentes consultadas").*

## Objetivo

Formalizar la aceptación del crédito aprobado mediante la firma electrónica del contrato, validar la identidad del cliente durante el proceso de firma, asignar el bono D1 y obtener la aprobación final de Core de Crédito/Originación* que activa el crédito y da inicio al funcionamiento de la calculadora.

> **Referencia comercial:** según el *Modelo Comercial B2B* (portal del cliente), la firma de contrato debe completarse de forma **digital, desde el celular, en minutos** — este journey detalla técnicamente cómo se logra esa experiencia.

---

## Journey

![Journey Colpatria B2B — página 5](imagenes/page-05.png)

![Journey Colpatria B2B — página 6](imagenes/page-06.png)

**Figura 5. Journey de Firma de Contrato y Activación del Crédito.**

En el journey, las cajas moradas ("Lee el contrato", "Se genera contrato firmado y se envía copia al cliente vía email" y "Se aprueba el crédito") corresponden a pasos ajustados en junio de 2026 (leyenda "Ajuste · jun 2026"); las demás cajas corresponden a pasos ya existentes del flujo.

> **⚠️ Pendiente de sincronizar:** las imágenes del journey (páginas 5 y 6) aún muestran el texto original de las cajas (incluyendo la mención a "pagaré" y los dos pasos separados de identificación/PIN). Este documento ya refleja los acuerdos de la reunión del 23 de julio, pero **las figuras deben actualizarse por separado** en la herramienta de diagramación para no quedar desincronizadas con el texto.

*Los elementos marcados con asterisco (\*) corresponden a puntos aún no definidos técnicamente o pendientes de confirmación con el dueño del proceso; se detallan en cada paso y se listan de forma consolidada en "Pendientes de validación".*

---

## Descripción general

Una vez el crédito ha sido aprobado durante la evaluación de riesgo, el Core Bancario habilita la operación para iniciar el proceso de firma electrónica. El cliente recibe una invitación para continuar con la firma, autentica su identidad mediante su Documento de Identificación y el PIN de seguridad creado durante el onboarding, y accede a una pantalla de bienvenida con la información general del crédito aprobado, incluyendo el cupo aprobado, el plan de pagos, la fecha de pago, la tasa de interés y la disponibilidad del cupo para su uso en tiendas D1. Después de aceptar las condiciones, el cliente visualiza el detalle completo del crédito (cupo, plan de pagos, valor, fecha, tasa e inicio de uso), revisa el contrato y confirma la firma mediante un código de verificación enviado a su correo electrónico.

Cuando la firma se completa correctamente, el sistema genera el contrato firmado y envía una copia al correo electrónico del cliente. Posteriormente, se genera la compra del bono con D1, D1 emite el bono y lo entrega al equipo de operaciones, quien lo asigna al cliente. Una vez completada la asignación, el sistema notifica al cliente que ya puede visualizar el código del bono desde su cuenta. Finalmente, un actor distinto al Core Bancario —Core de Crédito/Originación— aprueba el crédito, lo que da inicio al funcionamiento de la calculadora y permite continuar con la recepción y uso del bono.

Como parte del estándar de trazabilidad definido para el ciclo del crédito, el sistema notifica por Slack al equipo de operaciones los eventos relevantes de este proceso: firma exitosa, fallas reiteradas en la verificación y cualquier caso en que el crédito firmado no sea finalmente aprobado por el Core de Crédito/Originación.

---

## Explicación paso a paso

Cada paso incluye la descripción del proceso (qué ocurre a nivel técnico u operativo) y un tiempo estimado de referencia. Estas duraciones son estimaciones de planificación para UX, alineadas con el objetivo comercial de completar la firma del contrato "en minutos, desde el celular" (Modelo Comercial B2B). Los tiempos son referenciales y están sujetos a validación por parte de los equipos de Producto y Tecnología.

### 1. Habilitación del crédito

**Actor:** Core Bancario.

**Proceso:** Tras la aprobación del crédito en el proceso de KYC/riesgo (documento 3), el Core Bancario habilita la operación correspondiente para que el caso pueda continuar hacia la etapa de formalización. A partir de este momento el sistema puede solicitar al cliente la firma del contrato.

**Resultado:** Crédito es habilitado para iniciar la firma.

**Tiempo estimado:** Instantáneo (disparado automáticamente al recibir la aprobación de KYC/riesgo).

---

### 2. Solicitud de continuar con la firma del contrato

**Actor:** Web (sistema).

**Sistemas involucrados:** Sendgrid (correo) y Zenvia (WhatsApp).

**Proceso:** Al aprobarse el crédito, el sistema notifica al cliente por correo electrónico (plantilla Sendgrid `welcome`) y por WhatsApp (plantilla Zenvia `creditApproved`), informando que el crédito fue aprobado y que puede continuar con la firma electrónica. SMS y llamada quedan fuera del MVP.

**Resultado:** Cliente notificado por correo y WhatsApp para continuar el proceso de firma.

**Tiempo estimado:** Instantáneo del lado del sistema; la recepción depende del canal y es asíncrona.

> **Nota (Ajuste · sep 2026):** el contacto de aprobación dejó de ser un placeholder de canal. En código: correo `welcome` + WhatsApp `creditApproved` (catálogos de `communications`). SMS/llamada no aplican en el MVP.

---

### 3. Recepción del correo con el enlace

**Actor:** Cliente.

**Proceso:** El cliente recibe el correo enviado en el paso anterior y hace clic en el enlace, lo que lo redirige nuevamente a la plataforma web para retomar el proceso en el punto de la firma.

**Resultado:** El cliente ingresa nuevamente al proceso de firma.

**Tiempo estimado:** Asíncrono (depende de cuándo el cliente revisa su correo).

---

### 4. Autenticación del cliente

**Actor:** Cliente.

**Información utilizada:** Documento de identidad del cliente y PIN de seguridad de cuatro dígitos (creado durante el onboarding, paso 12 del documento 2).

**Proceso:** El cliente ingresa, en un mismo paso, su documento de identidad y su PIN de seguridad. El sistema valida ambos datos contra el registro creado durante el onboarding para confirmar que quien realiza la firma corresponde al titular de la solicitud. Si alguno de los dos datos es incorrecto, el sistema muestra un mensaje de error **genérico** (por ejemplo, "Documento o contraseña incorrectos"), sin indicar cuál de los dos campos falló, para evitar revelar información que pueda comprometer la seguridad de la cuenta.

**Resultado:** Identidad del cliente validada.

**Tiempo estimado:** ~15-20 segundos.

> **Excepción prevista en el journey:** si el cliente olvidó su PIN, el sistema lo redirige al canal de soporte; una vez recuperado el acceso, el cliente vuelve a recibir el correo con el enlace para continuar el proceso.

**Placeholder\*:** actualmente no existe un proceso automático de recuperación de PIN: el cliente debe contactar al canal de soporte, y es el equipo de soporte quien asigna uno nuevo. Falta definir el detalle operativo exacto de ese procedimiento (¿validación de identidad adicional?, ¿tiempo de resolución?).

> *Nota: este paso reemplaza y fusiona lo que en la versión anterior del documento eran dos pasos separados (identificación y PIN), según lo acordado en el check-in de Producto del 23 de julio de 2026.*

---

### 5. Condiciones de la aprobación*

**Actor:** Web (sistema).

**Proceso:** El sistema da la bienvenida al cliente en su cuenta y muestra en pantalla las condiciones del crédito aprobado:

1. Cupo aprobado.
2. Plan de pagos (cuota, valor de la cuota y día de pago).
3. Fecha de pago de la cuota.
4. Tasa de interés (E.A.), aclarando que la cuota ya incluye intereses y cargos administrativos.
5. Disponibilidad inmediata del cupo en la cuenta del cliente, utilizable en cualquier tienda D1.

**Resultado:** Cliente informado de las condiciones del crédito aprobado, incluyendo tasa de interés y detalle de la cuota mensuales.

**Tiempo estimado:** ~20-30 segundos de lectura.*

**Placeholder\*:** el cupo aprobado depende de los cálculos internos de la entidad; falta confirmar si existe algún mensaje adicional sobre congelación del cupo en caso de mora, y en qué casos aplicaría.

> **⚠️ Punto a validar:** la pantalla real de esta comunicación ya incluye tasa de interés, plan de pagos y fecha de pago (ver diseño de referencia). Esto se superpone con el contenido que el paso 7 ("Comunicación detallada") describe como una segunda pantalla posterior a la aceptación. Se recomienda confirmar con el equipo si el paso 7 sigue siendo necesario como pantalla independiente o si debe fusionarse con este paso.

---

### 6. Aceptación de condiciones

**Actor:** Cliente.

**Proceso:** El cliente revisa y acepta las condiciones del crédito comunicadas en el paso anterior. Esta aceptación es obligatoria antes de recibir el detalle completo del crédito; si no acepta, el flujo no permite continuar.

**Resultado:** Condiciones generales aceptadas.

**Tiempo estimado:** ~5-10 segundos.

**Regla de excepción (nueva):** si el cliente llega al momento de la firma sin haber aceptado previamente las condiciones, el sistema **no debe rechazar automáticamente** el flujo; debe permitirle aceptar en ese momento y generar una alerta en Slack (con enlace al caso en el panel de administración) indicando una falla en el flujo, para que el equipo de operaciones haga seguimiento.

---

### 7. Comunicación detallada de las condiciones del crédito

**Actor:** Web (sistema).

**Proceso:** El sistema comunica al cliente el detalle operativo completo del crédito:

- Cupo aprobado.
- Plan de pago.
- Valor a pagar.
- Fecha de pago.
- Tasa de interés.
- Cuándo podrá usarlo.

**Resultado:** Cliente informado del detalle completo del crédito.

**Tiempo estimado:** ~20-30 segundos de lectura.

*(Ver nota de validación en el paso 5 sobre la posible redundancia entre ambas comunicaciones.)*

---

### 8. Comunicación de siguientes pasos

**Actor:** Web (sistema, vía app).

**Proceso:** La aplicación web comunica al cliente los siguientes pasos del proceso: la firma del contrato.

**Resultado:** Cliente informado de que debe continuar hacia la revisión y firma del documento legal.

**Tiempo estimado:** Instantáneo.

---

### 9. Revisión del contrato

**Actor:** Cliente.

**Proceso:** El cliente lee el contrato desplegado en la plataforma antes de continuar hacia la verificación de la firma electrónica.

**Resultado:** Contrato revisado por el cliente.

**Tiempo estimado:** Variable según la disposición del cliente a leer el documento completo; no contabilizado dentro de un tiempo fijo por tratarse de un paso de lectura libre.

> **Nota (Ajuste · jun 2026):** este paso está marcado en el journey como un paso ajustado en junio de 2026.

---

### 10. Envío del código de verificación

**Actor:** Web (sistema).

**Información utilizada:** Correo electrónico registrado por el cliente.

**Proceso:** El sistema envía un código de verificación al correo electrónico del cliente para completar la firma electrónica. El flujo continúa en la página siguiente del journey (Figura 6).

**Resultado:** Código de verificación enviado.

**Tiempo estimado:** Instantáneo del lado del sistema; recepción asíncrona según el proveedor de correo.

---

### 11. Cliente firma el contrato

**Actor:** Cliente.

**Proceso:** El cliente ingresa el código recibido; el sistema valida su vigencia y coincidencia. Tiene la opción de reenviar el código si no lo recibe.

**Decisión:** ¿El código ingresado es válido?

- **Exitoso:** el proceso continúa hacia la generación del documento firmado.
- **Fallido:** el cliente es dirigido a contactar el servicio al cliente, y el sistema envía una alerta al canal de Slack de operaciones con enlace directo al caso en el panel de administración para dar trazabilidad al caso.

**Resultado:** Titularidad del correo confirmada para efectos de la firma.

**Tiempo estimado:** ~30-45 segundos (depende de la entrega del correo).

**Placeholder\*:** no está definido el número máximo de reenvíos del código ni el tiempo de vigencia antes de su expiración, ni si toda falla debe alertar por Slack o solo a partir de cierto número de intentos fallidos.

---

### 12. Gestión de código fallido

**Actor:** Web (sistema) / Cliente.

**Proceso:** En caso de que el código sea incorrecto o haya expirado, el cliente debe contactarse con el servicio de atención al cliente. El asesor evalúa el caso específico y decide la solución más adecuada (que puede ser el reenvío del código u otra gestión, según la causa del error) no se asume un reenvío automático en todos los casos.

**Resultado:** Cliente atendido por el canal de soporte para poder continuar con la firma.

**Tiempo estimado:** Depende del tiempo de atención del servicio al cliente (asíncrono, fuera del control directo de este journey).

---

### 13. Generación del documento firmado

**Actor:** Web (sistema).

**Proceso:** Una vez validado el código, el sistema genera automáticamente el contrato firmado electrónicamente, aplicando el mecanismo de firma/no repudio definido para el producto*, y notifica al cliente por dos canales: (1) correo con la plantilla Sendgrid *B2B - Firma de contrato* (`contract`), personalizado con el nombre del cliente y con el PDF firmado adjunto; (2) WhatsApp con la plantilla Zenvia `contractSigned` (sin adjunto; el PDF viaja solo por correo). Ambos envíos son fail-open respecto de la firma.

**Resultado:** Contrato firmado electrónicamente, con copia PDF por correo y aviso por WhatsApp.

**Tiempo estimado:** ~1 minuto (la generación del documento no es instantánea).

> **Nota (Ajuste · jun 2026):** este paso está marcado en el journey como un paso ajustado en junio de 2026.

> **Nota (Ajuste · sep 2026):** el correo post-firma usa la plantilla Sendgrid *B2B - Firma de contrato* (`contract`) con PDF adjunto. En paralelo se envía WhatsApp `contractSigned` (catálogo `whatsapp-templates.ts`).

**Placeholder\*:** el mecanismo de no repudio debe reformularse en términos de la **evidencia de la firma** que respalda la validez legal de la operación (por ejemplo, el registro del correo de verificación enviado, el código ingresado por el cliente y el registro de tiempos/timestamps de cada paso de la autenticación y la firma), en lugar de dejarlo como una referencia abierta a validar contra otros actores del mercado. Queda pendiente que el equipo legal/de producto confirme si esta evidencia es suficiente o si se requiere un mecanismo adicional.

---

### 14. Confirmación de firma exitosa

**Actor:** Cliente.

**Proceso:** El cliente recibe en pantalla la confirmación de que la firma del contrato fue exitosa. El sistema envía adicionalmente una alerta al canal de Slack de operaciones confirmando la firma exitosa con enlace directo al caso en el panel de administración, en línea con el estándar de trazabilidad del ciclo del crédito.

**Resultado:** Firma confirmada.

**Tiempo estimado:** Instantáneo.

---

### 15. Compra y emisión del bono D1

**Actor:** Colpatria – D1 – Sumz.

**Proceso:** Colpatria genera el giro a D1 para realizar la compra del bono. Sumz coordina con el equipo de operaciones de D1 la generación del bono. Finalmente, D1 genera el bono y se lo entrega a Sumz.

- El bono es único e intransferible.

**Resultado:** Compra y emisión del bono por parte de D1.

**Tiempo estimado (D1 emitiendo el bono):** Pendiente de definir.

**Tiempo estimado (Colpatria girando los fondos):** Pendiente de definir.

*(Se mantienen ambos tiempos separados porque corresponden a procesos distintos: el giro de Colpatria y la emisión de D1.)*

---

### 16. Asignación del bono D1

**Actor:** Web (sistema) + Operario Sumz.

**Proceso:** El operario de Sumz recibe el código de parte de D1 y lo ingresa manualmente en el panel de administración del cliente específico. Luego el sistema envía de forma automática el correo notificando al cliente.

**Resultado:** Bono D1 asignado.

**Tiempo estimado:** Menos de 5 minutos, una vez el código está del lado de Sumz.

**Regla adicional:** si el cliente ya tiene un código asignado, el sistema no debe permitir el ingreso de otro código que lo reemplace; casos especiales solo podrían gestionarse mediante ajustes directos en base de datos.

**Placeholder\*:** queda pendiente definir formalmente qué persona o rol del equipo de operaciones de Sumz será responsable de este ingreso manual.

---

### 17. Notificación del bono asignado

**Actor:** Web (sistema).

**Proceso:** El sistema notifica al cliente, a través de su cuenta, que el bono ya se encuentra asignado y disponible para consulta.

**Resultado:** Cliente notificado de la asignación del bono.

**Tiempo estimado:** Instantáneo.

---

### 18. Visualización del código del bono

**Actor:** Cliente.

**Proceso:** El cliente ingresa a su cuenta y visualiza el código correspondiente al bono, el cual podrá canjear en cualquier tienda D1 (consistente con el paso 6 del portal del cliente descrito en *Modelo Comercial B2B*: "Usa su cupo — código numérico para canjear en cualquier tienda D1").

**Resultado:** Código del bono visible y disponible para su uso.

**Tiempo estimado:** ~10 segundos.

---

### 19. Aprobación del crédito por Core de Crédito/Originación

**Actor:** Core de Crédito/Originación*.

**Proceso:** Este actor —distinto del Core Bancario que habilitó el proceso en el paso 1— aprueba formalmente el crédito ya firmado. El inicio del crédito queda marcado por el **uso efectivo del bono** por parte del cliente; el sistema debe conectarse con las bases de datos de D1 y consultar dicho uso.

**Decisión:** ¿El bono se usó?

- **Sí:** inicia el crédito por el valor del uso.
- **No:** no inicia el crédito.

**Resultado:** Se notifica al cliente que el crédito acaba de iniciar, junto con el valor a pagar y la fecha próxima de pago.

**Tiempo estimado:** Instantáneo, si el proceso es automático (pendiente de confirmar).

**Periodicidad:** todos los días, en una hora inactiva fija, se consultan las bases de datos de D1 para identificar qué bonos han sido utilizados e iniciar los créditos correspondientes.

> **Nota (Ajuste · jun 2026):** este paso está marcado en el journey como un paso ajustado en junio de 2026.

**Placeholder\*:** no está definida con precisión la diferencia funcional y de sistema entre **Core Bancario** (paso 1) y **Core de Crédito/Originación** (este paso): qué reglas aplica cada uno, si esta aprobación es automática o requiere alguna validación adicional, y qué ocurriría —y qué pasos concretos seguiría el flujo— si Core de Crédito/Originación no aprobara un crédito que ya fue firmado por el cliente.

---

### 20. Cierre del journey

**Proceso:** Con la aprobación de Core de Crédito/Originación finaliza el journey de firma de contrato y activación.

**Resultado:** El flujo continúa hacia la recepción y uso del bono (documento 6, Dispersión de fondos) y hacia el proceso de Calculadora y Cobro del Crédito (documento 5).

**Tiempo estimado:** Instantáneo (transición automática entre procesos).

---

## Reglas de negocio

- El crédito debe haber sido aprobado previamente (KYC + riesgo) antes de iniciar la firma.
- El cliente debe autenticarse mediante documento de identidad y PIN, en un único paso, para acceder al proceso.
- Si la autenticación falla, el sistema debe mostrar un mensaje de error genérico (sin indicar si el dato incorrecto fue el documento o el PIN).
- Si el cliente olvida el PIN, deberá recuperar el acceso a través del canal de soporte antes de continuar; actualmente no existe recuperación automática.
- La comunicación de las condiciones del crédito ocurre en dos momentos: una primera comunicación general antes de la aceptación (ver paso 5) y una segunda comunicación con el detalle operativo después de la aceptación (ver paso 7) — pendiente de confirmar si ambas siguen siendo necesarias.
- Si el cliente llega al momento de la firma sin haber aceptado los términos y condiciones, el sistema debe permitirle aceptarlos en ese momento (no rechazar automáticamente) y generar una alerta en Slack para el equipo de operaciones.
- La aceptación de las condiciones del crédito es obligatoria antes de acceder al detalle completo y al documento legal.
- El contrato debe visualizarse antes de la firma electrónica.
- La firma solo se completa cuando el código de verificación es validado correctamente; si falla, el cliente puede reintentar o contactar al servicio al cliente, quien evalúa el caso puntual antes de decidir la solución.
- Una vez finalizada la firma, el sistema genera automáticamente el documento legal firmado, envía copia PDF al correo del cliente y un aviso por WhatsApp (`contractSigned`).
- El mecanismo de firma electrónica debe garantizar la evidencia de la firma (registro del código de verificación, del correo y de los tiempos de cada paso) como respaldo de su validez legal.
- El bono D1 únicamente se asigna cuando la firma ha sido completada exitosamente.
- El bono D1 es único e intransferible; el sistema no debe permitir reemplazarlo una vez asignado a un cliente.
- Dependemos de actores externos (Colpatria, D1) para la emisión del bono.
- El inicio del crédito queda marcado por el uso efectivo del bono, validado contra las bases de datos de D1.
- Según el modelo comercial del producto, todo el proceso de firma debe poder completarse de forma digital, desde el celular, en minutos.
- En línea con el estándar de trazabilidad acordado para el ciclo del crédito, toda falla de verificación, la firma exitosa, y cualquier caso de no aprobación por parte de Core de Crédito/Originación se notifican en tiempo real por el canal de Slack de operaciones, incluyendo un enlace directo al caso en el panel de administración.

---

## Entradas

- Crédito aprobado.
- Datos de autenticación del cliente (documento de identidad y PIN).
- Condiciones del crédito.
- Contrato.
- Correo electrónico registrado.
- Código de verificación.

---

## Salidas

- Contrato firmado electrónicamente.
- Confirmación de firma exitosa.
- Bono D1 asignado.
- Crédito aprobado por Core de Crédito/Originación.
- Inicio del funcionamiento de la calculadora.
- Cupo disponible para utilización.
- Alertas registradas en Slack, con enlace al caso en el panel de administración (fallas de verificación, aceptación tardía de términos, firma exitosa, no aprobación de Core de Crédito/Originación).

---

## Excepciones

- El cliente no accede al enlace enviado por correo.
- El cliente olvidó el PIN.
- El código de verificación es incorrecto o expira, y el cliente debe contactar al servicio al cliente o reintentar (con alerta Slack asociada).
- El cliente llega al momento de la firma sin haber aceptado previamente las condiciones (se le permite aceptar en ese momento y se genera alerta Slack).
- El cliente no acepta las condiciones del crédito.
- Se presenta un error durante la generación del contrato.
- No es posible asignar el bono D1.
- Se produce un error durante la aprobación del crédito por parte de Core de Crédito/Originación.
- Core de Crédito/Originación no aprueba un crédito ya firmado por el cliente (caso alertado por Slack; flujo de resolución pendiente de definir).

---

## Consideraciones

- La autenticación mediante documento de identidad y PIN busca garantizar que únicamente el titular pueda acceder al proceso de firma; los mensajes de error genéricos protegen contra ataques que buscan confirmar la existencia de una cuenta o cuál dato es incorrecto.
- El journey distingue dos actores del Core distintos: Core Bancario, que habilita el proceso al inicio, y Core de Crédito/Originación*, que aprueba el crédito al final y da inicio a la calculadora. Esta distinción es uno de los principales placeholders del proceso.
- La firma electrónica requiere una validación adicional mediante código enviado al correo electrónico del cliente; la validez legal se sustenta en la evidencia generada durante ese proceso (registro del código, del correo y de los tiempos), no en un mecanismo externo adicional.
- El contrato se genera automáticamente una vez finaliza la firma, y se envía copia al correo del cliente mediante la plantilla Sendgrid *B2B - Firma de contrato*, con el PDF firmado adjunto.
- La asignación del bono D1 y la aprobación final del crédito ocurren únicamente después de completar exitosamente todo el proceso de firma.
- El objetivo comercial (firma digital, desde el celular, en minutos) debe validarse frente a los tiempos estimados detallados en este documento una vez el flujo esté implementado.
- Por consistencia con el proceso de KYC (documento 3), se incorporó la trazabilidad por Slack para los eventos críticos de este journey; sin embargo, a diferencia de KYC —que ya cuenta con un estado de "posible rechazo" y revisión manual definidos—, este documento aún no define un flujo concreto para el caso en que Core de Crédito/Originación no apruebe un crédito ya firmado. Se recomienda evaluar con el dueño del proceso si aplica un patrón similar (estado intermedio + revisión manual) para mantener consistencia entre ambos journeys.
- Se eliminó del documento toda referencia al pagaré: el proceso legal se maneja únicamente con el contrato.
- Se eliminaron las menciones al término "circuito cerrado" por ser ambiguas; queda pendiente confirmar con el dueño del proceso si existe algún contenido de "congelación del cupo por mora" que deba comunicarse en el paso 5, y con qué redacción.

---

## Pendientes de validación

> **Pendiente de validar con el dueño del proceso:**
>
> - Confirmar si las dos comunicaciones de condiciones del crédito (paso 5 y paso 7) deben mantenerse como pantallas independientes o si deben fusionarse, dado que la pantalla real ya muestra tasa, plan de pagos y fecha desde el primer momento. *(placeholder — pasos 5 y 7)*
> - ~~Confirmar la plataforma técnica exacta usada para el contacto inicial del paso 2~~ **Resuelto (sep 2026):** correo Sendgrid `welcome` + WhatsApp Zenvia `creditApproved`; SMS/llamada fuera del MVP.
> - Confirmar el procedimiento operativo exacto de recuperación de PIN vía canal de soporte. *(placeholder — paso 4)*
> - Confirmar si existe contenido de "congelación del cupo por mora" a comunicar en el paso 5, y su redacción definitiva (se eliminó "circuito cerrado" del documento). *(placeholder — paso 5)*
> - Confirmar el número máximo de reenvíos del código de verificación, el tiempo de vigencia del código antes de su expiración, y si toda falla de verificación debe alertar por Slack o solo a partir de cierto número de intentos. *(placeholder — paso 11)*
> - Validar con el equipo legal si la evidencia de firma descrita (código + correo + timestamps) es suficiente como mecanismo de no repudio, o si se requiere algo adicional. *(placeholder — paso 13)*
> - Definir el rol/persona responsable, del lado de Sumz, de ingresar manualmente el código del bono D1 en el panel de administración. *(placeholder — paso 16)*
> - Confirmar la diferencia funcional y de sistema entre Core Bancario y Core de Crédito/Originación, si la aprobación final (paso 19) es automática o requiere validación adicional, y **definir el flujo de resolución cuando Core de Crédito/Originación no apruebe un crédito ya firmado** (posible alineación con el patrón de "posible rechazo" usado en KYC). *(placeholder — paso 19)*

---

## Fuentes consultadas

- *Journeys Colpatria B2B* (junio de 2026), páginas 5 y 6.
- Documento funcional del proceso de originación.
- Documento de Alcance del Producto.
- *Modelo Comercial B2B.pptx* (Sumz, junio de 2026) — diapositiva 8, "El portal: simple, digital, en WhatsApp".
- Estándar de trazabilidad por Slack y manejo de excepciones (posible rechazo/revisión manual) acordado en la reunión "Producto: Check-in" (21 de julio de 2026), aplicado por consistencia a este documento.
- Transcripción y resumen de la reunión "Producto: Check-in" (23 de julio de 2026) — acuerdos sobre: eliminación de referencias al pagaré, unificación de los pasos de autenticación, mensajes de error genéricos, eliminación del término "circuito cerrado", ajuste del mecanismo de no repudio, inclusión de enlace al panel de administración en las alertas de Slack, tiempo estimado de generación del documento firmado, y flujo operativo de emisión y asignación del bono D1.
- Diseño de referencia de pantalla de condiciones del crédito (imagen aportada por el usuario), usado para validar el contenido real mostrado al cliente en el paso 5.
