# Vercel Hobby: build logs, runtime logs e deployments

## Fontes nativas

- Build Logs: abrir o deployment e consultar a etapa de instalação/build. A documentação informa que ficam associados ao deployment e armazenados indefinidamente; logs podem ser truncados quando excedem o limite documentado.
- Runtime Logs: consultar Logs no projeto; incluem invocações de Functions e middleware, stdout/stderr e metadados da requisição.
- Observability: visão agregada de eventos e insights com limites específicos por plano.
- Deployments: acompanhar estado, commit, autor, ambiente, alias e resultado do build.

## Retenção Free/Hobby

A documentação Vercel consultada em 2026-10-31 distingue:
- Runtime Logs no Hobby: 1 hora de retenção, com limites de linhas/volume por request.
- Observability base no Hobby: 12 horas de retenção, com métricas e dimensões limitadas.
- Observability Plus, Log Drains e retenção ampliada não fazem parte da baseline Free-only.
- Build Logs permanecem associados aos deployments; confirmar limites e disponibilidade atuais no painel.

As páginas gerais de logs podem exibir números diferentes conforme atualização do produto e superfície (Runtime Logs versus Observability). Para retenção, usar a página específica do recurso e do plano, registrar a data e não prometer histórico maior.

## Rotina por deployment

1. Confirmar commit SHA e ambiente.
2. Conferir Build Logs: instalação, versão Node, warnings, build e geração de rotas.
3. Conferir se o deployment ficou Ready e se o alias esperado foi atribuído.
4. Após release, fazer smoke test de rota pública, autenticação e operação de leitura no banco.
5. Se houver 5xx, filtrar Runtime Logs por status, route, requestId e deployment.
6. Comparar com último deployment saudável e mudanças de env vars.
7. Registrar incidente e decisão de rollback.

## Limites e política Hobby

Hobby é destinado a uso pessoal e não comercial. Não deve ser adotado para aplicações comerciais/SaaS. Quotas e métricas incluídas podem mudar; conferir Limits e Usage no dashboard. Não pressupor que uma métrica detalhada ou alerta disponível em plano superior exista no Hobby.

Não depender de Log Drains, Observability Plus, Monitoring Plus ou serviço de terceiros pago. Mudanças em variáveis de ambiente exigem novo deployment para refletirem no código implantado.

## Runbook: build falhou

1. Identificar o deployment e commit.
2. Localizar a primeira falha real, distinguindo warning de erro fatal.
3. Conferir Node.js, lockfile, versão de dependências e variáveis obrigatórias sem revelar valores.
4. Reproduzir localmente com ambiente sintético.
5. Corrigir em branch e validar CI.
6. Abrir novo deployment; não editar produção manualmente.

## Runbook: 5xx após deploy

1. Identificar janela, rota, requestId e release.
2. Comparar com logs do deployment e mudanças recentes.
3. Verificar Auth0, Neon e dependências externas separadamente.
4. Avaliar rollback conforme política de continuidade e limites Hobby.
5. Após rollback, confirmar smoke tests e registrar causa-raiz.

## Referências

- https://vercel.com/docs/logs/runtime
- https://vercel.com/docs/deployments/logs
- https://vercel.com/docs/observability
- https://vercel.com/docs/observability/observability-plus
- https://vercel.com/docs/limits
- https://vercel.com/docs/plans/hobby
