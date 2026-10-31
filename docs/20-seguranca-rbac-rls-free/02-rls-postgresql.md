# RLS e segregação de tenants no PostgreSQL

## 1. Limite de confiança
RLS restringe linhas por query. É uma segunda barreira após autenticação/autorização da aplicação.

Quando o backend usa uma única credencial para vários usuários, PostgreSQL não autentica cada usuário final. O backend define contexto transacional (\`tenant_id\`, \`user_sub\`). Um comprometimento total do processo/credencial pode permitir alterar GUCs; esse padrão reduz falhas de filtro na aplicação, mas não é fronteira criptográfica independente do backend.

Se for exigida identidade final verificada pelo próprio banco, desenhar e validar mecanismo de JWT assinado/verificação no PostgreSQL separadamente. Não presumir extensões, Data API ou recursos pagos no Neon Free.

## 2. Policies

\`\`\`sql
ALTER TABLE app.projects ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.projects FORCE ROW LEVEL SECURITY;

CREATE POLICY projects_tenant_select
ON app.projects FOR SELECT TO app_runtime
USING (
  tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid
);

CREATE POLICY projects_tenant_insert
ON app.projects FOR INSERT TO app_runtime
WITH CHECK (
  tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid
  AND created_by_sub = current_setting('app.user_sub', true)
);

CREATE POLICY projects_tenant_update
ON app.projects FOR UPDATE TO app_runtime
USING (tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid)
WITH CHECK (tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid);

CREATE POLICY projects_tenant_delete
ON app.projects FOR DELETE TO app_runtime
USING (tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid);
\`\`\`

Sem contexto, \`current_setting(..., true)\` retorna NULL e a comparação não permite acesso. Validar casts e nomes no schema real.

## 3. Contexto na mesma transação

Padrão ilustrativo com \`@neondatabase/serverless\`:

\`\`\`ts
import { neon } from "@neondatabase/serverless";
const sql = neon(process.env.DATABASE_URL!);

export async function listTenantProjects(tenantId: string, auth0Sub: string) {
  const [, projects] = await sql.transaction([
    sql\`SELECT
      set_config('app.tenant_id', \${tenantId}, true),
      set_config('app.user_sub', \${auth0Sub}, true)\`,
    sql\`SELECT id, name, created_at
        FROM app.projects
        ORDER BY created_at DESC\`,
  ]);
  return projects;
}
\`\`\`

O terceiro argumento \`true\` torna o setting local à transação. Contexto e query precisam estar na mesma transação/conexão. Não executar SET em chamada independente e presumir persistência no pool. Validar a API e semântica da versão instalada do driver; operações interativas podem exigir modo de conexão compatível.

## 4. Tabelas relacionadas

\`\`\`sql
ALTER TABLE app.documents ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.documents FORCE ROW LEVEL SECURITY;

CREATE POLICY documents_tenant_access
ON app.documents FOR ALL TO app_runtime
USING (tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid)
WITH CHECK (tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid);
\`\`\`

Adicionar regras de ownership/role quando necessárias. Revisar views, funções SECURITY DEFINER, triggers, jobs e rotinas administrativas.

## 5. Roles PostgreSQL
- \`app_owner\`: NOLOGIN; dono do schema/tabelas; usado apenas em migrations.
- \`app_migrator\`: LOGIN NOINHERIT; pode executar SET ROLE app_owner no pipeline controlado; nunca atende requests.
- \`app_runtime\`: LOGIN NOINHERIT NOBYPASSRLS; SELECT/INSERT/UPDATE/DELETE estritamente necessários; não é owner; sem CREATE, TRUNCATE, REFERENCES, DDL ou grants administrativos.

Owner normalmente ignora RLS. FORCE aplica policies ao owner, mas superuser e BYPASSRLS continuam ignorando. Runtime nunca deve herdar essas capacidades.

## 6. Testes negativos
- Sem contexto: zero linhas e escrita rejeitada.
- Tenant A não lê/atualiza/exclui registros de B.
- INSERT de tenant B sob contexto A falha.
- UPDATE que move linha de A para B falha em WITH CHECK.
- Contexto não vaza entre transações.
- Runtime não altera RLS/policies, não cria roles, não assume owner e não executa TRUNCATE.
- Revisar canais laterais de FKs/constraints, views, funções e jobs.

## 7. Cuidados
- RLS não concede privilégios SQL: GRANT e policies precisam estar corretos.
- Sem policy aplicável, default deny.
- Policies permissive combinam por OR; restrictive por AND.
- TRUNCATE e REFERENCES não são filtrados por RLS; não conceder ao runtime.
- Não concatenar SQL; usar parâmetros.
- Nunca permitir que o browser defina diretamente o contexto RLS.

## Referências
- [PostgreSQL Row Security](https://www.postgresql.org/docs/17/ddl-rowsecurity.html)
- [CREATE POLICY](https://www.postgresql.org/docs/16/sql-createpolicy.html)
- [CREATE ROLE — BYPASSRLS](https://www.postgresql.org/docs/current/sql-createrole.html)
- [Neon Serverless Driver](https://neon.com/docs/serverless/serverless-driver)
- [Neon Data API e Postgres RLS](https://neon.com/blog/a-postgrest-compatible-data-api-now-on-neon)
