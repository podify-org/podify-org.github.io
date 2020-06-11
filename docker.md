---
layout: default
---


## Installation using Docker

Here's a [sample `docker-compose.yml`](docker/docker-compose.example.yml).

## Configuration via Environment Variables

| Variable            | Required | Description                                                                                           |
|:--------------------|:---------|:------------------------------------------------------------------------------------------------------|
| DATABASE_URL        | Yes      | `postgres://` url pointing to postgres database                                                       |
| REDIS_URL           | Yes      | `redis://` url pointing to redis instance                                                             |
| URL_HOST            | Yes      | Where the app will be available, e.g. `https://podify.yourdomain.com`                                 |
| SECRET_KEY_BASE     | Yes      | This is used to encrypt sessions, see [below](#secret_key_base)                                       |
| STORAGE_DIR         | Yes      | Where downloads will be stored                                                                        |
| RAILS_LOG_TO_STDOUT | No       | Write logs to stdout so they end up in the docker container logs. `"yes"` or `"no"` (default: `"no"`) |

### `SECRET_KEY_BASE`

This is utilized to encrypt and sign sessions. It's recommended that you generate a new one instead of using the one from the example `docker-compose.yml`. You can generate a new one using

    docker run --rm maxhollmann/podify-worker rails secret
