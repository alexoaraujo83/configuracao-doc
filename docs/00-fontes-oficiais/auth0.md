# Auth0 Docs — referência operacional detalhada

Fonte primária: [Auth0 Docs](https://auth0.com/docs). O Dashboard muda ao longo do tempo; registrar rótulo observado, tenant, região, plano e data de inspeção.

## 1. Modelo de identidade

Documentar:
- Tenant: domínio, região, ambiente, plano, custom domains e políticas globais.
- Application: cliente OAuth/OIDC, tipo, grant types, callbacks, logout URLs, web origins, credenciais e owners.
- API/resource server: identifier/audience, algoritmo de assinatura, token lifetime, RBAC, scopes e consent.
- Connection: Database, Social, Enterprise, realms e applications autorizadas.
- User/profile: atributos, metadata, lifecycle, bloqueios e recuperação de conta.
- Organization: membership, invitations, roles, connections e branding.

## 2. Configuração padrão inicial

1. Separar tenants por ambiente quando isolamento, risco ou governança justificar; múltiplas Applications no mesmo tenant não equivalem necessariamente a isolamento de tenant.
2. Criar aplicações distintas por tipo de cliente e ambiente quando precisarem de callbacks, credenciais ou políticas independentes.
3. Escolher fluxo OIDC/OAuth compatível com o tipo de aplicação; evitar implicit grant legado.
4. Configurar callback, logout e web origins com URLs exatas, sem curingas amplos.
5. Habilitar apenas as connections necessárias para cada application.
6. Criar API identifier estável e scopes mínimos.
7. Definir políticas de token, sessão, MFA e proteção contra ataques.
8. Armazenar client secret e secrets de Actions apenas em cofre/variáveis server-side.
9. Testar login, logout, renovação, autorização e erros antes de produção.

## 3. Inventário do Dashboard

| Domínio | Configuração detalhada | Evidência/teste |
|---|---|---|
| Tenant Settings | domínio, região, custom domain, branding, suporte, limites | valores não secretos + data |
| Applications | tipo, grant types, client auth, callbacks, logout, web origins, credenciais | login/callback/logout |
| APIs | identifier, signing algorithm, token expiration, RBAC, permissions, consent | validar issuer, audience, assinatura, expiração e scopes |
| Authentication / Connections | Database, Social, Enterprise, realms, app assignments | testar cada connection aprovada |
| Database | signup, password policy, breach detection, passwordless se aplicável | cadastro, login, reset e bloqueio |
| Universal Login | experiência, branding, templates, prompts, custom domain | redirect, domínio e responsividade |
| Actions / Triggers | trigger, ordem, runtime, dependencies, secrets, deploy/version | testes e rollback |
| Organizations | organizations, membership, invitations, roles, connections | isolamento entre organizações |
| User Management | users, roles, permissions, metadata, import/export, blocking | least privilege e auditoria |
| Security | MFA, attack protection, bot detection, breached password, suspicious IP, sessions | testes de desafio e bloqueio |
| Monitoring | logs, event types, log streams, alerts, retention | evento sintético e entrega |
| Extensions / Integrations | apps, integrations, grants, webhooks e permissões | revisão de acesso |

## 4. Applications e OAuth/OIDC

Para cada application registrar:
- Application Type e SDK/runtime;
- Client ID (identificador, não segredo) e localização segura do Client Secret (não copiar o valor);
- Allowed Callback URLs, Allowed Logout URLs e Allowed Web Origins por ambiente;
- grant types e client authentication;
- connection assignments;
- token endpoint authentication method;
- consent e classificação first-party/third-party;
- owners e data de revisão.

Não reutilizar callback de produção em Preview de forma genérica. Wildcards ampliam a superfície de ataque; se inevitáveis, limitar domínio e caminho e registrar justificativa.

## 5. APIs, tokens e autorização

- Access Token é destinado à API/audience; ID Token comunica autenticação ao cliente. Não tratar um como substituto do outro.
- A API deve validar assinatura, issuer, audience, expiração e claims aplicáveis.
- Definir scopes/permissions por operação e verificar autorização no servidor.
- Modelar RBAC com papéis mínimos; controles visuais no frontend não são autorização.
- Não inserir segredos ou dados sensíveis no payload do token.
- Definir access token lifetime conforme risco, UX e requisitos de revogação.
- Para Refresh Tokens, avaliar habilitação, rotation, absolute lifetime, inactivity lifetime e reuse detection.
- Documentar consent e não desabilitá-lo sem análise de confiança e contexto.

## 6. Sessões, MFA e proteção contra ataques

Distinguir três camadas: sessão do tenant Auth0, sessão da aplicação e validade de access/refresh tokens. Um timeout no tenant não implica necessariamente expiração simultânea do cookie de sessão do SDK.

Avaliar inactivity e absolute session lifetime, persistent session, logout, MFA por política/contexto, brute-force protection, breached password detection, suspicious IP throttling, bot detection, recuperação de conta, rate limits e alertas.

## 7. Universal Login e custom domains

Documentar experiência de login, branding, templates, prompts, idioma, custom domain, certificados e DNS. Validar issuer e discovery metadata no domínio efetivo. Mudanças de domínio exigem revisar callbacks, cookies, SDK e tokens emitidos.

## 8. Actions

Para cada Action registrar trigger/ordem, runtime/versão, dependências fixadas, nomes de secrets (nunca valores), timeout/limites, chamadas externas, logs sem PII/token, testes e rollback.

Actions podem adicionar claims customizadas. Usar namespace próprio e evitar colisão com claims padrão. Validar os tokens após cada mudança.

## 9. Integração Next.js / Vercel

A versão instalada de @auth0/nextjs-auth0 é determinante. O SDK v4 tem arquitetura e APIs diferentes das versões anteriores; exemplos legados com handleAuth e /api/auth/[...auth0] não devem ser aplicados automaticamente a projetos v4. Consultar o guia e exemplos atuais do SDK, incluindo proxy.ts quando aplicável à versão do Next.js.

Nomes de variáveis variam por versão. Exemplos usados em diferentes gerações/integrações incluem AUTH0_SECRET, AUTH0_BASE_URL ou APP_BASE_URL, AUTH0_ISSUER_BASE_URL ou AUTH0_DOMAIN, AUTH0_CLIENT_ID e AUTH0_CLIENT_SECRET. Não assumir que todos coexistem na mesma major version.

No Vercel:
- cadastrar secrets separadamente em Development, Preview e Production;
- definir base URL correta para cada deployment/ambiente;
- allowlistar callback e logout URL de forma controlada;
- não usar prefixo NEXT_PUBLIC_ em segredo;
- criar novo deployment após alterar env vars, pois deployments existentes não recebem automaticamente os novos valores.

## 10. Restrições específicas deste projeto

Requisitos a validar no tenant:
- não adicionar Google OAuth2;
- usar Database Connection Username-Password-Authentication quando aplicável;
- permanecer no plano Free;
- manter Maximum Access Token Lifetime = 900 segundos;
- manter Allow Skipping User Consent desabilitado.

Não declarar esses valores como ativos sem evidência direta do tenant. Se o nome ou posição da opção mudar, registrar a opção equivalente e a evidência.

## 11. Checklist
- [ ] Tenant, região e plano registrados.
- [ ] Applications separadas conforme tipo e ambiente.
- [ ] Callbacks/logout/web origins exatos.
- [ ] Connections mínimas e atribuídas explicitamente.
- [ ] API audience/scopes/RBAC validados.
- [ ] Token lifetime e consent conferidos.
- [ ] Sessão da aplicação e sessão do tenant testadas separadamente.
- [ ] MFA e attack protection avaliados.
- [ ] Actions revisadas e versionadas.
- [ ] Logs sem secrets e PII desnecessária.
- [ ] SDK e versão Next.js confirmados.

## Fontes oficiais
- [Auth0 Docs](https://auth0.com/docs)
- [Índice da documentação Auth0](https://auth0.com/llms.txt)
- [Next.js SDK quickstart](https://auth0.com/docs/quickstart/webapp/nextjs)
- [Next.js integration guide](https://developer.auth0.com/resources/get-started/web-app/nextjs)
- [Next.js App Router guide](https://developer.auth0.com/resources/guides/web-app/nextjs/basic-authentication)
- [Deploy Auth0 + Next.js to Vercel](https://developer.auth0.com/resources/guides/web-app/nextjs/vercel-deployment)

## Limites e observações
MFA, Organizations, log streaming, attack protection, custom domains e limites do tenant podem variar por plano/região. Verificar disponibilidade no tenant. Esta documentação não executa alterações no tenant.

## Baseline de compatibilidade — Auth0 Free (31/10/2026)

A página oficial de preços apresenta o Free com até **25.000 MAU** e até **5 Organizations**, além de Actions, Attack Protection básico e opções passwordless. O domínio personalizado exige verificação de cartão; quando a política for não cadastrar cartão, manter o domínio padrão do tenant.

| Recurso | Classificação Free | Decisão operacional |
|---|---|---|
| Até 25 mil MAU | Incluído | Monitorar consumo e limites do tenant |
| Até 5 Organizations | Incluído | Não projetar mais de cinco Organizations nativas |
| Actions | Incluído no Free | Restringir triggers, dependências e dados de entrada |
| Attack Protection básico | Incluído | Não confundir com recursos avançados |
| M2M Authentication | Incluída até 1.000 autenticações/tokens por mês no Free; M2M Add-on: Não | Usar Client Credentials dentro da quota, reutilizar tokens e monitorar consumo; o add-on para ampliar quota não está incluído |
| RBAC por Organization | **NÃO DISPONÍVEL NO FREE** — aparece em Essentials | Gerenciar memberships/permissions no PostgreSQL e autorizar no servidor |
| MFA com OTP/Duo | **NÃO DISPONÍVEL NO FREE** — aparece em Essentials | Avaliar passkeys/passwordless; não chamar passwordless de MFA automaticamente |
| Banco de usuários externo/custom database connection | **NÃO DISPONÍVEL NO FREE** — listado em Professional | Diferenciar isso da Database Connection integrada Username-Password-Authentication |
| Custom domain | Requer verificação de cartão | Usar auth0 domain padrão se nenhum cartão puder ser verificado |

### Consequência para a API IAM

O Auth0 Free inclui Machine-to-Machine Authentication até a quota de 1.000 autenticações/tokens por mês; a tabela de preços também indica M2M Add-on = No. Portanto, Client Credentials pode ser usado dentro desse limite, mas não se deve depender de M2M ilimitado nem de add-on para ampliar a quota no Free. Reutilizar/cachar Access Tokens até próximo da expiração, evitar solicitar novo token a cada operação, medir o consumo mensal e definir alerta e bloqueio seguro ao atingir o limite.

Para operações iniciadas por usuário, usar OIDC Authorization Code Flow com PKCE, Access Token destinado à API e validação server-side de assinatura, issuer, audience, expiração e scopes. Carregar membership e permission do banco após validar iss + sub; aplicar RLS com o contexto autenticado.

Não substituir M2M por ID Token, sessão compartilhada ou segredo estático no frontend. Se o requisito superar a quota Free, a arquitetura deve ser revista sem pressupor contratação de add-on; qualquer identidade de serviço alternativa requer revisão específica de segurança, chave privada server-side, validade curta, audience estrita, rotação e auditoria.

Os requisitos do projeto — não adicionar Google OAuth2; usar Username-Password-Authentication quando aplicável; Maximum Access Token Lifetime = 900 segundos; Allow Skipping User Consent desabilitado — continuam válidos como requisitos a verificar. Eles não comprovam que as opções estejam salvas; a quota M2M Free também deve ser monitorada no painel de uso.

Fontes oficiais: [Auth0 Pricing](https://auth0.com/pt/pricing) (a tabela compara M2M Authentication e M2M Add-on) e [What to Expect When Your Auth0 Startup Plan Expires](https://auth0.com/blog/what-to-expect-when-your-auth0-startup-plan-expires/) (referência da quota de 1.000 M2M no Free). Revalidar antes de uma mudança no tenant.
