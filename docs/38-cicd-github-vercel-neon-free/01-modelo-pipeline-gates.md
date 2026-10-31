# 01 — Modelo de pipeline e gates

## Pipeline por Pull Request

~~~text
Pull Request
  ├─ checkout read-only
  ├─ npm ci
  ├─ typecheck
  ├─ lint
  ├─ unit tests
  ├─ contract/integration tests sem secrets de produção
  ├─ build
  └─ resultado de CI
       └─ Vercel Preview (integração Git, se elegível)
~~~

## Pipeline de main
1. Pull Request revisado e aprovado conforme governança disponível.
2. CI concluído; em GitHub Free privado, alguns gates protegidos via Environment/required reviewers podem não estar disponíveis.
3. Merge autorizado para main.
4. Vercel cria Production deployment via Git integration, se projeto e repositório forem elegíveis no Hobby.
5. Smoke test sem expor dados.
6. Monitorar logs após release.
7. Em falha, seguir procedimento de rollback.

## Fora do PR workflow
- Secrets de produção.
- DATABASE_URL de produção.
- Migrations contra produção.
- Deploy de produção.
- Operações destrutivas ou seed de dados reais.

## Cache e lockfile
- Versionar package-lock.json e usar npm ci.
- Cachear somente dependências e artefatos não sensíveis.
- Não publicar .env, .vercel, browser state, tokens ou dumps como artifacts.
- Não depender de cache como fonte de verdade.

## Exemplo mínimo de CI

~~~yaml
name: CI
on:
  pull_request:
    branches: [main]
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
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@<REVIEWED_FULL_COMMIT_SHA>
        with:
          persist-credentials: false
      - uses: actions/setup-node@<REVIEWED_FULL_COMMIT_SHA>
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npx tsc --noEmit
      - run: npm run lint
      - run: npm test
      - run: npm run build
~~~

Substituir SHAs por commits completos revisados. Ajustar Node à matriz suportada pelo Next.js/Auth0 fixada no projeto. O comando de testes depende do runner instalado.

## Segurança
O workflow não declara secrets nem id-token. O checkout é read-only e desabilita persistência de credenciais. Não usar pull_request_target para checkout e execução de código de fork com privilégios do repositório.
