# molecule

This directory contains the [Molecule](https://ansible.readthedocs.io/projects/molecule/) test scenarios for the roles in this collection. Each subdirectory is a self-contained scenario that uses Docker to build a test image, converge the role under test, and verify the result.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- Python with `molecule[docker]` installed (see `requirements.txt` in the repo root)

## Running a Scenario

From the **repo root**, run:

```bash
molecule test -s <scenario>
```

For example:

```bash
molecule test -s haproxy
```

## Component Test Reference (Ubuntu 24)

All scenarios listed here use `ubuntu:24.04` as their base image.

### iRODS Configuration

Only the `irods_cfg_initialize` scenario targets Ubuntu 24.04 (its `provider-cfg` platform). The other `irods_cfg_*` scenarios (`irods_cfg_default`, `irods_cfg_update`, `irods_cfg_client`) use `ubuntu:bionic`.

```bash
molecule test -s irods_cfg_initialize
```

### HAProxy Installation & Configuration

```bash
# NOTE: this test needs to be fixed, does not work at the moment (even on main branch)
molecule test -s haproxy
```

### RabbitMQ Installation & Configuration

The `rabbitmq` scenario uses `ubuntu:24.04`. The `rabbitmq_vhost` scenario uses `rabbitmq:management-alpine` and is not Ubuntu 24.

```bash
molecule test -s rabbitmq
```

### PostgreSQL Installation & Configuration

Both PostgreSQL scenarios use `ubuntu:24.04`.

```bash
# Test the postgresql role (installation + configuration)
molecule test -s postgresql

# Test the postgresql_db role (database and user provisioning)
molecule test -s postgresql_db
```
