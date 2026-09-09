# Caddy Reverse Proxy

Este documento describe la configuracion para reemplazar Traefik por Caddy usando:

- https://github.com/lucaslorentz/caddy-docker-proxy
- https://github.com/mholt/caddy-ratelimit

Incluye:

- creacion de imagen custom (`Dockerfile`)
- configuracion base (`Caddyfile`)
- cambios requeridos en `docker-compose.yml` de cada API

## 1. Dockerfile custom de Caddy

Archivo: `caddy/Dockerfile`

```dockerfile
FROM caddy:2.11-builder-alpine AS builder

RUN xcaddy build \
  --with github.com/lucaslorentz/caddy-docker-proxy/v2 \
  --with github.com/mholt/caddy-ratelimit

FROM caddy:2.11-alpine

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

### Que aporta cada plugin

- `caddy-docker-proxy`: descubre contenedores Docker por labels y genera automaticamente la config de rutas/sitios.
- `caddy-ratelimit`: agrega directiva `rate_limit` para limitar trafico por IP, token, etc.

## 2. Configuracion de Caddy (docker-compose)

Archivo: `caddy/docker-compose.yml`

Puntos importantes:

- usar imagen custom construida con el Dockerfile
- ejecutar `docker-proxy` con `--caddyfile-path` para cargar snippets globales
- montar `./caddy_file` en `/etc/caddy`
- montar `./certs` en `/etc/caddy/certs` para usar certificados locales
- montar socket Docker en modo lectura

Ejemplo:

```yaml
services:
  reverse-proxy-caddy:
    build:
      context: .
      dockerfile: Dockerfile
    image: microservicios/reverse-proxy-caddy:2.11
    command: ["caddy", "docker-proxy", "--caddyfile-path", "/etc/caddy/Caddyfile"]
    restart: unless-stopped
    environment:
      - CADDY_INGRESS_NETWORKS=mired
    networks:
      - mired
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - ./caddy_file:/etc/caddy
      - ./certs:/etc/caddy/certs:ro
      - ./data:/data
      - ./config:/config
      - /var/run/docker.sock:/var/run/docker.sock:ro

networks:
  mired:
    external: true
```

## 3. Caddyfile base (snippets reutilizables)

Archivo: `caddy/caddy_file/Caddyfile`

```caddyfile
{
	admin 0.0.0.0:2019
	order rate_limit before reverse_proxy
}

(tls-local-certs) {
  tls /etc/caddy/certs/cert.pem /etc/caddy/certs/key.pem
}

(rate-limit-defaults) {
	rate_limit {
		zone per_ip {
			key {remote_host}
			events 120
			window 1m
		}
	}
}

(resilience-proxy) {
  reverse_proxy {args[0]} {
		lb_policy least_conn
		lb_try_duration 10s
		lb_try_interval 250ms
		lb_retries 3
		fail_duration 30s
		max_fails 3
		unhealthy_status 500 502 503 504
		unhealthy_request_count 100
	}
}
```

### Cobertura funcional (equivalente Traefik -> Caddy)

- Proxy inverso: `reverse_proxy`
- Balanceo de carga: `lb_policy least_conn`
- Retry: `lb_retries`, `lb_try_duration`, `lb_try_interval`
- Circuit breaker (equivalente operativo): `fail_duration`, `max_fails`, `unhealthy_status`
- Bulkhead (control de presion/concurrencia): `unhealthy_request_count`
- Rate limit: `rate_limit` (plugin `caddy-ratelimit`)

Nota: Traefik tiene `circuitbreaker.expression` con metricas avanzadas. En Caddy no existe el mismo DSL; la aproximacion recomendada es health checking pasivo como en el snippet `resilience-proxy`.

## 4. Cambios requeridos en cada API (docker-compose)

Cada servicio API que quiera exponerse por Caddy debe:

1. estar en la red Docker `mired`
2. tener labels `caddy.*`
3. remover labels `traefik.*`

### Plantilla minima (host + rate limit + resiliencia)

```yaml
services:
  mi-api:
    image: mi-api:latest
    networks:
      - mired
    labels:
      - "caddy=mi-api.universidad.localhost"
      - "caddy.import=tls-local-certs"
      - "caddy.import_1=rate-limit-defaults"
      - "caddy.import_2=resilience-proxy {{upstreams 8080}}"

networks:
  mired:
    external: true
```

### Con ruta especifica (PathPrefix equivalente)

```yaml
services:
  admin:
    image: admin:dev
    networks:
      - mired
    labels:
      - "caddy=admin.universidad.localhost"
      - "caddy.route=/admin-main*"
      - "caddy.route.import=rate-limit-defaults"
      - "caddy.route.import_1=resilience-proxy {{upstreams 8080}}"
```

## 5. Pasos de despliegue

Desde `caddy/`:

```bash
docker compose build reverse-proxy-caddy
docker compose up -d
```

Desde cada API:

```bash
docker compose up -d
```

## 6. Verificacion

### Ver logs de Caddy

```bash
cd caddy
docker compose logs -f reverse-proxy-caddy
```

No deberia aparecer:

- `File to import not found: rate-limit-defaults`
- `Caddyfile input is not formatted`

### Probar endpoint expuesto

```bash
curl -I http://circuit.universidad.localhost/
curl -I https://circuit.universidad.localhost/
```

Respuesta esperada:

- HTTP devuelve `308 Permanent Redirect` con `Location: https://...`
- HTTPS responde normalmente con el certificado local montado en `/etc/caddy/certs`

## 7. Troubleshooting rapido

- Si falla `import ...`: validar que el comando incluya `--caddyfile-path /etc/caddy/Caddyfile`.
- Si no enruta un servicio: validar labels `caddy.*`, puerto de `{{upstreams PORT}}` y red `mired`.
- Si falla TLS local: validar existencia de `certs/cert.pem` y `certs/key.pem` en host.
- Si no funciona `rate_limit`: validar que la imagen fue construida con `github.com/mholt/caddy-ratelimit`.
- Si hay warning de formato: ejecutar `caddy fmt --overwrite /etc/caddy/Caddyfile`.
