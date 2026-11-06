# Jade proxy

Caddy reverse proxy for the Chat API on the Jade droplet (`201.79.41.99`). Caddy obtains and renews a Let's Encrypt certificate for `API_DOMAIN`.

`API_DOMAIN` must be a hostname whose public A record is `201.79.41.99`. Use a bare IP only as the DNS target, never as the site name. Use `subratapc.net` only when that name's public A record is `201.79.41.99`.

The Chat API container `jade-ai-bot-prod` already publishes port `8080` from `/apps/jade-ai-bot/deployment`. This proxy forwards the public site to `host.docker.internal:8080`.

## DNS and firewall

1. Create an A record for your API hostname pointing at `201.79.41.99`.
2. Allow inbound TCP `80` and `443` (DigitalOcean cloud firewall, and `ufw` if it is enabled):

```bash
ufw allow 80/tcp
ufw allow 443/tcp
```

Port `8887` stays closed on the public firewall. The config server remains on the private network at `http://10.132.0.2:8887`.

## Run

Copy this folder to the droplet at `/apps/jade-proxy`, then:

```bash
cd /apps/jade-proxy
cp .env.example .env
nano .env
docker compose up -d
```

Set `API_DOMAIN` and `ACME_EMAIL` in `.env` before starting. Keep `.env` on the droplet.

## Check

```bash
docker compose logs -f caddy
curl -fsS "https://${API_DOMAIN}/actuator/health"
```

Replace `${API_DOMAIN}` with the hostname from `.env` if the shell has not loaded that file. A healthy response is JSON from the Chat API actuator.

Certificates persist in the named volume `caddy_data`, mounted at `/data` in the container (`/data/caddy/certificates`). The `caddy_config` volume holds Caddy's autosaved config. Both survive `docker compose down` without `-v`.

```bash
docker compose exec caddy ls /data/caddy/certificates
```
