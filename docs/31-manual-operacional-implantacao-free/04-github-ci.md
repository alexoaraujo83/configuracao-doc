# 4. Configurar GitHub Free e CI

## Criar repositório
1. Criar repositório público ou privado conforme política.
2. Definir main como branch padrão.
3. Adicionar README e .gitignore Node ou publicar estrutura revisada.
4. Definir responsáveis por escrita e revisão.
5. Não incluir .env, tokens ou connection strings.

## Publicação inicial
~~~bash
git init
git add .
git status
git commit -m "chore: estrutura inicial"
git branch -M main
git remote add origin https://github.com/OWNER/REPOSITORY.git
git push -u origin main
~~~

Substituir OWNER/REPOSITORY. Antes do push, revisar git status e diff staged para detectar segredos.

## Branches
| Padrão | Uso |
|---|---|
| main | linha principal |
| feature/nome | funcionalidade |
| fix/nome | correção |
| docs/nome | documentação |
| chore/nome | manutenção |

Criar PR para mudanças relevantes. Revisar diff, CI e documentação. Proteções de branch variam conforme visibilidade e plano; não presumir que recursos disponíveis em repositório público também estejam disponíveis em privado no Free.

## CI sem secrets
Arquivo .github/workflows/ci.yml:
~~~yaml
name: CI
on:
  pull_request:
  push:
    branches: [main]
permissions:
  contents: read
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc
          cache: npm
      - run: npm ci
      - run: npm run lint --if-present
      - run: npx tsc --noEmit
      - run: npm run build
~~~

Antes de adotar, confirmar .nvmrc, scripts e lockfile. Revisar versões das actions e fixar SHA conforme política. Não passar secrets de produção a workflows pull_request. Não executar código não confiável com token privilegiado.

## Quotas e integração
Verificar Billing/Usage e documentação atual do GitHub Actions. O CI de lint, tipos e build não precisa de credenciais Auth0 ou Neon. Conectar Vercel apenas ao repositório necessário e revisar permissões do GitHub App.

## Troubleshooting
- Workflow não inicia: YAML, evento, branch e Actions habilitado.
- npm ci falha: package-lock sincronizado.
- Node ausente: .nvmrc e versão suportada.
- Build falha: env pública de build, Node e dependências.
- Vercel não recebe PR: integração e permissões do repositório.

Referências: https://docs.github.com/pt/repositories/creating-and-managing-repositories e https://docs.github.com/pt/actions
