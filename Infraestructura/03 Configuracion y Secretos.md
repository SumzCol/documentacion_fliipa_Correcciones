# Configuración, variables de entorno y secretos

Toda la configuración de despliegue es declarativa y vive en `config/deploy/`, versionada en el repositorio (salvo valores sensibles, que residen en Secret Manager o en archivos locales *gitignored*).

## Archivos de configuración

| Archivo | Contenido |
|---|---|
| `gcp-prod.env` / `gcp-dev.env` | `PROJECT_ID`, `REGION`, `ARTIFACT_REPO` por entorno |
| `domains.prod.env` / `domains.dev.env` | `DEPLOY_PHASE` y orígenes públicos (único punto de cambio de dominio) |
| `prod-service-matrix.json` / `dev-service-matrix.json` | flota Cloud Run: orden de despliegue, *tiers* de capacidad, health checks, service accounts |
| `service-catalog.json` / `service-catalog.dev.json` | mapeo `slug` → `packageDir` / nombre del servicio Cloud Run |
| `url-bindings.json` | qué variable de entorno de cada servicio recibe la URL de qué otro servicio |
| `runtime-secrets.json` | catálogo de secretos de Secret Manager: nombre del secreto, variable de entorno destino y qué servicios lo montan |
| `iam-deploy-engineer-role.yaml` | permisos versionados del rol IAM custom `deployEngineer` |
| `runtime-secrets.local.env.example` | plantilla de valores locales (el archivo real, `runtime-secrets.local.env`, está *gitignored*) |

## Tipos de variables y cómo llegan a Cloud Run

| Tipo | Fuente | Mecanismo de entrega |
|---|---|---|
| Secretos | `runtime-secrets.json` + `runtime-secrets.local.env` (o `.dev.local.env` en STG) | *mount* desde Secret Manager |
| Literales | `serviceEnvLiterals` (+ `serviceEnvLiteralsDev` en STG) | `--update-env-vars` en el deploy |
| URLs servicio-a-servicio | `url-bindings.json` | `prod_urls.py apply` (registro de URLs) |
| Build-time (Next.js) | `staticBuildArgs` de la matriz + bindings marcados `buildTime` | *build-arg* de Docker |

Cualquier variable nueva declarada en `.env.example` de un paquete debe quedar cableada en secretos, en literales, en `url-bindings.json`, en `staticBuildArgs`, o explícitamente exceptuada; esto se valida con un *gate* automático:

```bash
python3 scripts/lib/verify_deploy_env_coverage.py --env dev
```

Sincronización ordenada (cobertura → secretos+literales → URLs → verificación en vivo):

```bash
pnpm env:sync:dev
pnpm env:sync:dev -- verify
bash scripts/sync-deploy-env.sh
```

## Secretos (Secret Manager)

`runtime-secrets.json` enumera cada secreto con su `secretName` en Secret Manager, la variable de entorno con la que se monta y la lista de servicios que lo reciben. Ejemplos verificados en el catálogo: `creds-jwt-secret` → `JWT_SECRET` (`b2b`, `admin-back`); `creds-impersonate-internal-secret` → `IMPERSONATE_INTERNAL_SECRET` (`admin-back`, `b2b`, `webhooks`); `creds-webhook-shared-secret` → `WEBHOOK_SHARED_SECRET` (`webhooks`); `creds-truora-api-key`, `creds-zenvia-api-key`, `creds-sendgrid-api-key` (`communications`); y un grupo marcado como **no obligatorio** (`required: false`) para credenciales de Experian y Score (`EXPERIAN_AUTH_CLIENT_SECRET`, `EXPERIAN_AUTH_PASSWORD`, `SCORE_AUTH_CLIENT_SECRET`, `SCORE_AUTH_PASSWORD`, `SCORE_PASSWORD`, certificados/llaves HPJ de Experian, todos en `b2b`) — hoy con valores *placeholder* pendientes de reemplazo antes de operar en vivo con esos proveedores (ver [Operación y pendientes](06 Operacion y Pendientes.md)).

Operación de secretos en PROD:

```bash
bash scripts/prod-runtime-secrets.sh put \
  --secret-name creds-foo \
  --env-var FOO \
  --value '…' \
  --services b2b,admin-back

bash scripts/prod-runtime-secrets.sh verify
```

Los valores viven únicamente en Secret Manager; no se requiere `runtime-secrets.local.env` para desplegar en PROD (ese archivo aplica a la siembra local/STG).

## Plantillas de correo y WhatsApp (no son secretos)

Los IDs de plantilla de **Sendgrid** y **Zenvia** no van en Secret Manager ni como literales de Cloud Run. Viven en catálogos versionados en el código de `services/communications`:

| Canal | Catálogo | Ejemplos de keys |
|---|---|---|
| Email (Sendgrid) | `src/constants/email-templates.ts` | `welcome`, `otp`, `contract`, `requestRejected` |
| WhatsApp (Zenvia) | `src/constants/whatsapp-templates.ts` | `otp`, `signatureOtp`, `creditApproved`, `contractSigned` |

En runtime, B2B (u otro caller) envía la **key** estable; communications resuelve el ID del proveedor. Lo que sí permanece en secretos/config de infra es la credencial (`SENDGRID_API_KEY`, `ZENVIA_API_KEY`) y el número de origen de Zenvia (`ZENVIA_PHONE_NUMBER`, literal).

> **Nota (sep 2026):** los OTP de WhatsApp dejaron de leer `OTP_WHATSAPP_TEMPLATE_ID` / `SIGNATURE_OTP_WHATSAPP_TEMPLATE_ID` desde env; esos IDs pasaron al catálogo `whatsapp-templates.ts`. Las notificaciones de crédito aprobado (`creditApproved`) y contrato firmado (`contractSigned`) también viven en ese catálogo (sin variables de entorno).

## URLs entre servicios

`url-bindings.json` define, por `packageDir`, qué variable de entorno recibe la URL de qué otro servicio (`from: "services.<slug>"`) o de un valor calculado (`from: "computed.<nombre>"`), con un `suffix` opcional de *path*. Ejemplos: `backends/b2b` recibe `CORE_BASE_URL`, `COMMUNICATIONS_SERVICE_URL` y `RULES_ENGINE_URL`; `apps/checkout` recibe `API_BASE_URL` y, como variable de **build-time** (`buildTime: true`, porque Next.js la *inyecta* en el bundle), `NEXT_PUBLIC_REDEMPTION_BASE_URL`.

## IAM del operador (laptop)

Rol custom de mínimo privilegio para ejecutar despliegues sin usar `Owner`/`Editor`/`IAM Admin`:

```bash
bash scripts/create-deploy-engineer-role.sh                       # crea/actualiza el rol
bash scripts/create-deploy-engineer-role.sh --bind user:you@sumz.co  # lo asigna a un usuario
```

Los permisos están versionados en `config/deploy/iam-deploy-engineer-role.yaml`; si un paso de despliegue falla con `403`, el permiso puntual se agrega ahí y se re-corre el script. Alternativa (más amplia, marcada como *legacy*): `bash scripts/mirror-iam-prod-to-dev.sh`, que copia los roles IAM de un usuario desde PROD hacia STG.

## Salud del sistema (panel admin)

La vista **System → Health** del panel admin llama a `admin-back` (`GET /api/v1/system/core-health`), que prueba la flota interna (backends/servicios/apps) con `authenticatedFetch` (token IAM en STG). Los orígenes vienen de las variables de `admin-back` (local: `dev.sh` + `.env`; STG: `url-bindings.json`). Si falta un origen, ese chequeo se marca **omitido**, no fallido. La lógica de Experian/Score corre dentro de `b2b` (cubierta por el *probe* `b2b-backend`).
