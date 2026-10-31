# 09 — GitHub: documentação operacional e governança

> Fonte normativa: documentação oficial do GitHub. Esta página descreve padrões recomendados para repositórios que alimentam deployments na Vercel e aplicações que usam Auth0 e Neon. Recursos podem variar por plano, tipo de conta e política da organização.

## 1. Configuração padrão inicial

1. Criar um repositório por unidade de versionamento/deploy, ou definir claramente os limites do monorepo.
2. Definir branch padrão e estratégia de branches; proteger `main` por regras de branch ou rulesets.
3. Exigir Pull Request para mudanças em `main`, checks automatizados e resolução de conversas.
4. Definir permissões mínimas para usuários, equipes, GitHub Apps e tokens.
5. Usar GitHub Actions com permissões explícitas e mínimas para `GITHUB_TOKEN`.
6. Guardar segredos em Actions Secrets por ambiente, repositório ou organização, limitando acesso.
7. Não executar código não confiável de Pull Requests de forks com secrets privilegiados.
8. Revisar dependências, Actions de terceiros e alertas de segurança.

## 2. Inventário completo do repositório

| Área do GitHub | Configurações a documentar |
|---|---|
| General | nome, descrição, visibilidade, URL, recursos habilitados, template, arquivo padrão |
| Access | colaboradores, equipes, roles, outside collaborators, GitHub Apps |
| Branches | branch padrão, regras, patterns, proteção, restrições de push |
| Rulesets | alvo, enforcement, bypass list, regras de branch/tag |
| Pull Requests | merge methods, auto-merge, reviews, CODEOWNERS, templates |
| Actions | permissões padrão, políticas de Actions, runners, workflows, cache e artifacts |
| Environments | nomes, reviewers, branch restrictions, secrets e variables |
| Secrets and variables | escopo, repositórios autorizados, rotação e responsáveis |
| Code security | Dependabot alerts/updates, secret scanning, push protection, CodeQL conforme plano |
| Deployments | ambientes, histórico, protection rules e status |
| Webhooks / Apps | endpoint, eventos, permissões, segredo e responsável |
| Pages / Releases | publicação, tags, release notes e artefatos |
| Insights / Audit | atividade, dependabot, traffic e audit log conforme disponibilidade |

## 3. Branches, regras e estratégia de integração

### Configuração padrão inicial

- Branch principal: `main`.
- Branches de trabalho: `feat/<assunto>`, `fix/<assunto>`, `docs/<assunto>`, `chore/<assunto>`.
- Toda alteração em `main` via Pull Request.
- Proibir force-push e exclusão da branch principal.
- Exigir checks de CI com nomes únicos e estáveis.
- Resolver comentários de revisão antes do merge.

### Proteção de branch

A documentação do GitHub descreve requisitos configuráveis como revisões aprovadas, status checks, resolução de conversas, commits assinados, histórico linear, merge queue, deployments bem-sucedidos e restrição de push. Nem todos são obrigatórios para todo repositório; selecionar conforme risco e plano.

Recomendação para `main`:
- Require a pull request before merging.
- Require at least one approving review quando houver equipe revisora disponível.
- Dismiss stale approvals quando novos commits alterarem o diff.
- Require status checks to pass.
- Require conversation resolution.
- Block force pushes e branch deletion.
- Avaliar linear history e signed commits conforme fluxo e capacidade da equipe.

**Rulesets versus branch protection:** rulesets permitem combinar políticas; regras aplicáveis são agregadas e, em conflito, a regra mais restritiva prevalece. Evitar políticas duplicadas sem inventário, pois podem criar bloqueios inesperados. Usar enforcement Disabled/Evaluate quando disponível para testar antes de ativar.

### Pull Requests

Modelo de PR deve conter:
- contexto e objetivo;
- escopo e arquivos relevantes;
- impacto por ambiente;
- evidência de teste;
- impacto de schema/migrations;
- riscos e rollback;
- confirmação de ausência de secrets/dados sensíveis.

Definir merge method (squash, rebase ou merge commit) de modo consistente. Auto-merge só deve ser habilitado após todos os checks e revisões necessários. Merge queue é recurso voltado a branches com alto volume de integração e exige checks compatíveis com evento `merge_group`.

## 4. GitHub Actions: workflows, jobs e segurança

### Modelo mental

- Workflow: arquivo YAML em `.github/workflows/`.
- Event: evento que inicia o workflow, por exemplo `pull_request`, `push` ou `workflow_dispatch`.
- Job: unidade executada em runner; jobs podem depender de outros jobs.
- Step: ação ou comando dentro de um job.
- Runner: ambiente de execução hospedado pelo GitHub ou self-hosted.
- Artifact: saída persistida de um workflow; cache é mecanismo distinto para acelerar dependências.

### Workflow mínimo de validação

Exemplo ilustrativo para Node.js; ajustar versão, package manager e scripts ao projeto:

```yaml
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
          node-version: 24
          cache: npm
      - run: npm ci
      - run: npm run lint --if-present
      - run: npm run typecheck --if-present
      - run: npm test --if-present
      - run: npm run build
```

Este snippet demonstra estrutura, não uma recomendação de pinagem final: para supply-chain hardening, fixar Actions de terceiros em SHA completo verificado e atualizar por processo controlado. Omitir permissões que não sejam necessárias; definir `permissions: {}` ou o conjunto mínimo no nível workflow/job. Não conceder `write-all` como padrão.

### Secrets e variables

- Preferir environment secrets para credenciais específicas de deployment.
- Repositório secrets para uso restrito a um repositório.
- Organization secrets com política de acesso por repositórios selecionados.
- Variables para configuração não sensível; secrets para credenciais.
- Segredos de ambiente só ficam disponíveis após as proteções do environment serem aprovadas.
- Nunca imprimir secrets; mascaramento não substitui prevenção.
- Não colocar secrets em artifacts, cache, outputs, command-line args ou logs.

### GITHUB_TOKEN

O GitHub cria um token de instalação por job, limitado ao repositório do workflow e com expiração ao término do job/limite efetivo. Definir permissões mínimas explicitamente. Se o workflow precisa publicar comentário, release ou deployment, conceder somente a permissão necessária para aquela tarefa.

### OIDC

Para autenticação entre GitHub Actions e provedores cloud, preferir OpenID Connect e credenciais temporárias em vez de secrets estáticos de longa duração, quando o provedor e a integração suportarem. Configurar trust policy com issuer, audience e claims de repositório/branch/environment; não confiar apenas no nome do workflow. Para Vercel, usar a integração e mecanismo de autenticação oficialmente suportados pelo projeto, sem assumir que OIDC genérico está disponível para qualquer operação.

### Riscos de workflows

- Evitar `pull_request_target` para executar código de PR não confiável com contexto privilegiado.
- Não interpolar input/branch/title controlado por usuário diretamente em scripts shell.
- Fixar Actions por SHA completo.
- Revisar permissões de Actions de terceiros.
- Separar jobs de teste de PR dos jobs que recebem secrets/deploy.
- Preferir runners hospedados para workloads não confiáveis; self-hosted exige isolamento e hardening adicionais.

## 5. Environments e implantação

Criar environments como `development`, `preview`, `staging` e `production` somente quando houver destino de implantação real correspondente. Cada job referencia um environment; as proteções são avaliadas antes do runner iniciar e antes dos environment secrets serem disponibilizados.

| Environment | Branches autorizadas | Aprovação | Secrets |
|---|---|---|---|
| development | branches de desenvolvimento | geralmente não | credenciais descartáveis |
| preview | branches/PRs elegíveis | conforme risco | nunca credenciais de produção |
| staging | branch de release/staging | revisão técnica | dados e serviços isolados |
| production | somente branch/release autorizado | reviewer obrigatório quando disponível | menor privilégio e rotação |

Regras de proteção podem restringir branches, exigir aprovação e controlar acesso a secrets. Confirmar a disponibilidade de reviewers obrigatórios e outras proteções de acordo com o plano GitHub e o tipo de repositório. Não assumir isolamento de runner self-hosted por usar environment.

## 6. Integração com Vercel, Auth0 e Neon

### GitHub → Vercel

- Conectar o repositório à Vercel por Git Integration.
- Definir branch de produção e comportamento para Preview.
- Confirmar que PRs geram deployment Preview e que merge na branch de produção gera deployment Production, conforme configuração do projeto.
- Verificar se o acesso do GitHub App está limitado aos repositórios necessários.
- Não duplicar pipeline de deploy em GitHub Actions se a integração nativa da Vercel já for o mecanismo escolhido, salvo necessidade explícita de gates ou tarefas adicionais.

### GitHub Actions → Neon

- Executar lint/test/build em PR sem acesso ao banco de produção.
- Para migrations, usar role específica e credenciais de ambiente protegido.
- Preferir branch Neon efêmera ou banco de teste para validar migrations.
- Separar job de validação de migration do job de aplicação em produção.
- Exigir aprovação de environment antes de liberar credencial de produção.

### GitHub Actions → Auth0

- Não armazenar client secret de aplicação em workflows que executam PRs não confiáveis.
- Para operações administrativas automatizadas, usar credencial de máquina com escopo mínimo, ambiente protegido e trilha de auditoria.
- Não colocar Management API token em logs/artifacts.
- Mudanças de tenant, Actions, connections e APIs devem ser revisadas e testadas fora de produção.

## 7. Segurança e cadeia de suprimentos

- Habilitar Dependabot alerts e revisar dependências vulneráveis.
- Avaliar Dependabot security updates e version updates.
- Habilitar secret scanning e push protection quando disponíveis.
- Usar CodeQL/code scanning conforme linguagem, plano e risco.
- Gerar e verificar artifact attestations quando a maturidade do pipeline justificar.
- Manter lockfiles e revisar mudanças transitivas.
- Revisar permissões de GitHub Apps, deploy keys, PATs e webhooks.
- Preferir fine-grained PATs com expiração e escopo de repositório quando PAT for inevitável; preferir GitHub App/OIDC para automação durável.

## 8. Checklist de configuração e auditoria

- [ ] Branch padrão definida e protegida.
- [ ] PR obrigatório para mudanças de produção.
- [ ] Status checks estáveis, únicos e obrigatórios.
- [ ] CODEOWNERS e templates de PR quando aplicáveis.
- [ ] Actions com permissões mínimas e Actions de terceiros fixadas em SHA.
- [ ] Secrets separados por ambiente e sem acesso a PRs não confiáveis.
- [ ] Environments com branch restrictions e aprovação conforme criticidade.
- [ ] Dependabot e secret scanning revisados.
- [ ] Integração GitHub App/Vercel com escopo mínimo.
- [ ] Migrations e deploys de produção com gate e rollback.
- [ ] Webhooks, tokens e apps inventariados e revisados.

## 9. Fontes oficiais

- [GitHub Docs — Português](https://docs.github.com/pt)
- [Sobre branches protegidos](https://docs.github.com/pt/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [Gerenciar implantações com GitHub Actions](https://docs.github.com/pt/actions/how-tos/deploy/configure-and-manage-deployments/control-deployments)
- [Usar segredos em GitHub Actions](https://docs.github.com/pt/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets)
- [GITHUB_TOKEN](https://docs.github.com/pt/actions/concepts/security/github_token)
- [Segurança em GitHub Actions](https://docs.github.com/en/actions/how-tos/secure-your-work)
- [Rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)

## 10. Estado e evidências

Os itens acima são procedimentos e padrões documentados. Para marcar uma configuração como **Verificada**, registrar repositório, data, plano, tela/API consultada, valor observado (sem secrets) e responsável. Não inferir configurações da conta apenas pelo conteúdo deste guia.
