# Vercel Docs — referência operacional detalhada

Fonte primária: [Vercel Docs](https://vercel.com/docs). Confirmar plano, runtime, limites e disponibilidade regional antes de aplicar cada procedimento.

## 1. Team e Projects

### Configuração padrão inicial
- Identificar team owner, membros, roles e projetos.
- Associar cada projeto ao repositório e diretório corretos.
- Definir framework preset, install/build/development commands, output directory, Node.js runtime e região de Functions.
- Registrar production branch e comportamento de deploy para PRs/branches.
- Restringir integração Git aos repositórios necessários.

### Inventário completo
| Área | Itens a documentar |
|---|---|
| Team | owner, members, roles, SSO/SCIM se disponíveis, spend controls |
| Project / General | nome, framework, root directory, build/install commands, output |
| Git | provider, repository, production branch, ignored build step, PR comments |
| Environments | Development, Preview, Production, custom environments |
| Environment Variables | nome, target, sensibilidade, scope, git branch |
| Domains | domínio, aliases, DNS, TLS, redirects e target environment |
| Deployments | status, commit, author, build logs, promotion, retention, rollback |
| Functions | runtime, região, duração, memória, cron e limites |
| Networking | firewall, trusted IPs, private networking, secure compute se disponível |
| Storage / Marketplace | integração, credenciais, permissões, recursos |
| Observability | runtime/build logs, analytics, speed insights, alerts, usage |
| Security | deployment protection, password protection, headers, firewall e access control |

## 2. Environments

A Vercel oferece três ambientes padrão:
- Development: execução local e desenvolvimento.
- Preview: deployments de branches/PRs, com URLs de preview.
- Production: deployment associado à production branch e domínios principais.

Custom Environments como staging e qa dependem do plano. A documentação consultada descreve disponibilidade em Pro e Enterprise e limites por projeto; confirmar os limites atuais da conta. Não presumir quatro ambientes nativos. Para quatro estágios, documentar se Staging usa Custom Environment, projeto separado ou estratégia equivalente.

Cada ambiente pode possuir variáveis próprias. URLs de Preview podem ser branch-specific ou commit-specific. A URL de branch acompanha o deployment mais recente daquela branch; a URL de commit identifica um deployment específico.

## 3. Environment Variables

- Podem existir em nível de team ou project; preferir escopo de projeto para secrets específicos.
- Selecionar targets: Development, Preview, Production e custom environments.
- Variáveis sensíveis são criptografadas em repouso, mas acessíveis a usuários com acesso ao projeto; aplicar RBAC.
- Alterações de env vars não são aplicadas retroativamente aos deployments existentes: criar novo deployment.
- Não prefixar secrets com NEXT_PUBLIC_ em Next.js.
- Registrar nome, descrição, target, git branch filter, owner e data de rotação; nunca incluir o valor no inventário.

Comandos CLI:
~~~bash
vercel env ls
vercel env add NOME development
vercel env add NOME preview
vercel env add NOME production
vercel env pull .env.local
vercel env run -- npm run dev
~~~
Antes de sincronizar variáveis, conferir team e project vinculados. Não versionar .env.local nem arquivos locais com dados sensíveis.

## 4. Git e Deployments

Fluxo recomendado:
1. Pull Request cria Preview deployment.
2. Checks e revisão são concluídos no GitHub.
3. Merge na production branch dispara Production deployment.
4. Verificar build/runtime logs, smoke tests, domínio e integrações.
5. Executar rollback ou promoção manual conforme política e risco.

Documentar auto-deploy, ignored build step, branch tracking, deployment checks, preview protection, promotion e rollback. Deployments são artefatos de código/configuração; alteração de env vars exige novo deployment.

## 5. CLI e automação

Instalar CLI pelo gerenciador aprovado e controlar versão no CI quando necessário.

Comandos comuns:
~~~bash
vercel login
vercel link
vercel project ls
vercel env ls
vercel env pull .env.local
vercel dev
vercel deploy
vercel deploy --prod
vercel inspect <deployment-url>
vercel logs <deployment-url>
vercel domains ls
vercel domains inspect example.com
~~~
Validar flags na versão instalada. vercel link cria configuração local em .vercel/. Para deploy de produção, exigir confirmação explícita e pipeline autorizado.

## 6. Domains, DNS e TLS

Para cada domínio registrar domínio/subdomínio, projeto, ambiente/branch alvo, registros DNS, provedor, responsável, estado de verificação, certificado TLS, redirects, canonical host e regras de rewrite. Registrar callbacks Auth0 que dependem do domínio.

Não alterar DNS de produção durante documentação. Mudanças de domínio devem coordenar Vercel, Auth0, cookies, callback URLs e SEO.

## 7. Functions e Runtime

Registrar Node.js runtime, framework adapter, região, duração máxima, memória, concurrency, cron e limites do plano. Conferir compatibilidade de pacotes nativos e APIs de runtime. Não depender de estado local persistente entre invocações. Para workloads de longa duração ou conexões persistentes, revisar arquitetura e limites específicos do runtime.

## 8. Networking e Security

Avaliar firewall/WAF, deployment protection, password protection de Preview, trusted IPs, secure compute/private networking, security headers, redirects e rate limiting. Recursos variam por plano. Proteger Preview deployments com dados ou endpoints internos; URL difícil de adivinhar não é controle de acesso.

## 9. Observability, custos e rollback

- Separar build logs e runtime logs.
- Definir alertas e owners para erros, latência e consumo.
- Revisar retenção, limites de logs e métricas do plano.
- Acompanhar function invocations, bandwidth, image optimization, build minutes e storage.
- Antes de rollback, verificar se schema Neon permanece compatível com o código restaurado.
- Usar migrations expand/contract para reduzir acoplamento entre release da aplicação e banco.

## 10. Integração Auth0 + Neon

- Guardar Auth0 client secret, AUTH0_SECRET e DATABASE_URL em variáveis server-side.
- Definir targets distintos para Preview/Staging/Production.
- Não permitir que Preview use credenciais ou dados de produção.
- Atualizar callbacks/allowed logout URLs Auth0 com domínios reais e controlados.
- Fazer novo deployment após atualizar variáveis.
- Usar pooled Neon connection para padrões serverless quando driver/ORM forem compatíveis.

## 11. Checklist
- [ ] Team e acesso revisados.
- [ ] Projeto conectado ao repo/diretório corretos.
- [ ] Framework/build/runtime confirmados.
- [ ] Production branch e Preview deploy definidos.
- [ ] Development, Preview, Production e Staging mapeados explicitamente.
- [ ] Environment variables com targets corretos e sem secrets públicos.
- [ ] Domínios, DNS e TLS validados.
- [ ] Preview protection e segurança avaliadas.
- [ ] Logs, alertas, uso e rollback documentados.
- [ ] Integração Auth0/Neon isolada por ambiente.

## Fontes oficiais
- [Vercel Docs](https://vercel.com/docs)
- [Environments](https://vercel.com/docs/deployments/environments)
- [Environment Variables](https://vercel.com/docs/environment-variables)
- [CLI Overview](https://vercel.com/docs/cli)
- [Deploying from CLI](https://vercel.com/docs/projects/deploy-from-cli)
- [Managing Projects](https://vercel.com/docs/projects/managing-projects)

## Limites e observações
Custom environments, limites de execução, segurança avançada, networking, observability e retenção dependem do plano. Confirmar pricing/limits e configurações reais do team antes de estabelecer baseline obrigatória.
