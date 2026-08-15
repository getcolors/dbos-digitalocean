# dbos-digitalocean

Desired state for `https://bigconfig.space`: DBOS SDK 4.25.14 embedded in a
TypeScript reference API, with PostgreSQL 17 colocated on one Amsterdam
DigitalOcean Droplet. OpenTofu discovers the account's default `ams3` VPC at
runtime; it neither creates a VPC nor configures its UUID. Cloudflare manages
the apex A record and valid public TLS is served through ONCE.

The `s-4vcpu-8gb` size provides 4 shared vCPUs and 8 GiB RAM for PostgreSQL,
Node, Docker/Caddy, image updates, and restart acceptance checks. PostgreSQL and
DBOS internals remain loopback-only. Only restricted SSH and public HTTP/HTTPS
are allowed through the DigitalOcean firewall.

```sh
./green build
./green create --dry-run
./green create
.agents/skills/package-dbos-green/scripts/acceptance.sh
```

Credentials are only the `COLORS_PAR_*` variables listed in `colors.yml`,
sourced from ignored `.envrc.private`. Never set `COLORS_PAR_PROFILE`.

The acceptance script proves HTTPS health, workflow completion, the intentional
activity retry, duplicate-ID deduplication, deterministic result retrieval, and
recovery after rebooting the entire Droplet during `DBOS.sleep`.

## Operations and recovery

Use `ssh dbos-digitalocean` for Docker/Caddy status and logs. The application
container stores PostgreSQL under the ONCE `/storage` volume and uploads daily
custom-format dumps to the configured R2 prefix. Inspect R2 and
`/storage/.last-backup`; periodically restore a selected dump into an isolated
replacement deployment with `pg_restore`, then verify workflow rows and API
results before any DNS cutover.

Upgrade only by changing exact versions in `colors.yml` and the upstream
package, reviewing DBOS workflow patching/versioning guidance, rebuilding the
image, and testing pending workflow recovery. PostgreSQL major upgrades require
a tested dump/restore or `pg_upgrade` rollback procedure.

This is not highly available. Droplet, disk, PostgreSQL, or region failure causes
downtime; R2 restore is operator-driven and recovery point is bounded by backup
frequency. Keep `compute-prevent-destroy: true`. Never delete the default VPC,
pre-existing Cloudflare zone, state bucket, backups, or unrelated records.
