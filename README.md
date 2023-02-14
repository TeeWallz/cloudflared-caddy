# Cloudflare Tunnel and Caddy Reverse Proxy

Sources:

- https://caddy.community/t/caddy-cloudflare-tunnel/15929
- https://www.sakowi.cz/blog/cloudflared-docker-compose-tutorial


# Create Tunnel


Log in with cloudflare and create env files
```bash
tunnel_name=nas-caddy-tunnel

docker network create cloudflared
docker network create caddy

docker run -v ./cloudflared/config:/root/.cloudflared msnelling/cloudflared cloudflared tunnel login
docker run -v ./cloudflared/config:/etc/cloudflared msnelling/cloudflared cloudflared tunnel create $tunnel_name

cp cloudflared/config/config.yml.template cloudflared/config/config.yml
cp cloudflared/.env.template cloudflared/.env
```

Bring up the tunnel
```bash
cd cloudflared
docker-compose up -d
```

# Caddy Cloudflare key
Enter a Cloudflare DNS api key id into ```caddy/.env```


# Adding a new service
Edit cloudflared/config/config.yml.template . Add:
```
  - hostname: "SUBDOMAIN.${DOMAIN}"
    service: https://caddy
    noTLSVerify: true
    originRequest:
      originServerName: "SUBDOMAIN.${DOMAIN}"
```

Add DNS Entry:
```bash
cd cloudflared
docker compose exec cloudflared cloudflared tunnel route dns nas-caddy-tunnel SUBDOMAIN.tomekwaller.com
docker compose restart
```

In the new service's docker container, add the cady network at the bottom

```yaml
networks:
  caddy:
    external: true
```

Add the following labels: to tell caddy to generate a cloudflare Certificate
```yaml
labels:
    caddy: "${CADDY_URL}"
    caddy.reverse_proxy: "{{upstreams 80}}"
```

Add the container to the caddy network, and keep the default stack network if needed
```yaml
networks:
    caddy:
        aliases:
            - "{$CADDY_ALIAS_PREFIX}-life-client"
    default:
```

Remember to add the other services to the default network as well
```yaml
networks:
    default:
```