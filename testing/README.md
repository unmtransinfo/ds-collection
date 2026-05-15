# testing

This folder contains the test harness for the DS playbooks. It consists of a simplified Data Store and a playbook runner.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) with the `buildx` plugin
- `docker compose` (v2) or `docker-compose` (v1)
- Bash

## The Environment

The environment consists of a set of containers. The `amqp` container hosts the RabbitMQ broker that in turn hosts the `irods` exchange, where the Data Store publishes messages to. The `dbms_configured` container hosts the PostgreSQL server that in turn hosts the ICAT DB. The `provider_configured` container hosts a configured iRODS catalog service provider. The `provider_unconfigured` container hosts an unconfigured service provider. The `consumer_configured_centos` container hosts a configured CentOS catalog service consumer acting as a resource server. The `consumer_configured_ubuntu` container hosts a configured Ubuntu catalog service consumer acting as a resource server. The `consumer_unconfigured` container hosts an unconfigured service consumer. Finally, `consumer_containerized_alma` and `consumer_containerized_ubuntu` hold unconfigured AlmaLinux and Ubuntu 24.04 hosts respectively.

The environment is configured by `config.inc`, which exports the environment variables consumed by the Docker Compose file and the helper scripts. The defaults in `config.inc` are sufficient for most use cases.

## Building the Harness

Run the `build` script from the `testing/` directory to build all Docker images — both the environment images (base OS images and Docker Compose services) and the `ansible-tester` image:

```bash
./build
```

To remove all images built by `build`, run:

```bash
./clean
```

`build` and `clean` both read their configuration from `config.inc` automatically.

## Running the Tests

### Testing a Playbook

`test-playbook` is a convenience script for running a playbook against the testing environment. It brings up the environment, runs the playbook (and any associated tests), and then tears down the environment.

```
Usage:
  test-playbook [options]

Options:
  -P, --playbook <playbook>    the name of the playbook to test
  -S, --setup <playbooks>      comma-separated list of playbooks to run before the playbook under test
  -I, --inventory <inventory>  the inventory hosts file to use (default: hosts-all)
  -L, --limit <pattern>        limit execution to hosts matching this Ansible host pattern
  -i, --inspect                after running, open a shell for manual inspection
  -p, --pretty                 expand output for easier reading
  -h, --help                   show help and exit
```

For example, to test the `irods_cfg` playbook:

```bash
./test-playbook -P irods_cfg
```

To run a setup playbook before the playbook under test:

```bash
./test-playbook -S irods_provision -P irods_cfg
```

To limit execution to only Ubuntu hosts:

```bash
./test-playbook -P irods_resource_server -L '*ubuntu*'
```

To bring up the environment for interactive inspection without running a playbook:

```bash
./test-playbook -i
```

The script performs the tests in the following order:

1. If one or more setup playbooks are provided (`-S`), they are run in order.
1. If a playbook to test is provided (`-P`), it does the following:
   1. Performs a syntax check on the playbook under test.
   1. Runs the playbook on the testing inventory.
   1. Runs any associated test playbook found in `playbooks/tests/`.
   1. Performs an idempotency check on the playbook.
1. If the inspect option (`-i`) was provided, opens a shell prompt for manual inspection.

### Testing a Plugin

`test-plugin` is a script for testing an individual Ansible plugin (e.g., a module or filter) against the testing environment.

```
Usage:
  test-plugin [options] <plugin_type> <plugin_name>

Positional Arguments:
  <plugin_type>  the plugin type subdirectory (e.g., module, filter)
  <plugin_name>  the plugin file name (.py extension is optional)

Options:
  -I, --inventory <hosts>  the inventory hosts file to use (default: hosts-all)
  -i, --inspect            after running, open a shell for manual inspection
  -p, --pretty             expand output for easier reading
  -v, --verbose            produce verbose Ansible output
  -h, --help               show help and exit
```

For example, to test the `irods_group` module:

```bash
./test-plugin module irods_group
```

The plugin's test playbook must exist at `plugins/<type>/tests/<plugin_name>.yml`.

### Inventory Hosts Files

The following inventory hosts files are available in `ansible-tester/inventory/` and can be selected with the `-I` option:

| Hosts file                    | Description                                    |
| ----------------------------- | ---------------------------------------------- |
| `hosts-all`                   | All containers (default)                       |
| `hosts-configured`            | Only the pre-configured provider and consumers |
| `hosts-unconfigured-consumer` | Only the unconfigured consumer                 |
| `hosts-unconfigured-dbms`     | Only the unconfigured DBMS hosts               |
| `hosts-unconfigured-provider` | Only the unconfigured provider                 |

### Defining Tests for a Playbook

Tests for a given playbook are written as a separate Ansible playbook and placed in `playbooks/tests/` with the same filename as the playbook under test. For example, tests for `playbooks/irods_cfg.yml` go in `playbooks/tests/irods_cfg.yml`.

### Skipping Tasks During Testing

Two special task tags control how `ansible-tester` handles tasks during testing:

- `non_idempotent` — tasks tagged with this are skipped during the idempotency check, since they are expected to always report a change (e.g., forced restarts).
- `no_testing` — tasks tagged with this are skipped entirely inside the test environment (e.g., tasks that modify `/etc/hosts`).
