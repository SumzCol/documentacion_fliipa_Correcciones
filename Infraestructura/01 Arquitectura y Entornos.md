# Arquitectura y entornos

## Proveedor y proyectos GCP

Toda la infraestructura corre en **Google Cloud Platform**, con dos proyectos completamente aislados entre sí (sin compartir Cloud SQL, Secret Manager, Artifact Registry ni service accounts):

| | **PROD** | **STG / demo** |
|---|---|---|
| Proyecto GCP | `product-fintech` | `product-fintech-dev` |
| Región | `us-central1` | `us-central1` |
| Prefijo de recursos | `creds-*` | `creds-*-dev` |
| Variable `ENVIRONMENT` de la app | `prod` | `staging` |
| Matriz de servicios | `config/deploy/prod-service-matrix.json` | `config/deploy/dev-service-matrix.json` |
| Secret GitHub (SA deployer) | `GCP_SA_KEY` | `GCP_SA_KEY_DEV` |
| Instancia Cloud SQL | `creds-prod-pg` | `creds-dev-pg` (`db-f1-micro`) |

Los scripts fijan el contexto activo (`pin_prod_deploy_context` / `pin_dev_deploy_context`) y verifican que la matriz coincida con el `projectId` del entorno activo antes de tocar Cloud Run, evitando cruzar recursos de un proyecto con el otro.

## Componentes cloud (PROD)

```text
product-fintech
├── Cloud SQL          creds-prod-pg (5 bases de datos lógicas)
├── Artifact Registry  creds-platform-repo
├── Secret Manager     DB, JWT, webhooks, communications, Slack ops, registro de URLs, …
├── Cloud Run Job      creds-migrate-prod        (migraciones TypeORM)
├── Cloud Run Job      creds-jobs-prod           (+ Cloud Scheduler por JOB_NAME)
└── Cloud Run (~9 servicios) *.run.app
    ├── acceso público   webhooks, b2b, admin-back, checkout, redemption, admin
    └── acceso IAM       rules-engine, core (backends/core), communications
```

## Flota de servicios (matriz PROD)

`config/deploy/prod-service-matrix.json` define, por servicio: directorio del paquete, imagen de Artifact Registry, nombre del servicio Cloud Run, tipo (`node` / `next`), *tier* de capacidad, service account de runtime, tipo de acceso, ruta de health check y base de datos lógica asociada. El campo `deployOrder` fija el orden de despliegue de la flota:

| Slug | Paquete | Tipo | Tier | Acceso | Service Account runtime | Base de datos |
|---|---|---|---|---|---|---|
| `rules-engine` | `services/rules-engine` | node | B | IAM | `creds-runtime-product` | `rules-engine` |
| `core` | `backends/core` | node | A | IAM | `creds-runtime-core` | `fliipa_banking_core` |
| `communications` | `services/communications` | node | C | IAM | `creds-runtime-product` | `communications` |
| `webhooks` | `services/webhooks` | node | B | público | `creds-runtime-public` | — |
| `b2b` | `backends/b2b` | node | A | público | `creds-runtime-public` | `fliipa_b2b` (incluye `product_settings`) |
| `admin-back` | `backends/admin` | node | B | público | `creds-runtime-public` | `fliipa_banking_core` (settings de crédito/core; settings de producto vía S2S a b2b) |
| `checkout` | `apps/checkout` | Next.js | A | público | `creds-runtime-public` | — |
| `redemption` | `apps/redemption` | Next.js | A | público | `creds-runtime-public` | — |
| `admin` | `apps/admin` | Next.js | B | público | `creds-runtime-public` | — |

Los servicios con acceso **IAM** (`rules-engine`, `core`, `communications`) no son invocables públicamente: solo aceptan llamadas de otros servicios cuya service account esté en su lista `invokers`, usando `authenticatedFetch` (ID token) desde `packages/kernel`. Los servicios **públicos** exponen su health check bajo `*.run.app`.

Los *tiers* de capacidad (`A`, `B`, `C`) fijan `minInstances`/`maxInstances`/`concurrency`/`cpu`/`memory`/`timeout` en `prod-service-matrix.json`; se aplican en cada `pdeploy` / workflow y pueden sobreescribirse por servicio (por ejemplo `admin-back` fuerza `minInstances: 0`).

En STG (`dev-service-matrix.json`) todos los servicios usan el tier `cheap` (min `0`, max `2`, cpu `1`, memoria `512Mi`, *scale-to-zero*) y Cloud SQL es `db-f1-micro` (~25 conexiones), por lo que los *pool sizes* de base de datos deben mantenerse bajos (`DB_POOL_MAX=2–3`) para no agotar conexiones.

## Mapa código ↔ cloud

```text
config/deploy/
  gcp-prod.env                 PROJECT_ID, REGION, ARTIFACT_REPO
  domains.prod.env             switch de dominio (hoy DEPLOY_PHASE=1)
  prod-service-matrix.json     flota Cloud Run (orden, tiers, health, SA)
  service-catalog.json         slug → packageDir / cloudRun
  url-bindings.json            variables de URL entre servicios
  runtime-secrets.json         catálogo Secret Manager → mounts

scripts/
  pdeploy-prod.sh              deploy manual / scaffold
  prod-runtime-secrets.sh      put | verify secretos
  prod-urls.sh                 sync → apply → verify
  pause|resume-prod-cloud-run  costo diario
  run-prod-seeds.sh            seeds de admin + settings de core (kernel);
                               seeds de producto (`product_settings`) viven en b2b
  provision-prod-jobs.sh       Cloud Run Job creds-jobs-prod
  provision-prod-jobs-schedulers.sh  Cloud Scheduler → JOB_NAME

.github/workflows/creds-*-prod.yml     mismo resultado que pdeploy, por servicio
.github/workflows/creds-jobs-prod.yml  job batch + schedulers

packages/kernel/.../cloud-run-fetch.ts  ID token para llamadas servicio-a-servicio
apps/*/next.config.*                   basePath /checkout | /redemption | /admin
```

Runtime service accounts: `creds-runtime-{public,core,hub,product}@…`. GitHub usa el secret `GCP_SA_KEY` (SA `creds-deployer@product-fintech`) para PROD y `GCP_SA_KEY_DEV` para STG (SA deployer en `product-fintech-dev`).

## Qué falta (estado a la fecha de esta revisión)

| Pieza | ¿Bloquea las URLs `*.run.app` actuales? |
|---|---|
| Custodia de DNS + cutover de Webflow | No |
| Load Balancer + NEGs + certificados + DNS → IP del LB | No (sí es necesario para `fliipa.com`) |
| `DEPLOY_PHASE=F` + rebuild de frontends | No |
| Secretos reales de Experian / Score | No (hoy son *placeholders*) |
| Registro de webhooks de terceros con el dominio final | No |

Ver detalle en [Dominio y Load Balancer](05 Dominio y Load Balancer.md) y en [Operación y pendientes](06 Operacion y Pendientes.md).
