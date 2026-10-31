# Observabilidade, monitoramento e capacidade — somente Free

> Estado: **Recomendado**. Pesquisa documental verificada em 2026-10-31. Não representa telemetria coletada nem configuração aplicada.

## Objetivo

Definir um modelo operacional sem dependência de SaaS pago para observar a stack Next.js + TypeScript, Auth0, Neon PostgreSQL, GitHub Actions e Vercel Hobby.

## Princípios

1. Usar primeiro os painéis e logs nativos incluídos nos planos Free.
2. Instrumentar a aplicação com logs estruturados JSON e identificadores de correlação.
3. Medir tendências por janela temporal e comparar com limites publicados no painel, não com números presumidos.
4. Alertas no Free podem depender de revisão manual e notificações nativas; não presumir alertas configuráveis, exportação de logs ou retenção histórica longa.
5. Não registrar tokens, cookies, senhas, connection strings, dados pessoais desnecessários ou payloads sensíveis.
6. Vercel Hobby é restrito a projetos pessoais e não comerciais. Não usar como base de SaaS comercial.

## Conteúdo

- [Modelo de telemetria e logs Next.js](01-nextjs-logs-correlacao.md)
- [Auth0: autenticação, erros e sessões](02-auth0-monitoramento.md)
- [Neon PostgreSQL: métricas e capacidade](03-neon-metricas-capacidade.md)
- [Vercel Hobby: builds, runtime e deployments](04-vercel-logs-deployments.md)
- [GitHub Actions: execução e segurança](05-github-actions-monitoramento.md)
- [SLI/SLO, quotas e alertas operacionais](06-indicadores-alertas-quotas.md)
- [Dashboards e runbooks de diagnóstico](07-dashboards-runbooks.md)
- [Validação e gate de observabilidade](08-checklist-gate.md)

## Arquitetura operacional

- Aplicação: emite logs JSON no stdout/stderr; cada request tem requestId e traceId quando disponível.
- Auth0: consultar Monitoring > Logs; usar filtros temporais e de tipo; exportação via Management API somente em volume moderado e com paginação/limites.
- Neon: painel Monitoring e Usage por projeto; registrar métricas e consumo manualmente em uma planilha versionada sem dados sensíveis.
- Vercel: Build Logs por deployment, Runtime Logs no projeto e Observability nativa dentro dos limites do Hobby.
- GitHub: Actions > workflow > run > job logs; Security tab conforme elegibilidade do repositório.

## Limitações Free confirmadas na pesquisa

- Auth0 Free: página oficial de preços consultada informa 1 dia de retenção de logs; Log Streaming não consta como incluído no Free. Conferir novamente no tenant e na tabela atual antes de depender disso.
- Neon Free: comunicação oficial publicada em 02-10-2026 informa 100 CU-hours por projeto/mês, 1 GB de storage por projeto, até 2 CU de autoscaling, 10 branches e janela de instant restore de 6 horas. A elegibilidade e os valores devem ser confirmados na conta.
- Vercel Hobby: Runtime Logs com 1 hora de retenção e limite de linhas; Observability tem retenção própria de 12 horas. Build Logs ficam associados aos deployments. O Hobby é não comercial.
- GitHub Free: limites de Actions e armazenamento de artefatos dependem do tipo de repositório/conta. A documentação atual de limites deve ser consultada no painel e na referência oficial.

## Fontes oficiais

- [Next.js Instrumentation](https://nextjs.org/docs/app/guides/instrumentation)
- [Next.js OpenTelemetry](https://nextjs.org/docs/15/pages/guides/open-telemetry)
- [Auth0 Pricing](https://auth0.com/pt/pricing)
- [Auth0 Logs](https://auth0.com/docs/deploy-monitor/logs)
- [Neon Monitoring](https://neon.com/docs/introduction/monitoring)
- [Neon Free Plan — atualização 02-10-2026](https://neon.com/blog/neon-free-plan-1-gb-per-project)
- [Vercel Runtime Logs](https://vercel.com/docs/logs/runtime)
- [Vercel Observability](https://vercel.com/docs/observability)
- [Vercel Hobby](https://vercel.com/docs/plans/hobby)
- [GitHub Actions limits](https://docs.github.com/en/actions/reference/limits)
- [GitHub Actions workflow monitoring](https://docs.github.com/en/actions/how-tos/monitor-workflows)

## Critério de adoção

A documentação deve ser adaptada ao projeto real e validada com evidência de painel. Não habilitar integrações externas pagas, Log Drains, Observability Plus, log streams não incluídos, nem provedores externos de monitoramento com cobrança.
