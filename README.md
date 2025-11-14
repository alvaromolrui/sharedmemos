```
This is a fork from [clnhlzmn/memos-public-proxy](https://github.com/clnhlzmn/memos-public-proxy)
```

# Sharedmemos

Share your public [Memos](https://github.com/usememos/memos) in a safe way without exposing your Memos server to the public

# Installation

## Settings

| Variable | Description | Default |
| --- | --- | --- |
| `MEMOS_LOG_LEVEL` | Log level for the proxy application and for the Gunicorn server. Uses [Gunicorn levels](https://docs.gunicorn.org/en/stable/settings.html#loglevel). | `error` |
| `MEMOS_HOST` | The URL for the Memos server instance. | http://memos:5230 |

# Example Docker Compose File

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
  sharedmemos:
    image: sharedmemos
    container_name: sharedmemos
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
# Dev Notes

## Building

`docker build -t sharedmemos .`

