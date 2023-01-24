# Upgrading from v10 to v11

## Prerequisites

The steps listed in this article require an existing local installation of InvenioRDM v10, please make sure that this is given!

If unsure, run `invenio-cli install` from inside the instance directory before executing the listed steps.

!!! warning "Backup"

    Always backup your database and files before you try to perform an upgrade.

!!! info "Older versions"

    In case you have an InvenioRDM installation older than v10, you can gradually upgrade using the existing instructions to v10 and afterwards continue from here.

## Upgrade Steps

!!! warning "Upgrade your invenio-cli"

    Make sure you have the latest `invenio-cli`, for InvenioRDM v11 the release is v1.0.14.

    ```bash
    $ invenio-cli --version
    invenio-cli, version 1.0.14
    ```

!!! info "Virtual environments"
    In case you are not inside a virtual environment, make sure that you prefix each `invenio` command with `pipenv run`.

### Python version change

As of InvenioRDM v11.0, only Python 3.9 is supported. This is because the official InvenioRDM Docker images now comes
with Python 3.9 only and it is not anymore configurable.

Developing with the same Python version as in deployed environments helps avoiding surprises when deploying
code and it increases predictability.

We strongly suggest recreating your virtual environment with Python 3.9. If you are already using Python 3.9,
you can simply skip this first part and change to the [Upgrade InvenioRDM](#upgrade-inveniordm) section.

The first step is to edit the `<my-site>/Pipfile`:

```diff
[requires]
---python_version = "3.x"
+++python_version = "3.9"
```

Delete the `Pipfile.lock`.

**Container images**

Update the base Docker image in the `Dockerfile`, and rebuild the image if necessary:

```diff
---FROM inveniosoftware/centos8-python:3.8
+++FROM registry.cern.ch/inveniosoftware/almalinux:1
```

**Local development**

Changing the Python version in your development environment highly
depends on your setup, and there is no golden rule.
One example would be to use [PyEnv](https://github.com/pyenv/pyenv).

You should delete your virtual env before running `invenio-cli` or `pipenv` commands below.

!!! warning "Risk of losing data"

    Your virtual env folder contains uploaded files in InvenioRDM, in `var/instance/data`.
    If you need to keep such files, make sure you copy them over to the new virtual env in the same location.

### Upgrade InvenioRDM

Make sure you that your virtual env is now running with Python 3.9.

Upgrade the RDM version:

```bash
cd <my-site>
# Upgrade to InvenioRDM v11
invenio-cli packages update 11.0.0
# Re-build assets
invenio-cli assets build
```

### Database migration

Execute the database migration:

```bash
# Execute the database migration
invenio alembic upgrade
```

### Data migration

Execute the data migration, note that there is no need to re-index the data:

```bash
pipenv run invenio shell $(find $(pipenv --venv)/lib/*/site-packages/invenio_app_rdm -name migrate_10_0_to_11_0.py)
```

The parsing of the ROR dump used for the Funders vocabulary has
been improved, adding the actual ROR identifier to the `identifiers` field.

You can see how to update your data in the [Funders vocabulary section](../../customize/vocabularies/funding.md).