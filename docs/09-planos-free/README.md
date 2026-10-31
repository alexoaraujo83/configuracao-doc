# Matriz de compatibilidade — somente planos Free

> Data de verificação: 31/10/2026. Regra do projeto: nenhum upgrade, trial, crédito promocional, add-on pago ou funcionalidade faturável pode ser pressuposto. Esta matriz deve ser revalidada antes de cada implantação relevante.

## 1. Resultado executivo

| Serviço | Plano permitido | Disponibilidade/limite Free | Restrições críticas |
|---|---|---|---|
| GitHub | Free | Repositórios públicos com conjunto amplo de recursos; privados ilimitados com conjunto limitado; Actions com franquia mensal e runners públicos gratuitos | Rulesets e branch protections nativos não abrangem repositórios privados no Free; reviewers obrigatórios, protected branches e Code Owners para privados exigem plano superior |
| Auth0 | Free | Até 25.000 MAU; 5 Organizations; até 1.000 autenticações M2M/mês; Actions e Attack Protection básico | M2M Add-on não incluído; MFA OTP/Duo e RBAC por Organization aparecem em Essentials; mais de 5 Organizations indisponível |
| Vercel | Hobby ($0) | Projetos pessoais/não comerciais com quotas Hobby | Uso comercial ou SaaS comercial não permitido pelo Hobby; não usar para produção de produto comercial |
| Neon | Free ($0) | 100 projetos; 10 branches/projeto; 100 CU-hours/projeto/mês; 1 GB/projeto e 20 GB agregados; 5 GB de transferência/projeto/mês | Branch protection nativo indisponível; escala para zero obrigatória; 6 h/1 GB de instant restore; quotas podem suspender compute |

## 2. Regras por fornecedor

### GitHub Free

**Permitido**
- Repositórios públicos e privados; o conjunto de recursos privados é limitado.
- GitHub Actions com cota mensal de compute incluída; a página de pricing informa 2.000 minutos/mês para Free, enquanto a documentação de planos descreve franquia de compute em dólares. Confirmar o saldo e consumo na área Billing da conta.
- Runners hospedados em repositórios públicos: uso de runners padrão gratuito segundo a documentação de billing.
- Self-hosted runners sem cobrança de minuto pelo GitHub; o hardware, energia, rede e manutenção continuam sendo responsabilidade do operador.
- Dependabot alerts e 2FA para a conta pessoal Free.

**NÃO DISPONÍVEL NO FREE para repositórios privados**
- Repository rulesets: a documentação disponibiliza rulesets no Free para repositórios públicos; privados exigem Pro, Team ou Enterprise Cloud.
- Branch protections e exigência nativa de revisores em repositórios privados.
- Code Owners e determinados controles avançados de segurança em repositórios privados.

**Alternativa Free**
- Se o código não puder ser público, manter repositório privado e reduzir escritores da branch padrão ao mínimo (idealmente só o proprietário), aplicar PRs e executar CI como gate de processo; documentar que isso **não equivale a enforcement server-side**.
- Executar scanners open source localmente e no CI (por exemplo, Gitleaks e Trivy), sem assumir recursos GitHub Advanced Security.
- Se a proteção server-side de branch for requisito inegociável, GitHub Free não atende esse requisito em repositório privado. Não publicar código sensível para obter um controle de plano gratuito.

### Auth0 Free

**Limites e recursos documentados**
- Até 25.000 Monthly Active Users.
- Até 5 Organizations.
- Actions, customização de formulários e Attack Protection básico.
- Conexões sociais ilimitadas e opções passwordless indicadas na página oficial.
- Custom domain exige verificação de cartão de crédito; para evitar essa etapa, usar inicialmente o domínio padrão do tenant.

**Limites e recursos Free**
- Machine-to-Machine Authentication: quota de até 1.000 autenticações/tokens por mês. A tabela atual também informa M2M Add-on = No para Free; não há ampliação gratuita presumida.
- Auth0 Database Connection integrada: incluída. Custom Database Connections externas: não incluídas no Free.
- Passwordless e passkeys aparecem como incluídos; avaliar o método concreto sem equiparar automaticamente passwordless a MFA.

**NÃO DISPONÍVEL NO FREE ou não deve ser pressuposto**
- M2M Add-on para ampliar quota: não incluído no Free. Não usar M2M sem cache e monitoramento da quota mensal.
- MFA com OTP/Duo e RBAC por Organization: aparecem no plano Essentials.
- Uso de banco de usuários externo/custom database connection: listado no Professional. Isso é diferente da Database Connection integrada Username-Password-Authentication, que está incluída.
- Mais de cinco Organizations.

**Alternativa Free**
- Para operações da API em nome de uma pessoa, usar OIDC Authorization Code Flow com PKCE e Access Token para a audience da API, com sessão server-side/BFF quando aplicável.
- Para operações sem usuário, usar o fluxo M2M do Auth0 somente dentro da quota de 1.000 autenticações/tokens por mês; reutilizar tokens, evitar emissão por requisição e acompanhar quota.
- Persistir tenants, memberships e permissions no PostgreSQL, associando o usuário por iss + sub; verificar membership e autorização no servidor e aplicar RLS.
- Não simular M2M com token de usuário. Ao atingir o limite Free, bloquear a operação com segurança e rever a arquitetura; não pressupor acesso ao M2M Add-on.
- Para autenticação forte sem depender do pacote MFA pago, avaliar passkeys/passwordless disponíveis no Free. Não declarar passwordless como equivalente automático a uma política MFA.
- Os requisitos do tenant permanecem: não adicionar Google OAuth2; usar Database Connection Username-Password-Authentication quando aplicável; API Maximum Access Token Lifetime = 900 segundos; Allow Skipping User Consent desabilitado. São requisitos a validar, não prova de configuração nem prova de M2M disponível.

### Vercel Hobby

**Restrição contratual**
- O Hobby é gratuito e direcionado a projetos pessoais; os termos restringem o uso a fins pessoais ou não comerciais.
- Não utilizar Hobby para hospedar ou operar um SaaS comercial, produto empresarial comercial ou serviço oferecido a clientes.
- A conta conectada foi observada em plano Hobby; isso não demonstra que a finalidade comercial do projeto seja permitida.

**Quotas e limites relevantes**
- A página Hobby consultada lista, entre outros limites, 4 CPU-hours ativas, 360 GB-hours de memória provisionada, até 1 milhão de invocações de Functions e 100 GB-hours de duração de Functions.
- Logs de runtime Hobby: 1 hora e até 4.000 linhas.
- Limites gerais incluem até 200 projetos, 100 deployments/dia e uma build concorrente; verificar sempre a página Limits porque a plataforma pode atualizar quotas.

**Alternativa Free**
- Para um produto comercial que precise custar zero em licenças, hospedar o Next.js em infraestrutura própria já disponível, com Node.js/Docker e reverse proxy, ou usar export estático quando a aplicação permitir.
- Só considerar outro serviço hospedado se seus termos autorizarem explicitamente uso comercial no seu tier gratuito e suas quotas forem suficientes. Não presumir que “$0” significa “uso comercial permitido”.
- A infraestrutura própria também tem custo operacional (hardware, energia, internet, manutenção, backups). Não declarar operação de produção sem teste de disponibilidade, segurança e recuperação.

### Neon Free

**Quotas atuais**
- 100 projetos na organização.
- 10 branches por projeto.
- 100 CU-hours por projeto/mês e autoscaling até 2 CU.
- 1 GB de PostgreSQL storage por projeto, 20 GB no agregado da organização.
- 5 GB de public network transfer por projeto/mês.
- Scale-to-zero após 5 minutos de inatividade, sem opção de desativar no Free.
- Instant restore com janela máxima de 6 horas e limite de 1 GB de change history; uma manual snapshot; monitoring history de 1 dia.

**NÃO DISPONÍVEL NO FREE**
- Protected branches nativas (indicadas para Launch/Scale).
- Desativar scale-to-zero.
- Exceder 10 branches sem excluir outra.
- Mais storage/transfer/compute que as quotas; o esgotamento pode suspender compute ou bloquear escritas.

**Alternativa Free**
- Separar roles administrativas, de migration e de runtime; criar roles SQL de privilégio mínimo sem membership em neon_superuser.
- Habilitar RLS e testar BYPASSRLS, memberships e grants efetivos com role real de runtime.
- Versionar migrations no GitHub, testar em branch temporária e limpar branches fora de uso.
- Manter backups externos criptografados e fazer restore de teste; o PITR curto não substitui backup independente.
- Em aws-sa-east-1 (São Paulo), PostgreSQL e Managed Better Auth estão listados; Object Storage, Functions, AI Gateway e Realtime não aparecem disponíveis. AI Gateway exige plano pago. Não basear arquitetura Free nestes recursos na região.

## 3. Arquitetura de referência sob restrição Free

1. **Código:** GitHub Free, repositório privado se confidencial; aceitar explicitamente que controles server-side de branch protegida não estão disponíveis para privados.
2. **Autenticação:** Auth0 Free somente com fluxo interativo de usuário; Auth0 Organizations limitado a cinco; autorização/membership no PostgreSQL.
3. **API:** validar Access Token (assinatura, iss, aud, exp, scopes) e membership no servidor. Não depender de M2M token do Auth0 Free.
4. **Dados:** Neon Free; usar branches com lifecycle controlado, roles SQL dedicadas, RLS, pooling e backup externo.
5. **Hospedagem:** Vercel Hobby apenas para projetos pessoais/não comerciais. Para SaaS comercial, escolher hospedagem própria ou outro serviço cujo plano gratuito autorize uso comercial; se nenhum serviço compatível for confirmado, declarar **BLOCKED — nenhum host comercial Free validado** em vez de implantar em Hobby.
6. **CI/CD:** GitHub Actions com permissões mínimas e cota controlada; não depender de Environment protection/review gates quando indisponíveis para repo privado. Não conectar produção até haver autorização e evidência.

## 4. Matriz de decisão: recurso requerido

| Requisito | Estado Free | Decisão |
|---|---|---|
| Repositório privado + ruleset/protected branch obrigatório | Não suportado pelo GitHub Free privado | BLOCKED; controles compensatórios não equivalem a enforcement |
| Auth0 M2M Client Credentials | Incluído até 1.000 autenticações/tokens por mês; M2M Add-on não incluído | Usar cache/reutilização, monitorar quota e bloquear com segurança ao atingir o limite |
| Mais de 5 Auth0 Organizations | Não incluído | Guardar tenant/membership no banco ou BLOCKED se Organizations nativas forem requisito |
| MFA nativo OTP/Duo do Auth0 | Essentials | Passkeys/passwordless ou provedor alternativo comprovadamente Free; não alegar equivalência |
| Vercel para SaaS comercial | Proibido pelos termos do Hobby | Não usar Hobby; self-host ou host alternativo com uso comercial gratuito confirmado |
| Neon protected production branch | Launch/Scale | Controles compensatórios SQL/migrations/backup; registrar ausência de proteção nativa |
| PITR/restore superior a 6h no Neon Free | Não incluído | Backup externo independente |
| Mais de 10 branches Neon por projeto | Não incluído | Reutilizar/limpar branches ou dividir projetos com cuidado |

## 5. Checklist de aprovação de uma nova integração Free

- [ ] Plano exato confirmado no Dashboard/API/CLI.
- [ ] Página oficial de preços/termos consultada na data atual.
- [ ] O recurso está explicitamente incluído no Free, não em trial ou add-on.
- [ ] Limites mensais, storage, compute, transfer, retenção e throttling documentados.
- [ ] Uso comercial permitido (quando aplicável) confirmado nos termos.
- [ ] Alternativa documentada para quota excedida e indisponibilidade.
- [ ] Não há cartão, pagamento, crédito promocional ou upgrade pressuposto.
- [ ] Segredos permanecem server-side e fora do Git.
- [ ] Isolamento de Development/Preview/Production comprovado.
- [ ] Backups e restauração testados, sem depender apenas de logs/CI.
- [ ] Status da implementação é Verificado, Recomendado ou Pendente, com evidência.

## Fontes oficiais

- [GitHub Pricing](https://github.com/pricing)
- [GitHub plans](https://docs.github.com/en/get-started/learning-about-github/githubs-plans)
- [GitHub ruleset availability](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)
- [GitHub Actions billing](https://docs.github.com/en/actions/concepts/billing-and-usage)
- [Auth0 Pricing](https://auth0.com/pt/pricing)
- [Vercel Hobby plan](https://vercel.com/docs/plans/hobby)
- [Vercel Terms of Service](https://vercel.com/legal/terms)
- [Vercel limits](https://vercel.com/docs/limits)
- [Neon Free quotas](https://neon.com/faqs/free-plan-limits-and-quotas)
- [Neon plans](https://neon.com/docs/introduction/plans)
- [Neon regions](https://neon.com/docs/introduction/regions)
- [Neon protected branches](https://neon.com/docs/guides/protected-branches)
- [Neon roles](https://neon.com/docs/manage/roles)
