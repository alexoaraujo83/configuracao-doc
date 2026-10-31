# Gestão de mudanças, releases e versionamento — somente Free

> Estado: Recomendada. Referência documental pesquisada em 2026-10-31. Nenhum pipeline, branch protection, migration ou deployment foi configurado ou executado.

## Objetivo

Definir processo reproduzível e auditável para Next.js + TypeScript, Auth0, Neon PostgreSQL, GitHub e Vercel Hobby, sem depender de recursos pagos.

## Política Free

- GitHub Free: usar branches, Pull Requests e Actions. Proteções de branch estão disponíveis em repositórios públicos; não presumir ambientes protegidos ou environment secrets em repositórios privados Free.
- Vercel Hobby: somente projetos pessoais e não comerciais. Git Integration fornece Preview para branches/PRs e Production para a branch configurada.
- Neon Free: branches apoiam validação isolada, mas quantidade e consumo são limitados. Não criar branch de banco automaticamente para todo PR sem avaliar quota.
- Auth0 Free: não presumir tenant isolado por ambiente nem recursos avançados de promoção/backup.

## Guias

1. [Branches e Pull Requests](01-branches-pull-requests.md)
2. [SemVer, tags e changelog](02-semver-changelog.md)
3. [CI/CD GitHub Actions + Vercel](03-pipeline-ci-cd.md)
4. [Migrations PostgreSQL com Neon](04-migrations-neon.md)
5. [Compatibilidade e rollback](05-compatibilidade-rollback.md)
6. [Aprovações, rastreabilidade e auditoria](06-aprovacoes-auditoria.md)
7. [Checklist de release Free-only](07-checklist-release.md)

## Fluxo de alto nível

Issue → branch curta → commits convencionais → Pull Request → CI → revisão humana → merge em main → Preview/Production conforme integração Vercel → validação pós-release → tag e changelog.

Para mudanças de schema, aplicar estratégia expand-contract e manter compatibilidade entre versão anterior e nova.

## Regras essenciais

- Nunca versionar secrets, arquivos .env reais ou dados de produção.
- Não executar migration de produção automaticamente em Preview ou workflow de PR.
- Não expor credenciais privilegiadas a código de PR não confiável.
- Não depender de GitHub Environments em repositório privado Free.
- Não usar Vercel Hobby para SaaS ou uso comercial.
- Não usar force-push em main nem apagar evidência de release.
- Não marcar release como verificada sem evidência e responsável.

## Fontes oficiais

- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches
- https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments
- https://semver.org/lang/pt-BR/spec/v2.0.0.html
- https://keepachangelog.com/pt-BR/1.1.0/
- https://vercel.com/docs/git
- https://vercel.com/docs/deployments/rollback-production-deployment
- https://neon.com/docs/get-started/workflow-primer

## Estado

Documentação proposta. Branches, tags, regras de proteção, pipelines, secrets e configurações de serviço não foram modificados.
