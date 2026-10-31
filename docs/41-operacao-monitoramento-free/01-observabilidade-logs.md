# 01 — Observabilidade e logs

## Sinais mínimos
- Aplicação: request id, latência, status HTTP, taxa de erro.
- Auth0: sucesso/falha de login, callbacks, bloqueios e eventos administrativos conforme retenção disponível.
- Neon: compute, storage, conexões, egress, latência e estado de compute.
- Vercel: build/deploy status, runtime errors, duração e consumo.
- GitHub: workflow failures, minutos e alertas de dependências.

## Logging seguro
Registrar evento, timestamp, request_id, módulo, status, latência e categoria de erro. Não registrar cookie, Authorization, token OIDC, client secret, AUTH0_SECRET, DATABASE_URL, SQL parameters sensíveis ou payload pessoal.

## Limites Vercel Hobby
A documentação atual indica retenção de runtime logs de 1 hora e até 4.000 linhas. Build logs têm retenção distinta. Exportação avançada/log drains não deve ser pressuposta no Hobby. Capturar evidências operacionais necessárias dentro da janela, sem criar cópias de dados pessoais.

## Neon
O painel de Monitoring mostra CPU, RAM, conexões e tamanho de banco. Retenção Free indicada: 1 dia. Metrics export é restrito a planos superiores; não basear operação Free em export externo integrado.

## Auth0
Verificar a janela de retenção do tenant/plano no Dashboard. Logs expirados podem ser permanentemente removidos; não prometer recuperação após expiração. Evitar dependência de Log Streams/add-ons sem verificar entitlement Free.

Fontes: https://vercel.com/docs/logs/runtime ; https://neon.com/docs/introduction/plans ; https://auth0.com/docs/deploy-monitor/logs