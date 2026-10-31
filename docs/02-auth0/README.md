# 02 — Auth0

## Escopo e estado

Esta seção cobre tenants, applications, APIs, connections, Universal Login, Actions, RBAC, tokens, sessões, segurança e monitoramento. O estado atual do tenant não é considerado verificado sem inspeção direta do Dashboard, Management API ou CLI.

## Configuração padrão inicial

1. Definir tenant e região adequados à residência de dados e latência.
2. Criar uma aplicação por cliente/runtime e ambiente quando o isolamento exigir credenciais independentes.
3. Para aplicações web tradicionais, usar o tipo de aplicação e fluxo recomendado pelo SDK oficial; não implementar implicit flow legado.
4. Habilitar apenas as connections necessárias. Para autenticação por senha, avaliar Database Connection com política de senha, proteção contra ataques e recuperação de conta.
5. Registrar URLs permitidas de callback, logout e web origins com valores exatos por ambiente; evitar curingas amplos.
6. Definir API Identifier/audience, scopes e política de autorização antes de consumir a API.
7. Configurar sessão, expiração e renovação conforme risco e arquitetura.
8. Guardar client secret e secrets de Actions exclusivamente no cofre/variáveis seguras do ambiente consumidor.

## Configuração completa — inventário do Dashboard

| Área | O que documentar | Validação |
|---|---|---|
| Tenant Settings | domínio, região, políticas, suporte e limites | comparar tenant e ambiente |
| Applications | tipo, grant types, callbacks, logout, origins, credenciais | teste de login/logout |
| APIs | identifier, signing algorithm, token lifetime, RBAC, scopes | validar issuer/audience/scopes |
| Authentication / Database | connection, signup, password policy, breach detection | fluxo de cadastro e login |
| Social / Enterprise | provedores habilitados e justificativa | teste apenas dos provedores aprovados |
| Universal Login | experiência, branding, custom domain | validar redirect e domínio |
| Actions / Flows | triggers, secrets, dependências, deploy | teste em ambiente não produtivo |
| Organizations | membership, invitations, roles, connections | teste de isolamento por organização |
| Security | MFA, attack protection, bot detection, session controls | teste negativo e revisão de risco |
| Monitoring | logs, alerts, exports e retenção | evento de teste e confirmação de entrega |

### Tokens e sessões

- Validar assinatura, `iss`, `aud`, `exp` e `nbf` no backend.
- Não confundir ID Token com Access Token; cada um tem finalidade e audiência próprias.
- Solicitar somente scopes necessários e documentar cada scope.
- Definir access-token lifetime e refresh-token rotation conforme risco, plano e suporte do cliente.
- Não armazenar tokens em `localStorage` por padrão; preferir sessão server-side/BFF quando aplicável.

### Integração Next.js

Usar versão suportada do SDK oficial e seguir o guia correspondente à versão instalada. Documentar middleware/handlers, rotas protegidas, sessão, logout, callback, variáveis necessárias e comportamento em Preview. Nunca copiar exemplos de SDK de outra major version sem revisar breaking changes.

### Validação e rollback

Testar login, logout, callback, token inválido/expirado, usuário sem permissão, MFA (se habilitado), recuperação de senha e URLs de Preview. Mudanças de Actions, policies, connection e credenciais devem ter plano de reversão.

## Restrições específicas do projeto

Manter as decisões explicitamente aprovadas no projeto: não adicionar Google OAuth2; usar a Database Connection `Username-Password-Authentication` quando aplicável; respeitar o plano Free; manter Maximum Access Token Lifetime em 900 segundos e Allow Skipping User Consent desabilitado. Esses itens são requisitos a validar no tenant, não evidência de que o painel já esteja configurado dessa forma.

## Referências

- [Auth0 Documentation](https://auth0.com/docs)
- [Auth0 Next.js SDK](https://auth0.com/docs/quickstart/webapp/nextjs)