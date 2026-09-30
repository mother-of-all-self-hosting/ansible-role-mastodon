<!--
SPDX-FileCopyrightText: 2018-2025 Slavi Pantaleev
SPDX-FileCopyrightText: 2019-2022 Aaron Raimist
SPDX-FileCopyrightText: 2019-2023 MDAD project contributors
SPDX-FileCopyrightText: 2023 QEDeD
SPDX-FileCopyrightText: 2024 Fabio Bonelli
SPDX-FileCopyrightText: 2024 Nikita Chernyi
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Molecule Testing

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

## Prerequisites

To utilize Molecule you need to prepare several requirements:

- **x86** computer running one of these operating systems that make use of [systemd](https://systemd.io/):
  - **Archlinux**
  - **CentOS**, **Rocky Linux**, **AlmaLinux**, or possibly other RHEL alternatives (although your mileage may vary)
  - **Debian** (10/Buster or newer)
  - **Ubuntu** (18.04 or newer, although [20.04 may be problematic](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/ansible.md#supported-ansible-versions) if you run the Ansible playbook on it)
- `root` access on the computer which Molecule runs against
- [Ansible](http://ansible.com/) program
- [Python](https://www.python.org/)
  - Most distributions install Python by default, but some don't (e.g. Ubuntu 18.04) and require manual installation (something like `apt-get install python3`)
- [Docker](https://www.docker.com)
  - Access to Docker UNIX socket (`/var/run/docker.sock`) is required by default

## Installation

To set up the environment for using Molecule, run the command below on the terminal:

```bash
python3 -m venv ./molecule/venv
source ./molecule/venv/bin/activate
pip3 install -r ./molecule/requirements.txt
```

## Scenarios

Currently there is one testing scenario available.

### `default`

Tests a standard Mastodon installation — the Sidekiq, web and streaming services — against a Postgres installed by [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres) and a Valkey installed by [ansible-role-valkey](https://github.com/mother-of-all-self-hosting/ansible-role-valkey), both reached over Unix sockets.

The verification does not stop at "the systemd services are active". It:

- waits for Mastodon's own `/health` and the streaming server's `/api/v1/streaming/health`, rather than settling for the units being `active`. Reaching `/health` also means the schema is migrated, since the web container runs `db:migrate` before starting Puma (`mastodon_web_migration_enabled`)
- asserts that the API and the image tag of every container report the version `mastodon_version` pins in `defaults/main.yml`, and that Mastodon reports the domain `mastodon_hostname` gave it
- checks that `templates/env.j2` reached the environment of all three containers
- asks Mastodon from inside itself which database it migrated, whether any migrations are still pending, whether a Sidekiq process is running, and which Redis database Sidekiq is connected to — which must be the non-default `mastodon_redis_database` the scenario configures
- creates an owner account with the same `tootctl accounts create` invocation the role's user tasks use, looks it up over the API, and asks for an account that was never created
- posts a status, waits for Sidekiq to fan it out onto its author's home feed in Redis, and reads it back over the API — which is evidence that Sidekiq is consuming jobs, not merely connected
- watches the services for long enough to catch a crash loop hiding behind `active`

The scenario deliberately uses a database name and a Redis database number that are not the role's defaults, so that a configuration which never reached the processes shows up as a failure rather than as a pass.

The user and secret-generation tasks in `tasks/main.yml` are tagged `molecule-notest`: the former runs mid-role, before the services it needs are started, and the latter only print values for an operator to copy. The scenario supplies fixed throwaway secrets instead.

## Running

By default it is configured to run the scenarios on Ubuntu 26.04.

```bash
molecule test --scenario-name default
```

You can utilize other distributions by setting one to the `MOLECULE_DISTRO` environment variable:

```bash
# Ubuntu 24.04
MOLECULE_DISTRO=ubuntu2404 molecule test --scenario-name default

# Debian 13
MOLECULE_DISTRO=debian13 molecule test --scenario-name default

# Debian 12
MOLECULE_DISTRO=debian12 molecule test --scenario-name default
```
