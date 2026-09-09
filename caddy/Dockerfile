FROM caddy:2.11-builder-alpine AS builder

RUN xcaddy build \
  --with github.com/lucaslorentz/caddy-docker-proxy/v2 \
  --with github.com/mholt/caddy-ratelimit

FROM caddy:2.11-alpine

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
