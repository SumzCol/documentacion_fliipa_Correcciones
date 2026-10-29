# Operación diaria, rollback y pendientes

## Contexto de entornos

PROD usa el proyecto GCP `product-fintech`; STG/DEV usa `product-fintech-dev`. Los flujos de GitHub Actions actuales son manuales y usan Workload Identity Federation:

| Entorno | Workflow | CLI desde laptop |
|---|---|---|
| PROD | `PROD - deploy` (`prod-deploy.yml`) | `bash scripts/pdeploy-prod.sh` |
| STG/DEV | `DEV - deploy` (`dev-deploy.yml`) | `bash scripts/deploy-dev.sh` |

Los wrappers fijan el proyecto esperado. No cambies el proyecto activo manualmente para intentar redirigir una operación. Detalle de comandos y entradas: [Pipeline CI/CD](./deployment-pipeline.md), [Checklist de release](./ship-checklist.md) y [Staging / demo](./dev-staging.md).

## Deploy PROD

El pipeline normal recomendado es el workflow `PROD - deploy`. Detecta servicios afectados, ejecuta `pnpm test`, construye las imágenes necesarias y aplica Terraform/OpenTofu. Es dispatch-only; no hay workflow PROD separado por servicio.

```bash
# Cambios desde el último deploy exitoso
gh workflow run prod-deploy.yml -f slugs=auto -f migrate=auto

# Seleccionar servicios explícitamente
gh workflow run prod-deploy.yml -f slugs=b2b,admin-back -f migrate=auto

# Revisar runs
gh run list --workflow=prod-deploy.yml --limit 5
gh run watch
```

La CLI sigue disponible para operación desde laptop y bootstrap:

```bash
bash scripts/pdeploy-prod.sh --all
bash scripts/pdeploy-prod.sh webhooks b2b
bash scripts/pdeploy-prod.sh checkout --force
bash scripts/pdeploy-prod.sh webhooks --dry-run
```

`pdeploy-prod.sh` ejecuta un preflight: si PROD está pausado, sigue la opción de reanudar que presente el script; no omitas este control sin entender el estado del entorno. Para el proceso completo, ver [Deploy PROD](../PROD-DEPLOY.md).

## URLs, secretos y configuración

Después de cambiar URLs públicas o bindings, sincroniza y verifica el registro:

```bash
bash scripts/prod-urls.sh
bash scripts/prod-urls.sh verify
```

Los secretos de runtime se administran en Secret Manager; el catálogo y la distinción entre secretos, literales y URLs están en [Configuración, variables de entorno y secretos](./environment-configuration.md). Ejemplos operativos:

```bash
bash scripts/prod-runtime-secrets.sh plan
bash scripts/prod-runtime-secrets.sh verify
bash scripts/prod-runtime-secrets.sh verify-s2s
```

Para registrar un secreto nuevo, usar `prod-runtime-secrets.sh put` con el nombre del secreto, variable destino y servicios consumidores. Evitar valores sensibles en repositorio o logs. Tras rotar un secreto montado en una revisión de Cloud Run, confirmar que la revisión nueva lo consume; consultar el catálogo para el mecanismo y alcance del mount.

Las credenciales de Experian/Score tienen entradas `required: false` en el catálogo, por lo que su ausencia no necesariamente bloquea un deploy. El repositorio no permite confirmar los valores actuales de Secret Manager ni que la revisión activa de B2B los tenga cargados. Validar ambos puntos con permisos operativos antes de probar esas integraciones; en DEV los literales apuntan a endpoints demo/UAT.

## Capacidad y estado de PROD

La capacidad declarativa por servicio se define en [`prod-service-matrix.json`](../../config/deploy/prod-service-matrix.json) y en la infraestructura del entorno. Los cambios de configuración se aplican mediante el flujo de deploy correspondiente. Los `DB_POOL_MAX` y otros literales están en [`runtime-secrets.json`](../../config/deploy/runtime-secrets.json); no copiar a STG los pools de PROD.

Para revisar o cambiar la capacidad operativa en PROD:

```bash
bash scripts/prod-cloud-run-status.sh
bash scripts/pause-prod-cloud-run.sh
bash scripts/resume-prod-cloud-run.sh
```

**Pausar PROD detiene el servicio:** escala a cero el mínimo de instancias de Cloud Run y configura Cloud SQL con `activation-policy=NEVER`. Ejecutarlo solo en una ventana planificada, con aprobación y sin tráfico que deba atenderse. Reanudar primero habilita Cloud SQL (`ALWAYS`, esperando `RUNNABLE`) y luego restaura los mínimos de Cloud Run desde la matriz. Los valores modificados manualmente pueden volver a alinearse con Terraform en el siguiente deploy.

En STG/DEV, Cloud SQL es `db-f1-micro` (aprox. 25 conexiones); los pools definidos para ese entorno son bajos (en general `DB_POOL_MAX=2–3`). Revisar los literales específicos de cada servicio antes de modificar concurrencia o pools.

## Migraciones, seeds y jobs batch

Las migraciones TypeORM usan el Cloud Run Job `creds-migrate-prod` / `creds-migrate-dev`; los seeds son una operación aparte. En PROD, el workflow crea snapshots SQL antes de migrar; el wrapper de laptop requiere Cloud SQL Auth Proxy iniciado por separado.

```bash
# PROD: requiere proxy activo y acceso a los secretos/DB
bash scripts/run-prod-migrations.sh
bash scripts/run-prod-seeds.sh
```

La detección de migración del workflow acepta `migrate=auto|on|off`. Antes de forzar una migración o ejecutar seeds, confirmar proyecto, bases objetivo y estado de respaldo. Ver [Jobs batch y migraciones](./jobs-and-migrations.md).

La imagen batch PROD se ejecuta en `creds-jobs-prod`; actualmente hay handlers `auto-debit` y `credit-state-ledger`. Sus schedulers están en PROD, no en STG/DEV:

```bash
bash scripts/provision-prod-jobs.sh
bash scripts/provision-prod-jobs-schedulers.sh --dry-run
bash scripts/provision-prod-jobs-schedulers.sh

gcloud run jobs execute creds-jobs-prod \
  --region=us-central1 --project=product-fintech --wait \
  --update-env-vars=JOB_NAME=auto-debit
```

No reconstruir la imagen para un cambio solo de cron; actualizar el scheduler. Para código, publicar la imagen antes de validar ejecución. Ver el runbook de [jobs y migraciones](./jobs-and-migrations.md).

## Rollback

### PROD

No hay un workflow PROD de rollback por servicio ni de restauración de base de datos.

1. **Rollback de revisión:** en Cloud Run, seleccionar una revisión anterior y asignarle el tráfico. Es una acción operativa directa; el siguiente apply de Terraform puede reconciliar el servicio con la imagen declarada en el state.
2. **Redeploy de código conocido:** disparar `PROD - deploy` desde una rama/tag que apunte al commit bueno, indicando explícitamente los slugs a desplegar y `migrate=off` si no se pretende repetir migraciones. El workflow no tiene un input `git_ref`; la referencia se selecciona al iniciar el workflow:

   ```bash
   gh workflow run prod-deploy.yml --ref <tag-o-rama-buena> \
     -f slugs=b2b,admin-back -f migrate=off
   ```

   Revisar la imagen/commit de cada servicio y el estado del workflow antes de ejecutar.
3. **Rollback de base de datos:** una reversión de código no deshace migraciones. Los snapshots SQL previos a migrar están en `gs://product-fintech-db-snapshots`; PROD no tiene workflow de restore. Restaurar requiere un procedimiento manual, coordinado y compatible con el esquema y la aplicación.

### STG/DEV

El workflow [`DEV - rollback`](../../.github/workflows/dev-rollback.yml) revierte un solo servicio a un SHA de imagen existente. Requiere `confirm=ROLLBACK`, `service` y `target`; no cambia el tag `dev-deployed` ni revierte migraciones. Para recuperar datos, el workflow separado [`dev-db-restore.yml`](../../.github/workflows/dev-db-restore.yml) restaura snapshots de DEV.

Ver [Rollback en el pipeline](./deployment-pipeline.md#rollback) y comprobar siempre la compatibilidad entre la versión objetivo y las migraciones ya aplicadas.

## Añadir un servicio

La flota se define en las matrices de deploy, el catálogo de servicios y Terraform; los endpoints públicos además necesitan reglas Firebase Hosting según corresponda. Para un paquete nuevo, `pdeploy-prod.sh --package <dir>` puede generar el scaffold desde laptop, pero no sustituye revisar y versionar las entradas declarativas necesarias ni desplegar la infraestructura por el pipeline vigente.

1. Añadir el paquete y sus metadatos de servicio/matriz, configuración Terraform y health check.
2. Añadir los bindings URL, secretos o literales requeridos.
3. Si será público, configurar y revisar rewrites de Firebase Hosting; no exponer servicios IAM-only.
4. Probar en DEV y desplegar mediante `DEV - deploy`; luego usar `PROD - deploy` con los slugs seleccionados.
5. Sincronizar URLs si cambian y validar health checks y acceso desde callers.

Para cambios pequeños, seguir el [Checklist de release](./ship-checklist.md); para las rutas públicas, ver [Dominios públicos y Load Balancer](./domain-lb.md).

## Pendientes y riesgos operativos

| Tema | Estado según el repositorio | Acción |
|---|---|---|
| Front door PROD | Fase F configurada con `fliipa.com`, `apps.fliipa.com` y `api.fliipa.com` vía Firebase Hosting; no hay HTTPS Load Balancer de GCP | Validar en consola Firebase/DNS el dominio, certificado y rewrites antes de cambios operativos |
| HTTPS Load Balancer | No está provisionado como front door | Es una alternativa futura; no hacer cutover DNS siguiendo un plan antiguo. Ver [Dominio y Load Balancer](./domain-lb.md) |
| Custodia y renovación del dominio | No se puede verificar desde el repositorio; la fecha estimada previa de renovación (~2026-09-25) ya pasó | Confirmar hoy custodio y renovación en registrador/WHOIS |
| Secretos de Experian/Score | Catálogo `required: false`; los valores reales y la revisión activa no se pueden comprobar en Git | Revisar Secret Manager, mount y smoke test de la integración en cada entorno |
| Batch | Dos handlers y dos schedulers definidos en PROD; ninguno provisionado en STG/DEV | Revisar horarios y habilitación del envío de autodebit antes de cambios |
| Restauración DB PROD | No hay workflow automatizado de restore; hay snapshots previos a las migraciones del workflow | Acordar y probar un runbook de recuperación antes de necesitarlo |

La configuración documental no prueba por sí sola el estado en vivo de GCP. Para incidentes, contrastar siempre este runbook con Cloud Run, Cloud SQL, Firebase Hosting, Secret Manager y los logs de la ejecución afectada.