# GitHub Docs — referência operacional detalhada

Fonte: [Documentação GitHub em português](https://docs.github.com/pt). Esta página organiza os domínios funcionais que devem ser inventariados em cada repositório e organização.

## 1. Repositórios e governança

### Configuração padrão inicial
- Definir finalidade, proprietário, visibilidade, branch padrão e política de contribuição.
- Manter README, licença quando aplicável, SECURITY.md e instruções de contribuição.
- Definir quem pode administrar, manter, escrever, triar issues e ler.
- Usar equipes e permissões mínimas em repositórios de organização.
- Revisar periodicamente colaboradores, apps instalados, deploy keys e webhooks.

### Inventário completo
| Área | Opções a documentar | Controle recomendado |
|---|---|---|
| General | nome, descrição, topics, default branch, features, archive/transfer | ownership e propósito |
| Access | collaborators, teams, roles, outside collaborators | menor privilégio |
| Features | Issues, Projects, Discussions, Wiki | habilitar só o necessário |
| Rules | branch protection, rulesets, tags, bypass actors | proteger branches críticas |
| Webhooks | URL, eventos, secret, estado e entregas | validar assinatura |
| Deploy keys | chave, read/write, finalidade | escopo mínimo |
| Environments | nome, secrets, variables, reviewers, branch restrictions | segredos isolados |
| Secrets and variables | escopo repository/org/environment | não usar variable para segredo |

Repositórios públicos são acessíveis na Internet. Não publicar dados de clientes, segredos, configurações privadas ou artefatos internos.

## 2. Git, branches e proteção

Modelo recomendado:
- Branch padrão protegida; desenvolvimento em branches curtas.
- Pull Request obrigatório para integrar mudanças.
- Exigir checks relevantes, revisão independente e resolução de conversas.
- Restringir force-push e deleção da branch protegida.
- Aplicar CODEOWNERS a workflows, infraestrutura e documentação sensível.

Registrar em cada ruleset: branches/tags alvo, padrões de inclusão/exclusão, número de aprovações, dismiss stale approvals, code owner review, status checks, regras de commit, restrições de push e bypass actors. Rulesets podem ser combinados; disponibilidade varia por plano e visibilidade.

Pull Requests devem usar template com contexto, motivação, risco, testes, impacto de dados, screenshots quando úteis e rollback. Distinguir Draft PR de PR pronto para revisão.

## 3. GitHub Actions e CI/CD

Conceitos a documentar:
- Workflow YAML em .github/workflows, eventos, permissões, jobs e steps.
- Runner hospedado pelo GitHub ou self-hosted; avaliar isolamento e superfície de ataque.
- Actions de terceiros: revisar origem e fixar versão por SHA imutável quando possível.
- Artifacts e cache: definir retenção e não incluir secrets.
- Environments: destinos como development, staging e production, com secrets e regras de proteção.

Pipeline recomendado:
1. PR: lint, typecheck, testes unitários/integração, build e verificações de segurança.
2. Preview: deploy isolado, credenciais não produtivas e dados sintéticos.
3. Staging: aprovação, smoke tests e validação de integração.
4. Production: branch protegida, checks aprovados, autorização e deploy auditável.

Segurança do workflow:
- Definir permissions explicitamente e conceder o mínimo ao GITHUB_TOKEN.
- Usar secrets para dados sensíveis e variables para configuração não sensível.
- Evitar interpolar entradas não confiáveis diretamente em scripts shell.
- Tratar pull requests de forks como contexto não confiável.
- Preferir OIDC para credenciais de cloud/terceiros quando suportado.
- Restringir ambientes de produção por branch, reviewers e regras disponíveis no plano.
- Usar concurrency groups para evitar deploys conflitantes.

Exemplo conceitual mínimo:
~~~yaml
permissions:
  contents: read
jobs:
  checks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<SHA_IMUTAVEL>
      - run: npm ci
      - run: npm run lint
      - run: npm test
~~~
Substituir o placeholder por SHA verificado de uma Action confiável; o exemplo não é executável enquanto mantiver o placeholder.

## 4. Segurança e cadeia de suprimentos

Inventariar Dependabot alerts/security updates, version updates, secret scanning, push protection, code scanning/CodeQL, dependency review, políticas de Actions e SECURITY.md.

Se um segredo for publicado, revogar/rotacionar imediatamente. Remover o arquivo do histórico não invalida a credencial exposta.

## 5. Autenticação e APIs

Tipos de credencial:
- Personal access token: limitar permissões e duração; preferir fine-grained PAT quando adequado.
- GitHub App: opção preferível para automação com permissões granulares e tokens de instalação.
- GITHUB_TOKEN: token do workflow, com permissões controladas pelo YAML e políticas.
- SSH/deploy key: uso específico para acesso Git; documentar escopo e rotação.

A API REST exige token com permissões compatíveis com o endpoint. 401 geralmente indica credencial ausente/inválida; 403 ou 404 podem indicar permissão insuficiente ou recurso não acessível. Nunca registrar tokens em logs.

Documentar versão da API, paginação, rate limits, retries com backoff, idempotência, tratamento de 403/429 e permissões por endpoint.

## 6. Integração com Vercel

- Autorizar GitHub ↔ Vercel somente nos repositórios necessários.
- Definir production branch e comportamento de Preview por PR/branch.
- Verificar checks e comentários de deployment.
- Evitar pipelines de deploy concorrentes sem definir a fonte de verdade.
- Preferir OIDC quando suportado; caso contrário, limitar e rotacionar secrets.

## 7. Checklist
- [ ] Visibilidade, owners e colaboradores corretos.
- [ ] Branch padrão e rulesets revisados.
- [ ] CODEOWNERS e templates presentes.
- [ ] Actions com permissões mínimas e dependências fixadas.
- [ ] Secrets/variables separados por escopo.
- [ ] Environments e aprovações configurados.
- [ ] Dependabot, secret scanning, push protection e code scanning avaliados.
- [ ] Integração Vercel restrita ao repositório necessário.
- [ ] Webhooks e GitHub Apps revisados.
- [ ] Processo de incidente e rotação documentado.

## Fontes oficiais
- [Repositórios](https://docs.github.com/pt/repositories)
- [Sobre repositórios](https://docs.github.com/pt/repositories/creating-and-managing-repositories/about-repositories)
- [Rulesets — regras disponíveis](https://docs.github.com/pt/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)
- [Conceitos do GitHub Actions](https://docs.github.com/pt/actions/concepts)
- [Implantando com GitHub Actions](https://docs.github.com/pt/actions/how-tos/deploy/configure-and-manage-deployments/control-deployments)
- [Gerenciar configurações do Actions](https://docs.github.com/pt/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository)
- [Autenticação da API REST](https://docs.github.com/pt/rest/authentication/authenticating-to-the-rest-api)
- [Permissões necessárias para GitHub Apps](https://docs.github.com/pt/rest/authentication/permissions-required-for-github-apps)
- [Secret scanning](https://docs.github.com/pt/code-security/secret-scanning/introduction)

## Limites
Runners, rulesets, controles avançados e recursos de segurança variam por plano, tipo de conta e visibilidade. Confirmar disponibilidade atual antes de definir baseline obrigatória.
