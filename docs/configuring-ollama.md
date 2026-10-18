<!--
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2020-2024 MDAD project contributors
SPDX-FileCopyrightText: 2020-2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Thomas Miceli
SPDX-FileCopyrightText: 2024 noah
SPDX-FileCopyrightText: 2024, 2025 MASH project contributors
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Ollama

This is an [Ansible](https://www.ansible.com/) role which installs [Ollama](https://ollama.com/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Ollama is a server for running large language models on your own hardware.

See the project's [documentation](https://docs.ollama.com/) to learn what Ollama does and why it might be useful to you.

## Adjusting the playbook configuration

To enable Ollama with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# ollama                                                               #
#                                                                      #
########################################################################

ollama_enabled: true

########################################################################
#                                                                      #
# /ollama                                                              #
#                                                                      #
########################################################################
```

### Exposing the instance (optional)

By default, the Ollama instance is not exposed externally, as it is mainly intended to be used in the internal network, connected to other services such as [Open WebUI](https://github.com/mother-of-all-self-hosting/ansible-role-open-webui).

To expose it to the internet, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
ollama_hostname: "example.com"

ollama_container_labels_traefik_enabled: true
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

>[!WARNING]
> Ollama's API has no authentication, so anyone who can reach it can use your models and download new ones. If you expose it, consider protecting it with the basic authentication middleware (see `ollama_container_labels_traefik_middleware_basic_auth_*` on [`defaults/main.yml`](../defaults/main.yml)).

**Note**: hosting Ollama under a subpath (by configuring the `ollama_path_prefix` variable) is not possible due to Ollama's technical limitations.

### Enable hardware acceleration (optional)

By default, Ollama runs models on the CPU. To run them on a GPU, add the following configuration to your `vars.yml` file:

```yaml
# Valid values: cpu, cuda, rocm, vulkan
ollama_hardware_acceleration: cuda
```

Set `cuda` for NVIDIA GPUs, which requires the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) on the server. Set `rocm` for AMD GPUs, and `vulkan` for Intel GPUs and other GPUs with Vulkan support.

As the container does not run as `root`, it also needs to be added to the group which owns the GPU devices on the server for `rocm` and `vulkan`. Otherwise Ollama runs models on the CPU without reporting an error. To do so, add the numeric ID of the group (usually `render`, see `getent group render`) to your `vars.yml` file as below:

```yaml
ollama_container_group_add_custom:
  - 991
```

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `ollama_environment_variables_additional_variables` variable

See [this page](https://docs.ollama.com/faq) for Ollama's config options that you can put in `ollama_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Ollama becomes available internally to other services on the same network at port `11434`. If the service is exposed to the internet, it becomes available at the specified hostname like `https://example.com`.

Ollama does not come with any models. To download one, log in to the server with SSH and run the command below (replace `ollama` with how you/your playbook named the container, e.g. `mash-ollama`):

```sh
docker exec -it ollama ollama pull llama3.2
```

See the [model library](https://ollama.com/library) for the models which are available.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu ollama` (or how you/your playbook named the service, e.g. `mash-ollama`).

#### Increase logging verbosity

If you want to increase the verbosity, add the following configuration to your `vars.yml` file:

```yaml
ollama_environment_variables_ollama_debug: true
```
