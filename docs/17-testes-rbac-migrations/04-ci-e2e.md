# 04 — CI GitHub Free e testes E2E

**Estado:** workflow e testes propostos; não configurados nem executados.
**Objetivo:** validar código em PR sem expor secrets nem exceder cotas gratuitas.

## 1. Pipeline recomendado
1. Checkout.
2. Setup Node compatível com a versão do projeto.
3. Instalação determinística com npm ci.
4. Typecheck.
5. Lint.
6. Testes unitários.
7. Build.
8. npm audit e revisão dos resultados.
9. E2E em workflow separado quando necessário.

Não executar migrations em produção no pipeline genérico de PR. Não disponibilizar secrets a workflows originados de forks.

## 2. Exemplo de CI sem secrets

Ajustar scripts ao package.json real:

~~~yaml
name: CI
on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 24
          cache: npm
      - run: npm ci
      - run: npm run typecheck
      - run: npm run lint
      - run: npm test
      - run: npm run build
      - run: npm audit
~~~

Antes de produção, revisar versões e fixar Actions por SHA completo confiável. Node 24 é exemplo; manter a versão suportada e efetivamente usada pelo projeto. O exemplo pressupõe que os scripts existam.

## 3. Segurança de workflows
- Usar permissions mínimos por workflow/job.
- Evitar pull_request_target para executar código não confiável do PR.
- Nunca executar código de fork em contexto com secrets.
- Não imprimir environment, headers, cookies ou tokens.
- Fixar Actions de terceiros por SHA completo após revisão.
- Usar concurrency para cancelar execuções obsoletas.
- Não conceder contents: write quando o job só valida.
- Revisar dependências, scripts npm e Actions de terceiros.
- Separar CI de release/deploy.
- Secrets de produção nunca devem estar disponíveis a PRs comuns.

## 4. Playwright E2E local

~~~typescript
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./tests/e2e",
  fullyParallel: true,
  retries: process.env.CI ? 1 : 0,
  reporter: "list",
  use: {
    baseURL: "http://127.0.0.1:3000",
    trace: "retain-on-failure",
  },
  projects: [{ name: "chromium", use: { ...devices["Desktop Chrome"] } }],
  webServer: {
    command: "npm run dev",
    url: "http://127.0.0.1:3000",
    reuseExistingServer: !process.env.CI,
  },
});
~~~

Traces podem conter dados de sessão e conteúdo sensível. Não anexar trace de autenticação real sem inspeção e sanitização.

## 5. E2E sem login real em cada PR
- Testar páginas públicas e redirecionamento para login sem autenticar.
- Usar session mock somente em ambiente de teste, isolado do build de produção.
- Executar teste Auth0 real apenas em workflow protegido, com tenant/usuário de teste, secrets protegidos e consentimento operacional.
- Nunca desativar autenticação em produção para facilitar testes.

Mocks não validam OIDC, callback, cookies ou configuração real do tenant. Manter smoke test autenticado em ambiente de teste.

## 6. Testes de integração Neon
Não conectar job de PR público a banco persistente com secrets. Executar unitários sem Neon no CI comum; integração manual em branch/database de teste; ou workflow protegido apenas para branches confiáveis. Garantir limpeza e quota antes de criar branch efêmera. Não imprimir URL de conexão.

## 7. Uso de minutos Free
Preferir job simples; cancelar workflows obsoletos; evitar builds duplicados; cachear dependências; não instalar browsers em todos os jobs; monitorar billing e reconfirmar cotas, que variam conforme visibilidade e runner.

## 8. Referências
- [GitHub Actions — Secure use](https://docs.github.com/pt/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions)
- [GitHub Actions — Limits](https://docs.github.com/pt/actions/reference/limits)
- [Vitest](https://vitest.dev/)
- [Playwright](https://playwright.dev/docs/intro)
