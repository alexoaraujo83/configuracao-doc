# 9. Workflow CI de referência

## 9.1 Finalidade

Exemplo mínimo de CI sem secrets de produção. Adaptar scripts e versão Node ao projeto. O workflow não foi instalado nem executado nesta etapa.

## 9.2 Exemplo YAML

~~~yaml
name: quality-ci

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: quality-ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  quality:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Typecheck
        run: npm run typecheck

      - name: Unit tests
        run: npm run test:unit

      - name: Dependency audit
        run: npm audit --audit-level=high

      - name: Build
        run: npm run build
~~~

**Importante:** referências @v4 são legíveis para exemplo, mas a política mais rígida de supply chain recomenda fixar cada Action a SHA completo validado e atualizado por processo controlado. O Dependabot pode não alertar Actions fixadas por SHA. Substituir as referências antes de adotar o workflow, documentando versão e SHA.

## 9.3 Etapas opcionais

Adicionar jobs separados, somente quando scripts/infra estiverem preparados:
- integração PostgreSQL com serviço descartável e fixtures sintéticas;
- Playwright com browser instalado e servidor Next.js de teste;
- análise de dependências/Dependency Review quando elegível no plano e visibilidade do repositório;
- verificação de migration em banco isolado;
- upload de relatórios sanitizados.

Não abrir conexão para banco de produção em pull_request. Não provisionar branch Neon por PR sem avaliar quota, cleanup e custo de compute.

## 9.4 Limites e segurança

- Sem AUTH0_CLIENT_SECRET, DATABASE_URL de produção ou VERCEL_TOKEN no job de PR.
- Se o build exige variáveis, fornecer valores fictícios e não funcionais exclusivamente para compile/test; não simular sucesso de integração real.
- Não executar deploy neste workflow.
- Não executar migration em produção.
- Restringir artifacts e evitar logs de environment.
- Em repo privado GitHub Free, este workflow pode reportar status, mas não presumir que o GitHub imponha required checks na branch padrão.
