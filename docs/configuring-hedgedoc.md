<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Thomas Miceli
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up HedgeDoc

This is an [Ansible](https://www.ansible.com/) role which installs [HedgeDoc 2](https://docs.hedgedoc.dev/) to run as [Docker](https://www.docker.com/) containers wrapped in systemd services.

HedgeDoc 2 is a self-hosted collaborative online markdown editor.

See the project's [documentation](https://docs.hedgedoc.dev/) to learn what HedgeDoc 2 does and why it might be useful to you.

>[!WARNING]
> HedgeDoc 2 is currently alpha. Alpha releases come with no guarantees regarding upgradeability. It is very likely that you will need to wipe the database between alpha releases. As there is currently no migration path from HedgeDoc 1, it is recommended to set up a separate instance to test HedgeDoc 2.

## Adjusting the playbook configuration

To enable HedgeDoc 2 with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# hedgedoc                                                             #
#                                                                      #
########################################################################

hedgedoc_enabled: true

########################################################################
#                                                                      #
# /hedgedoc                                                            #
#                                                                      #
########################################################################
```

### Set the hostname

To enable HedgeDoc 2 you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
hedgedoc_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your backend.

**Note**: hosting HedgeDoc 2 under a subpath (by configuring the `hedgedoc_path_prefix` variable) does not seem to be possible due to HedgeDoc's technical limitations.

### Set a random string for encryption key

You also need to specify a random string to encrypt session data. To do so, add the following configuration to your `vars.yml` file. The value can be generated with `pwgen -s 64 1` or in another way.

```yaml
hedgedoc_backend_environment_variables_hd_session_secret: YOUR_SECRET_KEY_HERE
```

### Specify database

It is necessary to select database used by HedgeDoc 2 from MariaDB and Postgres. Currently SQLite is not supported by this role.

To use Postgres, add the following configuration to your `vars.yml` file:

```yaml
hedgedoc_database_type: postgres
```

Set `mariadb` to use MariaDB.

For other settings, check variables such as `hedgedoc_database_*` on [`defaults/main.yml`](../defaults/main.yml).

### Enabling signing up

By default this role is configured to disable signing up for an account on the service. To enable it, add the following configuration to your `vars.yml` file:

```yaml
hedgedoc_backend_environment_variables_hd_auth_local_enable_register: true
```

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `hedgedoc_environment_variables_additional_variables` variable

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, HedgeDoc instance becomes available at `https://example.com`.

To get started, open the frontend's URL with a web browser, and register the account.

Since account registration is disabled by default, you need to enable it first by setting `hedgedoc_backend_environment_variables_hd_auth_local_enable_register` to `false` temporarily in order to create your own account.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the backend with SSH, and running `journalctl -fu hedgedoc-backend` (or how you/your playbook named the service, e.g. `mash-hedgedoc-backend`) for the backend and `journalctl -fu hedgedoc-frontend` (or how you/your playbook named the service, e.g. `mash-hedgedoc-frontend`) for the frontend, respectively.
