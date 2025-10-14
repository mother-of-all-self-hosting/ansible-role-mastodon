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

# Setting up Mastodon

This is an [Ansible](https://www.ansible.com/) role which installs [Mastodon](https://docs.mastodon.dev/) to run as [Docker](https://www.docker.com/) containers wrapped in systemd services.

Mastodon is a self-hosted collaborative online markdown editor.

See the project's [documentation](https://docs.mastodon.dev/) to learn what Mastodon does and why it might be useful to you.

>[!WARNING]
> Mastodon is currently alpha. Alpha releases come with no guarantees regarding upgradeability. It is very likely that you will need to wipe the database between alpha releases. As there is currently no migration path from Mastodon 1, it is recommended to set up a separate instance to test Mastodon.

## Adjusting the playbook configuration

To enable Mastodon with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# mastodon                                                             #
#                                                                      #
########################################################################

mastodon_enabled: true

########################################################################
#                                                                      #
# /mastodon                                                            #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Mastodon you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
mastodon_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your sidekiq.

**Note**: hosting Mastodon under a subpath (by configuring the `mastodon_path_prefix` variable) does not seem to be possible due to Mastodon's technical limitations.

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `mastodon_environment_variables_additional_variables` variable

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Mastodon instance becomes available at `https://example.com`.

To get started, open the frontend's URL with a web browser, and register the account.

Since account registration is disabled by default, you need to enable it first by setting `mastodon_sidekiq_environment_variables_hd_auth_local_enable_register` to `false` temporarily in order to create your own account.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the sidekiq with SSH, and running `journalctl -fu mastodon-sidekiq` (or how you/your playbook named the service, e.g. `mash-mastodon-sidekiq`) for the sidekiq and `journalctl -fu mastodon-frontend` (or how you/your playbook named the service, e.g. `mash-mastodon-frontend`) for the frontend, respectively.
