# Dashboards e runbooks de diagnóstico Free-only

## Painel operacional mínimo

Manter um registro manual privado com data/hora UTC, versão/commit/deployment, status do CI, status do deployment, eventos Auth0 relevantes, Neon storage/compute/conexões, taxa de erro/latência, quotas, incidentes e responsável. Não incluir logs brutos, tokens, PII ou connection strings.

## Rotina inicial em incidente

1. Definir início e impacto percebido.
2. Obter requestId, rota, status, deployment SHA e ambiente.
3. Localizar o domínio afetado: Next.js, Auth0, Neon, Vercel ou GitHub Actions.
4. Consultar logs nativos dentro da retenção disponível.
5. Comparar com mudanças recentes.
6. Mitigar com a menor alteração reversível.
7. Validar smoke tests e registrar causa-raiz.

## Runbook: aumento de latência

- Agrupar por rota e deployment.
- Separar tempo de aplicação, Auth0, Neon e dependências externas.
- Verificar p95 e tamanho da amostra.
- Consultar queries lentas, conexões e locks.
- Corrigir via PR e validar em ambiente não produtivo.

## Runbook: falhas de autenticação

- Identificar client/application e intervalo.
- Comparar erros de callback, connection, credencial, refresh e rate limit.
- Verificar alterações recentes de URL/configuração.
- Não registrar tokens nem desligar proteções para restaurar serviço.
- Em suspeita de abuso, seguir resposta a incidente.

## Runbook: Neon sem conexão

- Confirmar status do projeto/compute no Console.
- Conferir host, database, role e ambiente sem divulgar valores.
- Verificar endpoint pooled/direct e SSL.
- Medir conexões e transações ativas.
- Conferir limites de compute/usage.
- Não conceder superuser ou BYPASSRLS à aplicação.

## Runbook: deployment falhou

- Abrir Build Logs e identificar o primeiro erro.
- Conferir commit, Node, lockfile e nomes das env vars.
- Confirmar se alteração de variável exige novo deployment.
- Reproduzir localmente com valores fictícios.
- Corrigir em branch e validar CI.

## Runbook: suspeita de credencial exposta

Seguir [modelagem de ameaças e resposta a incidentes](../21-modelagem-ameacas-incidentes-free/README.md). Não colar secrets em evidência. Rotação real depende de autorização e deve ocorrer no painel do fornecedor.

## Referências cruzadas

- [Continuidade e recuperação Free](../22-continuidade-recuperacao-free/README.md)
- [Segurança, RBAC e RLS](../20-seguranca-rbac-rls-free/README.md)
- [Testes e CI](../17-testes-rbac-migrations/README.md)
- [Inventário de painéis](../14-inventario-paineis/README.md)
