```
This is a fork from from [clnhlzmn/memos-public-proxy](https://github.com/clnhlzmn/memos-public-proxy)
```
# Memos Public Proxy

Share your public [Memos](https://github.com/usememos/memos) in a safe way without exposing your Memos server to the public (inspired by [Immich Public Proxy](https://github.com/alangrainger/immich-public-proxy)).

[Live demo](https://memos.share.holzman.ch/memos/o7nBhM3gUPUqaxLBgxCj3X)

# Content

- [Memos Public Proxy](#memos-public-proxy)
- [Content](#content)
- [About](#about)
- [Usage](#usage)
  - [Create a public memo](#create-a-public-memo)
  - [Copy the link](#copy-the-link)
  - [Share it](#share-it)
- [Installation](#installation)
  - [Settings](#settings)
- [Example Docker Compose File](#example-docker-compose-file)
  - [Example Caddy Config](#example-caddy-config)
- [Dev Notes](#dev-notes)
  - [Building](#building)
  - [Running locally](#running-locally)



# About

I was inspired by the approach taken by Immich Public Proxy and I wanted something similar for Memos. The Memos app already has a concept of public and private visibility, and memos are identified by long random strings. What Memos Public Proxy does is provide a locked down route for the public to access those public memos without exposing the rest of the Memos instance (auth, api, etc..).

# Usage

## Create a public memo

![Create a memo](docs/SCR-20250801-mwyg.png)

## Copy the link

![Copy the link](docs/SCR-20250801-mwzq.png)

## Share it

![Share it](docs/SCR-20250801-myat.png)

# Installation

## Settings

| Variable | Description | Default |
| --- | --- | --- |
| `MEMOS_LOG_LEVEL` | Log level for the proxy application and for the Gunicorn server. Uses [Gunicorn levels](https://docs.gunicorn.org/en/stable/settings.html#loglevel). | `error` |
| `MEMOS_HOST` | The URL for the Memos server instance. | http://memos:5230 |

# Example Docker Compose File

> Note: Memos 0.25.1 includes support for using `MEMOS_INSTANCE_URL` when copying a memo link using the "Copy Link" button. What does this mean? You can visit your Memos instance at memos.private.example.com, create a public memo, click "Copy Link", and have a ready-to-share public link of the form `memos.public.example.com/memos/<memo id>`.

```yaml
services:
  memos:
    image: neosmemo/memos:stable
    container_name: memos
    ports:
      - 5230:5230
    volumes:
      - ./memos:/var/opt/memos
    environment:
      - MEMOS_MODE=prod
      - MEMOS_PORT=5230
      - MEMOS_INSTANCE_URL=${MEMOS_INSTANCE_URL}
    networks:
      - memosnet
    restart: unless-stopped
  memogram:
    env_file: .env
    build: ./memogram
    container_name: memogram
    networks:
      - memosnet
  memos_public_proxy:
    image: ghcr.io/clnhlzmn/memos-public-proxy:main
    restart: unless-stopped
    environment:
      MEMOS_LOG_LEVEL: info
    ports:
      - 8467:5000
    networks:
      - memosnet
networks:
  memosnet:
    driver: bridge

```

## Example Caddy Config

```yaml
*.private.example.com {
    @denied not remote_ip private_ranges
    abort @denied

    # The following proxies memos.private.example.com to
    # the Memos server (accessible from local IPs only).
    @memos host memos.private.example.com
    reverse_proxy @memos 127.0.0.1:5230
}

*.public.example.com {
    # The following proxies memos.public.example.com to
    # the sharedmemos server (accessible from any IP).
    @memos host memos.public.example.com
    reverse_proxy @memos 127.0.0.1:8467
}
```

# Dev Notes

## Building

`docker build -t sharedmemos .`

## Running locally

`docker run --rm --network host -e MEMOS_HOST=<memos host> -e MEMOS_PORT=80 sharedmemos`
