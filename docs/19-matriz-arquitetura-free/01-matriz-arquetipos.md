# Matriz comparativa dos arquétipos

## 1. Aplicação simples
Exemplos: ferramenta pessoal, portfólio autenticado, protótipo individual não comercial.

| Camada | Topologia | Isolamento/limite |
|---|---|---|
| GitHub | Um repo por app ou monorepo pequeno | CI sem environment secrets em repo privado Free |
| Vercel | Um projeto Hobby por app | Somente pessoal/não comercial |
| Auth0 | Tenant Free único; até 10 Applications por tenant para todos os projetos | Callbacks/logout/web origins estritos; client não equivale a tenant; inventariar slots antes de criar apps |
| Neon | Um projeto por aplicação; branches para dev/teste | Branch não é backup independente |

**Decisão:** RECOMENDADO se pessoal/não comercial, dados de baixo risco e quotas suficientes.

## 2. SaaS multiusuário
Exemplo: contas, organizações e dados separados por cliente.

| Camada | Topologia de protótipo | Limite |
|---|---|---|
| GitHub | Repo privado; CI sem segredos de produção | Environment secrets indisponíveis em repo privado Free |
| Vercel | Hobby apenas para protótipo pessoal/não comercial | **BLOQUEADO** para SaaS comercial |
| Auth0 | Tenant Free único; até 10 Applications por tenant | Não há isolamento de tenant por cliente/ambiente; quota global compartilhada |
| Neon | Projeto por domínio do SaaS; roles runtime/migrator; RLS | RLS requer testes negativos e contexto transacional seguro |

**Decisão:** CONDICIONAL para protótipo pessoal/não comercial; BLOQUEADO NO FREE para SaaS comercial na Vercel Hobby.

Controles mínimos: tenant_id obrigatório nas tabelas multi-tenant; backend deriva identidade do token validado; membership/role/action verificados no servidor; RLS com USING e WITH CHECK; contexto aplicado na mesma transação da query; runtime sem owner/superuser/BYPASSRLS; testes negativos cross-tenant; logs sem tokens ou dados sensíveis.

Isolamento lógico deixa de ser suficiente quando contrato/regulação exige separação física, restore seletivo por cliente, chaves/retention diferentes, blast radius inaceitável ou RLS não pode ser provada. Projeto Neon separado reduz acoplamento de dados, mas consome quotas; se quota insuficiente, BLOQUEADO NO FREE.

## 3. API/backend
Exemplos: API REST, integração, backend de processamento.

| Camada | Topologia | Limite |
|---|---|---|
| GitHub | Repo próprio ou workspace; workflow mínimo | Restringir permissões e secrets |
| Vercel | Projeto Hobby para API pessoal/não comercial, runtime compatível | API comercial bloqueada |
| Auth0 | API audience/resource server e clients consumidores | Scopes não substituem tenant separado |
| Neon | Projeto por domínio de dados | Validar pooling, concorrência, quotas e escala a zero |

Padrões: validar assinatura, issuer, audience, expiração e scopes; separar autenticação de autorização; rate limit e payload limits; role runtime separada de migration; nunca expor connection string/client secret em variáveis públicas; documentar timeout, retry idempotente e cold start.

## 4. Múltiplas aplicações compartilhando serviços
Exemplo: vários portais e frontends sob uma conta.

| Serviço | Compartilhar | Separar quando |
|---|---|---|
| GitHub | Templates/workflows; repo por app | Ownership, permissões ou release divergentes |
| Vercel | Um projeto por aplicação | Domínio, variáveis ou ciclo de release independente; Hobby segue não comercial |
| Auth0 | Tenant Free único; até 10 Applications por tenant | Necessidade de diretórios, políticas, conexões ou blast radius isolados |
| Neon | Projeto por app/domínio de dados | Ownership, classificação, restore ou acesso diferentes |

**Decisão:** CONDICIONAL. Reutilizar Auth0 e templates reduz duplicação, mas não cria isolamento forte. Preferir Neon separado quando os domínios de dados não compartilham a mesma fronteira de segurança.

## Resumo: o que cada fronteira entrega

| Fronteira | Controle | Não substitui |
|---|---|---|
| Aplicação | Projeto Vercel/client Auth0 distinto | Tenant Auth0 independente |
| Ambiente | URLs, clients, secrets e branches separados | Tenant separado |
| Dados multi-tenant | RBAC + RLS + contexto transacional | Isolamento físico quando exigido |
| App para app | Projetos Neon e credenciais distintos | Isolamento da conta/organização |
| CI | Repositórios/permissões/workflows mínimos | Environment secrets em repo privado Free |

## Fontes
[Auth0 tenant Free](https://support.auth0.com/center/s/article/Second-Tenant-on-a-Free-Plan) · [Limite de 10 Applications](https://support.auth0.com/center/s/article/Error-Problem-creating-clients-Forbidden-You-reached-the-limit-of-entities-of-this-type-for-this-tenant) · [Vercel Hobby](https://vercel.com/docs/plans/hobby) · [Vercel Fair Use](https://vercel.com/docs/limits/fair-use-guidelines) · [Neon quotas](https://neon.com/docs/introduction/plans) · [GitHub environments](https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments)
