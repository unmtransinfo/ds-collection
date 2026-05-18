# testing

This folder contains the test harness for the DS playbooks. It consists of a simplified Data Store and a playbook runner.

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

## Component Test Reference (Ubuntu 24)

All commands below are run from the `testing/` directory. Every environment container referenced here is built on Ubuntu 24.04. For the equivalent Molecule tests, see [`molecule/README.md`](../molecule/README.md).

### iRODS Installation & Configuration

Uses `-L '*ubuntu*'` to restrict execution to the Ubuntu 24.04 environment containers (`provider_unconfigured`, `consumer_configured_ubuntu`, etc.).

```bash
# Test iRODS package installation
./test-playbook -P irods_provision -L '*ubuntu*'

# Test iRODS configuration (provision first as setup)
./test-playbook -S irods_provision -P irods_cfg -L 'irods_resource_native:&*ubuntu*'
```

### HAProxy Installation & Configuration

Not applicable. There is no `proxy` group in any inventory hosts file and no test playbook for `proxy.yml` in `playbooks/tests/`.

### RabbitMQ Installation & Configuration

The `amqp` container is built from `test-env-base:ubuntu2404`.

```bash
# NOTE: this test fails with: fatal: [dstesting-amqp-1.dstesting_default]: FAILED! => {"changed": false, "msg": "Service is in unknown state", "status": {}}
# fails in main branch as well with same issue -- ignoring for now
./test-playbook -P amqp
```

### PostgreSQL Installation & Configuration

Both `dbms_configured` and `dbms_unconfigured` containers are built from `test-env-base:ubuntu2404`. Use `hosts-unconfigured-dbms` to test against a fresh node.

```bash
# Test DBMS installation and configuration
./test-playbook -P dbms

# Test iCAT database setup (requires DBMS to be set up first)
./test-playbook -S dbms -P dbms_icat
```
