# CLAUDE.md

## Repository

Desired state for the DBOS benchmark deployment at `https://bigconfig.space` on
one `s-4vcpu-8gb` DigitalOcean Droplet in `ams3`. Behavior is pinned in the
installed `package-dbos-green` payload and reuses ONCE for host/application
convergence and HTTPS.

Tracked source is non-secret desired state, copied launcher and Package Skill
payload, lockfile, toolchain files and documentation. Never read
`.envrc.private` or `.colors/`. Never set `COLORS_PAR_PROFILE`, weaken
`compute-prevent-destroy`, create/delete a VPC, or expose PostgreSQL.

## Commands

```sh
./green build
./green create --dry-run
./green create
.agents/skills/package-dbos-green/scripts/acceptance.sh
```

Real create and the acceptance reboot are authorized for this benchmark.
Deletion is not authorized by that create authorization and remains guarded.

## Coupling

`green` must byte-match
`.agents/skills/package-dbos-green/green`. The lockfile records the real install.
During development use `DBOS_LIB_ROOT=../dbos`; final runs use the pushed SHA in
both launcher copies.

## Git

Work on the current branch. Do not commit or push unless explicitly authorized.

## Keygen mode and retired keys since the package adopted the standards

`digitalocean-ssh-key-name`, `digitalocean-ssh-private-key`,
`digitalocean-ssh-authorized-keys`, `digitalocean-https-sources` and
`digitalocean-vpc-mode` are accepted and ignored since the package adopted the
SSH Keypair, SSH Config, Compute Name and Compute Provider standards, and can be
removed at the next desired-state edit. With no `digitalocean-ssh-keys` set the
deployment is in keygen mode: the next real `create` generates and owns
`~/.ssh/dbos-digitalocean` and registers a profile-named account key. The same
create writes a managed `Host dbos-digitalocean` block into `~/.ssh/config` and
refuses by design a hand-written stanza of that name — remove or rename it
rather than working around the refusal.
