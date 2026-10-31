# Pipeline CI/CD com GitHub Actions e Vercel

## Separação de responsabilidades

- GitHub Actions: validação automatizada de código em PR e push.
- Vercel Git Integration: Preview para branches/PRs e Production para branch configurada.
- Neon: banco isolado para testes; migration de produção é etapa controlada, não consequência automática de build Preview.
- Auth0: callbacks/URLs e configurações revisados por ambiente; não expor secrets a PR não confiável.

## Pipeline mínimo em Pull Request

1. Checkout com actions revisadas e fixadas.
2. Setup Node compatível com a versão Next.js.
3. Instalação reproduzível com npm ci.
4. Lint.
5. Typecheck.
6. Testes unitários.
7. Testes de integração com Postgres isolado/dados sintéticos.
8. Build Next.js.
9. Verificação de dependências e secrets.

Workflow ilustrativo; adaptar scripts, versão Node e SHA das actions:

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
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run lint --if-present
      - run: npx tsc --noEmit
      - run: npm test --if-present
      - run: npm run build
~~~

Antes de adotar, fixar actions de terceiros por SHA completo conforme política de segurança e revisar versões suportadas. Não copiar sem confirmar Node suportado e scripts existentes.

## Segurança do workflow

- permissions: contents: read como padrão; ampliar por job somente quando necessário.
- Evitar pull_request_target para executar código do PR com acesso a secrets.
- Não imprimir contextos completos, secrets ou env.
- Não executar migrations em PR contra produção.
- Usar concurrency para evitar execuções redundantes.
- Usar artifacts somente para outputs necessários e sem secrets.
- Não expor credenciais Vercel/Neon/Auth0 a workflows que executem código não confiável.

## Vercel Git Integration

Com Git conectado, Vercel cria Preview deployments para branches/PRs e Production para commits na branch de produção configurada (geralmente main). Validar:
- repositório e branch de produção;
- Root Directory e comandos de build;
- Node.js runtime e package manager;
- variáveis por ambiente, sem valores no repositório;
- URLs Preview e callbacks Auth0 compatíveis;
- isolamento de dados Neon.

No Hobby, não prometer recursos de aprovação avançada ou ambientes protegidos de planos superiores.

## Deploy de produção

No Hobby, merge/push na branch de produção dispara deployment automaticamente. Não adicionar workflow de produção duplicado sem desativar/entender o mecanismo existente.

## GitHub Actions → Vercel CLI

Não é o caminho padrão se Git Integration atende. Exige token e aumenta superfície de segredo. Em repositório privado Free, não presumir Environment secrets. Preferir integração Git e evitar deploy duplicado.

## Referências

- https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication
- https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments
- https://vercel.com/docs/git
- https://vercel.com/docs/deployments
