# Historias de Usuario — Fliipa

| Documento | Historias de Usuario |
|:---:|:---:|
| **Proyecto** | Fliipa |
| **Versión** | 1.7 |
| **Estado** | En revisión |
| **Responsable** | Producto y negocio |
| **Última actualización** | 2026-08-18 |

---

## Objetivo

Describir las necesidades de cada actor de Fliipa en lenguaje de negocio, como base para la priorización y el desarrollo de las funcionalidades del sistema.

## Alcance

Cubre historias de usuario para el cliente empresarial, el asesor comercial, el analista de riesgo, el analista de cartera, el agente de servicio al cliente y el administrador del producto. Usa la numeración `HU-XXX`.

---

## Control de versiones (detallado)

### v0.1 — 2026-07-06
Borrador vacío. Se define la plantilla de ficha (Actor, Historia, Prioridad, Criterios de aceptación, Relaciones, Referencias, Autor/Fecha/Versión, Comentarios) sin contenido cargado todavía.

### v1.0 — 2026-07-10
Primera versión completa: **28 historias de usuario** organizadas por actor (cliente empresarial, asesor comercial, analista de riesgo, analista de cartera, agente de servicio al cliente, administrador del producto), en línea con el documento de Actores y el de Casos de Uso.

### v1.1 — 2026-07-10
Se reorganiza el mismo contenido en **tablas por actor**, agregando de forma explícita la columna de prioridad y las referencias a casos de uso relacionados por cada historia.

### v1.2 — 2026-07-10
Se convierte cada historia de la tabla en una **ficha individual** (una tabla de campo/detalle por HU), adaptando el formato usado en los documentos de actores. Este es el formato de ficha que se mantiene hasta hoy.

### v1.3 — 2026-07-22
Primera revisión técnica: se **contrasta cada historia contra el código fuente real** del repositorio `fliipa-main`.
- Se corrigen rutas de archivo que no existían o estaban mal escritas.
- **Hallazgo crítico**: el backend administrativo (`backends/admin`), referenciado en las historias del administrador (HU-024 a HU-028 en la numeración de esa época), **no existía** en el repositorio analizado.

### v1.4 — 2026-07-30
Se corrige el hallazgo crítico de la v1.3: el repositorio de referencia correcto para el módulo administrativo es `credits-platform-main`, no `fliipa-main`. Se corrigen las **cinco fichas afectadas** con las rutas ya verificadas contra el repositorio correcto.

### v1.5 — 2026-08-13
Revisión funcional y técnica integral:
- Se elimina una referencia obsoleta al actor "tendero" (ya no existe en el modelo de negocio).
- Se unifica el repositorio de referencia en `credits-platform-main` para todas las fichas.
- Se corrige el SLA de resultado de solicitud a **24 horas**.
- Se precisa que la validación de identidad es por **código OTP**.
- Se matiza la referencia al buró **Datacrédito** (la evidencia en código no permitía confirmarlo como buró independiente usado en paralelo con Experian).

### v1.6 — 2026-08-13
Revisión funcional integral a partir de un documento de **retroalimentación de negocio con 24 observaciones numeradas** sobre la v1.5. Cambios principales:

- Se agrega una historia nueva (consulta de cupo preaprobado por documento) **antes** de la que ya existía, lo que obliga a **renumerar todas las historias posteriores**: de 28 se pasa a **36 historias (HU-001 a HU-036)**.
- Se separan historias que mezclaban dos procesos distintos (no atómicas):
  - Biometría ↔ carga de soportes bancarios.
  - Prepago por PSE ↔ débito automático de cartera vencida.
  - Atención por IA ↔ escalamiento a agente humano.
- Se corrigen criterios de aceptación que en realidad eran **precondiciones** o pasos del proceso, no condiciones verificables de cumplimiento (se agrega el campo "Precondiciones" donde aplica).
- Se elimina la restricción no justificada de que ciertos pasos del flujo solo pudieran hacerse "desde el celular".
- Se agregan historias que no existían: reintento/corrección de OTP, mensaje de no disponibilidad de crédito, registro general de contacto con el cliente (más allá de cobranza), consulta de clientes en blacklist, y registro del motivo de ingreso a blacklist.
- Se corrigen títulos poco representativos (p. ej. "Ver score, Experian e histórico D1 en un solo lugar" → "Análisis de KYC + evaluación de crédito") y errores tipográficos ("Soporto" → "soporte").

Ver el **detalle observación por observación** en la sección [Detalle de las 24 observaciones (v1.5 → v1.6)](#detalle-de-las-24-observaciones-v15--v16) y la [tabla de correspondencia de numeración](#tabla-de-correspondencia-de-numeración-v15--v16).

### v1.6 (cotejo técnico adicional) — 2026-08-14
En paralelo a las 24 observaciones de negocio, se realizó un **cotejo técnico** de los comentarios de cada ficha contra el comportamiento real de la plataforma (no del código en abstracto, sino de lo que el sistema efectivamente hace hoy). Este cotejo tiene dos partes:

- **Parte 1 — Corrección de comentarios**: de las 36 historias, se revisó el campo "Comentarios" de cada una. **25 se dejaron sin cambios** (HU-004, 006, 008, 010, 012, 013, 015–031, 033, 034, 036) y **11 se actualizaron** por tener comentarios desactualizados o imprecisos: HU-001, 002, 003, 005, 007, 009, 011, 014, 032, 035.
- **Parte 2 — Ocho historias nuevas propuestas** (HU-037 a HU-044), numeradas de forma tentativa, para cubrir funcionalidad real de la plataforma que no estaba documentada: consulta de soportes bancarios en el panel admin, consulta de centrales (Experian) desde la ficha del cliente, coincidencias Fliipa vs. Reconocer, reconexión de cuenta Druo, cambio manual de estado del crédito, y el motor KYC post-solicitud (inicio automático, evaluación de reglas, decisión final).

Ver el detalle completo en [Detalle del cotejo técnico (Parte 1 y Parte 2)](#detalle-del-cotejo-técnico-parte-1-y-parte-2).

### v1.7 — 2026-08-18
**Reestructuración documental**, decidida en la reunión *Weekly Sync* del 2026-08-14. Se abandona el archivo único y las 36 historias (HU-001 a HU-036) se organizan en capítulos por vertical de servicio. El contenido de cada ficha (identificador, criterios de aceptación, referencias, comentarios) **se conserva sin cambios** respecto a la v1.6; solo cambia su ubicación y agrupación. Este README pasa a ser el punto de entrada del documento.

> **Nota de esta versión del README:** además de documentar el historial de versiones, esta edición deja constancia de algunos puntos de la reestructuración v1.7 que quedaron **inconsistentes o incompletos** al momento de esta revisión (ver [Hallazgos pendientes](#hallazgos-pendientes-de-la-reestructuración-v17)), para que se corrijan antes de considerar la v1.7 cerrada.

---

## Índice de capítulos (estado real del repositorio)

| Capítulo | Carpeta | Historias que contiene hoy |
|:---|:---|:---|
| Onboarding | `Onboarding/` | HU-001, 002, 003, 004, 005, 019, 020 (fichas individuales) + `onboarding.md` con el capítulo completo HU-001 a HU-021 |
| KYC | `KYC/` | HU-006, 007, 008, 009, 022, 023, 037, 042, 043, 044 |
| Crédito | `Credito/` | HU-010, 011, 012 |
| Cobranza | `Cobranza/` | HU-013, 014, 015, 024, 025, 026, 027 |
| Servicio al Cliente | `Servicio al Cliente/` | HU-016, 017, 018, 028, 029, 030 |
| Gestión Plataforma del admin | `Gestión Plataforma del admin/` | HU-031, 032, 033, 034, 035, 036, 038, 039, 040, 041 |

> Esta tabla refleja lo que **existe físicamente** en el repositorio hoy, no lo que describía el README antes de esta revisión (ver hallazgos abajo: el índice anterior enlazaba a archivos de capítulo que no existen y omitía el capítulo "Crédito").

---

## Detalle de las 24 observaciones (v1.5 → v1.6)

| # | Observación de negocio | Resolución aplicada |
|:---:|:---|:---|
| 1 | Faltaba una HU de "consultar si tengo cupo preaprobado con mi documento" antes de la 002 | Se crea **HU-002** nueva; la antigua 002 pasa a ser HU-003 |
| 2 | HU-003 (ahora HU-004) hablaba de "el canal autorizado/definido" en singular | Se corrige a **ambos canales**: WhatsApp y correo electrónico |
| 3 | No existía HU para reintentar el OTP si no llega o se ingresó mal el contacto | Se crea **HU-005** nueva |
| 4 | HU-004 (v1.5) se llamaba "Completar KYC y cargar soportes desde el celular"; mezclaba biometría con carga de documentos y limitaba a celular | Se separa en **HU-006** (biometría) y **HU-007** (soportes bancarios); se elimina la restricción a celular |
| 5 | Las HU deben ser atómicas; la 004 juntaba dos pasos no relacionados | Resuelto junto con el punto 4 |
| 6 | La 004 no debía llamarse "flujo móvil" | Se corrige el título y la redacción en HU-006 y HU-007 |
| 7 | La HU-006 (v1.5, firma por OTP) se reescribió en v1.6 como proveedor externo | **Corrección sep 2026:** la firma **sigue siendo OTP**. HU-009 describe firma mediante OTP (el nombre de archivo histórico se mantiene). |
| 8 | En general, ninguna HU debía limitarse a "un teléfono celular" | Se revisa y corrige en todas las fichas afectadas (HU-006, HU-007, entre otras) |
| 9 | Faltaba una HU dedicada a clientes rechazados/en blacklist/sin crédito aprobado, con un mensaje de "no hay opciones disponibles" | Se crea **HU-011** nueva: "Ver mensaje de no disponibilidad de crédito" |
| 10 | HU-009 (v1.5) mezclaba pago por pasarela y débito automático; el débito automático no es "yo como usuario" sino "yo como Fliipa" | Se separa en **HU-013** (prepago por PSE) y **HU-014** (débito automático, reescrita en tercera persona de sistema) |
| 11 | HU-011 (v1.5) combinaba atención por IA con atención de agente humano | Se separa en **HU-016** (IA) y **HU-017** (escalar a humano), más **HU-028** (recibir caso escalado) |
| 12 | Error tipográfico "Soporto" en vez de "soporte" | Corregido en **HU-018** |
| 13 | Un criterio de aceptación de HU-013 (v1.5) era en realidad una precondición | Se agrega el campo **"Precondiciones"** en **HU-019** y se revisan todas las demás fichas |
| 14 | HU-014 (v1.5) era un buen ejemplo de criterio de aceptación | Sin cambios (ahora **HU-020**) |
| 15 | HU-015 (v1.5) se titulaba "seguimiento a primera compra" pero trataba sobre clientes que **no** habían comprado; su criterio de aceptación era un paso del proceso, no un resultado verificable | Se corrige título y criterios en **HU-021** |
| 16–21 | Historias sin observaciones relevantes | Sin cambios de fondo (renumeradas) |
| 22 | HU-022 (v1.5) era "la mitad" de la HU-004: gestión humana del servicio al cliente | Resuelto junto con el punto 11, vía HU-017/HU-028 |
| 23 | Faltaba una HU de registro de **todo** contacto con el cliente, no solo el de cobranza | Se crea **HU-030** nueva |
| 24 | HU-027 (v1.5, blacklist) no permitía registrar el motivo de ingreso; tampoco existía una HU para consultar la tabla de blacklist en el admin | Se actualiza **HU-034** (agrega motivo) y se crea **HU-035** nueva (consulta y gestión de blacklist) |

## Tabla de correspondencia de numeración (v1.5 → v1.6)

| v1.5 | v1.6 | Cambio |
|:---:|:---:|:---|
| HU-001 | HU-001 | Sin cambios de contenido |
| — | **HU-002** | Nueva: consultar cupo preaprobado por documento |
| HU-002 | HU-003 | Renumerada |
| HU-003 | HU-004 | Actualizada: validación por WhatsApp y correo |
| — | **HU-005** | Nueva: reintento/corrección de OTP |
| HU-004 | HU-006 + HU-007 | Dividida en biometría y soportes bancarios |
| HU-005 | HU-008 | Renumerada |
| HU-006 | HU-009 | Reescrita como HU-009; mecanismo vigente = OTP (sep 2026) |
| HU-007 | HU-010 | Renumerada |
| — | **HU-011** | Nueva: mensaje de no disponibilidad de crédito |
| HU-008 | HU-012 | Renumerada |
| HU-009 | HU-013 + HU-014 | Dividida en prepago (PSE) y débito automático (Fliipa) |
| HU-010 | HU-015 | Renumerada |
| HU-011 | HU-016 + HU-017 | Dividida en atención IA y escalamiento a humano |
| HU-012 | HU-018 | Corrección tipográfica |
| HU-013 | HU-019 | Corrección de criterio de aceptación (precondición → resultado) |
| HU-014 | HU-020 | Renumerada |
| HU-015 | HU-021 | Corrección de título y criterios de aceptación |
| HU-016 | HU-022 | Renumerada |
| HU-017 | HU-023 | Renombrada: "Análisis de KYC + evaluación de crédito" |
| HU-018 | HU-024 | Renumerada |
| HU-019 | HU-025 | Renumerada |
| HU-020 | HU-026 | Renumerada |
| HU-021 | HU-027 | Renumerada |
| HU-022 | HU-028 | Renumerada |
| HU-023 | HU-029 | Renumerada |
| — | **HU-030** | Nueva: registro general de contacto con el cliente |
| HU-024 | HU-031 | Renumerada |
| HU-025 | HU-032 | Renumerada |
| HU-026 | HU-033 | Renumerada |
| HU-027 | HU-034 | Actualizada: registro de motivo de ingreso a blacklist |
| — | **HU-035** | Nueva: consultar clientes en blacklist |
| HU-028 | HU-036 | Renumerada |

---

## Detalle del cotejo técnico (Parte 1 y Parte 2)

### Parte 1 — 11 fichas con comentarios actualizados

| HU | Qué estaba desactualizado | Qué se confirmó en la plataforma |
|:---:|:---|:---|
| HU-001 | El comentario decía que el sistema **rechazaba** la creación si ya había un cliente con ese documento | El sistema no rechaza: si hay una solicitud en `REQUEST_STARTED`, la **reutiliza**; si no, crea una nueva |
| HU-002 | Estatus "pendiente de verificar"; dudas sobre si era lo mismo que HU-003 | Confirmado en plataforma; distinta de HU-003 (esta valida existencia de preaprobación, HU-003 muestra el valor del cupo) |
| HU-003 | Parecía que solo existía "el modelo" a medias | La preaprobación y el cupo sugerido están operativos en el motor de riesgo, con carga de archivo de clientes preaprobados |
| HU-005 | Dudaba si existía reenvío o corrección de datos de contacto | Confirmado: el cliente puede reenviar el OTP (con tiempo de espera) y corregir teléfono/correo sin reiniciar la solicitud |
| HU-007 | Hablaba de un cargue "viejo" y dejaba duda sobre los extractos | Confirmado: se cargan 2 PDFs (certificación + extractos), la plataforma responde de inmediato y sube en segundo plano |
| HU-009 | Comentario v1.6 hablaba de proveedor externo | Confirmado en plataforma: la firma es OTP (`send-signature-otp.ts` + `sign-contract.ts`). No hay proveedor externo. |
| HU-011 | Decía "pendiente de verificar" | Confirmado el mensaje; se aclara que estar en blacklist **no** es lo que dispara este mensaje en el flujo actual |
| HU-014 | Daba por hecho el cobro automático completo con Druo | Solo está lista la **conexión** de cuenta con Druo; el débito automático de punta a punta **aún no** está implementado como producto |
| HU-032 | El rango de días de corte estaba incompleto | Confirmado: en la experiencia de usuario (admin y portal) el rango útil es **1–31**; a nivel interno la API aún admite 0 (detalle técnico, no de negocio) |
| HU-035 | El comentario sonaba como si ya estuviera casi en el panel | Aclarado: **no** existe hoy pantalla de blacklist consultable en el admin; queda **pendiente de implementar** |

### Parte 2 — 8 historias nuevas (HU-037 a HU-044)

| HU | Título | Estado en plataforma |
|:---:|:---|:---:|
| HU-037 | Ver y abrir soportes bancarios en el panel | Hecho |
| HU-038 | Consultar información de centrales (Experian) desde la ficha | Hecho |
| HU-039 | Ver coincidencias entre Fliipa y Reconocer | Hecho |
| HU-040 | Ver y reconectar la cuenta Druo del cliente | Hecho |
| HU-041 | Cambiar el estado del crédito (incluye pre-rechazado) | Hecho (cambio manual) |
| HU-042 | Iniciar la evaluación KYC al quedar la solicitud en REQUESTED | En desarrollo |
| HU-043 | Evaluar las reglas KYC y guardar el resultado de cada una | En desarrollo |
| HU-044 | Decidir approved/pre_rejected y avisar cuando corresponda | En desarrollo |

> Estas ocho historias documentan el **motor KYC post-solicitud**: hoy las 4 reglas duras ya corren en el motor con comparación onboarding vs. centrales, pero el disparo automático de la corrida, el expediente por cliente y la decisión final automática todavía están en desarrollo. El acuerdo de negocio vigente es que el motor **nunca** deja un crédito en `rejected` automático: si no aprueba, pasa a `pre_rejected` (cola de operaciones).

---

## Hallazgos pendientes de la reestructuración v1.7

Al verificar el repositorio contra lo que describe el README de la v1.7, quedaron los siguientes puntos sin resolver:

1. **Enlaces rotos.** El índice de capítulos apuntaba a `kyc.md`, `cobranza.md`, `servicio-cliente.md` y `administracion.md`, pero esos archivos de capítulo **no existen**; solo `Onboarding/onboarding.md` se creó como archivo consolidado.
2. **Falta el capítulo "Crédito" en el índice.** La carpeta `Credito/` (HU-010, 011, 012) existe pero no estaba listada en la tabla de capítulos del README.
3. **La descripción del capítulo KYC estaba desactualizada.** Decía "HU-022, HU-023" cuando la carpeta real contiene también HU-006, 007, 008, 009, 037, 042, 043 y 044.
4. **HU-006 conserva "(KYC)" en el título**, algo que la Parte 2 del cotejo técnico sugería quitar (ej. renombrar a "Completar la validación biométrica con OlimpiaIT").
5. **Duplicación de contenido.** `Onboarding/onboarding.md` repite el texto completo de HU-001 a HU-021 que también existe como fichas individuales — hoy están sincronizadas, pero es una fuente doble de verdad a vigilar.
6. **Detalles menores de nomenclatura**: la ficha de HU-041 conserva el encabezado "PROPUESTA" pese a estar marcada como implementada; algunos nombres de archivo tienen doble espacio (`HU-039  Ver...`, `HU-042  Iniciar...`).

Se recomienda cerrar estos puntos (crear los archivos de capítulo faltantes o corregir los enlaces del índice, agregar "Crédito" a la tabla, renombrar HU-006 y limpiar los nombres de archivo) antes de dar la v1.7 por completamente cerrada.

---

## Conclusión general

La revisión funcional v1.6 incorporó las 24 observaciones de negocio y el cotejo técnico agregó 8 historias nuevas y corrigió 11 comentarios desactualizados, para un total de **44 historias de usuario documentadas** (HU-001 a HU-044, sin contar los saltos de numeración de HU-016 al HU-021 original de v1.5 que ya fueron absorbidos en la renumeración). Se mantienen vigentes hallazgos técnicos de versiones anteriores que aún no están resueltos en código: el OTP comodín hardcodeado, el canal SMS simulado, y la ausencia de módulos de alivios, mora por buckets e IA conversacional en el repositorio revisado. Estos aspectos deben priorizarse junto con los hallazgos documentales de la sección anterior antes de llevar las historias pendientes a desarrollo.

## Fuentes consultadas

- Documento de retroalimentación funcional de negocio sobre la v1.5 (24 observaciones numeradas).
- Documento de cotejo técnico v1.6 (Parte 1: corrección de 11 comentarios; Parte 2: 8 historias nuevas HU-037 a HU-044).
- Acta y transcripción de la reunión *Weekly Sync* del 2026-08-14, base de la reestructuración documental de la v1.7.
- Revisión directa del repositorio `MariaFer2006/documentacion_fliipa`, carpeta `Funcional/Historias De Usuario`.
