# infra-blocks-influxdb

[![Blocks](https://img.shields.io/badge/infra--blocks-index-555?style=flat-square)](https://github.com/kamaslau/infra-blocks)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
![Repository size](https://img.shields.io/github/repo-size/kamaslau/infra-blocks-influxdb?color=56BEB8)

A unit, not a demo: one service, one port, one documented credential, disposable with a single command.

Typical use: sensors or gateways write a reading every few seconds and nobody wants those rows in the relational database that serves the website.

[InfluxDB 3 Core](https://docs.influxdata.com/influxdb3/core/install/#docker-image) with its own query UI. For the dashboard side of the same problem, [infra-blocks-prometheus](https://github.com/kamaslau/infra-blocks-prometheus) covers host metrics rather than device data.

## Service URL

- HTTP API `http://localhost:8181`
- Query UI `http://localhost:8888`

Ports can be changed in `.env`; see Usage.

## Default user

There is no username and no default password. Access is a bearer token, and the first one has to be created from inside the container — it is printed once and never again:

```bash
docker exec influxdb influxdb3 create token --admin
```

Copy it somewhere before closing the terminal. Running the command a second time fails with `token name already exists, _admin`, and nothing on the volume will print the token again; a lost bootstrap token means `docker compose down -v` and starting over.

| Item | Value |
| ---- | ----- |
| Token | created by the command above, shown once |
| Database | none until `influxdb3 create database` is run |

## Usage

### Start with [Docker Compose](https://docs.docker.com/compose/)

```bash
# Initiate .env file
cp .env.example .env
# Start services
docker compose up -d

# Create the admin token — the only copy you get
docker exec influxdb influxdb3 create token --admin
```

Update existing composed containers with latest images:

```bash
docker compose pull && \
docker compose down && \
docker compose up -d
```

### Create a database and write a point

```bash
docker exec influxdb influxdb3 create database mydb --token $INFLUX_TOKEN
docker exec influxdb influxdb3 write \
  --database mydb --token $INFLUX_TOKEN \
  'sensor,room=lobby temperature=21.4'
```

### Connect the UI

Open <http://localhost:8888> → **Configure Servers** → **Add Server**:

- Server Name: anything you will recognise later
- Server URL: `http://influxdb:8181` — the service name, not `localhost`; the UI runs in the same compose network and the published port is for your browser only
- Token: the admin token from above

The first window after a fresh `up` takes a few seconds to populate; the UI seeds a plugin registry from `github.com` on start, and on a host without a route to it the empty list is a network symptom, not an empty database.

## Verify

```bash
docker exec influxdb influxdb3 create token --admin
```

The token, printed once with a warning above it. Then, with the token in `$INFLUX_TOKEN`:

```bash
curl -s -H "Authorization: Bearer $INFLUX_TOKEN" http://localhost:8181/health
# OK
```

`OK` is the whole body: the server is up, the token is valid, and the query path is authorised. Without the header the same endpoint answers `401 {"error": "the request was not authenticated"}`, which is worth knowing — a failing health check on this unit is usually a missing token, not a stopped process.

```bash
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:8888/
# 200
```

`200` from the UI port, with `docker compose ps` showing `influxdb_ui` as `(healthy)`.

## Why this shape

Two containers, because InfluxDB 3 ships the database and the UI as separate images — there is no bundled web interface the way InfluxDB 1.x had one. `influxdb` holds the data and answers the API; `influxdb_ui` is a browser front end that talks to it over the compose network.

Object storage is a file on a named volume (`--object-store=file --data-dir=/var/lib/influxdb3/data`), which is the honest default for a single host. The same flag is what makes the migration path short: point it at S3-compatible storage later without changing the query side.

Authentication is on by default and there are no users, only tokens. That is fewer moving parts than a username/password table, and it means the credential in a client's `.env` is the same object as the one in the UI — one thing to rotate, not two.

The UI port maps to **8080** inside the container, not 80. The image listens on 8080 and a `:80` mapping produces a published port that answers nothing while `docker compose ps` reports the container healthy.

## Notes for deployments in China

Both images pull from Docker Hub and need a registry mirror. `influxdb:3-core` is the larger of the two and the one that fails first on a congested link.

The UI seeds its plugin registry from `github.com` at startup. On a mainland host that request fails and the log records `Seed influxdb_plugin_registry failed: … Unreachable { host: "github.com" }`; the UI still runs and still queries, only the plugin list is empty. No fix is required unless plugins are the point.

The UI also warns `SESSION_SECRET_KEY not provided - generated ephemeral key` when `SESSION_SECRET_KEY` is not set. Sessions then do not survive a restart, which matters only if you hand the UI to someone who expects to stay logged in across one.

---

Part of [infrastructure blocks](https://github.com/kamaslau/infra-blocks) · ← back to index · [what I do](https://www.kamaslau.com/works?utm_source=github&utm_medium=repo-readme&utm_campaign=always-on&utm_content=unit)
