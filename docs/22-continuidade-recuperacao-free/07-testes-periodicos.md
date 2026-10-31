# Testes periódicos de continuidade e recuperação

## Calendário recomendado
| Frequência | Exercício | Evidência |
|---|---|---|
| Mensal | revisar owners, contatos, DNS e inventário de env vars | checklist |
| Mensal | dump lógico e checksum | metadata/hash |
| Trimestral | restore Neon isolado | log, duração, validação |
| Trimestral | export Auth0 sem secrets e diff | export redigido |
| Trimestral | reconstruir Preview Vercel de commit limpo | deployment ID/smoke tests |
| Semestral | tabletop perda de conta GitHub/Auth0/Neon/Vercel | timeline/gaps |
| Anual | exercício BCP/DR integrado e revisão BIA/RTO/RPO | relatório |

Frequências são política interna, não automação dos planos Free.

## Restore Neon
Usar dados sintéticos/cópia aprovada; restaurar isoladamente; comparar schema, tabelas, constraints, sequences e extensões; testar RLS sem contexto e cross-tenant; executar smoke tests; medir duração/RPO; excluir ambiente temporário somente com autorização.

## Reconstrução da aplicação
Clone limpo de commit aprovado; instalação pelo lockfile; build Next.js/TypeScript; Preview com secrets de teste; Auth0 callback/login; Neon e testes RLS; smoke tests/logs; registrar passos manuais.

## Métricas
- RTO observado = fim do serviço mínimo menos início do incidente.
- RPO observado = instante do incidente menos timestamp da última transação recuperada.
- Integridade = checks schema/dados/policies aprovados.
- Cobertura = componentes recuperados / componentes no escopo.
- Backup success = backups válidos / programados.
- Restore success = restores validados / tentados.

## Relatório
Projeto/ambiente; cenário; UTC; participantes; dados sintéticos; backup/branch; início/fim; RTO alvo/observado; RPO alvo/observado; integridade; falhas; passos manuais; ações/owners/prazos; aprovação.

Falha crítica bloqueia declaração de prontidão.
