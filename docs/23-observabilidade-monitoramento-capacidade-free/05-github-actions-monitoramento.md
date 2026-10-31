# GitHub Actions: falhas, segurança e histórico

## Acompanhamento de workflows

No repositório, abrir Actions e selecionar o workflow. Para cada execução, analisar:
- status geral e status por job;
- commit SHA, branch, evento disparador e ator;
- duração total e duração por job;
- etapa com falha e log correspondente;
- artifacts gerados e sua necessidade de retenção;
- reexecuções e padrão de falha recorrente.

O gráfico de execução ajuda a localizar jobs bloqueados, paralelos e dependências. Usar status badge apenas como sinal resumido; o badge não substitui análise dos logs.

CLI opcional:

    gh run list --limit 20
    gh run view RUN_ID
    gh run view RUN_ID --verbose
    gh run view --job JOB_ID --log

## Indicadores

- taxa de sucesso por workflow;
- número de falhas consecutivas;
- duração p50/p95 por workflow;
- falhas por etapa (install, lint, typecheck, test, build, deploy);
- consumo de minutos e armazenamento de artifacts/cache;
- dependabot/security alerts abertos, conforme elegibilidade;
- alterações de workflow e permissões.

## Segurança

- Definir permissions mínimas em workflow/job.
- Preferir GITHUB_TOKEN com escopo restrito.
- Fixar actions de terceiros por SHA completo quando política exigir.
- Nunca imprimir secrets ou variáveis sensíveis.
- Não executar código de pull request não confiável com secrets privilegiados.
- Usar dados sintéticos e banco isolado em CI.
- Revisar dependabot alerts e atualizar lockfile via PR.
- Separar CI de deploy e exigir aprovação humana fora de automações não autorizadas.

## Elegibilidade Free

A documentação de secret scanning consultada informa secret scanning automático gratuito para repositórios públicos; recursos para repositórios privados de usuário/organização têm requisitos diferentes e podem exigir GitHub Secret Protection/planos superiores. Não afirmar que secret scanning avançado está incluído em todo repositório privado Free.

Para Actions, consultar Usage/limits na conta e o tipo de repositório. A página oficial lista limites de execução, concorrência e armazenamento de artifacts/cache, mas cotas incluídas podem variar entre GitHub Free pessoal e organizações. Não copiar números de uma modalidade para outra sem verificar.

## Runbook: workflow falhou

1. Identificar run ID, workflow, commit, evento e job.
2. Abrir o primeiro step com erro; distinguir falha primária de falhas em cascata.
3. Verificar dependências, cache, quotas, runner e serviços externos.
4. Reproduzir com dados sintéticos e sem secrets reais.
5. Corrigir em branch, revisar e reexecutar.
6. Documentar causa, duração, correção e recorrência.

## Runbook: possível secret exposto

1. Tratar o valor como comprometido; não copiá-lo para issue, chat ou log.
2. Identificar serviço, escopo e janela de exposição.
3. Revogar/rotacionar a credencial no fornecedor com aprovação apropriada.
4. Remover o secret do fluxo e do histórico conforme procedimento de incidente.
5. Revisar Actions logs, deployments e acessos.
6. Registrar evidência minimizada e confirmar que a credencial antiga não funciona.

## Referências

- https://docs.github.com/en/actions/how-tos/monitor-workflows
- https://docs.github.com/en/actions/reference/limits
- https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/monitoring-workflows/viewing-workflow-run-history
- https://docs.github.com/en/code-security/concepts/secret-security/about-alerts
