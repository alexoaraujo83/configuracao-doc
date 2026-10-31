# Checklist de aprovação Free

Este checklist é preenchido por projeto e por release. Marcar somente após verificar evidência real; a documentação não conta como execução.

## Elegibilidade e governança

- [ ] Finalidade e classificação do sistema aprovadas.
- [ ] Uso Vercel Hobby confirmado como pessoal e não comercial.
- [ ] Planos, termos, cotas e limites conferidos na data do release.
- [ ] Nenhum trial, crédito, cartão ou add-on requerido.
- [ ] Responsáveis e aprovadores designados.
- [ ] Exceções e riscos residuais registrados.

## Aplicação e identidade

- [ ] Typecheck, lint e build aprovados no SHA candidato.
- [ ] Unit, integration e E2E críticos aprovados.
- [ ] Auth0 login/callback/logout e sessão verificados.
- [ ] Secrets e configurações separados por ambiente lógico.
- [ ] Autorização de domínio implementada na aplicação.
- [ ] Casos 401/403 e spoofing de identidade testados.
- [ ] Nenhuma permissão baseada apenas em middleware ou UI.

## Banco de dados

- [ ] Roles owner/migrator/runtime revisadas.
- [ ] Runtime não é owner, superuser nem BYPASSRLS.
- [ ] RLS e FORCE RLS aplicados onde necessários.
- [ ] Testes multi-tenant sintéticos aprovados.
- [ ] SQL parametrizado e constraints revisados.
- [ ] Migrations reproduzíveis e checksum verificado.
- [ ] Restore isolado executado e integridade conferida.
- [ ] RTO/RPO definidos como metas internas, não SLA.

## Cadeia de dependências e CI

- [ ] package-lock versionado e npm ci reproduzível.
- [ ] npm audit executado e resultados triados.
- [ ] CVEs críticas/altas resolvidas ou release bloqueado.
- [ ] GitHub Actions com permissões mínimas.
- [ ] Actions de terceiros fixadas em SHA revisado.
- [ ] Nenhum secret em código, logs, artifacts ou PR.
- [ ] Quotas GitHub Actions dentro do orçamento Free.

## Vercel e produção

- [ ] Deployment vinculado ao SHA aprovado.
- [ ] Variáveis revisadas sem expor valores.
- [ ] HTTPS, callback URLs e logout URLs verificados.
- [ ] Smoke tests de aplicação, Auth0 e Neon aprovados.
- [ ] Logs sem tokens, PII ou dados sensíveis.
- [ ] Consumo de compute, storage, egress e runtime dentro das cotas.
- [ ] Plano de rollback compatível com schema e migrations.
- [ ] Monitoramento e runbook disponíveis.

## Decisão

- [ ] Todos os controles críticos aplicáveis estão Conforme.
- [ ] Todas as evidências críticas estão Verificadas e vigentes.
- [ ] Nenhuma não conformidade crítica aberta.
- [ ] Nenhuma exceção proibida ou dependência paga.
- [ ] Owner aprovou escopo e elegibilidade.
- [ ] Tech Lead e Security Reviewer registraram revisão.
- [ ] Release Owner registrou GO/GO condicionado/NO-GO.

**Decisão:** PENDENTE

**Projeto / release / SHA:** PREENCHER

**Aprovadores / data:** PREENCHER

**Riscos residuais / condições:** PREENCHER
