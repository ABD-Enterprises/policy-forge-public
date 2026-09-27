# Policy Forge container image contract

## Image and platform

| Item | Contract |
| --- | --- |
| Image | Combined Policy Forge image with nginx and API under supervisord. |
| Platforms | Release builds are published for `linux/amd64` and `linux/arm64`. |
| User | Runs as the non-root user `appuser`. |
| Serving port | `3000` for nginx. Configure the platform's container or target port as `3000`. |
| Internal API port | `8000` inside the container only. |
| `PORT` | Not honoured. |

nginx serves the UI at `/`, proxies `/api/` to the API with the `/api` prefix stripped, and proxies `/healthz`.

supervisord runs three programs: `engine` (API), `integration-scheduler`, and `nginx`. The scheduler needs CPU between requests: on platforms that throttle idle CPU, for example Cloud Run request-based billing, keep CPU always allocated and at least one instance.

## Health endpoints

All endpoints are under `/api` except `/healthz`.

| Endpoint | Contract |
| --- | --- |
| `/healthz` | Liveness, no database access, never `503`. |
| `/api/live` | Liveness, no database access, never `503`. |
| `/api/ready` | Pings the database and reports `schema_version`, `migrations_current`, and `schema_compatible`. Returns `503` only when the database check raises. It does not return `503` for an out-of-date schema; read `migrations_current`. |
| `/api/ready/deep` | Full health check against the database and expected schema. Returns `503` if any subcheck is unhealthy. Anonymous access is rate-limited. |

## Entrypoint modes

| Mode | Contract |
| --- | --- |
| Serve | Default command. Runs preflight unless `POLICY_FORGE_PREFLIGHT=skip`, then starts supervisord. Preflight refuses a database whose schema is not current, so a new database must be initialised first. |
| `init` | Run the image with the single argument `init` and entrypoint `policy-forge-entrypoint`. It runs `policy-forge db bootstrap-schema --accept-irreversible` (schema plus signed framework-pack catalog) and then `policy-forge bootstrap` (bootstrap admin). Exit codes are `0` on success, `1` on failure, and `2` if given extra arguments. Schema bootstrap takes a PostgreSQL advisory lock, so concurrent init runs serialize. Run it once after the first deploy and after upgrades that change the schema. If `schema_version` was stamped before the file migrations ran, typically `policy-forge bootstrap` run before `db bootstrap-schema`, `bootstrap-schema` refuses with a recovery message. |

Secret file staging applies in both modes. When `POLICY_FORGE_MASTER_KEY_FILE` or `POLICY_FORGE_BOOTSTRAP_ADMIN_PASSWORD_FILE` points at a readable file, the entrypoint copies it to `${TMPDIR:-/tmp}/policy-forge-secrets/<VARIABLE>` with directory mode `0700` and file mode `0600`, and points the variable at the copy. This lets read-only `0444` secret mounts from Key Vault, Secret Manager, or Kubernetes pass the key-file permission check. The original mount is never modified.

## Environment variables

| Area | Variables |
| --- | --- |
| Required database | `DATABASE_URL` (`postgres://` or `postgresql://`). The image already sets `POLICY_FORGE_DATABASE_BACKEND=postgresql` and `POLICY_FORGE_POSTGRES_DATA_PATH_ENABLED=1`; do not unset them. |
| First admin | `POLICY_FORGE_BOOTSTRAP_ADMIN_EMAIL` and `POLICY_FORGE_BOOTSTRAP_ADMIN_PASSWORD_FILE` as a mounted secret file. Production profiles require the email and the password file; the plain `POLICY_FORGE_BOOTSTRAP_ADMIN_PASSWORD` value is legacy and refused on production profiles. Minimum password length follows `POLICY_FORGE_DEPLOYMENT_PROFILE`: default `dod-il4` is 15, `fedramp-moderate` is 14, and `commercial` is 12. |
| Production profile | `POLICY_FORGE_DEPLOYMENT_PATH` (`aws`, `azure`, or `gcp`) selects a customer profile. `POLICY_FORGE_ENVIRONMENT` then defaults to production, which requires `POLICY_FORGE_TRUSTED_HOSTS` as comma-separated Host names. Customer profiles reject `POLICY_FORGE_VAULT_BACKEND=local_secretbox`; use the cloud key service backend and set `POLICY_FORGE_VAULT_DEFAULT_KEK_REF` to the key reference. Secret-classed variables must be supplied as `*_FILE` mounts or secret references, never plaintext values. |
| Networking | `CORS_ORIGINS` is unprefixed; unset means cross-origin browser requests are refused in production. `FORWARDED_ALLOW_IPS` is the uvicorn proxy trust list. The image default is `127.0.0.1,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16`. Set it to include the load balancer's source range if that range is outside the default, and never set it to `*`. |
| Unknown or removed variables | Unknown or removed `POLICY_FORGE_*` variables are fatal at startup. Do not set `POLICY_FORGE_*` names that are not documented. |

## Storage

`POLICY_FORGE_DATA_DIR` defaults to `/var/lib/policy-forge`, is declared as a `VOLUME`, and holds reports, evidence files, and local artifacts. PostgreSQL is the system of record.

In a container on the default path, the data directory must be a mounted volume unless `POLICY_FORGE_ALLOW_EPHEMERAL_DATA=true`, which is for dev/test only and forbidden on customer profiles. A deployment may instead point `POLICY_FORGE_DATA_DIR` at container-local storage if it accepts that artifact files do not survive a restart.

Writable paths needed besides the data directory are `/tmp`, `/run/nginx`, `/run/supervisor`, `/var/lib/nginx`, and `/var/log/nginx`. A read-only root filesystem with these as tmpfs mounts has not yet been validated.

## Database

| Item | Contract |
| --- | --- |
| PostgreSQL | PostgreSQL 16, 17, or 18, tested in CI. |
| Extensions | No extensions are required for core features. `pgvector` is optional and used only by the AI embeddings features. |
| Role privileges | The database role must be able to create tables and indexes in the target database and take advisory locks. Owner of the database is sufficient. Superuser is not required. |

## Verification

Each release attaches docker-save tarballs for the engine, UI, and combined images (`linux/amd64` by default, `-arm64.tar` for `linux/arm64`), SPDX JSON SBOMs, an OpenAPI 3.1 document, and `SHA256SUMS`. The tarballs, the OpenAPI document, and `SHA256SUMS` are signed keyless with cosign (`.sig` and `.pem` next to each file).

Verify with:

```sh
cosign verify-blob --certificate-identity-regexp 'https://github\.com/ABD-Enterprises/policy-forge/\.github/workflows/release-containers\.yml.*' --certificate-oidc-issuer https://token.actions.githubusercontent.com --signature <file>.sig --certificate <file>.pem <file>
```

A public container registry is not yet available; load tarballs with `docker load`.
