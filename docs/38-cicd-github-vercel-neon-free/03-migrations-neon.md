# 03 — Migrations e Neon no CI/CD

## Princípio
Migrations alteram estado do banco e não devem rodar automaticamente em qualquer Preview ou PR com credenciais privilegiadas.

## Fluxo seguro
1. Criar migration SQL versionada e revisável.
2. CI faz validação estática, lint SQL e revisão de naming/ordem.
3. Se houver ambiente disponível, testar em branch Neon descartável, sem dados pessoais de produção.
4. Revisar impacto de lock/downtime.
5. Executar migration aprovada com role migrator e conexão apropriada.
6. Validar schema e integridade.
7. Promover código somente após compatibilidade do schema.
8. Remover branches temporárias conforme política, sem exceder quota Free.

## Expand-contract
- Expand: adicionar coluna/tabela/índice compatível.
- Deploy de código que entende schema antigo e novo.
- Backfill em lotes controlados.
- Migrar leituras/escritas.
- Contract: remover objetos antigos em release posterior.

## Credenciais
- DATABASE_URL de Preview aponta para branch não produtiva.
- CI de PR não recebe DATABASE_URL de produção.
- app_runtime nunca executa DDL.
- app_migrator é usado somente em procedimento aprovado.
- Conexão direta para migration quando necessário; pooled para runtime serverless.
- Não imprimir connection string nem SQL com dados sensíveis.

## Transação e atomicidade
Nem toda migration é transacional (por exemplo, certos comandos de índice). Marcar cada migration como transacional ou não transacional, definir prechecks, pós-checks e compensação. Não prometer rollback automático de DDL sem verificar semântica PostgreSQL.

## Branches Neon e limites Free
Neon branching é útil para testes isolados, mas branches, compute, storage e retenção têm quotas. Não criar branch para todo PR sem política de expiração e orçamento. Se não houver capacidade Free, documentar ausência de integração real.

## Exemplo de workflow somente de validação

~~~yaml
name: Migration review
on:
  pull_request:
    paths:
      - "database/migrations/**"
permissions:
  contents: read
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<REVIEWED_FULL_COMMIT_SHA>
        with:
          persist-credentials: false
      - name: List migration files
        run: find database/migrations -maxdepth 1 -type f -name "*.sql" -print | sort
      - name: Reject empty migrations
        run: |
          find database/migrations -maxdepth 1 -type f -name "*.sql" -size 0 -print -quit | grep . && exit 1 || true
~~~

Este gate não valida SQL semanticamente. Adicionar linter/test harness somente após escolher ferramentas compatíveis sem dependência paga.

## Rollback
Preferir migration corretiva forward quando dados já foram alterados. Scripts down são arriscados quando destrutivos ou não reversíveis. Backup/PITR não é plano de rollback por si só.
