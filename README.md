# caddy-cloudflare

A custom [Caddy](https://caddyserver.com/) build with the [Cloudflare DNS plugin](https://github.com/caddy-dns/cloudflare) pre-installed, for use in a homelab. The image is built automatically via GitHub Actions and published to the GitHub Container Registry (GHCR).

## Why

The Cloudflare DNS plugin enables Caddy to solve ACME DNS-01 challenges, allowing you to obtain TLS certificates for internal services without exposing port 80 to the internet.

## Versions

Versions are pinned in [`versions.env`](./versions.env):

```env
CADDY_VERSION=2.9.1
CLOUDFLARE_PLUGIN_VERSION=v0.2.3
```

To update, edit that file and push a new tag to trigger a build:

```bash
# Edit versions.env, then:
git add versions.env
git commit -m "chore: bump to caddy 2.x.x"
git tag v2.x.x
git push origin main --tags
```

The `CLOUDFLARE_PLUGIN_VERSION` must be a Go module pseudo-version. To get the latest:

```bash
curl https://proxy.golang.org/github.com/caddy-dns/cloudflare/@latest
```

## Usage

Pull the image from GHCR:

```bash
docker pull ghcr.io/<owner>/caddy-cloudflare:latest
```

Reference it in your `docker-compose.yml`:

```yaml
services:
  caddy:
    image: ghcr.io/<owner>/caddy-cloudflare:latest
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
    environment:
      - CLOUDFLARE_API_TOKEN=your_token_here

volumes:
  caddy_data:
  caddy_config:
```

Configure DNS-01 challenge in your `Caddyfile`:

```
example.internal {
  tls {
    dns cloudflare {env.CLOUDFLARE_API_TOKEN}
  }
  reverse_proxy localhost:8080
}
```

## Image tags

| Tag | Description |
|-----|-------------|
| `latest` | Latest build from `main` |
| `v2.9.1` | Exact Caddy release version |
| `2.9` | Major.minor |
| `sha-<commit>` | Specific commit |
