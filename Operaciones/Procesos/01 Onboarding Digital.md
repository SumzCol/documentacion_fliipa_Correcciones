# 1. Onboarding digital

## Objetivo

Registrar al cliente empresarial mediante un proceso completamente digital, identificar su cupo preaprobado utilizando la información transaccional disponible, recopilar la información básica del negocio y del representante legal, validar el correo electrónico y el PIN de seguridad, realizar la validación biométrica, registrar la cuenta bancaria para el débito automático y recopilar la documentación necesaria para dejar la solicitud preparada para el análisis crediticio y continuar con el proceso de Validación de Identidad (KYC).

> **Actualización (Check-in 16 jul 2026):** el estudio de crédito inicial queda completamente automatizado a través del motor de riesgo; no interviene un asesor humano en esta etapa. El flujo final del onboarding queda enfocado en dejar la solicitud lista para KYC + motor de riesgo.
>
**Actualización (sep 2026):** la firma del contrato **sí usa OTP** (ver proceso 03). En onboarding se mantienen además los códigos de verificación por WhatsApp y correo electrónico. La validación específica del número telefónico dentro del onboarding queda pendiente de confirmar con el dueño del proceso.

---

## Journey

![Journey Colpatria B2B — página 2](imagenes/page-02.png)

**Figura 2. Journey de Onboarding Digital (Parte 1).**

![Journey Colpatria B2B — página 3](imagenes/page-03.png)

**Figura 3. Journey de Onboarding Digital (Parte 2).**

En el journey, las cajas en color morado corresponden a pasos nuevos o ajustados en junio de 2026 (leyenda "Ajuste · jun 2026"); las cajas en gris corresponden a pasos ya existentes del flujo.

---

## Descripción general


El onboarding digital constituye el primer contacto del cliente con el producto y concentra todas las actividades necesarias para crear su perfil dentro de la plataforma.

El proceso inicia cuando el cliente recibe una invitación personalizada por correo electrónico(Sendgrid), SMS o WhatsApp ( Zenvia) para acceder al producto. A partir de este momento registra la información básica del negocio y suministra los datos del representante legal. Posteriormente ingresa un código de verificación enviado a su correo electrónico, configura un PIN de seguridad, realiza la validación biométrica con el proveedor externo *Olimpia*, registra la cuenta bancaria desde la cual se autorizarán los débitos automáticos (validada ante Drúo), adjunta la documentación bancaria requerida y selecciona su localidad habitual de compra. Finalmente, toda la información recopilada es enviada al proceso de Validación de Identidad (KYC).

> **Nota de estado:** tras la creación de la cuenta, el cliente queda en el estado **"pendiente de biometría"**, visible para seguimiento interno del funnel.

---

## Explicación paso a paso



### 1. Recepción de la invitación

**Actor:** Cliente.

**Sistemas involucrados:** Sendgrid (correo electrónico) y Zenvia (SMS/WhatsApp).

**Información utilizada:** Correo electrónico y/o número de teléfono del cliente.

**Proceso:** Al activar la campaña de invitación, la plataforma dispara automáticamente el envío a través de Sendgrid (correo) y/o Zenvia (SMS/WhatsApp), incluyendo un enlace a la landing page de onboarding. El envío es orquestado por el backend según el canal de contacto disponible para cada cliente.

**Resultado:** El cliente recibe un enlace a la landing page de onboarding por correo, SMS o WhatsApp para iniciar el proceso de forma segura.

**Tiempo estimado:** Instantáneo del lado del sistema; la recepción depende del proveedor de mensajería (asíncrono, fuera del control directo de la plataforma).

> **Nota técnica (Ajuste · jun 2026):** integración de envío con Sendgrid y Zenvia.

---

### 2. Registro del documento de identificación

**Actor:** Cliente.

**Sistemas involucrados:** Portal De Usuario.

**Información utilizada:** Documento de Identificación.

**Proceso:** El cliente ingresa su documento en la landing page. El backend consulta contra las bases de datos existentes para identificar si el cliente ya tiene información previa (persona natural o jurídica) y determinar el tipo de flujo a seguir.

**Resultado:** El sistema identifica el tipo de cliente y recupera la información previamente disponible de las bases de datos.

**Tiempo estimado:** ~15 segundos.

> **Nota técnica (Ajuste · jun 2026):** el journey señala la necesidad de cambiar la URL de la página; validar con Tecnología si ya fue implementado.

---

### 3. Identificación del cupo preaprobado

**Actor:** Web (sistema).

**Sistemas involucrados:** Información transaccional de D1.

**Información utilizada:** Documento de Identidad registrado; cupo preaprobado por Colpatria.

**Proceso:** El cupo preaprobado **no se calcula en este momento**: corresponde a un valor ya calculado previamente a partir del historial de consumos en D1 y almacenado en base de datos. En este paso el sistema únicamente consulta y recupera ese valor.

**Decisión:** ¿La validación del cupo fue exitosa?

- **Exitoso:** el flujo continúa hacia el registro de ubicación.
- **Fallido:** el cliente vuelve a la pantalla de ingreso para intentarlo de nuevo.

**Resultado:** Cupo preaprobado recuperado de la base de datos. Esta evaluación corresponde únicamente a una preaprobación y no representa la aprobación definitiva del crédito.

**Tiempo estimado:** ~5 segundos (consulta directa, no cálculo en línea).

---

### 4. Registro de ubicación

**Actor:** Cliente.

**Información utilizada:** Departamento y ciudad/municipio (listas desplegables); dirección donde desarrolla su actividad comercial (campo de texto libre).

**Proceso:** El cliente selecciona departamento y ciudad de listas desplegables predefinidas y luego digita la dirección exacta en un campo abierto. La información se guarda como parte del perfil comercial inicial.

**Resultado:** La información queda registrada como parte del perfil inicial del negocio.

**Tiempo estimado:** ~20 segundos.

---

### 5. Clasificación del tipo de cliente

**Actor:** Sistema.

**Proceso:** El sistema distingue automáticamente, a partir del documento ingresado en el paso 2, si corresponde a persona jurídica (NIT) o persona natural (CC), y enruta el flujo sin necesidad de una pregunta explícita al cliente.

**Decisión:** ¿Tipo de cliente: NIT o CC?

- **NIT (persona jurídica):** el flujo continúa hacia el reconocimiento del representante legal.
- **CC (persona natural):** el flujo continúa directamente hacia la aceptación de términos y condiciones.

**Resultado:** En el flujo CC esta validación "pasa por detrás", es decir, se verifica automáticamente contra la base de datos, sin una pantalla adicional visible para el cliente.

**Tiempo estimado:** Instantáneo (automático, sin fricción para el cliente).

---

### 6. Validación del representante legal (flujo NIT)

**Actor:** Cliente.

**Proceso:** Cuando el flujo corresponde a una empresa (NIT), la plataforma muestra una pantalla donde el cliente confirma explícitamente que es el representante legal de la empresa antes de continuar.

**Resultado:** Confirmación registrada de que quien realiza el proceso es el representante legal, previo a la aceptación de términos y condiciones.

**Tiempo estimado:** ~10 segundos.

---

### 7. Aceptación de términos y condiciones

**Actor:** Cliente.

**Proceso:** El cliente accede a los términos y condiciones mediante un hipervínculo y marca la casilla de aceptación. Si no la marca, el sistema simplemente no permite continuar (no se dispara ningún mensaje de error específico; el flujo se detiene ahí).

**Resultado:** El cliente acepta los términos y condiciones del producto. Esta aceptación es obligatoria para continuar con la solicitud, tanto en el flujo NIT como en el flujo CC.

**Tiempo estimado:** ~15 segundos.

---

### 8. Registro del representante legal

**Actor:** Cliente.

**Información utilizada:** Nombre, cédula de ciudadanía y número de teléfono del representante legal (campos desglosados de forma independiente).

**Proceso:** El cliente diligencia en pantalla los tres datos del representante legal por separado (nombre / cédula / teléfono). El teléfono queda registrado como dato de contacto. El mecanismo específico de validación del número telefónico dentro del onboarding queda pendiente de confirmar.

**Resultado:** Queda registrada la información del representante legal necesaria para continuar con la confirmación de datos.

**Tiempo estimado:** ~30 segundos.

---

### 8.1 Confirmación de datos del representante legal

**Actor:** Cliente.

**Proceso:** Antes de continuar hacia la validación del correo electrónico, la plataforma muestra en pantalla un resumen de los datos ingresados en el paso 8 (nombre, cédula, teléfono) para que el cliente los revise y confirme que son correctos. Si detecta un error, puede volver atrás y corregirlo antes de continuar.

**Resultado:** Datos del representante legal confirmados por el cliente, reduciendo el riesgo de errores de digitación antes de continuar el flujo.

**Tiempo estimado:** ~10 segundos.

> **Actualización (sep 2026):** el OTP de firma del contrato **no se elimina**; vive en el proceso de firma (documento 03). Los OTP de WhatsApp y correo de este onboarding se mantienen para validar contacto. La definición específica de la validación del número telefónico dentro del onboarding queda pendiente de confirmar.

---

### 9. Envío y validación del código de verificación al correo electrónico

**Actor:** Cliente y Web (sistema).

**Información utilizada:** Correo electrónico registrado por el cliente.

**Proceso:** El sistema envía un código de verificación al correo electrónico del cliente. El cliente lo ingresa y el sistema lo valida antes de habilitar la creación del PIN de seguridad. El cliente puede solicitar el reenvío si no lo recibe.

**Decisión:** ¿El código de verificación ingresado es válido?

- **Exitoso:** el flujo continúa hacia la creación del PIN de seguridad.
- **Fallido:** el cliente vuelve a la pantalla de ingreso del código para intentarlo de nuevo.

**Resultado:** Correo electrónico validado, previo a la creación del PIN.

**Tiempo estimado:** ~30-45 segundos (depende de la entrega del correo).

---

### 10. Creación del PIN de seguridad

**Actor:** Cliente.

**Información utilizada:** PIN de cuatro dígitos definido por el cliente.

**Proceso:** El cliente define un PIN de cuatro dígitos, el cual queda guardado en base de datos para autenticación futura. Se acordó **no** aplicar reglas adicionales de validación de combinaciones inseguras (p. ej. secuencias o repeticiones) por el momento, para simplificar el piloto.

**Resultado:** El PIN queda guardado en base de datos para autenticación futura.

**Tiempo estimado:** ~15 segundos.

---

### 11. Confirmación de creación de la cuenta

**Actor:** Web (sistema).

**Proceso:** El sistema marca la cuenta del socio D1 como creada y actualiza el estado del cliente a **"pendiente de biometría"**, habilitando la siguiente etapa del proceso.

**Resultado:** El sistema notifica que la cuenta de socio D1 fue creada exitosamente. El cliente pasa a estado de biometría pendiente.

**Tiempo estimado:** Instantáneo.

> **Pendiente de producto:** agregar un copy que indique al cliente que debe revisar su correo para continuar.

---

### 12. Envío del enlace para biometría

**Actor:** Web (sistema).

**Sistemas involucrados:** Olimpia (proveedor externo de biometría).

**Proceso:** El sistema genera automáticamente un correo con el enlace único de biometría de Olimpia y lo envía al cliente. La biometría se realiza fuera de la plataforma web.

**Resultado:** Correo con enlace de biometría enviado al cliente.

**Tiempo estimado:** Instantáneo del lado del sistema (asíncrono en su recepción).


---

### 13. Validación biométrica *(placeholder — pendiente de definición técnica con Olimpia)*

**Actor:** Cliente y proveedor externo (Olimpia).

**Proceso:** El cliente completa el proceso biométrico directamente en la plataforma de Olimpia. El detalle técnico de esta integración aún está en definición.

**Resultado:** El resultado puede ser Aprobado (exitoso), En revisión o Rechazado.

**Tiempo estimado:** Variable, fuera del control de la plataforma y fuera del conteo de los 5 minutos del onboarding web.

---

### 14. Gestión del resultado biométrico *(placeholder — pendiente de definición técnica con Olimpia)*

**Actor:** Sistema-webhook.

**Proceso:** El sistema escucha, mediante un webhook, la respuesta que envía Olimpia una vez finalizado el proceso biométrico.

**Decisión:** ¿Cuál es el resultado de la biometría?

- **Exitoso:** el flujo continúa automáticamente hacia la vinculación de la cuenta bancaria.
- **En revisión o Rechazado:** el caso pasa a un analista de riesgo para revisión manual.

**Resultado:** Enrutamiento automático según el resultado recibido vía webhook.

**Tiempo estimado:** Instantáneo (procesamiento del webhook).

> **Pendiente de validación:** confirmar con el dueño del proceso si todo rechazo automático pasa siempre por revisión manual o si existen rechazos definitivos sin intervención del analista.

---

### 15. Revisión manual por analista de riesgo *(placeholder)*

**Actor:** Analista de riesgo.

**Información utilizada:** Evidencia del proceso biométrico.

**Proceso:** El analista revisa manualmente la evidencia del caso marcado como "en revisión" o "rechazado" y determina si aprueba o rechaza definitivamente la solicitud.

**Decisión:** ¿El analista aprueba o rechaza el caso?

- **Aprueba:** el flujo continúa hacia la vinculación de la cuenta bancaria.
- **Rechaza:** la solicitud finaliza.

**Tiempo estimado:** Asíncrono, fuera del journey de cliente; no cuenta dentro del límite de 5 minutos del onboarding web.

---

### 16. Vinculación de la cuenta bancaria

**Actor:** Cliente.

**Sistemas involucrados:** Drúo.

**Información utilizada:** Entidad financiera y número de cuenta bancaria.

**Proceso:** El cliente selecciona el banco e ingresa su número de cuenta. El sistema envía la información a Drúo para validación, la cual ocurre de forma **asíncrona** (no bloquea al cliente en pantalla mientras se resuelve).

**Resultado:** Cuenta bancaria enviada a validación ante Drúo antes de autorizar el débito automático.

**Tiempo estimado:** ~30 segundos de interacción del cliente; la validación ante Drúo corre en paralelo/asíncrona.

---

### 17. Carga de documentación bancaria

**Actor:** Cliente.

**Información utilizada:** Certificación bancaria (saldo actual) y extractos bancarios de los últimos tres meses.

**Proceso:** El cliente adjunta los archivos solicitados. 

**Resultado:** La documentación queda adjunta como insumo para el análisis crediticio.

**Tiempo estimado:** ~30-60 segundos si el cliente ya cuenta con los documentos a la mano (variable si debe buscarlos).

---

### 18. Selección de localidad habitual *(placeholder)*

**Actor:** Cliente.

**Información utilizada:** Localidad donde realiza habitualmente sus compras.

**Proceso:** El cliente selecciona su localidad habitual de compra, información que complementa el perfil comercial usado en la evaluación del crédito.

**Resultado:** Información registrada como parte del perfil comercial.

**Tiempo estimado:** ~10 segundos.

---

### 19. Envío al análisis de crédito

**Actor:** Web (sistema).

**Proceso:** El sistema envía automáticamente toda la información recopilada al motor de riesgo para el análisis de crédito, el cual queda **completamente automatizado** (ya no se envía una solicitud manual a un asesor). En su lugar, se genera una **notificación automática de funnel** para que el equipo interno monitoree el flujo, sin que esto sea un paso manual del proceso de aprobación.

**Resultado:** El sistema informa al cliente que debe esperar respuesta en un plazo de 24 horas.

**Tiempo estimado:** Instantáneo (envío automático); la respuesta del análisis toma 24 horas.

---

### 20. Continuación hacia KYC

**Proceso:** La solicitud pasa automáticamente al proceso de Validación de Identidad (KYC) más motor de riesgo, donde se realizan las verificaciones correspondientes (incluyendo servicios de Experian) antes de la originación del crédito.

**Resultado:** La solicitud continúa con el proceso de KYC.

**Tiempo estimado:** Asíncrono, fuera del onboarding web.

---

## Reglas de negocio

- Cada cliente inicia el proceso mediante un enlace a la landing page enviado por Sendgrid y Zenvia.
- El cupo preaprobado se consulta a partir de información transaccional de D1 **ya calculada y almacenada previamente**; si la identificación falla, el cliente debe reintentar el ingreso de su NIT o identificación.
- La aceptación de términos y condiciones es obligatoria, tanto en el flujo NIT como en el flujo CC; si no se acepta, el sistema simplemente no permite avanzar.
- - El cliente debe validar un código de verificación enviado a su correo electrónico antes de crear el PIN de seguridad; puede solicitar el reenvío si no lo recibe.
- El cliente debe confirmar en pantalla los datos del representante legal (paso 8.1) antes de continuar hacia la validación del correo electrónico.
- El cliente debe validar un código de verificación enviado a su correo electrónico antes de crear el PIN de seguridad; puede solicitar el reenvío si no lo recibe.
- El cliente debe crear un PIN de cuatro dígitos, sin reglas adicionales de validación de combinaciones inseguras por ahora.
- La biometría se realiza mediante el proveedor externo *Olimpia*, fuera de la plataforma web.
- Los resultados de biometría "en revisión" y "rechazado" son evaluados por un analista de riesgo, quien determina la aprobación o el rechazo final.
- La cuenta bancaria debe validarse ante Drúo, de forma asíncrona, antes de autorizar el débito automático.
- El cliente debe adjuntar certificación bancaria (saldo actual) y extractos bancarios de los últimos tres meses; este requerimiento puede ajustarse según las reglas de riesgo finales.
- El estudio de crédito inicial y el análisis de riesgo son **100% automatizados**; no interviene un asesor humano en esta etapa.
- El equipo de análisis de crédito informa su respuesta en un plazo de hasta 1 día.
- Solo las solicitudes que completen exitosamente el onboarding continúan hacia KYC.

---

## Entradas

- Enlace de invitación a la landing page.
- Correo electrónico.
- Número de teléfono.
- Documento de identidad (NIT o CC).
- Departamento, ciudad y dirección.
- Nombre del representante legal.
- Cédula de ciudadanía del representante legal.
- Teléfono de contacto (dato de contacto, sin validación OTP).
- Código de verificación (correo electrónico).
- PIN de seguridad.
- Cuenta bancaria.
- Certificación bancaria.
- Extractos bancarios.
- Localidad habitual de compra.

---

## Salidas

- Cliente registrado.
- Cupo preaprobado identificado.
- Correo electrónico validado.
- Datos del representante legal confirmados.
- PIN registrado.
- Estado "pendiente de biometría" asignado.
- Biometría completada (con o sin revisión manual).
- Cuenta bancaria enviada a validación ante Drúo.
- Documentación bancaria registrada.
- Notificación automática de funnel para seguimiento interno.
- Proceso preparado para continuar con KYC y motor de riesgo.

---

## Excepciones

- El cliente no abre la invitación.
- El cliente abandona el proceso.
- La consulta del cupo preaprobado falla.
- No acepta los términos y condiciones.
- El código de verificación por correo expira o es incorrecto.
- La biometría es rechazada o queda en revisión y el analista de riesgo la rechaza.
- Error durante la validación biométrica con Olimpia.
- Error en la validación de la cuenta bancaria ante Drúo.
- No se adjunta la documentación bancaria.

---

## Consideraciones

- El onboarding corresponde a un proceso completamente digital, en aplicación web (no app móvil descargable).
- La identificación del cupo corresponde únicamente a una preaprobación ya calculada, no a un cálculo en línea.
- La biometría se realiza mediante el proveedor externo Olimpia y reemplaza la validación in-house (foto de cédula, selfie y verificación manual). Su integración técnica aún está en definición (placeholder).
- **Riesgo estratégico a monitorear:** Olimpia pertenece a Colpatria, lo cual implica una dependencia que el equipo debe seguir de cerca junto con las coordinaciones de otros proveedores (Drúo, Experian).
- La validación manual del analista de riesgo aplica cuando la biometría queda "en revisión" o "rechazada".
- La cuenta bancaria se valida ante Drúo de forma asíncrona antes de autorizar el débito automático.
- La documentación bancaria hace parte de los insumos para el análisis crediticio y puede ajustarse según las reglas de riesgo finales.
- El análisis de crédito inicial es automatizado por el motor de riesgo; el plazo de respuesta es de hasta 1 día.
- Para un cliente que tenga todos los datos requeridos, el proceso de onboarding activo (interacción con el cliente) no debe tardar más de 5 minutos; los tiempos por paso listados en este documento son estimaciones de planeación, pendientes de validar con Producto/UX.
- El onboarding finaliza exitosamente cuando la solicitud queda preparada para continuar con KYC.

---


## Fuentes consultadas

- *Journeys Colpatria B2B* (junio de 2026), páginas 2 y 3.
- Documentación de integraciones (Sendgrid, Zenvia, Olimpia, Drúo).
- Documento de Alcance del Producto.
- Reunión *Producto: Check-in*, 16 de julio de 2026 (transcripción y notas de Gemini), con participación de Francisco Javier Martínez Vargas, Alejandra Suárez, Iván Aponte, Nicolás Ibagón y María Fernanda Herazo.
