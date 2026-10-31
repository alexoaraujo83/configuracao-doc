# GitHub Free — repositório, revisão e CI

## Repositório

Criar um repositório por projeto quando separação de acesso, ciclo de vida e releases justificar. Registrar visibilidade, owner, branch padrão e política de contribuição.

Convenção:
- main: linha principal.
- feature/<assunto>: funcionalidades.
- fix/<assunto>: correções.
- docs/<assunto>: documentação.
- release/<versão>: somente se a cadência exigir.

Não presumir branch protection, Environments ou environment secrets em repositórios privados no GitHub Free. Verificar matriz atual de recursos antes de depender desses controles. Para repositórios públicos, avaliar exposição de código e workflows.

## Pull Requests

Exigir descrição de objetivo e escopo, issue/requisito relacionado, impacto em variáveis e migrations, evidências de validação, análise de segurança, plano de rollback quando aplicável e revisão humana antes do merge.

Não mesclar PR automaticamente como parte do processo documental.

## CI mínimo sem secrets

~~~yaml
name: CI

on:
  pull_request:
  push:
    branches:
      - main

permissions:
  contents: read

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Type check
        run: npx tsc --noEmit

      - name: Lint
        run: npm run lint

      - name: Build
        run: npm run build
~~~

Antes de uso, revisar versões das Actions e fixar referências confiáveis conforme política do projeto. O workflow não usa secrets, não conecta a banco de produção e não executa migrations.

## Segurança de Actions

- Permissões mínimas no nível do workflow/job.
- Não executar código de PR não confiável com secrets privilegiados.
- Evitar pull_request_target para checkout/execução de código não confiável.
- Não imprimir variáveis ou contexts completos.
- Usar dados sintéticos.
- Controlar minutos e armazenamento de artefatos conforme quotas Free atuais.
- Não presumir runners, concorrência ou minutos ilimitados.

## Checklist

- [ ] Visibilidade e exposição do repositório aprovadas.
- [ ] Branch padrão definida.
- [ ] Revisão humana documentada.
- [ ] CI sem secrets e com contents: read.
- [ ] Dependências e Actions revisadas.
- [ ] Nenhuma credencial em commits ou artefatos.
- [ ] Limites Free de Actions e recursos de proteção verificados.

Referências: [GitHub Docs](https://docs.github.com/pt), [GitHub Actions](https://docs.github.com/pt/actions).
