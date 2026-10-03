# STRUCTURE.md

Compact map for agents working on an NS8 module.

## Root

- `README.md`: install/configure/use/test documentation. Keep examples valid.
- `build-images.sh`: builds module image and service images; owns NS8 image labels.
- `imageroot/`: files copied into the installed module environment.
- `ui/`: administrator UI source for cluster-admin.
- `tests/`: Robot Framework module lifecycle tests.
- `.github/`: CI/release workflows.
- `.devcontainer/`: optional UI/dev tooling.

## `imageroot/`

- `actions/<action>/`: executable action steps and JSON schemas.
- `events/<event>/`: executable event handlers.
- `systemd/user/*.service`: Systemd user units for rootless services.
- `bin/`: helper commands available in the agent environment.
- `etc/state-include.conf`: backup includes for state and volumes.
- `etc/state-exclude.conf`: backup excludes.

## Required/typical actions

- `configure-module`: validate input, persist config, render config, setup routes, start/reload units.
- `get-configuration`: return current config for admin UI.
- `get-status`: inherited from core unless extended.
- `destroy-module`: cleanup routes/firewall/system state.
- `restore-module`: extend after core restore when Redis keys or app-specific restore steps are needed.

## State files

- `state/environment`: non-secret agent env, managed by `agent.set_env()`.
- `state/secrets.env`: preferred generic secret env file for new modules.
- `state/*.env`: service-specific env files; document each one.
- `state/<service config>`: generated runtime config used by containers.

## Standard data flow

`UI/API -> action stdin JSON -> validation -> agent env/secret files/config render -> systemd user unit -> Podman container -> Traefik/firewall/service discovery`
