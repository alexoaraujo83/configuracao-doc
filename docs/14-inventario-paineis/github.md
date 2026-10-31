# GitHub Free — inventário do painel

> Revisão documental: 31/10/2026. A interface varia por tipo de conta, permissões e atualização do GitHub.

## 1. Escopo Free

GitHub Free oferece repositórios públicos e privados ilimitados, mas com conjuntos diferentes de recursos. Preferir repositório público somente quando isso for compatível com a política de dados. Em repositório privado, não presumir recursos avançados de proteção.

- Rulesets estão disponíveis em repositórios públicos no GitHub Free e GitHub Free for organizations.
- Rulesets e branch protection em repositórios privados exigem GitHub Pro, Team ou Enterprise, conforme o tipo de conta.
- GitHub Free inclui 2.000 minutos/mês de Actions para repositórios privados em contas pessoais/organizações, sujeitos às regras de cobrança e armazenamento. Runners padrão hospedados em repositórios públicos têm uso gratuito.
- Code Security e Secret Protection pagos não devem ser tratados como incluídos. Recursos gratuitos variam por visibilidade e tipo de conta.
- PRs, revisão humana e CI são controles úteis, mas não equivalem a enforcement técnico de branch protection.

## 2. Navegação do repositório

| Caminho | Finalidade | Revisão | Free |
|---|---|---|---|
| Repositório > Code | Arquivos e branches | README, estrutura e branch padrão | Incluído |
| Repositório > Issues | Trabalho e defeitos | Templates, labels, milestones | Incluído |
| Repositório > Pull requests | Revisar alterações | Base/head, Draft, checks, comentários e merge | Incluído |
| Repositório > Actions | CI/CD | Workflows, runs, logs, artifacts e GITHUB_TOKEN | Quotas |
| Repositório > Projects | Organizar trabalho | Views, campos e automações | Verificar limites |
| Repositório > Wiki | Documentação colaborativa | Ativação e política de uso | Varia; não substituir Markdown versionado |
| Repositório > Security / Security and quality | Alertas e segurança | Dependabot, secret scanning e code scanning | Varia por plano/visibilidade |
| Repositório > Insights | Métricas | Pulse, contributors, traffic, commits | Alguns recursos privados exigem plano superior |
| Repositório > Settings | Administração | General, Access, Rules, Actions, Webhooks e Security | Varia por plano/permissão |

## 3. Settings — áreas relevantes

| Caminho | Finalidade | Configuração recomendada |
|---|---|---|
| Settings > General | Nome, visibilidade, features, branch padrão e merge | Confirmar visibilidade e branch padrão |
| Settings > Collaborators / Access | Pessoas e equipes | Menor privilégio; revisar periodicamente |
| Settings > Branches | Regras clássicas de branch | Em repositório privado Free, não presumir proteção |
| Settings > Rules > Rulesets | Regras para branches/tags | Usar em repositório público Free; confirmar elegibilidade |
| Settings > Actions > General | Políticas de execução | Restringir origens e permissões do token |
| Settings > Actions > Runners | Runners hospedados/self-hosted | Self-hosted exige isolamento e manutenção |
| Settings > Secrets and variables > Actions | Secrets e variáveis | Limitar escopo; nunca imprimir secrets |
| Settings > Environments | Ambientes, variáveis e secrets | Não presumir deployment protection/reviewers no Free |
| Settings > Webhooks | Eventos e integrações | Validar assinatura, eventos e endpoint |
| Settings > Security / Code security | Dependabot e controles de segurança | Habilitar apenas recursos gratuitos elegíveis |

## 4. Pull Requests

Caminho: repositório > Pull requests > selecionar PR.

Revisar:
- Base e head branches.
- Draft versus Ready for review.
- Diff, commits, checks e conversas.
- Aprovação humana antes de merge.
- Estratégia permitida: merge, squash ou rebase.
- Status de CI e resultados de segurança.
- Dependências entre PRs empilhadas.

Política do projeto: manter PRs em Draft até revisão; não mesclar automaticamente. Para PRs dependentes, registrar ordem e validar o diff após retarget.

## 5. Actions

Caminho: repositório > Actions; configurações em Settings > Actions.

Checklist:
- Declarar permissions explicitamente no workflow/job.
- Usar contents: read como baseline; ampliar somente quando necessário.
- Fixar actions de terceiros por SHA quando viável.
- Não expor secrets em logs, artefatos ou comandos de debug.
- Executar npm ci, lint, typecheck, testes e build.
- Controlar concorrência de workflows duplicados.
- Monitorar minutos e armazenamento incluídos no Free.
- Não permitir que workflows criem/aprovem PRs sem justificativa.

## 6. Segurança

Caminho: repositório > Security; configurações em Settings > Security / Code security.

- Dependabot alerts: revisar dependências vulneráveis.
- Dependabot security updates: automatizar PRs quando disponível.
- Secret scanning e push protection: confirmar elegibilidade por visibilidade e plano.
- Code scanning: distinguir ferramentas gratuitas em repositórios públicos de produtos Code Security pagos.
- Ausência de alertas não prova ausência de vulnerabilidades.

## 7. Conta pessoal e organização

Menu do avatar > Settings:
- Profile: identidade pública.
- Account: segurança e preferências.
- Password and authentication: senha, passkeys e MFA.
- Emails: endereços e verificação.
- SSH and GPG keys: chaves.
- Applications: OAuth Apps e GitHub Apps autorizados.
- Security: opções de segurança da conta.
- Billing and licensing: plano, uso e orçamento, conforme permissões.

Em organizações, Settings pode conter People, Teams, Repository defaults, Actions, Security e Billing. O conteúdo varia por papel e plano.

## 8. Fluxo recomendado

1. Criar branch descritiva.
2. Fazer commits pequenos e rastreáveis.
3. Abrir PR em Draft.
4. Executar CI com permissões mínimas.
5. Revisar arquivos e diff completo.
6. Confirmar dependências e conflitos.
7. Solicitar revisão humana.
8. Mesclar somente após autorização e validação.

## 9. Fontes oficiais

- [GitHub plans](https://docs.github.com/en/get-started/learning-about-github/githubs-plans)
- [Product usage included with each plan](https://docs.github.com/en/billing/reference/product-usage-included)
- [About rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)
- [Managing repository rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/managing-rulesets-for-a-repository)
- [GitHub Actions settings](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository)
- [Security features](https://docs.github.com/en/code-security/getting-started/github-security-features)
- [Security and analysis settings](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-security-and-analysis-settings-for-your-repository)

## 10. Pendências de validação

- [ ] Tipo de conta: pessoal ou organização.
- [ ] Plano efetivo e orçamento de Actions.
- [ ] Visibilidade do repositório.
- [ ] Rulesets visíveis e elegíveis.
- [ ] Permissões de workflow e GITHUB_TOKEN.
- [ ] Recursos de segurança habilitados e elegibilidade Free.
- [ ] Configurações reais de colaboradores e equipes.
