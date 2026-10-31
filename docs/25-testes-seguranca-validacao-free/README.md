# Testes, segurança da cadeia de dependências e validação de produção — Free

**Revisão documental:** 2026-10-31  
**Stack:** Next.js App Router + TypeScript + Auth0 SDK v4 + Neon PostgreSQL + GitHub Free + Vercel Hobby quando permitido.  
**Estado:** recomendado/documental. Nenhum teste, auditoria, migration, deploy ou validação de serviço foi executado nesta etapa.

## Objetivo e limites

Definir um processo reproduzível para demonstrar qualidade funcional, segurança da cadeia de dependências, integridade do banco e prontidão de release sem presumir recursos pagos.

A documentação não declara que o sistema atual está seguro ou pronto para produção. Cada controle deve receber um estado de evidência: **Documentado**, **Implementado**, **Executado**, **Aprovado**, **Bloqueado** ou **Não aplicável**, com data, commit, ambiente, executor e artefato não sensível.

## Documentos

1. [Estratégia de testes unitários, integração e E2E](./01-estrategia-testes.md)
2. [Testes Auth0, RBAC e RLS](./02-testes-auth0-rbac-rls.md)
3. [Auditoria npm e segurança de GitHub Actions](./03-cadeia-dependencias.md)
4. [Triagem e correção de CVEs](./04-vulnerabilidades-cves.md)
5. [Migrations e integridade PostgreSQL/Neon](./05-migrations-integridade-postgresql.md)
6. [Quality gates e bloqueios de release](./06-quality-gates-release.md)
7. [Validação final de produção](./07-validacao-producao.md)
8. [Modelo de evidências de conformidade Free](./08-modelo-evidencias.md)
9. [Workflow CI de referência](./09-workflow-ci-exemplo.md)

## Ordem recomendada

1. Confirmar requisitos de negócio, matriz de permissões e plano Free elegível.
2. Definir scripts de lint, typecheck, testes unitários, integração, E2E e build.
3. Criar fixtures sintéticas e banco isolado; nunca apontar testes para produção.
4. Testar autenticação real via navegador em aplicação/connection de teste e testar autorização com sessões controladas.
5. Executar testes RLS com a role efetiva de runtime, não com owner.
6. Auditar lockfile npm, dependências transitivas e Actions.
7. Testar migrations em banco vazio e em cópia/branch descartável representativa.
8. Aplicar quality gates; registrar exceções com prazo e responsável.
9. Concluir checklist de prontidão e reunir evidências antes de qualquer release autorizado.

## Matriz de evidência

| Estado | Significado |
|---|---|
| Documentado | Procedimento e critério descritos |
| Implementado | Código, teste ou workflow existe no repositório |
| Executado | Comando rodou; registrar commit, data e ambiente |
| Aprovado | Resultado revisado contra critério objetivo |
| Bloqueado | Dependência, quota, acesso, risco ou requisito impede aprovação |
| Não aplicável | Justificativa documentada e aprovada |

## Restrições Free obrigatórias

- **GitHub Free:** não presumir enforcement de branch protection, required reviewers, Environments ou environment secrets em repositório privado. O workflow CI deve funcionar sem secrets de produção em PR.
- **Auth0 Free:** usar tenant e aplicações de teste dentro das quotas existentes; não presumir Organizations, MFA/add-ons ou quotas ampliadas. Não automatizar o login por endpoints internos do Universal Login.
- **Neon Free:** respeitar limites atuais de projetos, branches, compute, armazenamento e transferência; usar branch descartável com lifecycle controlado. PITR curto não substitui backup independente.
- **Vercel Hobby:** somente uso pessoal/não comercial. Não declarar SaaS comercial apto para produção nesse plano. Para produção comercial, classificar hospedagem como **Bloqueada** até existir destino cujo plano gratuito autorize expressamente esse uso.
- Não incluir cartão, trial, crédito promocional, ferramenta paga ou upgrade na arquitetura.
- Não usar dados reais de clientes em fixtures, logs, snapshots ou artifacts.

## Fontes primárias

- [Next.js — Testing](https://nextjs.org/docs/app/guides/testing)
- [Next.js — Vitest](https://nextjs.org/docs/app/guides/testing/vitest)
- [Next.js — Playwright](https://nextjs.org/docs/app/guides/testing/playwright)
- [Auth0 — Mock session cookie para testes Next.js](https://support.auth0.com/center/s/article/Automate-testing-with-Next-js)
- [Auth0 — Automated testing do Universal Login](https://support.auth0.com/center/s/article/automated-testing-with-identifier-first-universal-login)
- [npm — npm audit](https://docs.npmjs.com/cli/v11/commands/npm-audit/)
- [GitHub — Secure use de Actions](https://docs.github.com/pt/actions/reference/security/secure-use)
- [GitHub — Dependabot alerts](https://docs.github.com/en/code-security/concepts/supply-chain-security/dependabot-alerts)
- [PostgreSQL — Row Security](https://www.postgresql.org/docs/17/ddl-rowsecurity.html)
- [PostgreSQL — pg_dump](https://www.postgresql.org/docs/current/app-pgdump.html)
- [Neon — Database branching](https://neon.com/blog/practical-guide-to-database-branching)

Revalidar links, quotas, termos e elegibilidade Free na data da implantação.
