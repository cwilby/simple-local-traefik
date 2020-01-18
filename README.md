# Simple Local Traefik

[Traefik](https://docs.traefik.io/) is an open-source edge router that intelligently proxies incoming network traffic to other services. You could run a hosting company with it, or run your local apps with `.test` domains and full HTTPS.

This project focuses on the simplest steps needed to setup a local Traefik environment so you can start testing with real domain names. Whether for vanity or otherwise.

## Instructions

1. Install [Docker](https://docs.docker.com/docker-for-mac/install/) and [Homebrew](https://brew.sh/).

2. Add `traefik.test` and any desired `.test` domains to `/etc/hosts`.

   ```bash
   127.0.0.1  traefik.test
   127.0.0.1  my-site.test
   ```

3. Create an external Docker network called `web` that will be used to connect traefik to other services (e.g. a Docker container running an nginx image).

   ```bash
   docker network create web
   ```

4. Install [`mkcert`](https://github.com/FiloSottile/mkcert) (and [`nss`](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS) if you're using Firefox).

   ```bash
   brew install mkcert
   brew install nss
   ```

5. Setup a local root certificate authority (CA) using `mkcert`.

   ```bash
   mkcert -install
   ```

6. Clone this repository

   ```
   git clone https://github.com/cwilby/simple-local-traefik.git
   cd simple-local-traefik
   ```

7. Create a TLS certificate/key file for `traefik.test` and store in `certs` folder within the repository. Repeat for each of your domains.

   ```bash
   mkcert -cert-file certs/local.crt -key-file certs/local-key.pem "traefik.test"
   ```

8. Start traefik and verify that [`https://traefik.test/dashboard/`](https://traefik.test/dashboard/) works. **Note that the final `/` is mandatory**.

   ```sh
   docker-compose up -d
   ```

9. To start a Docker container connected to Traefik:

   ```sh
   docker run nginx:latest \
     --network web \
     --label "traefik.enable=true" \
     --label "traefik.docker.network=web" \
     --label "traefik.http.routers.my-site.entryPoints=https" \
     --label "traefik.http.routers.my-site.rule=Host(`my-site.test`)" \
     --label "traefik.http.routers.my-site.tls=true"
   ```

10. To do the same in a seperate docker-compose file:

    ```yml
    version: "3.7"

    networks:
      web:
        external: true

    services:
      my-site:
        image: nginx:latest
        networks:
          - web
        labels:
          - traefik.enable=true
          - traefik.docker.network=web
          - traefik.http.routers.my-site.entryPoints=https
          - traefik.http.routers.my-site.rule=Host(`my-site.test`)
          - traefik.http.routers.my-site.tls=true
    ```

## Frequently Asked Questions

### What if a Docker container uses a port other than port 80?

You can add a label to your Docker container to set the port, but it's better to let Traefik auto-find the port by looking at the ports exposed by the container.

In other words, if you have a Docker container that exposes port 8080, Traefik will know to use port 8080.

## Contributing

Please submit issues if you think something can be improved, and preferably pull requests if you have the time.
