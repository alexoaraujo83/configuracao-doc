# CI/CD integrado — GitHub Free, Vercel Hobby e Neon Free

**Status:** padrão documental; não implementado. **Escopo:** somente planos Free.

## Estratégia recomendada

Preferir integração Git nativa GitHub → Vercel para Preview e Production, quando projeto e uso forem elegíveis ao Hobby. Usar GitHub Actions para validações independentes: npm ci, TypeScript, lint, testes e build. Evitar duplicar deploy na Actions se a integração Git nativa já atende.

**Atenção ao Hobby:** Vercel Hobby é para uso pessoal e não comercial. A documentação da Vercel informa restrições para deploy de repositório privado pertencente a organização GitHub em time Hobby; confirmar propriedade/autor do commit e elegibilidade antes de escolher integração.

## Documentos
- [01 — Modelo de pipeline e gates](01-modelo-pipeline-gates.md)
- [02 — GitHub Actions seguro](02-github-actions-seguro.md)
- [03 — Migrations e Neon](03-migrations-neon.md)
- [04 — Integração Git e Vercel Hobby](04-vercel-hobby-git.md)
- [05 — Fluxo de release e rollback](05-release-rollback.md)

## Limites GitHub Free
- GitHub Free informa 2.000 minutos mensais para Actions em repositórios privados; Actions em repositórios públicos têm política distinta de cobrança.
- Environments, secrets de environment, deployment branches e regras de proteção variam por visibilidade/plano. Não presumir required reviewers nem secrets de environment em repositório privado Free.
- Usar secrets de repositório apenas em workflows de código confiável; não disponibilizar secrets para PRs de forks.
- Conferir limites, armazenamento, retenção e minutos no painel e na página oficial.

## Regra de não execução
Este conjunto não cria workflow, secret, branch de banco, integração Vercel ou deployment. Os exemplos precisam de revisão de versões/SHAs e validação antes de adoção.

## Fontes oficiais
- https://github.com/pricing
- https://docs.github.com/en/actions/reference/security/secure-use
- https://docs.github.com/en/actions/reference/security/oidc
- https://vercel.com/docs/git
- https://vercel.com/docs/git/vercel-for-github
- https://neon.com/docs/guides/branching
- https://neon.com/docs/guides/schema-migrations

Consulta: 31/10/2026.
