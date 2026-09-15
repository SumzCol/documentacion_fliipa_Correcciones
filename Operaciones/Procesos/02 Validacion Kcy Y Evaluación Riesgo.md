# 2. Validación de identidad (KYC)


## Objetivo

Validar automáticamente la identidad del cliente y evaluar si cumple los criterios mínimos de riesgo utilizando información proveniente de Experian (servicios de Historia de Crédito para persona natural y jurídica, y el servicio Reconocer para validación de datos de contacto), el historial transaccional de D1 y las reglas de negocio (reglas de Colpatria)* definidas para el producto. Al finalizar este proceso se determina si la solicitud continúa hacia la firma del contrato, pasa a un estado de "posible rechazo" para revisión manual del equipo de operaciones, o el crédito es rechazado.

*Los elementos marcados con asterisco (\*) corresponden a puntos aún no definidos técnicamente o pendientes de confirmación con el dueño del proceso; se detallan en cada paso y se listan de forma consolidada en "Pendientes de validación".*

---

## Journey

![Journey Colpatria B2B — página 4](imagenes/page-04.png)

**Figura 4. Journey de Validación de Identidad (KYC) y Evaluación Inicial de Riesgo.**

En el journey, las dos cajas moradas ("Evalúa criterios de KYC automáticamente" y "Valida la información de la cuenta contra los productos reportados en Experian") corresponden a pasos ajustados en junio de 2026 (leyenda "Ajuste · jun 2026"); las demás cajas corresponden a pasos ya existentes del flujo. El journey debe actualizarse para reflejar el nuevo estado intermedio de "posible rechazo" y la eliminación de la consulta a RUES (pendiente de reflejar en la Figura 4).

---

## Descripción general

Una vez el cliente finaliza el proceso de onboarding digital, la solicitud ingresa automáticamente al proceso de Validación de Identidad (KYC). El motor de riesgo se activa de forma automática sin esperar respuesta de Druo, permitiendo una evaluación inmediata. En esta etapa el sistema recibe el caso en el Admin, consulta y almacena información de fuentes externas Experian (Historia de Crédito y servicio Reconocer)— y el historial transaccional de D1, y ejecuta de forma automática las reglas de negocio de Colpatria* establecidas para evaluar la elegibilidad del cliente. El resultado de la validación biométrica obtenida durante el onboarding, realizada por el proveedor externo (Olimpia)*, también queda registrado en la base de datos del producto.

Si el cliente cumple con todos los criterios definidos, el sistema valida la cuenta bancaria registrada contra los productos financieros reportados en Experian y aprueba o ajusta* el cupo de crédito antes de continuar con la firma del contrato. En caso de que alguna validación falle ya sea el cumplimiento de requisitos o la validez de la cuenta, la solicitud pasa a un estado de **"posible rechazo"**: el sistema emite una alerta interna por Slack y el equipo de operaciones revisa el caso antes de confirmar el rechazo definitivo desde el Admin. Solo una vez confirmado el rechazo por operaciones, el cliente recibe la notificación por correo electrónico. Este proceso de estudio de crédito es automático, con un punto de control manual estratégico para los posibles rechazos, y elimina el estudio manual que anteriormente realizaba un analista para la totalidad de los casos.

La cuenta bancaria ingresada por el cliente también se registra en Druo para habilitar futuros débitos; sin embargo, el motor de riesgo toma su decisión con base en la información contrastada contra Experian, no contra el resultado de open banking (a diferencia del modelo B2C).

---

## Explicación paso a paso

### 1. Recepción de la solicitud

**Actor:** Sistema.

**Sistemas involucrados:** Admin (módulo administrativo), Slack (canal de alertas internas).

**Información utilizada:** Solicitud finalizada durante el onboarding digital.

**Proceso:** Al cerrarse exitosamente el onboarding, el sistema dispara el motor de riesgo de forma inmediata, sin esperar respuesta de Druo, y adicionalmente lanza una alerta al canal de Slack de operaciones, cubriendo la llegada de nuevos clientes, aprobaciones, rechazos y alertas de fraude, sin que el cliente deba realizar ninguna acción adicional.

**Resultado:** El sistema recibe la solicitud y abre automáticamente el caso dentro del Admin. A partir de este momento inicia el proceso de KYC.

**Tiempo estimado:** Instantáneo (disparado automáticamente al cierre del onboarding).

---

### 2. Consulta y almacenamiento de información en Experian y D1

**Actor:** Sistema.

**Sistemas involucrados:** Experian (servicio de Historia de Crédito para persona natural y jurídica; servicio Reconocer para datos de contacto), base de datos transaccional de D1.

**Información utilizada:** Información del cliente registrada durante el onboarding.

**Proceso:** El sistema ejecuta, en el caso abierto en el Admin, las consultas correspondientes a Experian incluyendo el historial de crédito para personas, y el servicio Reconocer para contrastar los datos de contacto y el histórico transaccional de D1 del cliente. Cada resultado se almacena como parte del expediente del caso. Adicionalmente, el resultado de la validación biométrica obtenida durante el onboarding con el proveedor externo (Olimpia)* se registra en el Admin como un insumo más del expediente, sin volver a ejecutarse en esta etapa.

La cuenta bancaria informada por el cliente se registra también en Druo para habilitar futuros débitos; este registro es paralelo al expediente de KYC y no sustituye la validación contra Experian del Paso 5.

**Resultado:** El sistema consulta y almacena en el Admin la información de Experian y el historial transaccional de D1, junto con el resultado biométrico del onboarding.

**Tiempo estimado:** Procesamiento automático (instantáneo, vía APIs).

---

### 3. Evaluación automática de criterios KYC

**Actor:** Sistema.

**Sistemas involucrados:** Motor de reglas (reglas de Colpatria)*.

**Información utilizada:**

- Score (puntaje) mínimo requerido tanto para persona jurídica como persona natural*.
- Capacidad de endeudamiento*.
- Datos de contacto reportados mediante el servicio Reconocer de Experian vs. datos de contacto reportados durante el proceso de onboarding.
- Localidad habitual declarada por el cliente durante el onboarding, comparada de forma automática contra la tienda habitual registrada en el histórico transaccional de D1*.
- Comparación de la cuenta bancaria ingresada por el cliente vs. el servicio "Historia de Crédito" de Experian.

**Proceso:** El motor de reglas toma la información consolidada en el paso 2 (Experian, D1 y biometría) y la evalúa de forma automática contra los criterios de riesgo configurados para el producto. Este paso fue ajustado en junio de 2026 para incorporar la comparación automática entre la tienda habitual declarada y la registrada en D1.

**Resultado:** El sistema aplica automáticamente las reglas de Colpatria definidas para el proceso de estudio de crédito y determina si el cliente cumple o no los criterios.

**Tiempo estimado:** Instantáneo a segundos (procesamiento interno del motor de reglas).

> **Placeholder\*:** continúan pendientes de definición el valor exacto del score mínimo, la fórmula/límites de la capacidad de endeudamiento, y la metodología para comparar "tienda habitual declarada" vs. "tienda habitual registrada" (¿coincidencia exacta?, ¿radio geográfico?, ¿tolerancia?).

---

### 4. Verificación de cumplimiento de requisitos

**Actor:** Sistema.

**Proceso:** El sistema evalúa el resultado del paso 3 y bifurca el flujo según el cliente cumpla o no la totalidad de los criterios definidos.

**Decisión:** ¿El cliente cumple con los requisitos?

- **Sí:** si el cliente es aprobado se le envía de forma automática un correo (Sendgrid, plantilla `welcome`) y un WhatsApp (Zenvia, plantilla `creditApproved`) confirmando su aprobación. El rechazo definitivo sigue notificando solo por correo (paso 10).
- **No:** la solicitud pasa al estado de **"posible rechazo"** (ver Paso 7) en lugar de rechazarse de forma automática; se envía una alerta al canal de Slack de operaciones fliipa-ops-alerts para que el equipo revise el caso.

**Tiempo estimado:** Instantáneo.

---

### 5. Validación de la cuenta bancaria contra Experian (Historia de Crédito)

**Actor:** Sistema.

**Sistemas involucrados:** Experian, Druo (registro de la cuenta para débitos).

**Información utilizada:** Cuenta bancaria registrada durante el onboarding; productos financieros reportados en Experian.

**Proceso:** El sistema contrasta la cuenta bancaria registrada por el cliente (entidad + número de cuenta, capturados en el onboarding) contra los productos financieros que Experian reporta a nombre del mismo cliente, con el fin de confirmar que la cuenta le pertenece y que la información es consistente antes de continuar con la originación del crédito. A diferencia del modelo B2C, este modelo refuerza la seguridad contrastando contra Experian en lugar de basarse únicamente en el resultado de open banking. Si Druo reporta una discrepancia o falla sobre la titularidad de la cuenta, se realiza una conciliación interna verificando la información contra lo reportado en Experian.

**Resultado:** Confirmación de que la cuenta bancaria corresponde al cliente evaluado.

**Tiempo estimado:** Automático.

> **Placeholder\*:** no está definido el detalle exacto de qué campos de "productos financieros reportados en Experian" se usan para el cruce (¿tipo de producto, titularidad, antigüedad?) ni el criterio de coincidencia mínima aceptado. Pendiente de definición técnica.

---

### 6. Verificación de la cuenta bancaria

**Actor:** Sistema.

**Proceso:** El sistema evalúa el resultado del paso 5 y bifurca el flujo según la cuenta resulte válida o no.

**Decisión:** ¿La cuenta es válida?

- **Sí:** el proceso continúa hacia la aprobación o ajuste del cupo de crédito (Paso 8).
- **No:** el proceso pasa al estado de **"posible rechazo"** (Paso 7) en lugar de un rechazo automático directo.

**Tiempo estimado:** Instantáneo.

---

### 7. Estado de "posible rechazo" y revisión manual de operaciones

**Actor:** Sistema / Equipo de operaciones.

**Sistemas involucrados:** Admin, Slack.

**Proceso:** Cuando el cliente no cumple los requisitos (Paso 4) o la cuenta bancaria no resulta válida (Paso 6), el sistema ya no ejecuta el rechazo automático: marca la solicitud con el estado "posible rechazo" y envía una alerta al canal de Slack de operaciones. Un miembro del equipo revisa el caso en el Admin y confirma manualmente el rechazo, o determina que la solicitud puede continuar, evitando así rechazos erróneos.

**Resultado:** Solicitud marcada como "posible rechazo", pendiente de confirmación operativa; alerta de trazabilidad emitida en Slack.

**Tiempo estimado:** Instantáneo para la marca del estado; el tiempo de confirmación depende de la revisión manual del equipo de operaciones.

> **Pendiente\*:** no se han definido los SLA de revisión manual ni los criterios específicos que debe validar el analista antes de confirmar el rechazo.

---

### 8. Aprobación o ajuste del cupo de crédito

**Actor:** Sistema.

**Proceso:** Cuando todas las validaciones (pasos 3 a 6) son satisfactorias, el sistema calcula la decisión final: aprueba la solicitud con el cupo preaprobado del onboarding, o lo ajusta* de acuerdo con las reglas de negocio y los resultados obtenidos durante la evaluación (score, capacidad de endeudamiento, información de Experian).

**Resultado:** Solicitud aprobada o con cupo ajustado; la solicitud continúa hacia la firma del contrato.

**Tiempo estimado:** Instantáneo (cálculo automático).

> **Placeholder\*:** no está definido si el ajuste del cupo se realiza mediante reglas parametrizadas adicionales o si se maneja caso a caso; pendiente de validar con el dueño del proceso.

---

### 9. Rechazo del crédito y emisión de alerta

**Actor:** Sistema / Equipo de operaciones.

**Proceso:** Una vez el equipo de operaciones confirma el rechazo desde el estado de "posible rechazo" (Paso 7), el sistema ejecuta el rechazo definitivo de la solicitud, registra el causal correspondiente en la base de datos del producto, y mantiene el registro de la alerta emitida en Slack para trazabilidad. Ambos caminos ("No" del paso 4 y "No" del paso 6) confluyen en este mismo paso.

**Resultado:** Crédito rechazado, alerta registrada en Slack, y causal de rechazo guardado en la base de datos del producto.

**Tiempo estimado:** Instantáneo una vez confirmado el rechazo por operaciones.

> **Placeholder\*:** no está definido si el rechazo por incumplimiento de requisitos (Paso 4) y el rechazo por cuenta bancaria inválida (Paso 6) requieren distinto nivel de severidad de alerta dentro del mismo canal de Slack.

---
### 10. Notificación al cliente

**Actor:** Sistema.

**Sistemas involucrados:** Sendgrid.

**Proceso:** Una vez que el equipo de operaciones confirma el rechazo en el Paso 9, el sistema envía automáticamente una notificación al cliente informando que la solicitud de crédito no fue aprobada. La comunicación no incluye información sobre las razones o causales asociados a la decisión.

**Resultado:** Notificación de rechazo enviada al cliente.

**Tiempo estimado:** Instantáneo del lado del sistema tras la confirmación operativa; recepción asíncrona según el proveedor de correo.

> **Placeholder*:** no está definido el contenido/copy exacto de la notificación de rechazo, ni si existe algún canal adicional al correo para esta notificación.


## Reglas de negocio

- El proceso KYC se ejecuta de forma automática, con un punto de control manual para los casos de posible rechazo.
- La consulta a Experian (Historia de Crédito y servicio Reconocer) y al historial transaccional de D1 es obligatoria; el resultado se almacena en el Admin junto con el resultado de la biometría del onboarding.
- El cliente debe cumplir el puntaje mínimo definido para el producto*.
- Se evalúa automáticamente la capacidad de endeudamiento del cliente*.
- La localidad habitual declarada por el cliente se compara automáticamente contra la tienda habitual registrada en el histórico transaccional de D1*.
- La evaluación de criterios de KYC se realiza mediante las reglas de Colpatria*.
- La cuenta bancaria debe coincidir con los productos financieros reportados en Experian*.
- Si el cliente no cumple los requisitos o la cuenta bancaria no es válida, la solicitud pasa a estado de "posible rechazo" y requiere confirmación manual del equipo de operaciones antes del rechazo definitivo.
- Toda alerta relevante del proceso (llegada de solicitud, aprobación, posible rechazo/rechazo, alerta de fraude) se notifica en tiempo real por el canal de Slack de operaciones.
- Solo las solicitudes aprobadas continúan hacia la firma del contrato.
- Los causales de rechazo se guardan en la base de datos del producto para trazabilidad y futura revisión.

---

## Entradas

- Información del cliente registrada durante el onboarding.
- Resultado de la validación biométrica (almacenado en el Admin).
- Historial transaccional de D1.
- Información consultada en Experian (Historia de Crédito persona natural/jurídica y servicio Reconocer).
- Cuenta bancaria registrada por el cliente.
- Reglas de negocio de Colpatria definidas para el proceso de estudio de crédito.

---

## Salidas

- Cliente aprobado para continuar con la firma del contrato.
- Cupo de crédito aprobado o ajustado.
- Solicitud marcada en estado de "posible rechazo", pendiente de confirmación operativa.
- Solicitud rechazada (tras confirmación del equipo de operaciones).
- Alerta emitida en Slack y registrada en el sistema.
- Notificación de rechazo enviada al cliente por correo electrónico.

---

## Excepciones

- El cliente no cumple el puntaje mínimo requerido.
- La capacidad de endeudamiento supera los límites permitidos.
- La tienda habitual declarada no coincide con la registrada en el histórico transaccional.
- La información consultada en Experian presenta inconsistencias.
- Druo reporta una discrepancia o falla sobre la titularidad de la cuenta bancaria (se resuelve mediante conciliación interna contra Experian).
- La cuenta bancaria registrada no coincide con la información validada en Experian.
- Error durante la consulta a Experian o al historial transaccional de D1.
- Error en la ejecución de las reglas automáticas de Colpatria del proceso KYC.

---

## Consideraciones

- El proceso de Validación de Identidad (KYC) es mayoritariamente automático; esto elimina el estudio manual que antes realizaba un analista para la totalidad de los casos, y conserva una revisión manual estratégica únicamente para los posibles rechazos.
- El resultado de la biometría obtenida durante el onboarding (proveedor externo Olimpia) queda almacenado en el Admin y es uno de los insumos utilizados en esta evaluación.
- La validación de la cuenta bancaria se realiza contra los productos reportados en Experian, antes de aprobar definitivamente el cupo de crédito.
- Cualquier validación fallida —requisitos de KYC o validez de la cuenta— genera el estado de "posible rechazo" y una alerta en Slack; el rechazo definitivo y la notificación al cliente solo ocurren tras la confirmación del equipo de operaciones.
- Las reglas de evaluación (reglas de Colpatria) pueden modificarse conforme evolucione la estrategia de riesgo del producto.
- Varios de los parámetros críticos de este proceso (score mínimo, límites de endeudamiento, metodología de comparación de tienda habitual, reglas de ajuste de cupo, y SLA de revisión manual del posible rechazo) todavía son placeholders* y deben confirmarse antes de pasar este journey a desarrollo.

---

## Pendientes de validación

> **Pendiente de validar con el dueño del proceso:**
>
> - Confirmar si el ajuste del cupo de crédito puede realizarse mediante reglas parametrizadas adicionales antes de la firma del contrato, o si es un ajuste manual caso a caso. *(placeholder — paso 8)*
> - Validar si existen criterios complementarios no reflejados en el journey. *(placeholder — paso 3)*
> - Confirmar cómo se define y mide la comparación entre "tienda habitual declarada" y "tienda habitual registrada" (coincidencia exacta, radio geográfico, tolerancia). *(placeholder — paso 3)*
> - Confirmar el valor del score mínimo y la fórmula/límites de la capacidad de endeudamiento. *(placeholder — paso 3)*
> - Confirmar qué campos específicos de "productos financieros reportados en Experian" se usan para validar la cuenta bancaria y el criterio de coincidencia mínima. *(placeholder — paso 5)*
> - Definir el SLA y los criterios operativos para la revisión manual del estado de "posible rechazo" antes de confirmar el rechazo definitivo. *(placeholder — paso 7)*
> - Confirmar el copy/contenido exacto de la notificación de rechazo y si existe algún canal adicional al correo (SMS/WhatsApp). *(placeholder — paso 10)*

---

## Fuentes consultadas

- *Journeys Colpatria B2B* (junio de 2026), página 4.
- Documentación funcional del proceso KYC.
- Documento de reglas de negocio del producto.
- Documento de Alcance del Producto.
- Indicación de Iván Aponte (16 de julio de 2026): detallar el proceso de KYC paso a paso, incorporando los procesos técnicos de cada paso y marcando con asteriscos los puntos aún no definidos (placeholders).
- Acta y transcripción de la reunión "Producto: Check-in" (21 de julio de 2026), con asistencia de Francisco Javier Martínez Vargas, Alejandra Suárez, Iván Aponte, Oscar Ivan Briceño, Nicolás Ibagón y María Fernanda Herazo — fuente de las correcciones incorporadas en esta versión (eliminación de RUES, documentación de servicios Experian, estado de "posible rechazo", alertas Slack y almacenamiento de causales de rechazo).
