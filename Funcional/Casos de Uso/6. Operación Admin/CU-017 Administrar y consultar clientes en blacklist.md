### CU-017: Administrar y consultar clientes en blacklist

![Diagrama de caso de uso CU-017](imagenes/diagrama_CU-017.svg)

| Campo | Detalle |
|---|---|
| **Actores** | Administrador del producto |
| **Descripción** | El administrador incluye o retira un documento de la lista negra. El bloqueo aplica en la landing, al crear el checkout y en el login del portal, aunque no exista un cliente. |
| **Precondiciones** | Existe una decisión de negocio para incluir o retirar un documento. No se requiere un cliente previo. |
| **Flujo principal** | 1. El administrador abre Producto → Lista negra, o la ficha de un cliente existente.<br>2. Ingresa el documento y el motivo (mínimo 10 caracteres). En la baja también registra motivo.<br>3. El sistema guarda el documento, el motivo y quién lo hizo. Si hay un único cliente con ese documento, vincula el registro; si no hay cliente, el registro queda igual.<br>4. Desde la misma pantalla consulta el listado y puede quitar un documento. |
| **Flujos alternativos / excepciones** | A1. El documento ya está en la lista: el sistema no crea otro registro activo.<br>A2. Sin el permiso de gestionar lista negra, el administrador no puede dar de alta ni de baja.<br>A3. La persona intenta la landing, el onboarding o el login: ve el mensaje genérico de contactar a Fliipa y no se le dice que está en lista negra. Operaciones recibe una alerta con el motivo cuando el intento es de onboarding. |
| **Postcondiciones** | El documento queda incluido o retirado. Mientras esté activo, no puede entrar por la landing, no inicia onboarding y no entra al portal de redemption. |
| **Reglas de negocio** | Todo ingreso exige motivo de al menos 10 caracteres y guarda el email de quien lo hizo. La búsqueda de bloqueo es por número de documento, sin tipo de documento. La evaluación de riesgo y el desembolso no son, por sí solos, el punto de bloqueo: el freno está en la entrada (landing, alta de checkout y login). |
| **Historias de usuario relacionadas** | [HU-034](../../Historias De Usuario/5. Operaci�n admin/HU-034 Administrar la lista negra.md) (Administrar la lista negra)<br>[HU-035](../../Historias De Usuario/5. Operaci�n admin/HU-035 Consultar clientes en blacklist.md) (Consultar y gestionar clientes en blacklist) |
| **Estado en plataforma** | [HU-034](../../Historias De Usuario/5. Operaci�n admin/HU-034 Administrar la lista negra.md) : implementada: alta por documento, con cliente opcional, motivo y autor. [HU-035](../../Historias De Usuario/5. Operaci�n admin/HU-035 Consultar clientes en blacklist.md): **implementada** en Producto → Lista negra. |
| **Referencias** | Fuente: fichas [HU-034](../../Historias De Usuario/5. Operaci�n admin/HU-034 Administrar la lista negra.md)  y [HU-035](../../Historias De Usuario/5. Operaci�n admin/HU-035 Consultar clientes en blacklist.md) — *Historias de Usuario — Fliipa*, carpeta "6. Gestión Plataforma del admin" (repositorio `documentacion_fliipa`, María Fernanda Herazo). |
