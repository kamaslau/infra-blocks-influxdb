# infra-blocks-influxdb

Trial or micro-service unit of [InfluxDB](https://docs.influxdata.com/).

## Service Endpoints

- Database http://localhost:8181
- UI http://localhost:8888

## Default Account

| DATABASE | TOKEN                                                   |
| -------- | ------------------------------------------------------- |
| mydb     | docker exec -it influxdb influxdb3 create token --admin |

To add a server in UI, go to Configure Servers page → the Add Server button

- Server Name: YOU_NAME_IT
- Server URL: http://influxdb:8181
- Token: as instructed already

## Usage

### Start with [Docker Compose](https://docs.docker.com/compose/)

```bash
# Initiate .env file
cp .env.example .env
# Start services
docker compose up -d
# Create an admin token
docker exec -it influxdb influxdb3 create token --admin
```

Update existing composed containers with latest images:

```bash
docker compose pull && \
docker compose down && \
docker compose up -d
```

Further operations

```bash
# Enter container and initiate shell
docker exec -it trial-influxdb bash

# Create an admin token
docker exec -it influxdb influxdb3 create token --admin

# Create new database (Write admin token to env variable first)
docker exec -it influxdb influxdb3 create database test
```

## 参考资料

- [DockerHub: InfluxDB](https://hub.docker.com/_/influxdb)
- https://docs.influxdata.com/influxdb3/core/install/#docker-image
- https://docs.influxdata.com/influxdb/cloud/api-guide/client-libraries/nodejs
- https://docs.influxdata.com/influxdb/cloud/api-guide/client-libraries/nodejs/write/
- https://docs.influxdata.com/influxdb/cloud/api-guide/client-libraries/nodejs/query/
- https://docs.influxdata.com/influxdb/cloud/write-data/
- https://docs.influxdata.com/influxdb/cloud/query-data/
