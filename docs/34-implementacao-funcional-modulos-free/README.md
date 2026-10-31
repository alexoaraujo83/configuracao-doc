# Implementação funcional por módulos — Next.js + Auth0 + Neon (Free)

Status: documentação recomendada. Exemplos não compilados/executados; nenhum pacote instalado, serviço provisionado, migration, teste ou deploy realizado.

Stack: Next.js App Router + TypeScript, Auth0 SDK v4 e Neon PostgreSQL. Somente Free. Não depender de Auth0 Organizations/add-ons pagos nem usar Vercel Hobby em aplicações comerciais (Hobby é pessoal/não comercial). Revalidar quotas e elegibilidade.

~~~text
Browser -> Auth0 Universal Login -> Next.js BFF
                                  |-- sessão / sub
                                  |-- membership / autorização
                                  |-- validação / services
                                  |-- repositories SQL
                                  v
                              Neon PostgreSQL
                              app_user / tenant
                              tenant_member / project
                              grants + RLS
~~~

Princípios: Auth0 autentica; a aplicação autoriza; sub é identidade externa estável; tenant é validado no servidor; RLS é defesa em profundidade; role runtime não é owner/BYPASSRLS; secrets e tokens nunca chegam ao browser.

Estrutura sugerida: src/lib/auth0.ts, src/lib/db/pool.ts, src/server/identity, authorization, users, projects, http; Route Handlers em src/app/api. Next.js 15 usa middleware.ts; Next.js 16 adota proxy.ts.

Variáveis: AUTH0_DOMAIN, AUTH0_CLIENT_ID, AUTH0_CLIENT_SECRET, AUTH0_SECRET, APP_BASE_URL e DATABASE_URL. Valores não versionados; sem prefixo NEXT_PUBLIC_ para secrets; separar ambientes e registrar callback/logout URLs exatas.

## Módulos
1. [Usuários](01-usuarios.md)
2. [Login, logout e sessões](02-autenticacao-sessoes.md)
3. [Perfis, permissões e autorização](03-perfis-permissoes.md)
4. [CRUD Neon](04-crud-neon.md)
5. [Multi-tenancy e RLS](05-multitenancy-rls.md)
6. [Validação e erros](06-validacao-erros.md)
7. [Testes](07-testes.md)

Fluxo: login → callback → sessão → sub → upsert idempotente do perfil → seleção e validação do tenant. Operação: sessão → membership/permissão → contexto transacional → SQL parametrizado → RLS → resposta.

Antes de implementar, definir convites, suspensão, exclusão, retenção, permissões e quantidade de tenants por usuário. Confirmar versões exatas do Next.js, SDK e driver. Toda execução real exige revisão e ambiente autorizado.
