# 8. Modelo de evidências de conformidade Free

## 8.1 Registro por controle

Usar um registro por teste, scanner, migration ou validação:

| Campo | Conteúdo |
|---|---|
| ID | TEST-xxx / SEC-xxx / DB-xxx / REL-xxx |
| Requisito | requisito ou risco coberto |
| Controle | comando, teste ou verificação |
| Estado | Documentado / Implementado / Executado / Aprovado / Bloqueado / N/A |
| Commit | SHA completo |
| Data/hora | UTC ou timezone explicitado |
| Ambiente | local / CI / Preview / produção |
| Plano | Free exato e visibilidade do repo |
| Ferramentas | versões Node, npm, Vitest, Playwright, PostgreSQL |
| Resultado | pass/fail, contagem, duração, achados |
| Evidência | URL do workflow/PR ou artefato sanitizado |
| Responsável | pessoa/time responsável |
| Revisão | aprovador e data |
| Exceção | justificativa, expiração e mitigação |

Nunca incluir tokens, cookies, secrets, connection strings, dados pessoais, dumps ou logs brutos com informação sensível.

## 8.2 Checklist Free

- [ ] GitHub: plano/visibilidade e recursos de branch/Actions confirmados.
- [ ] GitHub: nenhum workflow depende de Environment secrets/review gate indisponível.
- [ ] GitHub Actions: permissões mínimas, Actions revisadas e sem secrets em PR/fork.
- [ ] npm: lockfile versionado e npm ci reproduzível.
- [ ] npm: npm audit executado e achados triados.
- [ ] Auth0: quotas do tenant Free verificadas; aplicação de teste não excede limite.
- [ ] Auth0: login real validado por browser; mocks não são usados como prova de login.
- [ ] RBAC: autorização server-side testada em todas as ações sensíveis.
- [ ] PostgreSQL: app_runtime sem owner/BYPASSRLS/DDL.
- [ ] RLS: SELECT/INSERT/UPDATE/DELETE e contexto ausente testados com tenants sintéticos.
- [ ] Neon: quota/branches/compute/storage verificadas; sem pressupor branch ilimitada.
- [ ] Migrations: fresh install, upgrade e integridade validados em DB isolado.
- [ ] Backup: restore isolado comprovado quando exigido pelo RPO/RTO.
- [ ] Vercel: Hobby não usado para SaaS ou finalidade comercial.
- [ ] Produção: env vars, deployment, smoke tests e rollback documentados.
- [ ] Nenhuma evidência contém secrets ou dados reais desnecessários.

## 8.3 Resumo executivo de release

~~~text
Release:
Commit:
PR/tag:
Data:
Ambiente:
Plano GitHub:
Plano Auth0:
Plano Neon:
Host e plano:
Finalidade (pessoal/não comercial/comercial):
Testes unitários:
Testes integração:
Testes E2E:
npm audit:
Actions audit:
Migrations:
RBAC/RLS:
Smoke tests:
Exceções abertas:
Bloqueadores:
Decisão: GO / NO-GO
Responsável:
Aprovador:
Evidências:
~~~

## 8.4 Linguagem permitida

- “Documentado” significa procedimento descrito.
- “Implementado” significa artefato de código/configuração existe.
- “Executado” exige log ou resultado de execução.
- “Aprovado” exige revisão do resultado contra critério.
- “Conforme Free” exige evidência de plano, elegibilidade, quotas e termos na data de referência.
- “Pronto para produção” só pode ser declarado para finalidade/host/ambiente específicos, após todos os gates aplicáveis.

Evitar expressões como “100% seguro”, “sem vulnerabilidades”, “backup garantido” ou “produção validada” sem escopo e evidência verificável.
