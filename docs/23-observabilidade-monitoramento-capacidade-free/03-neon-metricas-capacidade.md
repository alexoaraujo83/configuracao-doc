# Neon PostgreSQL: métricas, conexões e capacidade

## Painéis

Por projeto, consultar Monitoring e Usage no Neon Console. A visibilidade de métricas e histórico pode variar por plano e evolução do produto. Registrar a data da observação e a janela exibida. Não assumir que o painel mantém histórico indefinido.

## Indicadores PostgreSQL

| Indicador | Sinal de risco | Diagnóstico inicial |
|---|---|---|
| CPU/compute e CU-hours | consumo acelerado ou compute ativo continuamente | verificar carga, queries e conexões ociosas |
| Storage | crescimento próximo do limite Free | identificar tabelas/índices maiores e dados temporários |
| Connections | conexões próximas do teto do compute/role | revisar pooling, concorrência e conexões idle |
| Query latency | p95/p99 crescente | EXPLAIN (ANALYZE, BUFFERS) em ambiente seguro e query sintética |
| Slow queries | consultas repetidas ou longa duração | revisar plano, índices e cardinalidade |
| Locks | bloqueios e espera | identificar transação longa e ordem de locks |
| Errors | timeout, too many connections, deadlocks | correlacionar requestId e logs da aplicação |
| Egress | crescimento anormal de tráfego | revisar payloads, paginação e chamadas repetidas |

Não executar EXPLAIN ANALYZE sobre consultas de escrita em produção sem avaliar efeitos. Preferir réplica/branch de diagnóstico ou consulta somente leitura.

## Quotas Free verificadas na data

A publicação oficial da Neon de 02-10-2026 descreve Free com 100 CU-hours por projeto/mês, 1 GB de armazenamento por projeto, autoscaling até 2 CU, 10 branches por projeto e instant restore de 6 horas. Esses valores são referência de pesquisa, não evidência da quota efetiva da conta. Confirmar Usage no Console antes de planejar capacidade.

A janela de instant restore não equivale a backup externo nem a SLA. Ver seção de continuidade de negócio para dumps independentes e ensaios documentais.

## Conexões e pooling

- Usar connection string pooled para aplicações serverless quando apropriado e disponibilizada pelo Neon.
- Não criar pool persistente por invocation; reutilizar cliente conforme runtime e recomendações do driver.
- Limitar concorrência de tarefas e jobs.
- Definir timeouts coerentes para connection, statement e transação.
- Monitorar conexões active/idle e erros de esgotamento.
- Encerrar transações rapidamente; não manter transação aberta durante chamadas HTTP.
- Evitar criar uma conexão por operação dentro de loops.

O pooler reduz pressão sobre conexões PostgreSQL, mas não corrige queries lentas, locks ou concorrência excessiva.

## Consultas de diagnóstico seguras

Conexões por estado:

    SELECT state, count(*)
    FROM pg_stat_activity
    GROUP BY state
    ORDER BY count(*) DESC;

Consultas ativas e idade:

    SELECT pid, state, now() - query_start AS query_age,
           wait_event_type, wait_event,
           left(query, 160) AS query_sample
    FROM pg_stat_activity
    WHERE state <> 'idle'
    ORDER BY query_start;

Tamanho do banco:

    SELECT pg_database_size(current_database()) AS database_bytes;

Essas consultas devem ser executadas com credencial de observabilidade de privilégio mínimo, sem conceder superuser ou BYPASSRLS à aplicação. A amostra de SQL deve ser revisada para não expor valores pessoais.

## Capacidade e otimização

1. Registrar consumo atual e tendência semanal.
2. Identificar branches abandonadas e dados de teste, removendo somente após revisão e autorização.
3. Ajustar autosuspend/scale-to-zero conforme comportamento da aplicação.
4. Investigar processos que mantêm conexões ativas.
5. Paginar resultados e evitar SELECT * em endpoints de grande volume.
6. Adicionar índices com base em planos e medições, via migration revisada.
7. Definir limite operacional interno abaixo da quota máxima.
8. Reavaliar crescimento antes de cada release.

Não executar DELETE, VACUUM FULL, migration, ajuste de compute ou exclusão de branch como parte deste guia documental.

## Runbook: esgotamento de conexões

1. Capturar hora UTC, deployment SHA e requestIds afetados.
2. Verificar conexões active/idle e mensagens de erro.
3. Comparar pooler versus conexão direta.
4. Identificar transações longas e jobs concorrentes.
5. Reduzir concorrência de forma controlada no código; não matar sessões indiscriminadamente.
6. Revisar após release e documentar resultado.

## Referências

- https://neon.com/docs/introduction/monitoring
- https://neon.com/docs/connect/connection-pooling
- https://neon.com/docs/introduction/usage
- https://neon.com/blog/neon-free-plan-1-gb-per-project
