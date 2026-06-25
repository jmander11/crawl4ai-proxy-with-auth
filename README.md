# crawl4ai OpenWebUI proxy

A simple proxy server that enables [OpenWebUI](https://github.com/open-webui/open-webui)'s web page fetching feature (fetch_url) to work with [crawl4ai](https://github.com/unclecode/crawl4ai). This makes crawling faster and avoids paying for a cloud API service. 🎉

## Features

- **Auth forwarding**: Forwards `CRAWL4AI_AUTH_TOKEN` as a Bearer token to crawl4ai (required for v0.9.0+)
- **Content-Type forwarding**: Ensures `application/json` is sent to the upstream crawl4ai server
- **OpenWebUI format**: Transforms crawl4ai's response into the format OpenWebUI expects

## Usage

> **Security note**: This proxy forwards credentials to crawl4ai but does not add its own authentication layer. Do not expose it publicly without a TLS-terminating reverse proxy or other access controls.

### Docker Compose

Build and run with a similar `docker-compose.yml` as below:

```yaml
services:
    crawl4ai-proxy:
        build: .
        environment:
            - LISTEN_PORT=8000
            - CRAWL4AI_ENDPOINT=http://crawl4ai:11235/crawl
            - CRAWL4AI_AUTH_TOKEN=your_secret_token  # Must match crawl4ai's API token
        networks:
            - openwebui

    openwebui:
        image: ghcr.io/open-webui/open-webui:ollama
        ports:
            - "8080:8080"
        deploy:
            resources:
                reservations:
                    devices:
                        - driver: nvidia
                          count: all
                          capabilities: [gpu]
        networks:
            - openwebui

    crawl4ai:
        image: unclecode/crawl4ai:latest
        shm_size: 1g
        environment:
            - CRAWL4AI_API_TOKEN=your_secret_token
        networks:
            - openwebui

networks:
    - openwebui
```

Then run:

```bash
docker compose up -d --build
```

### Environment Variables

| Variable | Default | Description |
|---|---|---|
| `LISTEN_PORT` | `8000` | Port the proxy listens on |
| `LISTEN_IP` | `""` (all interfaces) | Interface to bind to |
| `CRAWL4AI_ENDPOINT` | `http://crawl4ai:11235/crawl` | URL of the crawl4ai server |
| `CRAWL4AI_AUTH_TOKEN` | `""` | Bearer token forwarded to crawl4ai (required for v0.9.0+) |

### OpenWebUI Configuration

After starting, visit your OpenWebUI instance and navigate to **Admin Panel → Web Search**. Under the "Loader" section, set:

- Web Loader Engine: `external`
- External Web Loader URL: `http://<your-host>:8000/crawl`
- External Web Loader API Key: `*` (required field but value doesn't matter)
