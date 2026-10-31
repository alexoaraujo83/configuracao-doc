# Operação, monitoramento e recuperação — somente Free

**Status:** runbooks documentais, não executados. Abrange logs, métricas, quotas, incidentes e recuperação na stack Next.js + Auth0 + Neon + GitHub + Vercel.

## Documentos
- 01 — Observabilidade e logs
- 02 — Quotas Free e capacidade
- 03 — Incidentes e resposta
- 04 — Backup, restore e continuidade
- 05 — Rotina operacional e checklist

## Princípios
- Monitorar antes de atingir limites; não presumir alertas avançados incluídos no Free.
- Não armazenar tokens, cookies, secrets ou dados pessoais em logs.
- Tratar limites de quota como limites operacionais, não SLAs.
- Testar restauração em ambiente isolado antes de depender dela.
- Documentar incidentes e ações sem executar mudanças sem autorização.

## Referências oficiais consultadas em 31/10/2026
- Vercel Limits: https://vercel.com/docs/limits
- Vercel Hobby: https://vercel.com/docs/plans/hobby
- Vercel Runtime Logs: https://vercel.com/docs/logs/runtime
- Neon Free quotas: https://github.com/neondatabase/website/blob/main/content/faqs/free-plan-limits-and-quotas.md
- Neon Plans: https://neon.com/docs/introduction/plans
- Neon history window: https://neon.com/docs/postgres/backup-restore/history-window
- Neon Monitoring: https://neon.com/docs/introduction/monitoring
- Auth0 log retention: https://auth0.com/docs/deploy-monitor/logs

Nenhum monitoramento, backup, restore ou incidente foi executado nesta etapa.