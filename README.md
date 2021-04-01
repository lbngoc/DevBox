# DevBox

Devbox by Docker for local development

This guide is for MacOS, but will work on Linux with minor modifications.

## Requirements

- [Homebrew](https://brew.sh/)
- [Docker](https://docs.docker.com/docker-for-mac/install/)

## Included 

- Dnsmasq
- Traefik v2
- MySQL
- PostgreSQL (optional)
- Mailhog

## Configuration

- Clone this repository to your local before we start

```sh
git clone https://github.com/lbngoc/DevBox.git && \
cd DevBox
```

### 1. Setup local DNS, Root CA

- Setup MacOS to take into account our local docker resolver

```sh
sudo mkdir -p /etc/resolver && \
echo "nameserver 127.0.0.1" | sudo tee -a /etc/resolver/test > /dev/null
```

- Setup a local trusted Root CA and create a TLS certificate for using https in local (shout out to [mkcert](https://github.com/FiloSottile/mkcert)).

```sh
brew install mkcert
brew install nss # only if you use Firefox
mkcert -install
mkcert -cert-file certs/local.crt -key-file certs/local.key "this.test" "*.this.test"
```

### 2. Docker & Traefik

```sh
touch traefik/acme.json && chmod 600 acme.json && \
docker network create proxy
```

Setup dashboard authentication (optional)

- Generate `USER:PASSWORD` for access Traefik dashboard

```sh
echo $(htpasswd -nb <USER> <PASSWORD>) | sed -e s/\\$/\\$\\$/g
```

- Replace USER:PASSWORD in `docker-compose.yml:42`
- Uncomment `docker-compose.yml:41` `docker-compose.yml:42`

Or we can create new file `docker-compose.override.yml`

```yml
version: '3'

services:
  traefik:
    labels:
      - "traefik.http.middlewares.traefik-auth.basicauth.users=USER:PASSWORD"
      - "traefik.http.routers.traefik.middlewares=traefik-auth"
```

Done, we need to start Traefik at first time

```sh
docker-compose up -d
```

Go on https://traefik.this.test you should have the traefik web dashboard serve over https

## Usage

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
      - "traefik.http.routers.wordpress.rule=Host(`wordpress.this.test`)"

networks:
  default:
    external:
      name: proxy
```

Then run `docker-compose up` then open browser and go to http://wordpress.this.test


## Example

I also put a sample config to start whoami container, that will prints OS information and HTTP request to output

```sh
docker-compose -f whoami.yml up
```

Go on https://whoami.this.test to see the result

### References

- https://github.com/SushiFu/traefik-local
- https://medium.com/@containeroo/traefik-2-0-docker-a-simple-step-by-step-guide-e0be0c17cfa5
