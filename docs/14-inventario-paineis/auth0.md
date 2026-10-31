# Auth0 Free — inventário do Dashboard

> Revisão documental: 31/10/2026. A navegação redesenhada está em Early Access e pode ser ativada por usuário. A navegação clássica pode continuar diferente; usar breadcrumbs e busca.

## 1. Navegação documentada

| Seção/menu | Submenus | Finalidade |
|---|---|---|
| Primary | Home | Visão inicial e atalhos |
| Primary | Agents | Configuração de agentes; não presumir que todos os recursos sejam Free |
| Primary | Users | Pesquisa e gestão de usuários |
| Primary | Organizations | Organizações e membros |
| Primary | Settings | Configuração do tenant e conta |
| Integration | Applications | Registro e gestão de aplicações |
| Integration | APIs | Resource Servers, identifiers, scopes e políticas |
| Integration | Authentication | Database e Passwordless |
| Integration | Connections | Conexões de identidade, incluindo social |
| Integration | Enterprise SSO | Conexões enterprise; não presumir disponibilidade Free |
| Integration | Authorization | Roles e autorização |
| Customization | Branding | Tema do Universal Login |
| Customization | Pages | Universal Login e Forms |
| Customization | Portals | Editor de portais; Developer Preview, não presumir Free |
| Customization | Domains | Domínios customizados |
| Messaging | Emails | Templates e configuração de email |
| Messaging | Phone | Mensagens por telefone |
| Automation & Extensibility | Actions | Actions e triggers |
| Automation & Extensibility | Events | Envio de eventos a serviços externos |
| Automation & Extensibility | Extensions | Extensões oficiais/terceiros; validar custo |
| Automation & Extensibility | Rules | Rules legadas |
| Automation & Extensibility | Hooks | Hooks legados |
| Security & Monitoring | Attack Protection | Bot, IP throttling, brute force e breached password |
| Security & Monitoring | Access Control | ACLs do tenant |
| Security & Monitoring | Security Dashboard | Métricas e visão de ameaças |
| Security & Monitoring | Logs | Eventos e logs do tenant |
| Security & Monitoring | Analytics | Uso e relatórios |

A navegação redesenhada é documentada como Early Access em todos os planos. Isso não significa que cada funcionalidade listada esteja incluída no Free. Validar a elegibilidade de cada recurso.

## 2. Tenant e contexto

Antes de configurar, confirmar:
- Tenant ativo e domínio.
- Região/ambiente do tenant.
- Plano Free efetivo.
- Usuário e papel administrativo.
- Navegação nova ou clássica.
- Aplicação e API alvo.

Não confundir conta Auth0, organização de conta, tenant e aplicação.

## 3. Applications

Caminho: Applications > Applications.

Para Next.js:
- Tipo Regular Web Application.
- Identificar Client ID e Client Secret sem copiá-los para documentação.
- Revisar Application Type, Grant Types e propriedades.
- Allowed Callback URLs: somente callback exato do ambiente.
- Allowed Logout URLs: somente origens autorizadas.
- Allowed Web Origins: origens estritamente necessárias.
- Separar aplicações por ambiente quando o isolamento justificar.

URLs locais de referência:
- Callback: http://localhost:3000/auth/callback.
- Logout: http://localhost:3000.
- Origin: http://localhost:3000.

Não reutilizar URL local em produção. Para Preview, controlar hosts e callbacks autorizados; evitar curingas amplos.

## 4. APIs

Caminho: Applications > APIs (na navegação clássica pode aparecer como APIs).

Revisar:
- Nome e Identifier/audience.
- Signing Algorithm.
- Token Expiration / Token Lifetime.
- Allow Offline Access, somente se necessário.
- RBAC e Add Permissions in the Access Token.
- Scopes por operação.
- Application Access / Client Grants para M2M.

Requisito do projeto: Max Access Token Lifetime = 900 segundos, pendente de confirmação no painel e teste do token emitido.

Aplicações clientes e APIs são objetos diferentes: a aplicação solicita tokens; a API os valida e autoriza.

## 5. Authentication e Connections

Caminhos: Authentication > Database; Authentication > Passwordless; Connections > categorias disponíveis.

Revisar:
- Connection habilitada para a aplicação correta.
- Signup e login.
- Campos de perfil.
- Password policy e proteção de credenciais.
- Configuração de email/telefone e provedores.
- Aplicações autorizadas a usar a conexão.

Requisitos específicos:
- Database Connection: Username-Password-Authentication, quando autenticação por senha for usada.
- Não configurar Google OAuth2 como conexão social.
- São requisitos documentais, não prova do estado atual do tenant.

## 6. Users e Organizations

Users:
- Pesquisar por email/identificador.
- Verificar status, bloqueios, metadados e identidades.
- Administrar sessões/refresh tokens quando disponível.
- Não exportar dados pessoais para logs compartilhados.

Organizations:
- Útil para arquitetura B2B/multi-tenant.
- Revisar nome, display name, branding, domains, connections, membros e convites.
- Free tem limite de Organizations; confirmar quota vigente.
- Não presumir RBAC por Organization incluído no Free.

## 7. Authorization

Caminho: Authorization.

Revisar roles, permissions, associações usuário-role, role-permission, API/scopes e claims. Não confundir RBAC global com RBAC por Organization. A autorização efetiva deve permanecer no servidor e no PostgreSQL, com escopo do usuário validado em cada operação.

## 8. Branding, Pages e Domains

- Branding: logotipo, cores e identidade visual do Universal Login.
- Pages: páginas hospedadas e Forms, conforme disponibilidade.
- Domains: domínio customizado e validação de propriedade/certificados.

No Free, não assumir domínio customizado, editor avançado, portais ou recursos de preview incluídos. Se não constar no plano, marcar NÃO DISPONÍVEL NO FREE e manter o domínio padrão.

## 9. Actions, Events e extensibilidade

Actions:
- Revisar trigger, código, dependências, secrets e versão implantada.
- Implantar Action após alterações, conforme fluxo do produto.
- Evitar chamadas lentas/não confiáveis no login.
- Testar falhas e logs.

Events:
- Configurar destinos necessários.
- Validar autenticação, retry, duplicidade e proteção de payloads.

Rules e Hooks:
- Superfícies legadas; não iniciar implementação nova.
- Migrar lógica existente para Actions em projeto separado e testado.

Extensions:
- Verificar custo, manutenção e permissões. Não usar extensão paga ou trial.

## 10. Security & Monitoring

Attack Protection:
- Brute-force Protection.
- Breached Password Detection.
- Bot Detection.
- Suspicious IP Throttling.

Security Dashboard: revisar métricas, janela de retenção e filtros disponíveis no plano.

Logs: investigar login, logout, falhas, bloqueios e alterações administrativas. Não registrar tokens ou senhas; minimizar PII.

Access Control: ACLs e regras de acesso ao tenant; recursos avançados podem depender de elegibilidade específica.

MFA: não presumir MFA OTP/Duo incluído no Free. Validar fatores efetivamente disponíveis; não ativar add-on pago ou trial.

## 11. Settings e governança

Caminho: Settings e menu de perfil no topo.

Revisar:
- Tenant settings.
- Session lifetime e timeout.
- Tenant members e papéis.
- MFA de administradores do Dashboard.
- Logs de atividade e auditoria, se disponíveis.
- Configuração de email/telefone.
- Feature previews.
- Região e configurações do tenant.

A preferência pela navegação nova é individual e pode ser alternada pelo menu de perfil. A nomenclatura pode diferir da navegação clássica.

## 12. Restrições Free e requisitos

- Auth0 Free: confirmar quotas vigentes de MAU, Organizations e M2M no pricing oficial.
- M2M Add-on: não incluído no baseline Free; respeitar quota base.
- MFA OTP/Duo: não assumir incluído.
- RBAC por Organization: não assumir incluído.
- Custom Domains e recursos avançados: confirmar elegibilidade.
- Não recomendar trials, add-ons ou upgrades.

Checklist do tenant:
- [ ] Plano Free efetivo.
- [ ] Sem conexão Google OAuth2.
- [ ] Username-Password-Authentication habilitada quando aplicável.
- [ ] API com token lifetime máximo de 900 s.
- [ ] Allow Skipping User Consent desabilitado, se aplicável.
- [ ] Callbacks/logout URLs exatos por ambiente.
- [ ] MFA administrativa conforme opção incluída.
- [ ] Logs sem secrets.

## 13. Fontes oficiais

- [Navigate Auth0 Dashboard](https://auth0.com/docs/get-started/auth0-overview/dashboard/use-dashboard-navigation)
- [Auth0 Dashboard Overview](https://auth0.com/docs/get-started/auth0-overview/dashboard)
- [Auth0 Pricing](https://auth0.com/pricing)
- [Applications](https://auth0.com/docs/get-started/applications)
- [APIs](https://auth0.com/docs/get-started/apis)
- [Database Connections](https://auth0.com/docs/authenticate/database-connections)
- [Actions](https://auth0.com/docs/customize/actions)
- [Attack Protection](https://auth0.com/docs/secure/attack-protection)
- [Auth0 Next.js SDK](https://auth0.com/docs/libraries/nextjs-auth0)

## 14. Pendências de validação do tenant

- [ ] Registrar navegação clássica/nova e data.
- [ ] Confirmar plano e quotas.
- [ ] Conferir Application Type, callbacks, logout e origins.
- [ ] Confirmar ausência de Google OAuth2.
- [ ] Conferir API e token lifetime de 900 s.
- [ ] Conferir consent settings.
- [ ] Verificar elegibilidade Free de MFA, Organizations, domains e Actions.
- [ ] Registrar evidência não sensível, sem Client Secret, token ou PII.
