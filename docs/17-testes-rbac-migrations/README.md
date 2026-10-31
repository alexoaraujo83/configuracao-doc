# Testes, RBAC, migrations e recuperação — Free

**Estado:** documentação técnica; nenhum teste foi executado e nenhum banco foi alterado.
**Base:** [manuais de implementação](../16-manuais-implementacao/README.md).
**Política:** somente recursos Free, sem trials, upgrades ou add-ons.

## Objetivo
Definir estratégia verificável para Next.js + TypeScript + Auth0 + Neon PostgreSQL + GitHub Free: testes automatizados, autorização por papéis, migrations auditáveis e exercícios de recuperação.

## Documentos
1. [Estratégia de testes automatizados](./01-estrategia-testes.md)
2. [RBAC e autorização multiusuário](./02-rbac-autorizacao.md)
3. [Migrations e evolução do schema](./03-migrations.md)
4. [Integração, CI e testes E2E](./04-ci-e2e.md)
5. [Backup, restore e recuperação](./05-backup-restore.md)

## Ordem de adoção
1. Definir requisitos de acesso e modelo de dados.
2. Implementar testes unitários de autorização e validação.
3. Implementar migrations em branch/database descartável.
4. Executar testes de integração com role runtime e dois usuários fictícios.
5. Executar testes E2E localmente.
6. Habilitar CI sem secrets em PRs não confiáveis.
7. Ensaiar backup/restore em ambiente isolado.
8. Registrar evidências e limitações do plano Free.

## Critério de evidência
- **Documentado:** procedimento descrito e revisado.
- **Executado:** execução registrada com data e ambiente.
- **Validado:** resultado reproduzível aprovado.
- **Bloqueado:** depende de acesso, quota, autorização ou recurso indisponível.

Não registrar como validado apenas por existir um teste ou workflow no repositório.

## Restrições
- Não executar migrations nem testes destrutivos em produção.
- Não usar credenciais administrativas em testes da aplicação.
- Não incluir tokens, connection strings ou secrets em fixtures, logs ou artifacts.
- Neon Free tem cotas de compute, armazenamento, branches e transferência; confirmar limites atuais.
- GitHub Actions Free possui limites de uso; manter jobs curtos e monitorar consumo.
- Vercel Hobby permanece limitado a uso pessoal e não comercial; não é destino de produção para SaaS comercial.

## Referências
- [Vitest](https://vitest.dev/guide/)
- [Playwright](https://playwright.dev/docs/intro)
- [GitHub Actions — Secure use](https://docs.github.com/pt/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions)
- [PostgreSQL — Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html)
- [PostgreSQL — Row Security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [Neon — Serverless Driver](https://neon.com/docs/serverless/serverless-driver)
