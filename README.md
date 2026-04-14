# openrelik-deploy

##### Obligatory Fine Print

This is not an official Google product (experimental or otherwise), it is just code that happens to be owned by Google.

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
