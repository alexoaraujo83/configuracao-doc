# Segurança integrada — Next.js, Auth0 e Neon (somente Free)

**Status:** padrão recomendado; pendente de validação em aplicação real.  
**Escopo:** documentação técnica, sem alterações em tenants, banco, secrets, workflows ou deployments.  
**Stack:** Next.js App Router + TypeScript, Auth0 SDK v4, Neon PostgreSQL e GitHub Free/Vercel Hobby.

## Objetivo e princípios

Definir controles coerentes entre identidade, sessão web, autorização de domínio, isolamento de dados, transporte, segredos e pipeline de mudanças.

Princípios:
1. Autenticação não equivale a autorização.
2. A identidade confiável vem da sessão validada no servidor; nunca de campos enviados pelo browser.
3. A autorização é verificada em cada operação e para cada recurso/tenant.
4. PostgreSQL grants e RLS são controles complementares.
5. O contexto de tenant deve existir somente durante a transação que executa a consulta.
6. Segredos e dados pessoais não entram no bundle cliente, logs, commits ou artefatos.
7. Deploy Preview não recebe credenciais de produção.
8. O plano Free não justifica reduzir controles de segurança.

## Mapa dos documentos

| Documento | Conteúdo |
|---|---|
| [01 — Modelo de ameaças e fronteiras](01-modelo-ameacas-fronteiras.md) | Ativos, ameaças, limites de confiança |
| [02 — Auth0, OIDC e sessão](02-auth0-oidc-sessao.md) | Fluxos, cookies, tokens e callbacks |
| [03 — Next.js, APIs e CSRF](03-nextjs-api-csrf.md) | Server Components, Route Handlers, Server Actions |
| [04 — Autorização e multi-tenancy](04-autorizacao-multitenancy.md) | RBAC de domínio, membership e tenant |
| [05 — PostgreSQL grants e RLS](05-postgresql-rls.md) | Roles, policies e contexto transacional |
| [06 — Segredos, headers e logging](06-segredos-headers-logging.md) | Env, cache, CSP, auditoria |
| [07 — Checklist e resposta](07-checklist-resposta.md) | Aceite, testes e incidentes |

## Modelo de confiança

~~~text
Browser não confiável
  │ HTTPS + cookie de sessão
  ▼
Next.js (fronteira server-side)
  ├─ valida sessão Auth0
  ├─ deriva sub do usuário
  ├─ verifica membership e permissão
  ├─ valida payload/CSRF
  └─ abre transação com contexto de tenant
       ▼
Neon PostgreSQL
  ├─ role runtime sem privilégios de owner
  ├─ grants mínimos
  └─ RLS ENABLE + FORCE
~~~

## Convenções de evidência

- **Recomendado:** padrão proposto para novos projetos.
- **Pendente:** requer confirmação de versão, painel, plano ou execução autorizada.
- **Não disponível no Free:** recurso que não pode ser pressuposto sem confirmação oficial da elegibilidade.
- Não declarar controle como ativo apenas porque existe um exemplo neste repositório.

## Decisões obrigatórias

- Auth0 SDK v4 em aplicação Next.js server-rendered.
- Não utilizar APIs legadas v3 como handleAuth ou /api/auth/[...auth0].
- Next.js 15 usa middleware.ts; Next.js 16 usa proxy.ts conforme documentação da versão instalada.
- Não depender de Auth0 Organizations ou add-ons como requisito arquitetural.
- Papéis e memberships do domínio ficam no PostgreSQL; claims Auth0 são identidade/atributos, não substituto de autorização de recurso.
- Neon runtime role não é owner, não tem BYPASSRLS e não executa DDL.
- Toda resposta autenticada ou que leia sessão é privada e não deve ser armazenada em cache compartilhado.

## Referências oficiais

- Auth0 Next.js SDK: https://auth0.github.io/nextjs-auth0/
- Auth0 SDK security notes: https://github.com/auth0/nextjs-auth0/blob/main/SECURITY.md
- Next.js security: https://nextjs.org/docs/app/guides/authentication
- Next.js data security: https://nextjs.org/docs/app/guides/data-security
- PostgreSQL Row Security: https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- PostgreSQL SET / set_config: https://www.postgresql.org/docs/current/functions-admin.html
- Neon serverless driver: https://neon.com/docs/serverless/serverless-driver
- GitHub Actions secure use: https://docs.github.com/en/actions/reference/security/secure-use
- OWASP CSRF Prevention: https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html

**Data de consulta das referências web:** 31/10/2026. Revalidar antes de implementar, pois SDKs, políticas e limites Free mudam.
