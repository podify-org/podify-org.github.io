---
layout: default
---

## Tags

| Tag     | Description                                                                                             |
|:--------|:--------------------------------------------------------------------------------------------------------|
| latest  | Latest stable release (but note that the software is still in early development, so stable is relative) |
| preview | Unstable development releases                                                                           |


## Docker Compose

This is a fully working `docker-compose.yml`, but it is recommended
that you switch out at least the `SECRET_KEY_BASE` environment
variable as explained [below](#secret_key_base).

```yaml
version: '3.4'

x-app-defaults: &app-defaults
  restart: always
  environment: &app-env
    URL_HOST: https://podify.yourdomain.com
    DATABASE_URL: postgres://podify:verysecurepassword@db/podify
    REDIS_URL: redis://redis
    SECRET_KEY_BASE: a57d57661ef5df58b46fab6f04304e89108f22f89b31d2242b31891102da87d519a1f3c6459c1d2716b3b8c5438ef43e06ed4c29c8fb059eb650dc2ec0062d57
    RAILS_LOG_TO_STDOUT: "yes"
    STORAGE_DIR: /storage

  volumes:
    - storage:/storage

  depends_on:
    - db
    - redis

services:
  web:
    <<: *app-defaults
    image: maxhollmann/podify-web:latest
    ports:
      - 3000:3000
    environment:
      <<: *app-env

  worker:
    <<: *app-defaults
    image: maxhollmann/podify-worker:latest
    environment:
      <<: *app-env

  db:
    image: postgres:12.3
    restart: always
    environment:
      POSTGRES_USER: podify
      POSTGRES_PASSWORD: verysecurepassword
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
      - pgdata:/var/lib/postgresql/data/pgdata

  redis:
    image: redis:6
    restart: always

volumes:
  pgdata:
  storage:
```

If you're using Traefik you will need to set up the
CustomRequestHeaders middleware using the following labels in addition
to the labels setting up the endpoint on the `web` service:

```
- "traefik.http.routers.podify.middlewares=sslheader@docker"
- "traefik.http.middlewares.sslheader.headers.customrequestheaders.X-Forwarded-Proto=https"
```

The realtime download status updates are sent via websockets and won't
work without this.

## Configuration via Environment Variables

| Variable            | Required | Description                                                                                           |
|:--------------------|:---------|:------------------------------------------------------------------------------------------------------|
| DATABASE_URL        | Yes      | `postgres://` url pointing to postgres database                                                       |
| REDIS_URL           | Yes      | `redis://` url pointing to redis instance                                                             |
| URL_HOST            | Yes      | Where the app will be available, e.g. `https://podify.yourdomain.com`                                 |
| SECRET_KEY_BASE     | Yes      | This is used to encrypt sessions, see [below](#secret_key_base)                                       |
| STORAGE_DIR         | Yes      | Where downloads will be stored                                                                        |
| RAILS_LOG_TO_STDOUT | No       | Write logs to stdout so they end up in the docker container logs. `"yes"` or `"no"` (default: `"no"`) |

### SECRET_KEY_BASE

This is utilized to encrypt and sign sessions. It's recommended that you generate a new one instead of using the one from the example `docker-compose.yml`. You can generate a new one using

    docker run --rm maxhollmann/podify-worker rails secret
