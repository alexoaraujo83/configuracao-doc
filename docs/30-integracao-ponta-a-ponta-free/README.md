# Integração ponta a ponta — planos Free

Documentação de referência para Next.js App Router + TypeScript, Auth0, Neon PostgreSQL, GitHub e Vercel. Os exemplos são modelos não executados. O escopo é documental.

**Restrição Vercel:** Hobby somente para uso pessoal e não comercial. Não utilizar para SaaS ou atividade comercial. Não depender de planos pagos, trials ou add-ons. Conferir quotas e elegibilidade diretamente nas fontes oficiais na data de implantação.

## Módulos
1. [Arquitetura e fluxos](01-arquitetura-e-fluxos.md)
2. [Auth0 + Next.js](02-auth0-nextjs.md)
3. [Neon, persistência e autorização](03-neon-persistencia-autorizacao.md)
4. [GitHub Actions + Vercel](04-github-vercel.md)
5. [Erros, observabilidade e privacidade](05-erros-observabilidade-privacidade.md)
6. [Testes e critérios de aceitação](06-testes-criterios-aceitacao.md)

## Premissas
- Auth0 Next.js SDK v4; não reutilizar exemplos legados v3, como handleAuth ou rota catch-all /api/auth/[...auth0].
- Next.js 15 usa middleware.ts; Next.js 16 usa proxy.ts. Confirmar API e convenção na versão instalada.
- Neon acessado exclusivamente no servidor por @neondatabase/serverless.
- CI com dados sintéticos, sem secrets de produção.
- Separar configurações e credenciais de local, preview e produção.

## Fluxo de entrega
1. Configurar Auth0 e URLs por ambiente.
2. Configurar Neon com papéis separados para migração e runtime.
3. Implementar sessão, identidade, membership e autorização server-side.
4. Aplicar RLS e queries parametrizadas.
5. Validar lint, tipos e build em CI.
6. Revisar preview e PR manualmente antes de qualquer publicação.

## Fontes oficiais
- https://nextjs.org/docs
- https://auth0.com/docs
- https://neon.com/docs
- https://docs.github.com/pt/actions
- https://vercel.com/docs/plans/hobby
