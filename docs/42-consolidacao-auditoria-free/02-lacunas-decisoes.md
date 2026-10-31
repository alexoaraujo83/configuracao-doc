# 02 — Lacunas, inconsistências e decisões técnicas

## Lacunas ainda abertas
1. Não há evidência de configuração real dos tenants Auth0, projetos Vercel ou projetos Neon; a documentação é padrão de referência.
2. Nenhum exemplo TypeScript/SQL/YAML foi compilado, executado ou testado.
3. Os conteúdos dos PRs #29–#36 foram reunidos na branch de integração #37; sua incorporação à main depende do merge desse PR.
4. Auth0 Free e Neon Free têm quotas/recursos que mudam; verificar o painel e os termos na data de uso.
5. Vercel Hobby não pode ser usado para aplicação comercial; isso pode impedir deploy para certos repositórios privados de organizações.
6. GitHub Free em repositórios privados não deve pressupor environments, secrets de environment ou required reviewers.
7. Testes E2E autenticados precisam de conta de teste e segredos; PRs de fork não recebem secrets e não devem ser contornados.

## Decisões que devem permanecer explícitas
- Next.js App Router + TypeScript.
- Auth0 SDK v4; evitar API legada v3.
- Next.js 15: middleware.ts; Next.js 16: proxy.ts, conforme versão fixada.
- Neon PostgreSQL com driver serverless; validar transações e contexto RLS na API exata do driver.
- Runtime role PostgreSQL não é owner e não tem BYPASSRLS.
- Autorização por tenant é verificada no servidor; RLS é defesa adicional.
- Contexto tenant/user precisa ser local à transação que executa as consultas protegidas.
- Migrations executadas com role migrator e conexão apropriada, nunca pelo runtime.
- Preview nunca usa credenciais de produção.
- Vercel Hobby somente uso pessoal/não comercial.

## Divergência Neon storage
Em 31/10/2026, FAQ Free e anúncio de 02/10/2026 indicam 1 GB PostgreSQL por projeto; a página Plans ainda apresenta 0,5 GB em trechos. Até confirmação direta no Console, a documentação operacional usa 0,5 GB como teto conservador. Fontes: https://github.com/neondatabase/website/blob/main/content/faqs/free-plan-limits-and-quotas.md ; https://neon.com/blog/neon-free-plan-1-gb-per-project ; https://neon.com/docs/introduction/plans.

## Limites Vercel Hobby
Retenção de runtime logs indicada como 1 hora e uso pessoal/não comercial. Revalidar quotas em https://vercel.com/docs/limits e https://vercel.com/docs/plans/hobby.

## Risco de snippets
Os exemplos de Auth0 callback dinâmico, Neon transação HTTP, policies RLS, GitHub Actions e Playwright exigem compatibilidade com versões fixadas. Placeholders de SHA e scripts de migration não são executáveis sem substituição/revisão.
