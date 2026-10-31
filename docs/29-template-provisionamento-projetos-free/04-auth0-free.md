# Auth0 — configuração de referência no plano Free

## Escopo e limites

Planejar a solução para um tenant Free e validar limites vigentes antes de criar aplicações e conexões. Não depender de role management gerenciado, add-ons, múltiplos tenants, Organizations avançadas ou retenção longa de logs. O Free pode limitar recursos, administradores, conexões, MAU e funcionalidades; conferir painel e página oficial de preços na data de adoção.

## Configuração padrão inicial

1. Criar/selecionar tenant autorizado do projeto.
2. Registrar região, domínio e owner do tenant.
3. Criar application apropriada para aplicação web Next.js.
4. Definir callback, logout URL e web origin para Development.
5. Habilitar somente a connection necessária.
6. Configurar Universal Login com política de consentimento adequada.
7. Gerar e armazenar secrets fora do Git.
8. Configurar SDK Auth0 v4 no servidor.
9. Testar login, callback, sessão e logout em ambiente local com conta de teste.

Não reutilizar uma application entre projetos sem avaliar isolamento, callbacks, responsáveis e blast radius.

## URLs de referência

Para Development local:
- Allowed Callback URL: http://localhost:3000/auth/callback
- Allowed Logout URL: http://localhost:3000
- Allowed Web Origin: http://localhost:3000

Para Preview e Production, registrar domínios exatos, sem curingas amplos desnecessários. Confirmar endpoints exigidos pela versão instalada do SDK.

## SDK v4

~~~typescript
// src/lib/auth0.ts
import { Auth0Client } from "@auth0/nextjs-auth0/server";

export const auth0 = new Auth0Client();
~~~

A configuração por variáveis de ambiente deve seguir a versão instalada do SDK. Não copiar APIs legadas do SDK v3, como handleAuth e rotas catch-all antigas.

A convenção de middleware/proxy depende da versão principal do Next.js, conforme o guia de bootstrap. Revisar documentação do SDK v4 e do Next.js na versão exata.

## Autenticação e autorização

- Autenticação confirma identidade; autorização decide o que essa identidade pode fazer.
- Obter sub exclusivamente da sessão/token validado no servidor.
- Não aceitar sub, role, tenant ou permission enviados pelo browser como autoridade.
- Verificar autorização em cada Route Handler, Server Action e operação sensível.
- Não presumir que RBAC do Auth0 está disponível no Free.
- Se roles forem mantidas na aplicação, armazenar e administrar permissões no banco com controles server-side.
- Email não é identificador estável; verificar email_verified quando confirmação for requisito.
- Manter consentimento explícito; não ativar opções de pular consentimento sem análise.

## Checklist

- [ ] Tenant e application identificados.
- [ ] Callback/logout/web origins exatos por ambiente.
- [ ] Connection elegível e habilitada.
- [ ] SDK v4 e versão do Next.js compatíveis.
- [ ] Sessão criada e encerrada corretamente.
- [ ] Rotas protegidas verificam sessão no servidor.
- [ ] Autorização de domínio não depende de dados do browser.
- [ ] Secrets não são versionados nem exibidos.
- [ ] Limites Free e recursos utilizados registrados com data.

Referências: [Auth0 Docs](https://auth0.com/docs), [Next.js SDK](https://auth0.com/docs/quickstart/webapp/nextjs), [Auth0 Pricing](https://auth0.com/pricing).
