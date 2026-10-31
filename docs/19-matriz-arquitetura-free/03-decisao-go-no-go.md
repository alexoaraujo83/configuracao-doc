# Árvore de decisão e critérios go/no-go

## Árvore de decisão

1. A aplicação/deployment tem finalidade comercial ou ganho financeiro?
   - **Sim:** Vercel Hobby não elegível. **BLOQUEADO NO FREE**.
   - **Não:** continuar.
2. Precisa de tenant Auth0 independente por ambiente, cliente ou aplicação?
   - **Sim:** Auth0 Free (um tenant/conta) insuficiente. **BLOQUEADO NO FREE**.
   - **Não:** clients separados e callbacks estritos; documentar risco compartilhado.
3. Precisa de isolamento físico ou restore independente de dados?
   - **Sim:** projeto Neon separado; verificar quota. Se exceder, **BLOQUEADO NO FREE**.
   - **Não:** RBAC + RLS + roles + testes negativos; risco residual aceito formalmente.
4. Repo privado depende de GitHub Environment secrets?
   - **Sim:** indisponível no Free privado. Remover dependência sem reduzir segurança; se obrigatório, **BLOQUEADO NO FREE**.
   - **Não:** seguir com permissões mínimas.
5. Quotas e capacidade foram validadas?
   - **Não:** **PENDENTE**.
   - **Sim:** GO condicionado aos testes e evidências.

## Critérios

| Critério | GO | NO-GO |
|---|---|---|
| Vercel | Pessoal/não comercial, dentro dos termos | Comercial/ganho financeiro |
| Auth0 | Tenant compartilhado aceito, clients e callbacks separados | Exige tenant separado |
| GitHub | CI não depende de environment secrets em repo privado | Deploy depende desse recurso |
| Neon | Quotas confirmadas e topologia cabe | Excede quotas ou exige recurso não Free |
| Multi-tenancy | Backend authz + RLS + testes negativos | Isolamento não demonstrável |
| Secrets | Ausentes de Git/browser/logs/artifacts | Exposição ou compartilhamento sem controle |
| Recuperação | RPO/RTO e restore testado | Requisito não atendido |
| Operação | Owner, runbook e rollback definidos | Sem responsável/reversão |

## Gates por arquétipo
- **Aplicação simples:** GO se pessoal/não comercial, dados minimizados e quotas confirmadas.
- **SaaS multiusuário:** GO apenas para protótipo pessoal/não comercial; NO-GO para SaaS comercial na Vercel Hobby.
- **API/backend:** GO se pessoal/não comercial e tokens/autorização validados; NO-GO para API comercial.
- **Múltiplas aplicações:** GO se ownership, client Auth0, projeto Vercel, credenciais e domínio de dados estiverem documentados; aceitar explicitamente o risco do tenant Auth0 compartilhado.

## Evidências obrigatórias
Registrar data/URLs de consulta de quotas; finalidade pessoal/não comercial; diagrama de confiança; owners e identificadores não secretos; nomes/local de variáveis; testes RBAC/RLS negativos; prova de que Preview não usa Production; ensaio de backup/restore; decisão, aprovador e riscos aceitos.

## Estados
- **GO:** gates satisfeitos e evidenciados.
- **GO CONDICIONAL:** pendência não crítica com responsável e prazo; não autoriza produção com risco de segurança aberto.
- **PENDENTE:** faltam dados, quota, evidência ou decisão.
- **NO-GO / BLOQUEADO NO FREE:** incompatível; não contornar com trial, crédito, contas artificiais ou redução de controles.
