---
name: faro-devops
description: |
  Pattern DevOps: Docker, CI/CD, deploy, monitoring, infrastruttura.
  Trigger: "docker", "deploy", "ci/cd", "pipeline", "infrastruttura", "monitoring setup"
---

# faro-devops — Docker · CI/CD · Deploy · Monitoring

**Lingua:** Sempre italiano. Consulta sempre la doc ufficiale prima di scrivere config:
- Docker: https://docs.docker.com/build/building/best-practices
- GitHub Actions: https://docs.github.com/en/actions
- Nginx: https://nginx.org/en/docs

## QUANDO USARE

- Deploy a produzione (VPS, Vercel, Railway, Render)
- Configurazione Docker + docker-compose
- Setup CI/CD con GitHub Actions
- Configurazione Nginx + SSL Let's Encrypt
- Setup monitoring (Prometheus, Grafana, Sentry)

## FLUSSO DECISIONALE — Piattaforma

```
Tipo progetto → Piattaforma raccomandata
Next.js/Astro  → Vercel (zero config, CI/CD integrato)
Node.js/Python → Railway o Render (PaaS semplice)
Docker custom  → VPS + Docker Compose (controllo totale)
Mobile         → EAS Build + App Store/Play Store
```

## FLUSSO OPERATIVO

1. **Scegli piattaforma** in base al tipo progetto (vedi tabella sopra)
2. **Configura env production** — separate da staging, mai in git
3. **Dockerfile multi-stage** — build layer + runtime layer, user non-root
4. **CI/CD pipeline**: lint → typecheck → test → build → deploy
5. **Health check** `/api/health` implementato prima del deploy
6. **Monitoring** attivo prima del lancio (uptime + error rate minimi)

## REGOLE CHIAVE

1. **Mai usare `latest` per Docker tag** — usa sempre semver + SHA (`v1.2.0` + `sha-abc123`)
2. **Backup PRIMA di ogni migration DB** — `pg_dump` obbligatorio
3. **Staging identico a prod** — solo le chiavi sono diverse, mai il codice
4. **Smoke test dopo ogni deploy** — verifica `/api/health` entro 10 secondi
5. **Mai testare su prod** — usa staging per ogni verifica
6. **`output: 'standalone'`** in `next.config.js` obbligatorio per Docker
7. **Tutti e 3 i COPY** nel Dockerfile Next.js: standalone + static + public
8. **Variabili d'ambiente** in `.env.production` (permessi 600), mai in git
9. **HSTS + security headers** su Nginx sempre attivi
10. **Rinnovo SSL automatico** via crontab certbot

## CHECKLIST PRE-DEPLOY

- [ ] Piattaforma scelta (Vercel / Railway / VPS+Docker)
- [ ] Variabili env production configurate e verificate
- [ ] DB production separato da staging
- [ ] Docker build: tutti i COPY presenti, non gira come root
- [ ] Nginx: SSL, HSTS, rate limiting su auth
- [ ] CI/CD: test → build → push → deploy
- [ ] Health check endpoint `/api/health` implementato
- [ ] Backup DB automatico configurato
- [ ] Monitoring attivo (almeno uptime + error rate)
- [ ] Rollback testato (sai come tornare indietro)

## TABELLA — Quando usare cosa

| Scenario | Strumento | Note |
|---|---|---|
| Next.js semplice | Vercel | Zero config, CI integrato |
| Node/Python API | Railway | PaaS, db incluso |
| Full control | VPS + Docker | Costi fissi, più lavoro |
| Mobile | EAS Build | Profili dev/preview/production |
| Pipeline CI | GitHub Actions | Gratis su repo pubblici |
| Monitoring base | Uptime Robot | Gratis fino a 50 monitor |
| APM | Sentry | Error tracking + performance |

## REFERENCES

Per dettagli tecnici completi, leggi:
- [references/devops-patterns.md] — Dockerfile, docker-compose, GitHub Actions, Nginx, SSL, monitoring completo
