# AGENTS.md

This repository is an NS8 module. Follow official NS8 docs first, then local repo conventions, then inferred patterns from maintained NS8 modules.

## Source order

1. Local files: `README.md`, `build-images.sh`, `imageroot/`, `ui/`, `tests/`, `.github/`.
2. NS8 developer manual: modules, agents/actions, images, validation, user domains, firewall, proxy/certificates, backup/restore, release process.
3. Current maintained modules: `ns8-core`, `ns8-mail`, `ns8-prometheus`, `ns8-traefik`.

## Resource map
- NethServer 8 administrator manual: https://docs.nethserver.org/projects/ns8
- NethServer 8 developer manual: https://nethserver.github.io/ns8-core/
- NethServer 8 issue tracker: https://github.com/NethServer/dev
- NethServer 8 core source code: https://github.com/NethServer/ns8-core

## Non-negotiable NS8 model

- A module is a deployable NS8 application unit, normally rootless, made of one or more Podman containers managed by Systemd.
- Keep module automation in `imageroot/actions/`.
- Keep long-running services in `imageroot/systemd/user/*.service`.
- Keep administrator UI in `ui/`; end-user HTTP UI/API must be exposed through Traefik routes.
- Prefer rootless. Use rootfull only when the module truly needs host-level privileges.
- Do not add a separate orchestrator, daemon supervisor, or custom admin backend unless explicitly required.

## Actions

- An action is a directory of executable steps. Steps run alphabetically; use numeric prefixes to make order explicit.
- Keep steps small and single-purpose: validation, env write, config render, route setup, service restart, cleanup.
- Use `configure-module` for administrator-supplied configuration.
- Use `get-configuration` to return current config in a shape equal/similar to `configure-module` input.
- Use `destroy-module` for cleanup: routes, firewall services, generated host/system state.
- Make actions idempotent. Re-running `configure-module` must converge, not duplicate state.
- Validate public action input/output with `validate-input.json` / `validate-output.json`.
- Put semantic validation in an early step such as `01validation`; on validation failure set `validation-failed` and exit non-zero.
- Read action input from stdin JSON. Write machine-readable result to stdout. Write diagnostics to stderr.

## Environment and secrets

- Persist non-secret config with the NS8 `agent` library, usually `agent.set_env()`.
- Do not edit `state/environment` manually from action code unless there is no agent API for the operation.
- Do not store secrets in `state/environment`.
- New modules should use `secrets.env` for generic secrets:
  - read with `agent.read_envfile("secrets.env")`
  - write with `agent.write_envfile("secrets.env", values)`
- Use service-specific secret env files only when separation is useful, and document them.
- Create secret files with restrictive permissions; never print secrets to stdout/stderr/logs.
- Name sensitive action input keys with suffixes `password`, `secret`, or `token` so NS8 task-context redaction applies.
- Pass env to containers through Systemd `EnvironmentFile=%S/state/environment` and Podman `--env-file`, not ad hoc shell exports.

## Systemd and containers

- Systemd units own container lifecycle.
- Use `%S/state` as `WorkingDirectory`.
- Use `Environment=PODMAN_SYSTEMD_UNIT=%n`.
- Use `--replace --name=%N`, `--cidfile`, `--conmon-pidfile`, `--cgroups=no-conmon`.
- Bind HTTP backends to `127.0.0.1:${TCP_PORT}` unless the service must be directly reachable.
- Use `ExecStop` and `ExecStopPost` cleanup.
- Use `ExecReload` when the service supports reload.
- Do not start long-running containers directly from actions except through `systemctl --user`.

## Traefik and HTTP

- Request the minimum Traefik authorization in `build-images.sh`, usually `traefik@node:routeadm` or `traefik@any:routeadm`.
- Use `agent.set_route()` or the Traefik `set-route` action.
- Route `instance` names must be stable and unique. Use `${MODULE_ID}` as prefix/suffix for multiple routes.
- Use `configure-module` for routes depending on user input. `create-module` is acceptable only for generated/internal routes.
- For public FQDN routes, prefer `lets_encrypt: true` and `http2https: true`.
- Backend services should normally be plaintext HTTP behind Traefik TLS termination.
- Remove module-owned routes on destroy when core automatic cleanup is not enough.

## User domains

- If the module consumes LDAP users/groups, add `cluster:accountconsumer`.
- Discover domains with `agent.ldapproxy`; do not hard-code LDAP host/port.
- Bind selected domains with `agent.bind_user_domains([...])`.
- Store LDAP bind passwords in `secrets.env` or a documented service secret env file.
- Listen for `user-domain-changed` or `module-domain-changed` when LDAP settings affect running services.
- Use NS8 LDAP helper filters when listing users/groups so hidden objects stay hidden.

## Service discovery and cross-module calls

- Publish service endpoints with `srv` Redis keys only when this module is a provider.
- Raise documented service-changed events when provider data changes.
- Consume services with `agent.list_service_providers()` or `list-service-providers`.
- Module-to-module task calls must go through the API server / `agent.tasks`, with explicit roles.
- Request minimal authorizations in image labels. Do not use broad roles for convenience.

## Firewall

- Prefer Traefik for HTTP/HTTPS.
- For direct public TCP/UDP services, request `node:fwadm`.
- Add public services with `agent.add_public_service()` and remove them in `destroy-module`.
- Do not rely on unsupported rootfull DNAT/private-network patterns.

## Backup and restore

- `state/environment` is included automatically.
- Put persistent data in Podman volumes where possible.
- Add `imageroot/etc/state-include.conf` for required state/volumes.
- Dump Redis keys with `module-dump-state` and clean temporary dumps with `module-cleanup-state`.
- Restore Redis keys in `restore-module` steps after the core restore step.
- Do not include volatile caches, build outputs, or downloaded artifacts unless needed for restore.

## UI

- Module UI is for administrators inside cluster-admin.
- Keep standard pages where useful: Status, Settings, About.
- UI must call NS8 actions/API; avoid adding a bespoke backend just for configuration.
- Update UI labels/translations when changing user-visible fields.

## Tests

- Keep or extend Robot Framework lifecycle tests.
- Cover at least install, configure, status, route reachability, reconfigure, restart, uninstall.
- Add tests for validation failures and secret non-disclosure when relevant.
- Run the local test wrapper before declaring a change complete.
- Update tests with behavior changes; do not weaken tests to match broken behavior.

## Commits

- Review `git status --short` and relevant diffs before staging.
- Stage only intended files.
- Use Conventional Commit format: `<type>(<scope>): <Subject>`.
- Subject: imperative, capitalized, <= 50 chars, no final period.
- Common types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `ci`, `build`, `perf`.
- Add a body only when it clarifies what changed and why.
- Do not push, amend, rebase, or rewrite history unless explicitly requested.

## Agent workflow

- Before editing, identify affected actions, units, labels, UI, tests, and docs.
- Prefer small, reviewable changes.
- Preserve NS8 conventions over generic container-app habits.
- When behavior is unclear, inspect upstream docs/code and record the assumption in the change summary.
- Never invent NS8 standards. Mark unsupported or unclear rules as gaps.
