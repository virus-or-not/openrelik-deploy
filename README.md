# openrelik-deploy

##### Obligatory Fine Print

This is not an official Google product (experimental or otherwise), it is just code that happens to be owned by Google.

## Post-installation steps for custom builds

If you are using custom images (e.g. `ghcr.io/virus-or-not/openrelik-server`) instead of the official ones, follow these steps after running `install.sh`.

### 1. Replace image references in docker-compose.yml

Open `openrelik/docker-compose.yml` and update the `openrelik-server` and `openrelik-ui` service definitions. Comment out the official `image:` line and add your custom image or a `build:` context:

```yaml
openrelik-server:
  # image: ghcr.io/openrelik/openrelik-server:${OPENRELIK_SERVER_VERSION}
  image: ghcr.io/virus-or-not/openrelik-server:latest
  # — or, to build from source —
  # build:
  #   context: /path/to/openrelik-server

openrelik-ui:
  # image: ghcr.io/openrelik/openrelik-ui:${OPENRELIK_UI_VERSION}
  image: ghcr.io/virus-or-not/openrelik-ui:latest
```

### 2. Pull and restart the custom containers

```bash
cd openrelik/
docker compose pull openrelik-server openrelik-ui
docker compose up -d openrelik-server openrelik-ui
```

### 3. Run database migrations

Custom builds may include Alembic migrations that are not present in the official images (for example, schema changes required by external storage support). Apply them after starting the containers:

```bash
docker exec openrelik-server bash -c \
  "cd /app/openrelik/datastores/sql && alembic upgrade head"
```

### 4. Verify the migration was applied

```bash
docker exec openrelik-server bash -c \
  "cd /app/openrelik/datastores/sql && alembic current"
```

The output should show the latest revision hash followed by `(head)`. If it does not, re-run the upgrade command and check the container logs for errors.

---

## External Storage Override

The `docker/docker-compose.external-storage.yml` Compose override mounts a
host directory as a read-only volume into all services that process artifacts
(server, mediator, and all workers). Use this when your evidence lives on a
separate drive, NAS mount, or shared path that you do not want to copy into
the OpenRelik data directory.

### Configuration

In your `openrelik/` deployment directory, open `.env` (or `config.env`) and
uncomment the two variables at the bottom:

```env
EXTERNAL_STORAGE_HOST_PATH=/path/to/evidence/on/host
EXTERNAL_STORAGE_CONTAINER_PATH=/mnt/external
```

| Variable | Description |
|---|---|
| `EXTERNAL_STORAGE_HOST_PATH` | Absolute path on the Docker host to the directory you want to expose. |
| `EXTERNAL_STORAGE_CONTAINER_PATH` | Path inside every container where the directory will be mounted (read-only). |

### Starting the stack with the override

```bash
cd openrelik/
docker compose -f docker-compose.yml \
               -f docker-compose.external-storage.yml \
               up -d --wait
```

Compose merges the override additively — all existing volume mounts are
preserved and the external directory is appended.

### Connecting evidence in the UI

When creating a task in the OpenRelik UI, set the **Mount point** field to the
same value as `EXTERNAL_STORAGE_CONTAINER_PATH` (e.g. `/mnt/external`).
If the values do not match, the server will not be able to locate your files.
