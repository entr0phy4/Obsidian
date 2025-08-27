Ready for production.

**Dockerfile**:

```yml
# Dockerfile
FROM quay.io/keycloak/keycloak:26.3.3 AS builder

ENV KC_DB=postgres \
    KC_HEALTH_ENABLED=true \
    KC_METRICS_ENABLED=true

WORKDIR /opt/keycloak

# Build an optimized server image
RUN /opt/keycloak/bin/kc.sh build

FROM quay.io/keycloak/keycloak:26.3.3
COPY --from=builder /opt/keycloak/ /opt/keycloak/
ENTRYPOINT ["/opt/keycloak/bin/kc.sh"]
```

**docker-compose.yml**:

```yml
version: "3.9"

services:
  postgres:
    image: postgres:17-alpine
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?set in .env}
    volumes:
      - pg_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U keycloak -d keycloak"]
      interval: 10s
      timeout: 3s
      retries: 30
    restart: unless-stopped

  keycloak:
    build: .
    command: ["start"]
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      # DB (use a private network; do not expose Postgres)
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: ${POSTGRES_PASSWORD:?set in .env}

      # Hostname and proxy (edge TLS terminated at your LB/proxy)
      KC_HOSTNAME: ${KC_HOSTNAME}
      KC_HOSTNAME_ADMIN: ${KC_HOSTNAME_ADMIN:-}
      KC_HTTP_ENABLED: "true"
      KC_PROXY_HEADERS: xforwarded
      KC_PROXY_TRUSTED_ADDRESSES: ${KC_PROXY_TRUSTED_ADDRESSES:-127.0.0.1/32,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16}

      # Optional hardening/tuning
      KC_HTTP_MAX_QUEUED_REQUESTS: ${KC_HTTP_MAX_QUEUED_REQUESTS:-1000}
      JAVA_OPTS_APPEND: "-Djava.net.preferIPv4Stack=true"

      # First boot admin (remove after first start)
      KC_BOOTSTRAP_ADMIN_USERNAME: ${KC_BOOTSTRAP_ADMIN_USERNAME:-admin}
      KC_BOOTSTRAP_ADMIN_PASSWORD: ${KC_BOOTSTRAP_ADMIN_PASSWORD:?set in .env}

      # Health/metrics (also enabled at build time)
      KC_HEALTH_ENABLED: "true"
      KC_METRICS_ENABLED: "true"
    ports:
      - "127.0.0.1:8080:8080"   # reverse proxy -> 8080 (HTTP, edge TLS)
      - "127.0.0.1:9000:9000"   # local health/metrics only; do NOT expose publicly
    healthcheck:
      # Simple TCP check for management port; do not require curl
      test: ["CMD-SHELL", "timeout 2 bash -lc '</dev/tcp/127.0.0.1/9000'"]
      interval: 10s
      timeout: 3s
      retries: 30
    restart: unless-stopped

volumes:
  pg_data:
```

**.env**:

```
POSTGRES_PASSWORD='strong-db-pass'
KC_HOSTNAME='https://keycloak.tst'
KC_BOOTSTRAP_ADMIN_PASSWORD='strong-admin-pass'
```

**Nginx**:

```
server {
  listen 127.0.0.1:80;
  server_name keycloak.tst;
  return 301 https://$host$request_uri;
}

server {
  listen 127.0.0.1:443 ssl http2;
  server_name keycloak.tst;

  ssl_certificate     /etc/ssl/localcerts/keycloak.tst.pem;
  ssl_certificate_key /etc/ssl/localcerts/keycloak.tst-key.pem;

  location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
  }
}
```