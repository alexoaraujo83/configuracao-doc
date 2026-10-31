# Manual operacional de implantação — stack Free

## Escopo e status
Manual documental para repetir a configuração de referência em um novo projeto Next.js App Router + TypeScript. Não comprova criação de contas, tenants, bancos, repositórios ou deployments. Nenhum comando foi executado.

**Perfil Free:** Auth0 Free, Neon Free, GitHub Free e Vercel Hobby. Hobby somente para uso pessoal e não comercial; não utilizar para SaaS, clientes ou atividade empresarial. Revalidar quotas e termos antes de cada implantação.

## Procedimentos
1. [Preparar estação e criar projeto Next.js](01-preparacao-estacao-nextjs.md)
2. [Configurar Auth0](02-configuracao-auth0.md)
3. [Configurar Neon PostgreSQL](03-configuracao-neon.md)
4. [Configurar GitHub e CI](04-github-ci.md)
5. [Configurar Vercel Hobby](05-vercel-hobby.md)
6. [Validar, diagnosticar e reverter](06-validacao-troubleshooting-rollback.md)
7. [Checklist](07-checklist-implantacao.md)

## Três perspectivas
- **Padrão recomendado:** configuração de referência proposta.
- **Configuração existente:** estado observado diretamente em painel/API/CLI, com data e evidência. Não foi inventariado nesta etapa.
- **Procedimento de implantação:** sequência a executar somente após revisão e autorização.

## Convenções
- Exemplos usam placeholders, nunca credenciais reais.
- Segredos ficam server-side; não usar prefixo NEXT_PUBLIC_.
- Preview deve ser isolado de produção.
- Migrations são revisadas e executadas por fluxo controlado.
- Não automatizar deploy de produção ou exclusão de recursos sem autorização.
- Não assumir quotas fixas com base em documentos antigos.

## Referências
- Next.js: https://nextjs.org/docs/app/getting-started/installation
- Auth0 SDK v4: https://auth0.github.io/nextjs-auth0/
- Auth0 migration guide: https://github.com/auth0/nextjs-auth0/blob/main/V4_MIGRATION_GUIDE.md
- Neon: https://neon.com/docs
- Neon branching: https://neon.com/branching
- GitHub Actions: https://docs.github.com/pt/actions
- Vercel Hobby terms: https://vercel.com/legal/terms
- Vercel environment variables: https://vercel.com/docs/environment-variables
