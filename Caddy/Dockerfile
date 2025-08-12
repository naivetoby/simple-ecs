FROM caddy:2.10.0-builder-alpine AS builder

RUN go env -w GOPROXY=https://goproxy.io,direct

RUN xcaddy build --with github.com/caddy-dns/alidns

FROM caddy:2.10.0-alpine

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
