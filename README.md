# DevBox
---
Devbox by Docker for local development

## Requirements

- Docker
- Dnsmasq (optional - for redirecting `*.test` to `127.0.0.1`)

## Included 

- Traefik v2
- MySQL
- PostgreSQL (optional)
- Mailhog

## Configuration

- Generate `USER:PASSWORD` for access Traefik dashboard

```sh
echo $(htpasswd -nb <USER> <PASSWORD>) | sed -e s/\\$/\\$\\$/g
```

then replace in `docker-compose.yml:24`

Open browser and check if traefik dashboard is working https://traefik.test

## Usage

- Before we need start devbox first (just only run once)

```sh
touch acme.json && chmod 600 acme.json && \
docker network create proxy && docker-compose up -d
```

- When starting any new projects, just use this sample config

```yml
version: '3'

services:
  wordpress:
    image: wordpress
    container_name: wordpress
    restart: unless-stopped
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: root
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - ./public_html:/var/www/html
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.wordpress.entrypoints=http"
      - "traefik.http.routers.wordpress.rule=Host(`wordpress.test`)"

networks:
  default:
    external:
      name: proxy
```

Then run `docker-compose up` then open browser and go to http://wordpress.test

### References

- https://medium.com/@containeroo/traefik-2-0-docker-a-simple-step-by-step-guide-e0be0c17cfa5

