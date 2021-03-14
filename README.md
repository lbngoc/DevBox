# DevBox

## Included 

- Traefik v2
- MySQL
- PostgreSQL (optional)
- Mailhog

## Configuration

- Generate `USER:PASSWORD` for access Traefik dashboard

	$ echo $(htpasswd -nb <USER> <PASSWORD>) | sed -e s/\\$/\\$\\$/g

then replace in `docker-compose.yml:24`

- Sample `docker-comsose.yml` file

```
  mailhog:
    restart: unless-stopped
    image: mailhog/mailhog
    container_name: mailhog
    networks:
      - proxy
    ports:
      - "1025:1025"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.mailhog.entrypoints=http"
      - "traefik.http.routers.mailhog.rule=Host(`mailhog.test`)"
      - "traefik.http.services.mailhog.loadbalancer.server.port=8025"
```

## Start

```
touch acme.json && chmod 600 acme.json
docker network create proxy && docker-compose up -d
```

### References

- https://medium.com/@containeroo/traefik-2-0-docker-a-simple-step-by-step-guide-e0be0c17cfa5

