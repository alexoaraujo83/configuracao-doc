# Revisão técnica aprofundada — Next.js 16, Auth0 SDK v4 e Neon PostgreSQL

**Estado:** revisão documental concluída em 2026-10-31.  
**Escopo:** documentação e exemplos de referência; nenhuma implementação foi executada ou validada operacionalmente.  
**Planos:** Auth0 Free, Neon Free, GitHub Free e Vercel Hobby, sem trials, créditos, add-ons ou recursos pagos.  
**Stack:** Next.js 16 App Router, TypeScript, Node.js, `@auth0/nextjs-auth0` v4 e `@neondatabase/serverless`.

## 1. Resultado executivo

A revisão identificou pontos que precisavam de precisão adicional nos exemplos e os corrigiu nos manuais de implementação e segurança:

1. Diferenciar a convenção `proxy.ts` do Next.js 16 da convenção legada `middleware.ts`.
2. Reforçar que o Proxy do Auth0 não protege rotas automaticamente e não substitui autorização em Server Actions, Route Handlers e DAL.
3. Derivar o subject exclusivamente de `auth0.getSession()`; tratar o tenant recebido da rota como seletor não confiável até a validação de membership.
4. Corrigir escapes literais que quebravam os exemplos de template SQL e de formatação Markdown.
5. Explicitar a semântica da transação HTTP do driver Neon: lote não interativo, múltiplas consultas na mesma transação e resultados na ordem de entrada.
6. Separar transações HTTP em lote de operações que exigem sessão PostgreSQL persistente/interativa.
7. Explicitar que `SET ROLE` depende da mesma sessão e que `SET LOCAL ROLE` deve estar na transação que contém a migration.
8. Reforçar a policy de RLS com membership ativa no tenant, além de `tenant_id` e `user_sub` no contexto transacional.
9. Distinguir privilégios SQL (`GRANT`) de filtragem de linhas (RLS), e documentar o bypass de owner, superuser e `BYPASSRLS`.

As alterações são documentais. Os exemplos devem ser adaptados ao schema, às versões efetivamente escolhidas e às políticas do projeto antes de qualquer implementação.

## 2. Compatibilidade Next.js 16

### 2.1 Proxy

No Next.js 16, a convenção de interceptação de requisições é `proxy.ts`, exportando uma função `proxy`. A documentação do framework informa que Middleware foi renomeado para Proxy; o runtime de Proxy é Node.js e não deve ser confundido com Edge Middleware de versões/padrões anteriores.

Padrão documentado:

```typescript
import { auth0 } from "@/lib/auth0";

export async function proxy(request: Request) {
  return auth0.middleware(request);
}

export const config = {
  matcher: [
    "/((?!_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)",
  ],
};
```

A SDK v4 aceita `Request` e monta endpoints como `/auth/login`, `/auth/logout`, `/auth/callback` e `/auth/profile`. O matcher amplo permite ao SDK gerir o ciclo de sessão e os endpoints. Isso **não significa** que as rotas de negócio estejam protegidas.

### 2.2 Server Actions e Route Handlers

Cada Server Action deve ser tratada como uma superfície de entrada invocável por requisição direta. Cada Route Handler também deve verificar a sessão e autorização. Não confiar em:
- botão oculto ou UI condicional;
- layout protegido;
- redirecionamento no Proxy;
- `tenant_id`, `user_id`, `sub`, `role` ou `permissions` enviados pelo browser.

Fluxo mínimo em cada operação sensível:
1. Obter sessão com `await auth0.getSession()`.
2. Rejeitar sessão ausente.
3. Validar schema/entrada.
4. Obter o subject da sessão validada.
5. Consultar membership ativa e permissão no servidor.
6. Restringir a query ao tenant e ao recurso autorizados.
7. Revalidar permissões na mesma transação/statement para operações críticas, reduzindo TOCTOU.

### 2.3 Node.js e dependências

A SDK Auth0 v4 documenta Node.js 20 LTS ou superior. A versão do Node escolhida deve satisfazer simultaneamente Next.js, SDK Auth0, driver Neon e runtime de hospedagem. Fixar versão LTS e lockfile; não tratar `latest` como pin reproduzível.

## 3. Auth0 SDK v4

### 3.1 Cliente e variáveis

Padrão simples suportado pela documentação atual:

```typescript
import "server-only";
import { Auth0Client } from "@auth0/nextjs-auth0/server";

export const auth0 = new Auth0Client();
```

Variáveis de referência do SDK v4:
- `AUTH0_DOMAIN`
- `AUTH0_CLIENT_ID`
- `AUTH0_CLIENT_SECRET`
- `AUTH0_SECRET`
- `APP_BASE_URL` (quando necessária para URL/base dinâmica)

A configuração explícita via opções do construtor também é possível, mas não misturar nomes de variáveis v3 (`AUTH0_ISSUER_BASE_URL`, `AUTH0_BASE_URL`) com o padrão v4 sem uma decisão documentada. Segredos permanecem exclusivamente no servidor.

### 3.2 Sessão e tokens

- `auth0.getSession()` é a API server-side para obter a sessão no App Router e pode retornar `null`.
- ID Token representa a autenticação OIDC do usuário perante o cliente; não é um Access Token de API.
- Access Token deve ser emitido para a audience do recurso e validado pelo serviço destinatário.
- Não registrar tokens, cookies ou objetos completos de sessão.
- A SDK v4 habilita rolling sessions no middleware/Proxy por padrão; não alterar o matcher de modo a excluir endpoints `/auth/*` ou quebrar a manutenção da sessão.
- O endpoint `/auth/access-token` é habilitado por padrão na documentação atual; desabilitá-lo (`enableAccessTokenEndpoint: false`) se o browser não precisar obter tokens diretamente. Preferir BFF quando o token puder permanecer no servidor.

### 3.3 Autorização no plano Free

Não presumir que Auth0 Free inclua Role Management ou todos os recursos de RBAC gerenciado. A autorização de domínio desta arquitetura deve residir na aplicação, usando tabelas de membership/roles no Neon e regras server-side. Claims de token podem expressar scopes/permissions quando configuradas, mas não substituem membership, tenant, status e ownership atuais no banco.

## 4. Neon Serverless Driver — consultas e transações

### 4.1 HTTP query function

`neon(DATABASE_URL)` fornece uma função SQL tag para consultas parametrizadas por HTTP. Uma chamada simples executa uma consulta. Chamadas independentes não compartilham automaticamente estado de sessão.

```typescript
import "server-only";
import { neon } from "@neondatabase/serverless";

const sql = neon(process.env.DATABASE_URL!);

const rows = await sql`SELECT id FROM app.projects WHERE id = ${projectId}`;
```

Valores interpolados pelo template tag são parâmetros SQL. Para identificadores dinâmicos, usar allowlist e composição segura documentada pelo driver; não interpolar texto arbitrário como nome de tabela/coluna.

### 4.2 Transação HTTP em lote

A API documentada `sql.transaction([...])` recebe uma lista de consultas conhecidas antecipadamente e executa o lote numa transação não interativa. O array retornado mantém a ordem das consultas submetidas.

```typescript
const [contextResult, projectRows] = await sql.transaction([
  sql`SELECT
    set_config('app.tenant_id', ${tenantId}, true),
    set_config('app.user_sub', ${userSub}, true)`,
  sql`SELECT id, name
      FROM app.projects
      ORDER BY created_at DESC`,
]);
```

O terceiro argumento `true` de `set_config` faz o valor ser local à transação. Assim, o contexto e a query protegida precisam pertencer ao mesmo `sql.transaction([...])`. Não dividir a configuração de contexto e a consulta em chamadas HTTP independentes.

Essa API é **não interativa**: o conjunto de consultas é definido antes da execução. Não usar quando o fluxo depende de ler um resultado e decidir qual SQL enviar em seguida dentro da mesma transação.

### 4.3 Sessões e transações interativas

Quando a aplicação ou ferramenta de migration precisa de estado de sessão, cursores ou decisões condicionais entre statements, usar `Pool`/`Client` compatível via WebSocket ou uma conexão PostgreSQL persistente, conforme a documentação do driver e do runtime. Implementar `BEGIN`, `COMMIT`, `ROLLBACK`, liberação de client e encerramento de pool em `finally`. Não criar pools globais sem avaliar o ciclo de vida do runtime serverless.

## 5. Roles, grants e migrations

### 5.1 Separação de papéis

| Role | LOGIN | Uso |
|---|---:|---|
| `app_owner` | Não | Dono de schema/objetos; DDL sob procedimento controlado |
| `app_migrator` | Sim | Credencial dedicada de migration; pode assumir `app_owner` |
| `app_runtime` | Sim | Consultas da aplicação; sem ownership, DDL ou BYPASSRLS |

Regras:
- Runtime nunca usa a role administrativa padrão do projeto Neon.
- Runtime não é owner, superuser ou `BYPASSRLS`.
- Runtime não recebe membership de `app_owner`.
- Migrator não atende requests web.
- Grants de tabelas/sequences devem ser explícitos e mínimos.
- `ALTER DEFAULT PRIVILEGES FOR ROLE app_owner` só afeta objetos futuros criados por `app_owner` no escopo definido.

### 5.2 SET ROLE

`SET ROLE` muda o papel efetivo da sessão e depende de membership com permissão `SET`. `NOINHERIT` impede herança automática, mas não significa, por si só, que a troca explícita esteja proibida.

A troca de role deve ocorrer na mesma conexão/transação que executa DDL. Padrão documental:

```sql
BEGIN;
SET LOCAL ROLE app_owner;
-- DDL revisado da migration
COMMIT;
```

Se ocorrer erro, executar rollback na mesma conexão. Não enviar `SET ROLE` numa chamada HTTP isolada e esperar que o estado permaneça na chamada seguinte. A criação de roles e membership é uma tarefa administrativa separada.

## 6. RLS e isolamento multi-tenant

### 6.1 Princípios

- `GRANT` determina se a role pode executar a operação; policy RLS limita as linhas.
- Habilitar RLS em cada tabela com dados tenant-scoped.
- Usar `FORCE ROW LEVEL SECURITY` quando o owner também deve ser sujeito às policies.
- Superuser e roles com `BYPASSRLS` sempre ignoram RLS; owner normalmente ignora, salvo FORCE.
- Definir policies específicas para SELECT/INSERT/UPDATE/DELETE conforme a necessidade.
- UPDATE exige validação da linha existente (`USING`) e do novo estado (`WITH CHECK`).
- Não conceder `TRUNCATE` ou `REFERENCES` ao runtime como substituto de policies; essas operações não são filtradas por RLS.

### 6.2 Contexto transacional e limite de confiança

O backend valida a sessão Auth0 e obtém `sub` do SDK. O tenant informado na URL/formulário é somente um seletor. A aplicação deve validar membership ativa no banco e a policy RLS deve reforçar que o subject possui membership ativa no tenant no statement protegido.

GUCs customizadas como `app.tenant_id` e `app.user_sub` são úteis para evitar filtros esquecidos, mas não constituem uma fronteira criptográfica contra um backend/credencial comprometidos: quem controla a mesma credencial pode tentar definir outros valores. A segurança depende também da validação server-side, grants mínimos, policies e redução da superfície SQL.

### 6.3 Padrão de policy

Exemplo conceitual de predicate:
```sql
tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid
AND EXISTS (
  SELECT 1
  FROM app.tenant_memberships m
  WHERE m.tenant_id = projects.tenant_id
    AND m.auth0_sub = current_setting('app.user_sub', true)
    AND m.status = 'active'
)
```

A tabela de membership também requer grants mínimos e uma policy que permita ao runtime consultar somente a membership do subject atual. Revisar índices para `(tenant_id, auth0_sub)`. Evitar policies recursivas entre tabelas; verificar views, funções `SECURITY DEFINER`, triggers e rotinas administrativas.

### 6.4 Transação com contexto

```typescript
const session = await auth0.getSession();
if (!session) throw new Error("UNAUTHENTICATED");

const [contextResult, projects] = await sql.transaction([
  sql`SELECT
    set_config('app.tenant_id', ${requestedTenantId}, true),
    set_config('app.user_sub', ${session.user.sub}, true)`,
  sql`SELECT id, name, created_at
      FROM app.projects
      ORDER BY created_at DESC`,
]);
```

O exemplo só é seguro quando a policy também confirma membership ativa e o runtime possui apenas os grants necessários. A camada de aplicação ainda deve validar a permissão de domínio (por exemplo, leitura/edição) em cada operação.

## 7. Compatibilidade e pontos que exigem validação no projeto

| Área | Regra documental | O que precisa ser confirmado antes da implementação |
|---|---|---|
| Next.js | Proxy convention do Next.js 16 | Versão minor/patch e runtime escolhido |
| Node.js | Node LTS compatível com framework e SDK | Versão selecionada no ambiente local e Vercel |
| Auth0 SDK | API v4, `Auth0Client`, `getSession`, `middleware` | Versão exata do pacote no lockfile e opções configuradas |
| Neon driver | `neon()`, `sql.transaction([...])`, `Pool`/`Client` | Versão do driver, runtime Node e modalidade HTTP/WebSocket |
| PostgreSQL | Policies, roles, grants, SET LOCAL ROLE | Major version do projeto Neon e privilégios efetivos |
| Migrations | Executor persistente ou lote transacional | Ferramenta única, concorrência, ledger e tratamento de rollback |
| RLS | Contexto transacional + membership ativa | Policies para todas as tabelas/views/funções e testes com app_runtime |

Não declarar compatibilidade operacional até que o projeto escolha versões fixas e execute validações isoladas autorizadas. Esta revisão não executou tais validações.

## 8. Checklist de revisão documental

- [x] Next.js 16 documentado com `proxy.ts` e export `proxy`.
- [x] SDK Auth0 v4 documentada com `Auth0Client`, `getSession()` e rotas `/auth/*`.
- [x] Autorização descrita como responsabilidade server-side em cada action/handler/DAL.
- [x] Driver Neon HTTP e transação em lote diferenciados de conexão interativa.
- [x] Contexto GUC definido com `set_config(..., true)` na mesma transação da query.
- [x] SET ROLE associado à mesma sessão/transação de migration.
- [x] Runtime separado de owner/migrator e sem BYPASSRLS.
- [x] Policies incluem membership ativa, além do tenant e subject.
- [x] Escapes literais encontrados nos exemplos foram corrigidos.
- [ ] Typecheck/build/testes de integração/RLS executados — **não executados por escopo**.
- [ ] Compatibilidade operacional confirmada no tenant/branch Neon — **não verificada**.

## 9. Fontes oficiais e referências

- Next.js 16 Proxy: https://nextjs.org/docs/app/getting-started/proxy
- Next.js Authentication guide: https://nextjs.org/docs/app/guides/authentication
- Auth0 Next.js SDK v4 API e Getting Started: https://auth0.github.io/nextjs-auth0/
- Auth0 v4 migration guide: https://github.com/auth0/nextjs-auth0/blob/main/V4_MIGRATION_GUIDE.md
- Neon Serverless Driver — transactions and sessions: https://github.com/neondatabase/serverless#sessions-transactions-and-node-postgres-compatibility
- Neon Serverless Driver — transaction options: https://github.com/neondatabase/serverless/blob/main/CONFIG.md
- PostgreSQL Row Security: https://www.postgresql.org/docs/17/ddl-rowsecurity.html
- PostgreSQL SET ROLE: https://www.postgresql.org/docs/18/sql-set-role.html
- PostgreSQL GRANT: https://www.postgresql.org/docs/current/sql-grant.html

**Aviso:** esta página é registro de revisão documental, não atestado de segurança, validação de código executável, certificação, execução de migration ou confirmação de comportamento em produção.
