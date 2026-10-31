# 05 — E2E, CI e critérios de aceite

Playwright deve cobrir login/logout, acesso privado, CRUD autorizado, acesso cross-tenant negado, expiração de sessão e erros controlados.

Boas práticas: browser context isolado, dados dedicados, Chromium quando suficiente, workers=1 inicialmente no CI, traces/screenshots apenas em falha e sem dados sensíveis, playwright/.auth no .gitignore. Nunca versionar storageState, cookies ou headers.

GitHub Free: typecheck, lint e unit tests em PR; integração/E2E em PR confiável e main, controlando minutos. Não fornecer secrets de produção a workflows de PR/fork. Fixar Actions por SHA completo. Artifacts excluem .env, .vercel, playwright/.auth, dumps e logs sensíveis.

Critérios de aceite:
- Cada regra crítica de autorização tem teste positivo e negativo.
- RLS é testada com runtime role e cross-tenant.
- Migrations aplicam do zero e sobre schema anterior em banco descartável.
- E2E de login/logout/rotas privadas passa em ambiente de teste.
- Testes são determinísticos e isolados.
- Nenhum secret/estado de sessão é versionado.
- CI cabe nas quotas Free observadas.

Nenhum teste foi executado nesta entrega. Fixar versões de Vitest, Playwright, Node e browsers após compatibilidade verificada.

Fontes: https://playwright.dev/docs/ci ; https://playwright.dev/docs/best-practices ; https://nextjs.org/docs/app/guides/testing