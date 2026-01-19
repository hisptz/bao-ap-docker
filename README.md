# Analytics Platform Setup With Docker

## Overview

A docker compose setup for the analytics platform with all its components and dependencies.

## Prerequisites

This setup requires the latest version of docker (> 28.4.0) and docker compose (> 2.39.4).

## Components

- Postgres - Database for data-pipeline, identity, and superset services' metadata
- Pulsar - Message broker for the data-pipeline service
- ClickHouse - Database for the data-pipeline and superset services' data
- Redis - Caching for the identity, data-pipeline, and superset services
- Superset - Data visualization tool
- Data-Pipeline - Data ingestion and transformation pipeline (Analytics Platform component)
- Identity - User management and authentication service (Analytics Platform component)
- Gateway - API gateway for the analytics platform
- DHIS2 Superset Gateway - API gateway for connecting DHIS2 to Superset
- Proxy - Reverse proxy for the analytics platform

## Migration from the old setup

If you already cloned the old setup, you can migrate to the new setup by downloading the release file from the
[releases page](https://github.com/Baosystems/bao-ap-docker/releases)

```shell
wget https://github.com/hisptz/bao-ap-docker/releases/download/<latest-tag-here>/analytics-platform.zip
```

And then unzip the downloaded archive:

```shell
  unzip analytics-platform.zip
```

Then move or copy the `docker-compose.yml` and the `docker-compose.pulsar.yml` files to the root of the cloned
repository in the old setup. The new `docker-compose.yml` should replace the old one.

To clean up the setup, you can delete the downloaded analytics-platform.zip file and the unzipped folder. You can also
delete the `Dockerfile` files in the `bao-data-pipeline`, `bao-identity`, `bao-api-gateway`, `dhis2-superset-gateway`
folders. You can also delete the pulsar folder.

## Installation

To install the setup, download the latest release from
the [releases page](https://github.com/Baosystems/bao-ap-docker/releases).

You can use wget to download the release:

```shell
 wget https://github.com/hisptz/bao-ap-docker/releases/download/<latest-tag-here>/analytics-platform.zip
```

And then unzip the downloaded archive:

```shell
  unzip analytics-platform.zip
```

An then navigate to the extracted folder:

```shell
cd analytics-platform
```

## Setup

First, pull all required images by running:

```shell 
docker compose pull 
```

## Configuration

For most services, the default configuration is sufficient. However, some services require additional configuration.

### Database (ap-db)

You can change the default database name, username, and password by modifying the `POSTGRES_USER`, and
`POSTGRES_PASSWORD` environment variables in the `db/.env` file. If these values are changed, you will need to update
the database connection string in the service configuration files for identity, data-pipeline, and superset.

### Data Pipeline (data-pipeline)

The data pipeline service configuration can be found in the `bao-data-pipeline/config/bao-data-pipeline.conf` file. The
default database connection should work as is unless you use a different database setup, or you change the database
name, username, and password as explained in the database section.

You can configure the rest of the data pipeline service as explained in
the [data pipeline documentation](https://docs.ap.baosystems.com/sysadmin/analytics-platform-installation/#data-pipeline).

The data pipeline services also requires an encryption key to be set. You can generate a new key as explained in
the [documentation](https://docs.ap.baosystems.com/sysadmin/analytics-platform-installation/#encryption) and place the
resulting JSON key in the `bao-data-pipeline/config/bao-data-pipeline-key.json` file.

### Identity (identity)

The identity service configuration can be found in the `bao-identity/config/bao-identity.conf` file. The default
database connection should work as is unless you use a different database setup, or you change the database
name, username, and password as explained in the database section.

You can configure the rest of the identity service as explained in
the [identity documentation](https://docs.ap.baosystems.com/sysadmin/analytics-platform-installation/#identity).

### API Gateway (gateway)

The API gateway service configuration can be found in the `bao-api-gateway/config/bao-api-gateway.conf` file. The
default identity and data-pipeline connections should work as is.

You can configure the rest of the gateway service as explained in
the [gateway documentation](https://docs.ap.baosystems.com/sysadmin/analytics-platform-installation/#api-gateway).

### Superset (superset)

The superset service configuration can be found in the `superset/config/superset_config.py` file. The default
database connection should work as is unless you use a different database setup, or you change the database
name, username, and password as explained in the database section.

You can configure the rest of the superset service as explained in
the [superset documentation](https://docs.ap.baosystems.com/sysadmin/apache-superset-installation/#configuration).

### DHIS2 Superset Gateway (dhis2-superset-gateway)

The DHIS2 Superset Gateway service configuration can be found in the
`dhis2-superset-gateway/config/dhis2-superset-gateway.conf` file.

The following configuration values are required:

- `dhis2.base_url`: Base URL of the DHIS2 instance
- `superset.base_url`: Base URL of the superset instance (The exposed URL of the superset service, preferably hosted on
  secure http)
- `superset.username`: Username of the superset user to connect to superset from the DHIS2 instance
- `superset.password`: Password of the superset user to connect to superset from the DHIS2 instance

### ClickHouse (clickhouse)

The ClickHouse service configuration can be found in the `clickhouse/.env` file. The default configuration values are
sufficient as is unless you want to change them.

The performance configuration can be found in the `clickhouse/config/performance.xml` file. The default configuration is
as recommended in
the [documentation](https://docs.ap.baosystems.com/sysadmin/clickhouse-installation/#performance-tuning)

### NGINX (proxy)

The NGINX service configuration can be found in the `nginx/config/analytics-platform.conf` file. The default is as
provided in the [documentation](https://docs.ap.baosystems.com/sysadmin/middleware-installation/#nginx)

There is also a configuration file for superset included in the `nginx/config/superset.conf` file.

## Running the setup

To start the services, run:

```shell
docker compose up -d
```

To stop the services, run:

```shell
docker compose down 
```

To view the logs, run:

```shell
docker compose logs -f
```

To view logs for a specific service, run:

```shell
docker compose logs -f <service>
```

## Post-running setup

Some services require additional setup after running the services.

### Superset

Superset needs to be initialized and a superuser created when running for the first time as explained in
the [documentation](https://docs.ap.baosystems.com/sysadmin/apache-superset-installation/#configuration).

Run database migrations

```shell
docker compose exec superset superset db upgrade
```

Initialize the database

```shell
docker compose exec superset superset init
```

Create a superuser

```shell
docker compose exec superset superset fab create-admin
```

## Client setup in the analytics platform interface

To allow the analytics platform to work with the set up clickhouse service, when configuring clients in the analytics
platform interface use the following values;

Data warehouse configuration:

- Host: clickhouse
- Port: leave empty
- Public hostname: leave empty
- Username: bao (or whatever you set in the clickhouse service configuration)
- Password: analytics-platform (or whatever you set in the clickhouse service configuration)
- Database: baoanalytics (or whatever you set in the clickhouse service configuration)

## Database connection in the superset interface

To connect to the database from the superset interface, use the following values:

- Host: clickhouse
- Port: leave empty
- Username: bao (or whatever you set in the clickhouse service configuration)
- Password: analytics-platform (or whatever you set in the clickhouse service configuration)
- Database name: baoanalytics (or whatever you set in the clickhouse service configuration)

## Connecting to a DHIS2 database

If your database is hosted in the same server as the analytics platform, and is accessible from the localhost, you can
set it up using the following values:

- Hostname: 172.17.0.1 (Docker's host IP by default)
- Username: dhis2-db-user
- Password: dhis2-db-password
- Database name: dhis-db-name

