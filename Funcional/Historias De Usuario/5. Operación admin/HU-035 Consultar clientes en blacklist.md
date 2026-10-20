#### HU-035: Consultar y gestionar clientes en blacklist

| Campo | Detalle |
|:---:|:---:|
| **Actor** | Administrador del producto |
| **Historia** | Como administrador, quiero consultar los documentos en lista negra y gestionar sus altas y bajas, para impedir el acceso aunque la persona no exista como cliente. |
| **Prioridad** | Baja |
| **Criterios de aceptación** | El administrador consulta en Producto → Lista negra el documento, el motivo, la fecha y quién lo agregó. Puede dar de alta un documento sin cliente y quitarlo. Si el registro está vinculado a un cliente, el nombre enlaza a su ficha.  |
| **Relaciones** | Casos de uso: [CU-017](../../Casos de Uso/6. Operaci�n Admin/CU-017 Administrar y consultar clientes en blacklist.md). Historia relacionada: [HU-034](../5. Operaci�n admin/HU-034 Administrar la lista negra.md) . |
| **Referencias** | `apps/admin/src/app/product/blacklist/page.tsx`, `backends/admin/src/controllers/product-blacklist.controller.ts` |
 **Autor** | María Fernanda Herazo |
| **Fecha** | 18/08/2026 |
| **Versión** | V.1.8 |
| **Estado en plataforma** | **Implementado.** |
| **Comentarios** | **Corrección 2026-10-20:** la pantalla Producto → Lista negra está disponible. El listado es por documento. No hace falta que la persona exista en la plataforma para incluirla o para que el bloqueo aplique. |


