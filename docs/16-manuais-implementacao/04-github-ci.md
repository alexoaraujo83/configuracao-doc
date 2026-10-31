# 04 — GitHub Free, versionamento e CI

**Estado:** Documentado; workflow não executado.  
**Plano:** GitHub Free.

## 1. Pré-requisitos

- Conta GitHub Free.
- Repositório do projeto.
- Git instalado.
- Node.js LTS.
- `package-lock.json` versionado.
- Nenhum segredo de produção necessário para lint, typecheck e build básico.

## 2. Inicializar e publicar repositório

```bash
git init
git add .
git commit -m "chore: initial Next.js application"
git branch -M main
git remote add origin https://github.com/OWNER/REPOSITORY.git
git push -u origin main
```

Substituir OWNER e REPOSITORY. Antes de `git add`, revisar `.gitignore` e executar:

```bash
git status --short
git check-ignore .env.local
```

Se `.env.local` não estiver ignorado, interromper e corrigir antes do commit.

## 3. Estratégia de branches

- `main`: branch de integração e produção.
- `feature/<assunto>`: funcionalidades.
- `fix/<assunto>`: correções.
- `docs/<assunto>`: documentação.

Fluxo: branch → commits pequenos → Pull Request → CI → revisão → merge autorizado. Não assumir que todos os controles avançados de proteção estão disponíveis no Free para repositórios privados. Em repositórios públicos, confirmar a disponibilidade atual de rulesets e aplicar apenas controles compatíveis com o plano.

## 4. Workflow de CI

Criar `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
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
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install
        run: npm ci

      - name: Type check
        run: npm run typecheck

      - name: Lint
        run: npm run lint

      - name: Build
        run: npm run build
```

A versão Node deve ser alinhada ao `.nvmrc`/engines e à versão suportada pelo Next.js instalado. Revisar e fixar Actions em versões/SHAs confiáveis conforme política do repositório; `@v4` é uma referência legível, não imutabilidade de supply chain.

## 5. Permissões e segredos

- Definir `permissions: contents: read` como padrão mínimo.
- Não expor secrets a workflows de PR de forks.
- Não usar `pull_request_target` para executar código não confiável com secrets.
- Não armazenar secrets em YAML, logs, artefatos ou cache.
- Não disponibilizar credenciais de produção no job de build.
- Para workflows que não precisam de credenciais, não configurar secrets.
- Exigir revisão humana antes de qualquer workflow que altere infraestrutura.

## 6. Limites Free

GitHub Free oferece uma franquia de uso de Actions, mas minutos e armazenamento podem variar por sistema operacional, tipo de runner, visibilidade do repositório e política vigente. Consultar as páginas oficiais de uso incluído e limites antes de fixar um orçamento. Não converter valores em USD para minutos sem fórmula oficial aplicável.

Práticas de contenção:
- Cancelar execuções redundantes por concorrência.
- Evitar matrizes excessivas.
- Não publicar artefatos grandes sem necessidade.
- Preferir cache npm com lockfile.
- Acompanhar consumo no painel Billing.
- Configurar orçamento/limite de gastos quando a interface do plano oferecer essa opção; não contar com mecanismo pago como requisito.

## 7. Validação

1. Abrir PR de teste.
2. Confirmar que o workflow inicia em `pull_request`.
3. Confirmar `npm ci`, typecheck, lint e build.
4. Introduzir erro de tipo em branch temporária e confirmar falha do job.
5. Remover o erro e confirmar sucesso.
6. Conferir que nenhum secret foi impresso nos logs.

## 8. Segurança e governança

- Revisar dependências e alertas de segurança.
- Revisar autores, diffs e workflows antes do merge.
- Não aprovar automaticamente PRs de dependências sem avaliar changelog e impacto.
- Não usar Actions para publicar produção sem aprovação e desenho de credenciais.
- Registrar decisões e evidências sem expor dados sensíveis.

## 9. Referências

- [GitHub Docs PT](https://docs.github.com/pt)
- [GitHub Actions — limites](https://docs.github.com/pt/actions/reference/limits)
- [Uso incluído por plano](https://docs.github.com/pt/billing/reference/product-usage-included)
- [Segurança para Actions](https://docs.github.com/pt/actions/security-guides/security-hardening-for-github-actions)

