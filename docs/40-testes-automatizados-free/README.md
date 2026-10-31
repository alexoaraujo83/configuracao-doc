# Testes automatizados — Next.js, Auth0 e Neon (somente Free)

**Status:** estratégia documental, não executada. Stack Next.js App Router + TypeScript, Auth0 SDK v4 e Neon PostgreSQL.

## Pirâmide de testes
1. Unitários: Vitest para funções puras, validadores, regras de domínio.
2. Componentes: React Testing Library para Client Components e interação acessível.
3. Integração: services, repositories, Route Handlers e banco isolado.
4. Contratos: DTOs, status HTTP, schemas e paginação.
5. Segurança: autorização negativa, tenant isolation, RLS e CSRF.
6. E2E: Playwright em Preview ou ambiente local controlado.

## Documentos
- 01 — Estratégia e organização
- 02 — Unitários e componentes
- 03 — Integração, contratos e banco
- 04 — Segurança, autorização e RLS
- 05 — E2E, CI e critérios de aceite

## Custo Free
Executar primeiro testes rápidos. Playwright deve usar Chromium somente quando suficiente, workers limitados no CI e artifacts sem cookies/tokens. Não depender de serviços pagos ou banco de produção.

## Referências
- https://nextjs.org/docs/app/guides/testing
- https://nextjs.org/docs/app/guides/testing/vitest
- https://vitest.dev/guide/coverage
- https://playwright.dev/docs/best-practices
- https://playwright.dev/docs/ci
- https://playwright.dev/docs/auth

Consulta: 31/10/2026.