# Operación diaria, rollback y pendientes

## Operación diaria en PROD

Convención común: `source config/deploy/gcp-prod.env` (los scripts ya lo hacen internamente vía `pin_prod_deploy_context`).

### Pausar / reanudar (control de costos)

```bash
bash scripts/pause-prod-cloud-run.sh
bash scripts/resume-prod-cloud-run.sh
bash scripts/prod-cloud-run-status.sh
```

### Capacidad de Cloud Run

Los valores de `minInstances`/`maxInstances`/`concurrency`/CPU/memoria salen de `prod-service-matrix.json` y se aplican en cada `pdeploy` / workflow (`--min-instances`, etc.); no existe un script separado de "aplicar capacidad". Los *pool sizes* de base de datos (`DB_POOL_MAX`) están en `serviceEnvLiterals` dentro de `runtime-secrets.json`.

### Rollback

1. Cloud Run → seleccionar la revisión anterior → mover el 100% del tráfico a esa revisión, **o**
2. Re-ejecutar el workflow correspondiente indicando `git_ref` = el SHA conocido como bueno.

## Ciclo de vida de un servicio nuevo

1. Código en `apps/`, `backends/`, `gateways/` o `services/`.
2. `bash scripts/pdeploy-prod.sh --package <dir>` (scaffold + deploy).
3. Completar `url-bindings.json` si el servicio consume URLs de otros servicios.
4. `bash scripts/prod-urls.sh`.
5. Commitear el Dockerfile, la entrada de la matriz y el workflow generados.

(Ver detalle completo del pipeline en [Pipeline de Despliegue](02 Pipeline de Despliegue.md).)

## Pendientes identificados (a la fecha de esta revisión)

| Pieza | Estado | ¿Bloquea `*.run.app` hoy? |
|---|---|---|
| Custodia de DNS + *cutover* de Webflow | Pendiente de decisión de negocio | No |
| Load Balancer + NEGs + certificados + DNS → IP del LB | No creado | No (sí para `fliipa.com`, ver [Dominio y Load Balancer](05 Dominio y Load Balancer.md)) |
| `DEPLOY_PHASE=F` + rebuild de frontends | No activado (`DEPLOY_PHASE=1`) | No |
| Secretos reales de Experian / Score | Cargados en Secret Manager de PROD; literales de producción en `serviceEnvLiterals.b2b` y demo/UAT en `serviceEnvLiteralsDev.b2b` | No. La revisión de `b2b` los usa al redesplegar |
| Registro de webhooks de terceros con el dominio final | Pendiente hasta el *cutover* de dominio | No |
| Jobs batch reales | Solo existe el job de referencia `example-ping`, con dos schedulers de ejemplo | No |

## Preguntas abiertas

- ¿Quién tiene el panel de gestión de `fliipa.com`?
- ¿Se reemplaza Webflow o se redirige a `/checkout`?
- Experian / Score en PROD ya tiene credenciales reales en Secret Manager y hosts de producción en literales. STG sigue en demo/UAT. Falta redesplegar `b2b` para que la revisión en Cloud Run los monte.
- ¿Qué jobs batch reales (más allá de `example-ping`) se necesitan y con qué periodicidad?

## Riesgos a vigilar

- **Renovación de dominio** `fliipa.com`: alerta WHOIS estimada ~2026-09-25; sin renovación, cualquier plan de *cutover* queda bloqueado.
- **Mezcla de entornos**: los scripts verifican `matrix.projectId == PROJECT_ID` antes de operar, pero cualquier cambio manual de contexto (por ejemplo, exportar `PROJECT_ID` a mano) podría saltarse esa protección; seguir siempre los wrappers (`pdeploy-prod.sh`, `deploy-dev.sh`) en lugar de `gcloud` directo.
- **Capacidad de Cloud SQL en STG**: al ser `db-f1-micro` (~25 conexiones), *pool sizes* copiados de PROD (15–20) agotarían las conexiones disponibles; mantener `DB_POOL_MAX=2–3` en STG.
- **Secretos no obligatorios con `required: false`** (Experian/Score): no bloquean el despliegue. En PROD las credenciales reales ya están en Secret Manager; STG conserva las de demo/UAT. Si se despliega `b2b` sin redesplegar después de rotar un secreto, la revisión vieja sigue con el valor anterior.
