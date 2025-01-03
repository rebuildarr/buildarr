# Welcome to ReBuildarr!

[![Docker Version](https://img.shields.io/docker/v/callum027/buildarr?sort=semver)](https://hub.docker.com/r/callum027/buildarr) [![PyPI](https://img.shields.io/pypi/v/buildarr)](https://pypi.org/project/buildarr) ![PyPI - Python Version](https://img.shields.io/pypi/pyversions/buildarr) [![GitHub](https://img.shields.io/github/license/buildarr/buildarr)](https://github.com/buildarr/buildarr/blob/main/LICENSE) ![Test Status](https://img.shields.io/github/actions/workflow/status/buildarr/buildarr/test.yml?label=tests)

This is ReBuildarr - fork of [buildarr](https://github.com/buildarr/buildarr), a solution to automating deployment and configuration of your *Arr stack.

Have you spent many hours getting your setup for one or more linked Sonarr/Radarr/Prowlarr instances just right, only to have no way to reproduce this setup apart from UI screenshots and database backups?

Buildarr aims to alleviate those concerns by using a static configuration file to store settings for all your *Arr applications, and automatically configure them as defined. It can just once using an ad-hoc user command, or as a service to keep your application configurations up to date. Buildarr runs idempotently, only making changes to your instance if they are required.

It can also automatically retrieve optimal configuration values from TRaSH-Guides for many things such as quality definitions and release profiles, so not only is there no need to manually input them into your configuration, Buildarr will also continually keep them up to date for you.

The full documentation for Buildarr is available [here](http://buildarr.github.io).

## Similar projects

Buildarr attempts to fulfill some of the needs of users of the following projects.

* [Bobarr](https://github.com/iam4x/bobarr) - An all-in-one package containing Sonarr, Radarr, Jackett etc
    * Still requires manual configuration of many components, and there is no way to store the configuration as code.
* [Flemmarr](https://github.com/Flemmarr/Flemmarr) - Uses API parameters stored in YAML configuration files to push configuration changes to Sonarr, Radarr, Lidarr etc
    * Requires users to comprehensively learn how the APIs of each application work, going through often poor documentation.
    * Since the values are machine-oriented, configuration files are difficult to write and understand.
    * Does not support idempotent updates ([at this time](https://github.com/Flemmarr/Flemmarr/pull/14)).
* [Recyclarr](https://github.com/recyclarr/recyclarr) - Automatically syncs recommended TRaSH-Guides settings to Sonarr/Radarr instances
    * Buildarr has support for this built-in, and in the case of Sonarr release profiles, supports the same filtering syntax.

## Installation

Buildarr is available on Docker Hub as a Docker image.

```bash
$ docker pull callum027/buildarr:latest
```

[All available plugins](#plugins) at the time of release are bundled into the official Docker container for Buildarr, so you can manage instances of those types right away.

Buildarr can also be installed using `pip`. Python 3.8 or later is required. Windows is natively supported.

As of version 0.4.0, the Python package for Buildarr no longer includes plugins for applications. In order to use Buildarr to manage an application instance, you will also need to install its corresponding plugin.

```bash
$ python3 -m venv buildarr-venv
$ . buildarr-venv/bin/activate
$ python3 -m pip install buildarr
```

You can deploy Buildarr as a service within a [Docker Compose](https://docs.docker.com/compose) environment, or use configuration management tools such as [Ansible](https://www.ansible.com) to automatically deploy it.

For more information, check the [installation instructions](http://buildarr.github.io/installation).

## Plugins

Buildarr supports external plugins to allow additional applications to be supported.

At the time of this release the following plugins are available:

* [`buildarr-sonarr`](https://buildarr.github.io/plugins/sonarr) - [Sonarr](https://sonarr.tv) PVR for TV shows
* [`buildarr-radarr`](https://buildarr.github.io/plugins/radarr) - [Radarr](https://radarr.video) PVR for movies
* [`buildarr-prowlarr`](https://buildarr.github.io/plugins/prowlarr) - [Prowlarr](https://prowlarr.com) indexer manager for Arr applications
* [`buildarr-jellyseerr`](https://buildarr.github.io/plugins/jellyseerr) - [Jellyseerr](https://github.com/Fallenbagel/jellyseerr) media request manager for Jellyfin, Sonarr and Radarr

For more information on installing plugins, check the [plugin documentation](http://buildarr.github.io/plugins).

## Configuration

Buildarr uses YAML as its configuration file format. By default, Buildarr looks for `buildarr.yml` in the current directory.

It contains not only the settings for Buildarr itself, but also the application instances to be managed. Multiple instances of the same application type can be defined (for example, a common use case would be separate Sonarr instances for HD TV shows, 4K TV shows, and anime).

Any configuration on the remote instance not explicitly defined in the Buildarr configuration is not modified.

For more information on how Buildarr uses configuration and how to configure Buildarr itself, check the [configuration documentation](https://buildarr.github.io/configuration).

Here is an example of a simple Buildarr configuration that changes some settings on a Sonarr instance:

```yaml
---
# buildarr.yml
# Buildarr example configuration file.

# Buildarr configuration (all settings have sane default values)
buildarr:
  watch_config: true
  update_days:
    - "monday"
    - "tuesday"
    - "wednesday"
    - "thursday"
    - "friday"
    - "saturday"
    - "sunday"
  update_times:
    - "03:00"

# Sonarr instance configuration
sonarr:
  hostname: "localhost"
  port: 8989
  protocol: "http"
  settings:
    # General settings (all options supported except for changing the API key)
    general:
      host:
        instance_name: "Sonarr (Buildarr Example)"
```

If you have an already configured application instance, its configuration can be dumped. For example, to get the configuration of a Sonarr instance, this can be done using the following command (Buildarr will prompt for your API key):

```bash
$ docker run -it --rm callum027/buildarr:latest sonarr dump-config http://sonarr.example.com:8989
```

Once you have this configuration, you can insert it into `buildarr.yml` and ensure this configuration is maintained.

## Running Buildarr

Once you have a valid configuration file, you can try Buildarr on your local machine using the Docker image.

The following command will mount the current folder into the Docker container so `buildarr.yml` can be read, and start Buildarr in daemon mode.

```bash
$ docker run -d --name buildarr -v $(pwd):/config -e PUID=$(id -u) -e PGID=$(id -g) callum027/buildarr:latest
```

If installed using `pip`, simply run the `buildarr` CLI command.

```bash
$ buildarr daemon
```

On startup, Buildarr daemon will do an initial sync with the defined instances, updating their configuration immediately.
After this initial run, Buildarr will wake up at the scheduled times to periodically run updates as required.

```txt
2023-11-12 10:00:29,220 buildarr:1 buildarr.cli.daemon [INFO] Buildarr version 0.7.0 (log level: INFO)
2023-11-12 10:00:29,220 buildarr:1 buildarr.cli.daemon [INFO] Loading configuration file '/config/buildarr.yml'
2023-11-12 10:00:29,775 buildarr:1 buildarr.cli.daemon [INFO] Finished loading configuration file
2023-11-12 10:00:29,775 buildarr:1 buildarr.cli.daemon [INFO] Daemon configuration:
2023-11-12 10:00:29,776 buildarr:1 buildarr.cli.daemon [INFO]  - Watch configuration files: No
2023-11-12 10:00:29,776 buildarr:1 buildarr.cli.daemon [INFO]  - Update at:
2023-11-12 10:00:29,776 buildarr:1 buildarr.cli.daemon [INFO]    - Monday 03:00
2023-11-12 10:00:29,776 buildarr:1 buildarr.cli.daemon [INFO]    - Tuesday 03:00
2023-11-12 10:00:29,777 buildarr:1 buildarr.cli.daemon [INFO]    - Wednesday 03:00
2023-11-12 10:00:29,777 buildarr:1 buildarr.cli.daemon [INFO]    - Thursday 03:00
2023-11-12 10:00:29,778 buildarr:1 buildarr.cli.daemon [INFO]    - Friday 03:00
2023-11-12 10:00:29,778 buildarr:1 buildarr.cli.daemon [INFO]    - Saturday 03:00
2023-11-12 10:00:29,778 buildarr:1 buildarr.cli.daemon [INFO]    - Sunday 03:00
2023-11-12 10:00:29,778 buildarr:1 buildarr.cli.daemon [INFO] Scheduling update jobs
2023-11-12 10:00:29,779 buildarr:1 buildarr.cli.daemon [INFO] Finished scheduling update jobs
2023-11-12 10:00:29,779 buildarr:1 buildarr.cli.daemon [INFO] Config file monitoring is already disabled
2023-11-12 10:00:29,779 buildarr:1 buildarr.cli.daemon [INFO] Applying initial configuration
2023-11-12 10:00:29,932 buildarr:1 buildarr.cli.run [INFO] Loaded plugins: jellyseerr (0.3.0), prowlarr (0.5.0), radarr (0.2.0), sonarr (0.6.0)
2023-11-12 10:00:29,932 buildarr:1 buildarr.cli.run [INFO] Loading instance configurations
2023-11-12 10:00:29,973 buildarr:1 buildarr.cli.run [INFO] Finished loading instance configurations
2023-11-12 10:00:29,973 buildarr:1 buildarr.cli.run [INFO] Running with plugins: prowlarr, sonarr, radarr, jellyseerr
2023-11-12 10:00:29,973 buildarr:1 buildarr.cli.run [INFO] Resolving instance dependencies
2023-11-12 10:00:29,973 buildarr:1 buildarr.cli.run [INFO] Finished resolving instance dependencies
2023-11-12 10:00:29,973 buildarr:1 buildarr.cli.run [INFO] Fetching TRaSH metadata
2023-11-12 10:00:36,723 buildarr:1 buildarr.cli.run [INFO] Finished fetching TRaSH metadata
2023-11-12 10:00:36,723 buildarr:1 buildarr.cli.run [INFO] Rendering instance configuration dynamic attributes
2023-11-12 10:00:36,739 buildarr:1 buildarr.cli.run [INFO] Finished rendering instance configuration dynamic attributes
2023-11-12 10:00:37,273 buildarr:1 buildarr.cli.run [INFO] <sonarr> (default) Fetching instance secrets
2023-11-12 10:00:37,273 buildarr:1 buildarr.cli.run [INFO] <sonarr> (default) Finished fetching instance secrets
2023-11-12 10:00:37,273 buildarr:1 buildarr.cli.run [INFO] <sonarr> (default) Running connection test
2023-11-12 10:00:37,343 buildarr:1 buildarr.cli.run [INFO] <sonarr> (default) Connection test successful
2023-11-12 10:00:38,112 buildarr:1 buildarr.cli.run [INFO] Performing post-initialisation configuration render
2023-11-12 10:00:39,292 buildarr:1 buildarr.cli.run [INFO] Finished performing post-initialisation configuration render
2023-11-12 10:00:39,292 buildarr:1 buildarr.cli.run [INFO] Updating configuration on remote instances
2023-11-12 10:00:39,292 buildarr:1 buildarr.cli.run [INFO] <sonarr> (default) Fetching remote configuration to check if updates are required
2023-11-12 10:00:39,738 buildarr:1 buildarr.cli.run [INFO] <sonarr> (default) Finished fetching remote configuration
2023-11-12 10:00:39,810 buildarr:1 buildarr.cli.run [INFO] <sonarr> (default) Updating remote configuration
2023-11-12 10:00:39,850 buildarr:1 buildarr.config.base [INFO] <sonarr> (default) sonarr.settings.general.host.instance_name: 'Sonarr' -> 'Sonarr (Buildarr Example)'
2023-11-12 10:00:39,933 buildarr:1 buildarr.cli.run [INFO] <sonarr> (default) Remote configuration successfully updated
2023-11-12 10:00:40,574 buildarr:1 buildarr.cli.run [INFO] <sonarr> (default) Finished updating remote configuration
2023-11-12 10:00:49,722 buildarr:1 buildarr.cli.run [INFO] Finished updating configuration on remote instances
2023-11-12 10:00:49,722 buildarr:1 buildarr.cli.run [INFO] Deleting unmanaged/unused resources on remote instances
2023-11-12 10:00:52,579 buildarr:1 buildarr.cli.run [INFO] <sonarr> (default) Refetching remote configuration to delete unused resources
2023-11-12 10:00:52,714 buildarr:1 buildarr.cli.run [INFO] <sonarr> (default) Finished refetching remote configuration
2023-11-12 10:00:52,771 buildarr:1 buildarr.cli.run [INFO] <sonarr> (default) Deleting unmanaged/unused resources on the remote instance
2023-11-12 10:00:52,843 buildarr:1 buildarr.cli.run [INFO] <sonarr> (default) Remote configuration is clean
2023-11-12 10:00:52,843 buildarr:1 buildarr.cli.run [INFO] <sonarr> (default) Finished deleting unmanaged/unused resources on the remote instance
2023-11-12 10:00:52,843 buildarr:1 buildarr.cli.run [INFO] Finished deleting unmanaged/unused resources on remote instances
2023-11-12 10:00:52,843 buildarr:1 buildarr.cli.run [INFO] Deleting downloaded TRaSH metadata
2023-11-12 10:00:52,873 buildarr:1 buildarr.cli.run [INFO] Finished deleting downloaded TRaSH metadata
2023-11-12 10:00:52,874 buildarr:1 buildarr.cli.daemon [INFO] Finished applying initial configuration
2023-11-12 10:00:52,874 buildarr:1 buildarr.cli.daemon [INFO] Setting up signal handlers
2023-11-12 10:00:52,875 buildarr:1 buildarr.cli.daemon [INFO] Finished setting up signal handlers
2023-11-12 10:00:52,875 buildarr:1 buildarr.cli.daemon [INFO] The next run will be at 2023-11-13 03:00
2023-11-12 10:00:52,875 buildarr:1 buildarr.cli.daemon [INFO] Buildarr ready.
```

For more information on how to interfact with Buildarr, check the [usage documentation](https://buildarr.github.io/usage).

## To-do list

* Test updates for all available attributes in the existing Sonarr plugin
* Unit tests and code coverage
* Split Sonarr plugin to its own repository (completed in [version 0.4.0](https://buildarr.github.io/release-notes/#v040-2023-03-31))
* Create plugins for the following applications:
    * Sonarr V4
    * Radarr (now available as [`buildarr-radarr`](https://buildarr.github.io/plugins/radarr))
    * Prowlarr (now available as [`buildarr-prowlarr`](https://buildarr.github.io/plugins/prowlarr))
    * Jellyseerr (now available as [`buildarr-jellyseerr`](https://buildarr.github.io/plugins/jellyseerr))
    * Bazarr
    * Unmanic
    * Tdarr (maybe)
    * Unpackerr
    * Lidarr
* Instance linking (e.g. Prowlarr-to-Sonarr/Radarr) and dependency resolution (added in [version 0.3.0](https://buildarr.github.io/release-notes/#v030-2023-03-15))
* Stable plugin API between major versions
* Auto-generation of Docker Compose environment files reflecting the Buildarr configuration (added in [version 0.4.0](https://buildarr.github.io/release-notes/#v040-2023-03-31))

## Contributions

Buildarr is still early in development, and even currently implemented features still require testing and fixing. There are so many possible configurations to cover that I simply cannot feasibly test every feature at this time.

If you encounter an issue or error while using Buildarr, please do a Buildarr ad-hoc run with verbose log output by executing `buildarr --log-level DEBUG run` and making an issue on [our GitHub repository](https://github.com/buildarr/buildarr/issues/new) explaining the issue and attaching the output. (Please ensure that any API keys or other sensitive information are obfuscated before submitting.)

```bash
$ docker run -d --name buildarr -v $(pwd):/config -e PUID=$(id -u) -e PGID=$(id -g) callum027/buildarr:latest --log-level DEBUG run
```

Bug reports and pull requests for Buildarr itself are welcome in the Buildarr base package repository. For reporting issues and making contributions to application plugins, check out their repositories:

* Sonarr plugin: [https://github.com/buildarr/buildarr-sonarr](https://github.com/buildarr/buildarr-sonarr)

For developers looking to make a contribution to this project, thank you! Documentation of the internal APIs is still in the works, so for now, the best way to learn how Buildarr works is to clone the project and have a look at the comments and docstrings.

Pre-commit hooks are configured for this project. In this pre-commit hook, [Black](https://black.readthedocs.io/en/stable), [Ruff](https://beta.ruff.rs/docs) and [Mypy](https://mypy-lang.org) are run to automatically format source files, ensure grammatical correctness and variable type consistency.

To enable them, ensure the `pre-commit` Python package is installed in your local environment and run the following command:

```bash
$ pre-commit install
```

Poetry is used to manage the Python package definition and dependencies in this project.

If you're looking to develop a new plugin for adding support for a new application, please develop it as a new package and configure entry points in your Python package definitions to allow Buildarr to load your plugin.

Setuptools `setup.py` entry point definition example:
```python
from setuptools import setup

setup(
    # ...,
    entry_points={
        "buildarr.plugins": [
            "example = buildarr_example.plugin:ExamplePlugin",
        ],
    },
)
```

Setuptools `setup.cfg` entry point definition example:
```ini
[options.entry_points]
buildarr.plugins =
    example = buildarr_example.plugin:ExamplePlugin
```

Setuptools `pyproject.toml` entry point definition example:
```toml
[project.entry-points."buildarr.plugins"]
"example" = "buildarr_example.plugin:ExamplePlugin"
```

Poetry plugin definition example:
```toml
[tool.poetry.plugins."buildarr.plugins"]
"example" = "buildarr_example.plugin:ExamplePlugin"
```
