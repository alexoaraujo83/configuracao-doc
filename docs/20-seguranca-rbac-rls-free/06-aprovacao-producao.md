# Critérios de aprovação para produção

## 1. Gate
Aprovar somente após todos os critérios obrigatórios terem evidência. Esta documentação não substitui auditoria, revisão de código, pentest ou confirmação de termos dos fornecedores.

## 2. Checklist

### Identidade e sessão
- [ ] Auth0 Regular Web Application adequada a Next.js SSR.
- [ ] Callback/logout/web origins exatos e mínimos.
- [ ] Sessão validada no servidor; cookie seguro conforme fluxo.
- [ ] Token validado: assinatura, issuer, audience, expiração e algoritmo.
- [ ] ID token não usado como access token de API.
- [ ] Client secret e AUTH0_SECRET nunca chegam ao browser.

### Autorização
- [ ] Matriz RBAC aprovada pelo owner.
- [ ] Deny-by-default para roles/permissões desconhecidas.
- [ ] Membership/status verificados no servidor.
- [ ] Cada Server Action, Route Handler e DAL verifica autorização.
- [ ] Exportação e ações administrativas têm regra/auditoria.
- [ ] Sem confiança em tenant_id/role/user_id enviados pelo browser.

### PostgreSQL
- [ ] RLS habilitada em todas as tabelas multi-tenant.
- [ ] Policies cobrem SELECT/INSERT/UPDATE/DELETE conforme necessidade.
- [ ] WITH CHECK impede insert/mudança de tenant indevida.
- [ ] FORCE ROW LEVEL SECURITY avaliado/aplicado.
- [ ] Runtime NOINHERIT/NOBYPASSRLS, não owner e sem DDL.
- [ ] Migrator/owner não são usados em requests.
- [ ] Sem TRUNCATE, BYPASSRLS ou superuser no runtime.
- [ ] Contexto transacional não vaza entre requests.
- [ ] Views, SECURITY DEFINER, jobs, FKs e rotinas admin revisados.

### Credenciais e aplicações
- [ ] Secrets distintos por ambiente/app quando possível.
- [ ] Preview não acessa Production.
- [ ] Ownership de clients/projetos/credenciais documentado.
- [ ] Tokens restritos à audience correta.
- [ ] Banco compartilhado justificado e testado contra cross-access.
- [ ] Requisitos de isolamento físico atendidos ou NO-GO.

### Testes e operação
- [ ] SEC-001 a SEC-020 executados; exceções aprovadas formalmente.
- [ ] CI bloqueia merge em falha de autorização/RLS.
- [ ] Revisão independente do modelo de ameaça/policies.
- [ ] Logs sem tokens, cookies, PII desnecessária ou connection strings.
- [ ] Rotação/revogação ensaiadas.
- [ ] Backup/restore testado; RPO/RTO documentados.
- [ ] Runbook de incidente e rollback disponível.
- [ ] Quotas e elegibilidade Free revalidadas.

## 3. Bloqueios absolutos
NO-GO se houver: acesso cross-tenant/cross-app; RLS desabilitada ou bypass pelo runtime; owner/migrator em requests; autorização apenas na UI/layout/Proxy; token sem validação; segredo exposto; teste negativo obrigatório falhando; isolamento físico não atendido; Vercel Hobby para workload comercial; quota excedida ou dependência de recurso pago.

## 4. Registro da decisão

| Campo | Registro |
|---|---|
| Aplicação/versão/commit | |
| Ambiente | |
| Data | |
| Responsável técnico | |
| Revisor independente | |
| Testes/resultados | |
| Exceções/riscos aceitos | |
| Quotas Free confirmadas | |
| Resultado | GO / GO CONDICIONAL / PENDENTE / NO-GO |
| Aprovação formal | |

GO CONDICIONAL não autoriza expor dados sensíveis nem hospedar workload comercial na Vercel Hobby. Riscos críticos e falhas de isolamento não podem ser aceitos como pendência.

## 5. Auditoria
Guardar commit, workflow run, relatório de testes, versão das migrations, revisão de roles/policies e evidência de configuração sem anexar secrets ou dados pessoais. Reavaliar quando role, policy, tabela, API, client, credencial ou fornecedor mudar.
