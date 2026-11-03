# Infraestructura

| Documento | Infraestructura |
|---|---|
| **Proyecto** | Fliipa |
| **Estado** | Documentación técnica basada en la configuración versionada; validar el estado operativo directamente en GCP |
| **Responsable** | Tecnología / Infraestructura |

---

## Objetivo

Documentar la infraestructura técnica y la operación de Fliipa: proveedor cloud y entornos, pipeline de despliegue, configuración y secretos, jobs batch y migraciones, dominios públicos y operación diaria.

## Alcance

Cubre los aspectos técnicos de infraestructura y operación de la plataforma. No reemplaza las reglas de negocio (consultar [Negocio](../Negocio/README.md)) ni las especificaciones funcionales (consultar [Funcional](../Funcional/README.md)). Para documentación de producto y alcance, consultar [Producto](../Producto/README.md).

La documentación describe principalmente la configuración versionada en el repositorio `SumzCol/credits-platform`. No confirma por sí sola el estado actual de recursos, permisos, DNS, certificados o secretos en vivo; esos elementos deben verificarse en GCP y en los sistemas externos antes de operaciones.

## Documentos

| # | Documento | Contenido |
|---|---|---|
| 1 | [Arquitectura y entornos](01 Arquitectura y Entornos.md) | Proveedor GCP, aislamiento PROD y STG/DEV, flota, entrada pública y mapa código-cloud. |
| 2 | [Pipeline de despliegue](02 Pipeline de Despliegue.md) | Workflows vigentes, despliegues manuales, selección de servicios, migraciones y rollback. |
| 3 | [Configuración, variables de entorno y secretos](03 Configuracion y Secretos.md) | Configuración declarativa, URLs, variables literales, secretos, Firebase domains e IAM. |
| 4 | [Jobs batch y migraciones de base de datos](04 Jobs y Migraciones.md) | Handlers batch registrados, schedulers PROD, aprovisionamiento y migraciones TypeORM. |
| 5 | [Dominios públicos y Load Balancer](05 Dominio y Load Balancer.md) | Firebase Hosting activo, hosts y rutas por entorno; estado del Load Balancer como alternativa futura. |
| 6 | [Operación diaria, rollback y pendientes](06 Operacion y Pendientes.md) | Comandos operativos, pause/resume, secretos, recuperación y riesgos por validar. |

## Documentos relacionados

- [Índice general de documentación](../README.md)
- [Documentación de producto](../Producto/README.md)
- [Documentación de negocio](../Negocio/README.md)
- [Documentación funcional](../Funcional/README.md)
- [Operaciones](../Operaciones/README.md)
- [Autorizaciones, roles y permisos](../Autorizacion Roles y Permiso/README.md)

## Fuentes de verdad

Si un documento y la configuración del repositorio de código (`SumzCol/credits-platform`) discrepan, verificar primero el código y estos directorios en `main`:

- Proyectos, servicios, capacidad y dominios: [`config/deploy/`](https://github.com/SumzCol/credits-platform/tree/main/config/deploy)
- Infraestructura declarativa: [`terraform/envs/prod/`](https://github.com/SumzCol/credits-platform/tree/main/terraform/envs/prod) y [`terraform/envs/dev/`](https://github.com/SumzCol/credits-platform/tree/main/terraform/envs/dev)
- Workflows: [`.github/workflows/`](https://github.com/SumzCol/credits-platform/tree/main/.github/workflows)

Para estado de recursos, DNS, certificados, IAM y secretos, consultar directamente las consolas y herramientas operativas de los entornos correspondientes.
