# 01 — Estratégia e organização

Estrutura sugerida:

~~~text
src/
  app/
  server/
  lib/
  components/
__tests__/
  unit/
  components/
  integration/
  contracts/
  security/
e2e/
playwright/
  .auth/  # ignorado pelo Git
~~~

Convenções:
- Um teste verifica comportamento observável ou regra de negócio.
- Isolar dados, cookies, storage e fixtures entre testes.
- Usar fixtures determinísticas e idempotentes.
- Testar comportamento público, não detalhes internos.
- Para async Server Components, preferir E2E enquanto suporte unitário não for adequado.

Scripts esperados: test:unit, test:integration, test:e2e, test:coverage e test:ci. Declarar somente após selecionar e configurar os runners no projeto real.