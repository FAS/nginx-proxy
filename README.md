# nginx-proxy

Centralized reverse proxy with automatic Let's Encrypt SSL certificates for all FAS projects.

## Components

- **nginx-proxy** - Routes traffic to containers based on VIRTUAL_HOST
- **acme-companion** - Automatically provisions and renews Let's Encrypt certificates

## First-time Setup (on server)

```bash
git clone https://github.com/FAS/nginx-proxy.git
cd nginx-proxy
docker network create nginx-proxy
docker-compose up -d
```

Verify containers are running:
```bash
docker ps | grep nginx-proxy
```

## Adding a New Site

For any container to get automatic HTTPS, add these to its `docker run` command:

```bash
docker run --name mysite \
  --network nginx-proxy \
  -e VIRTUAL_HOST=example.com,www.example.com \
  -e LETSENCRYPT_HOST=example.com \
  myimage:tag
```

| Variable | Description |
|----------|-------------|
| `--network nginx-proxy` | Connect to the proxy network |
| `VIRTUAL_HOST` | Domain(s) to route to this container |
| `LETSENCRYPT_HOST` | Domain(s) to obtain SSL certificate for |

## WWW Redirects

To redirect www to non-www, create a file in `vhost.d/`:

```bash
# vhost.d/www.example.com
return 301 https://example.com$request_uri;
```

## Verification

```bash
# Check HTTP redirects to HTTPS
curl -I http://example.com

# Check HTTPS works
curl -I https://example.com

# Check certificate details
openssl s_client -connect example.com:443 -servername example.com
```

## Logs

```bash
# Proxy logs
docker logs nginx-proxy

# ACME companion logs (certificate operations)
docker logs nginx-proxy-acme
```

## Certificate Renewal

Certificates are automatically renewed by acme-companion before expiration. No manual intervention required.

To force renewal:
```bash
docker exec nginx-proxy-acme /app/force_renew
```
