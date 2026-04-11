# DevOps Patterns — Riferimento Tecnico

Pattern completi per Docker, CI/CD, Nginx e monitoring in progetti Faro.

## DOCKERFILE — Next.js multi-stage

```dockerfile
# syntax=docker/dockerfile:1.6
FROM node:20-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --frozen-lockfile

FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

# Utente non-root
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# I 3 COPY obbligatori per standalone
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
COPY --from=builder --chown=nextjs:nodejs /app/public ./public

USER nextjs
EXPOSE 3000
ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/api/health || exit 1

CMD ["node", "server.js"]
```

**Requisito `next.config.js`:**
```js
module.exports = { output: 'standalone' }
```

## DOCKER-COMPOSE — Stack completa

```yaml
version: '3.9'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: faro-app:${TAG:-latest}
    restart: unless-stopped
    env_file: .env.production
    depends_on:
      db:
        condition: service_healthy
    ports:
      - "127.0.0.1:3000:3000"
    networks: [internal]

  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./backups:/backups
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "${DB_USER}"]
      interval: 10s
    networks: [internal]

  nginx:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/letsencrypt:ro
    depends_on: [app]
    networks: [internal]

volumes:
  pgdata:

networks:
  internal:
    driver: bridge
```

## GITHUB ACTIONS — CI/CD completo

`.github/workflows/deploy.yml`:

```yaml
name: Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm test
      - run: npm run build

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build and push Docker image
        env:
          REGISTRY: ${{ secrets.REGISTRY_URL }}
        run: |
          docker build -t $REGISTRY/faro-app:${{ github.sha }} .
          docker build -t $REGISTRY/faro-app:latest .
          echo "${{ secrets.REGISTRY_PASSWORD }}" | \
            docker login $REGISTRY -u ${{ secrets.REGISTRY_USER }} --password-stdin
          docker push $REGISTRY/faro-app:${{ github.sha }}
          docker push $REGISTRY/faro-app:latest

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd /opt/faro-app
            docker compose pull
            docker compose up -d
            sleep 5
            curl -f http://localhost:3000/api/health || (docker compose logs && exit 1)
```

## NGINX — Config produzione

`nginx.conf`:

```nginx
worker_processes auto;

events {
  worker_connections 1024;
}

http {
  include /etc/nginx/mime.types;
  sendfile on;
  tcp_nopush on;
  gzip on;
  gzip_types text/plain text/css application/json application/javascript;

  limit_req_zone $binary_remote_addr zone=auth:10m rate=5r/m;
  limit_req_zone $binary_remote_addr zone=api:10m rate=100r/m;

  upstream app {
    server app:3000;
    keepalive 32;
  }

  server {
    listen 80;
    server_name faro.example.com;
    return 301 https://$host$request_uri;
  }

  server {
    listen 443 ssl http2;
    server_name faro.example.com;

    ssl_certificate /etc/letsencrypt/live/faro.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/faro.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;

    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';" always;

    client_max_body_size 10M;

    location /api/auth {
      limit_req zone=auth burst=5 nodelay;
      proxy_pass http://app;
      include /etc/nginx/proxy_params;
    }

    location /api {
      limit_req zone=api burst=20 nodelay;
      proxy_pass http://app;
      include /etc/nginx/proxy_params;
    }

    location / {
      proxy_pass http://app;
      include /etc/nginx/proxy_params;
    }
  }
}
```

## SSL LET'S ENCRYPT — Setup iniziale

```bash
# Installazione certbot
sudo apt install -y certbot python3-certbot-nginx

# Primo rilascio (con nginx attivo su :80)
sudo certbot --nginx -d faro.example.com --non-interactive --agree-tos -m admin@example.com

# Rinnovo automatico (già configurato in systemd timer)
sudo systemctl status certbot.timer
```

## HEALTH CHECK — `/api/health`

```ts
// app/api/health/route.ts
import { NextResponse } from 'next/server'
import { db } from '@/lib/db'

export async function GET() {
  try {
    await db.$queryRaw`SELECT 1`
    return NextResponse.json({
      status: 'ok',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
    })
  } catch (error) {
    return NextResponse.json(
      { status: 'error', error: String(error) },
      { status: 503 }
    )
  }
}
```

## MONITORING — Sentry + Uptime

```ts
// sentry.server.config.ts
import * as Sentry from '@sentry/nextjs'

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  tracesSampleRate: 0.1,
  environment: process.env.NODE_ENV,
  beforeSend(event) {
    // Maschera PII
    if (event.user) {
      delete event.user.email
      delete event.user.ip_address
    }
    return event
  },
})
```

**Uptime monitoring (Uptime Robot / Better Stack):**
- Monitor `https://faro.example.com/api/health` ogni 1 minuto
- Alert email + Slack su 2 fail consecutivi
- SSL certificate expiry alert (30 giorni prima)

## BACKUP DB AUTOMATICO

```bash
#!/bin/bash
# /opt/faro-app/scripts/backup.sh
set -euo pipefail

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="/opt/faro-app/backups/db_${TIMESTAMP}.sql.gz"

docker compose exec -T db pg_dump -U "${DB_USER}" "${DB_NAME}" \
  | gzip > "${BACKUP_FILE}"

# Mantieni ultimi 30 giorni
find /opt/faro-app/backups -name "db_*.sql.gz" -mtime +30 -delete

# Upload offsite (S3/R2)
aws s3 cp "${BACKUP_FILE}" "s3://faro-backups/db/"
```

Crontab:
```
0 3 * * * /opt/faro-app/scripts/backup.sh >> /var/log/faro-backup.log 2>&1
```

## DEPLOY CHECKLIST

- [ ] `.env.production` presente e con permessi `600`
- [ ] Dockerfile `USER` non-root
- [ ] `docker compose config` valido
- [ ] Nginx SSL attivo, HSTS preload, rate limiting
- [ ] `/api/health` testato
- [ ] Backup cron attivo e testato (restore incluso)
- [ ] Monitoring uptime + errori configurato
- [ ] DNS puntato, TTL basso per emergenze
- [ ] Rollback plan scritto (tag previous + `docker compose up -d`)
- [ ] Smoke test post-deploy: login → operazione principale → logout
