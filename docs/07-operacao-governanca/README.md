# 07 — Operação e governança

## Configuração padrão inicial

- Toda alteração de infraestrutura/configuração tem responsável, justificativa e evidência.
- Mudanças em produção exigem revisão e plano de rollback.
- Incidentes devem registrar impacto, linha do tempo, causa raiz e ações preventivas.
- Segredos devem ser rotacionados sem exposição em logs ou tickets.

## Runbooks

### Deploy com falha

1. Identificar deployment, commit e ambiente afetado.
2. Inspecionar build/runtime logs, variáveis ausentes e mudanças recentes.
3. Comparar configuração com último deployment saudável.
4. Reverter deployment se necessário, avaliando compatibilidade de schema.
5. Registrar causa, correção e validação.

### Falha de autenticação

1. Verificar domínio, issuer, audience, callback e clock skew.
2. Confirmar estado da application/connection e Actions.
3. Inspecionar logs sem copiar tokens ou dados pessoais.
4. Testar em ambiente não produtivo e aplicar correção revisada.

### Falha de conexão Neon

1. Confirmar endpoint/branch/database e status do compute.
2. Verificar pooling, limites de conexão, TLS e credencial ativa.
3. Conferir role/grants e migrations recentes.
4. Rotacionar credencial se houver suspeita de exposição.

## Governança

- Revisão periódica de acessos e integrações OAuth.
- Inventário de recursos órfãos e branches temporárias.
- Revisão de custos e limites do plano.
- Teste periódico de restore e rollback.
- Atualização de versões de SDK/CLI com registro de breaking changes.

## Registro de mudança

| Campo | Descrição |
|---|---|
| ID / data | identificador e data da mudança |
| Escopo | serviço, projeto e ambiente |
| Motivo | requisito ou incidente |
| Risco | impacto e probabilidade |
| Execução | passos e responsável |
| Validação | testes e evidências |
| Rollback | procedimento e gatilho |

## Checklist de revisão operacional

- [ ] Acessos revisados.
- [ ] Alertas e logs testados.
- [ ] Custos/limites revisados.
- [ ] Backups/restores validados.
- [ ] Runbooks atualizados.

## Governança de planos Free

- Revisar quotas e termos oficiais antes de cada implantação e periodicamente durante a operação.
- Criar alertas/processo de observação para compute, storage, transferência, minutos CI e retenção.
- Definir comportamento quando a quota acabar: pausar, limitar ou bloquear operação de forma segura; não ativar cobrança automática.
- Manter backups independentes quando a retenção Free for insuficiente e testar restore.
- Registrar recursos indisponíveis no Free e controles compensatórios sem classificá-los como equivalentes.
- Para Vercel Hobby, confirmar que o uso é pessoal/não comercial antes de deploy.
