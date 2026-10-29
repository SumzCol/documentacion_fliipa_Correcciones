Infraestructura

| Documento | Infraestructura |
|---|---|
| **Proyecto** | Fliipa |
| **Estado** | Documentación técnica basada en la configuración versionada; validar el estado operativo directamente en GCP |
| **Responsable** | Tecnología / Infraestructura |

---

## Objetivo

Documentar la infraestructura técnica y la operación de Fliipa: proveedor cloud y entornos, pipeline de despliegue, configuración y secretos, jobs batch y migraciones, dominios públicos y operación diaria.

## Alcance

Cubre los aspectos técnicos de infraestructura y operación de la plataforma. No reemplaza las reglas de negocio ni las especificaciones funcionales. Para la arquitectura de software, consultar [architecture/](../architecture/); para documentación de producto y flujos, consultar [product/](../product/).

La documentación describe principalmente la configuración versionada en el repositorio. No confirma por sí sola el estado actual de recursos, permisos, DNS, certificados o secretos en vivo; esos elementos deben verificarse en GCP y en los sistemas externos antes de operaciones.

## Documentos

| # | Documento | Contenido |
|---|---|---|
| 1 | [Arquitectura y entornos](./README.md) | Proveedor GCP, aislamiento PROD y STG/DEV, flota, entrada pública y mapa código-cloud. |
| 2 | [Pipeline de despliegue](./deployment-pipeline.md) | Workflows vigentes, despliegues manuales, selección de servicios, migraciones y rollback. |
| 3 | [Configuración, variables de entorno y secretos](./environment-configuration.md) | Configuración declarativa, URLs, variables literales, secretos, Firebase domains e IAM. |
| 4 | [Jobs batch y migraciones de base de datos](./jobs-and-migrations.md) | Handlers batch registrados, schedulers PROD, aprovisionamiento y migraciones TypeORM. |
| 5 | [Dominios públicos y Load Balancer](./domain-lb.md) | Firebase Hosting activo, hosts y rutas por entorno; estado del Load Balancer como alternativa futura. |
| 6 | [Operación diaria, rollback y pendientes](./day-to-day.md) | Comandos operativos, pause/resume, secretos, recuperación y riesgos por validar. |
| 7 | [Staging / demo](./dev-staging.md) | Particularidades de DEV, límites de capacidad y operaciones del entorno. |
| 8 | [Checklist de release](./ship-checklist.md) | Decisión de qué construir y desplegar según los archivos modificados. |
| 9 | [Acceso a APIs internas](./internal-api-access.md) | Acceso IAM, autenticación servicio-a-servicio y límites de exposición. |
| 10 | [Observabilidad desde admin](./observabilidad-admin.md) | Funciones y configuración de observabilidad del panel administrativo. |

## Documentos relacionados

- [Índice general de documentación](../README.md)
- [Arquitectura de software](../architecture/README.md)
- [Documentación de producto](../product/)
- [Runbook de deploy PROD](../PROD-DEPLOY.md)
- [Catálogo de configuración de deploy](../../config/deploy/README.md)
- [Terraform / OpenTofu](../../terraform/README.md)

## Fuentes de verdad

Si un documento y la configuración del repositorio discrepan, verificar primero el código y estos archivos:

- Proyectos, servicios, capacidad y dominios: [`config/deploy/`](../../config/deploy/)
- Infraestructura declarativa: [`terraform/envs/prod/`](../../terraform/envs/prod/) y [`terraform/envs/dev/`](../../terraform/envs/dev/)
- Workflows: [`.github/workflows/`](../../.github/workflows/)

Para estado de recursos, DNS, certificados, IAM y secretos, consultar directamente las consolas y herramientas operativas de los entornos correspondientes.