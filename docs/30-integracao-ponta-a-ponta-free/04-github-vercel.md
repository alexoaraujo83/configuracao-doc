# 4. GitHub Actions + Vercel

## Pipeline

~~~mermaid
flowchart TD
 C[Commit] --> P[Pull Request]
 P --> A[Actions: lint, tipos, build]
 A --> R{Revisão humana}
 R -->|Ajustar| C
 R -->|Aprovado| V[Preview Vercel]
 V --> H[Validação]
 H --> M[Publicação autorizada]
~~~

A automação não deve mesclar PR nem publicar produção sem autorização explícita.

## CI mínimo

Arquivo de referência .github/workflows/ci.yml:

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

Revisar versões das actions e política de atualização. O workflow não necessita secrets de produção. Usar dados sintéticos e valores de teste não sensíveis. Nunca imprimir DATABASE_URL, secrets Auth0, tokens ou variáveis secretas.

## GitHub Free
- Não presumir recursos exclusivos de planos pagos, sobretudo em repositórios privados.
- Conferir quotas de Actions e armazenamento na documentação vigente.
- Usar permissões mínimas e revisar actions de terceiros.
- Não executar código não confiável de forks com secrets privilegiados.
- Preferir SHA imutável para fixar actions ou manter processo formal de atualização das tags.

## Vercel
1. Importar repositório GitHub.
2. Confirmar preset Next.js, root directory, comandos e versão Node.
3. Configurar variáveis separadas para Development, Preview e Production.
4. Registrar callback e origins Auth0 por ambiente.
5. Isolar banco e credenciais de preview.
6. Revisar preview antes de publicar.
7. Ao alterar variáveis, criar novo deployment para aplicar configuração nova.

## Restrição Hobby
Vercel Hobby é somente para uso pessoal e não comercial. Não documentar como opção para SaaS, empresa ou atividade comercial. Para finalidade comercial, este perfil Free não atende ao requisito; manter publicação fora do escopo até existir alternativa elegível, sem presumir upgrade.

| Variável | Local | Preview | Produção |
|---|---|---|---|
| APP_BASE_URL | localhost | URL preview | domínio final |
| AUTH0_DOMAIN | tenant definido | ambiente definido | ambiente definido |
| AUTH0_CLIENT_ID | credencial local | credencial isolada | credencial prod |
| AUTH0_CLIENT_SECRET | segredo local | segredo isolado | segredo prod |
| AUTH0_SECRET | aleatório local | aleatório isolado | aleatório prod |
| DATABASE_URL | Neon dev | Neon isolado | Neon prod |

Nenhum valor secreto deve entrar em README, workflow, issue, PR ou artefato. Rollback de aplicação não reverte schema; migrations precisam de estratégia própria e compatível.
