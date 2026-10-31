# 05 — Isolamento multi-tenant e Row-Level Security

Tenant da URL é seletor solicitado, não prova de acesso. Backend deriva identidade da sessão e valida membership/permissão. RLS é defesa em profundidade, não substitui autorização, grants ou segurança do backend. Um backend/credencial comprometido que define contexto arbitrário pode atravessar policies baseadas em GUC.

Roles: app_owner NOLOGIN (owner das tabelas/policies); app_migrator LOGIN NOINHERIT (assume app_owner explicitamente); app_runtime LOGIN NOINHERIT NOBYPASSRLS, não owner, sem DDL/TRUNCATE/BYPASSRLS.

~~~sql
CREATE ROLE app_owner NOLOGIN;
CREATE ROLE app_migrator LOGIN NOINHERIT;
GRANT app_owner TO app_migrator;
CREATE ROLE app_runtime LOGIN NOINHERIT NOBYPASSRLS;
~~~

~~~sql
ALTER TABLE project ENABLE ROW LEVEL SECURITY;
ALTER TABLE project FORCE ROW LEVEL SECURITY;
CREATE POLICY project_tenant_isolation
ON project FOR ALL TO app_runtime
USING (
  tenant_id = nullif(current_setting('app.tenant_id', true), '')::uuid
)
WITH CHECK (
  tenant_id = nullif(current_setting('app.tenant_id', true), '')::uuid
);
~~~

Sem contexto, a expressão não é verdadeira e o acesso é negado. Criar policies para todas as tabelas tenant-scoped. Revisar views, funções, triggers, FKs e tabelas associativas. RLS não governa TRUNCATE; superuser/BYPASSRLS ignora; owner normalmente ignora salvo FORCE.

Contexto deve ser definido e consumido na mesma transação/conexão. set_config(..., true) limita o valor à transação:

~~~ts
export async function withTenantContext(context, work) {
  const client = await pool.connect();
  try {
    await client.query("BEGIN");
    await client.query("SELECT set_config('app.tenant_id', $1, true)", [context.tenantId]);
    await client.query("SELECT set_config('app.user_sub', $1, true)", [context.userSub]);
    const result = await work(client);
    await client.query("COMMIT");
    return result;
  } catch (error) {
    await client.query("ROLLBACK");
    throw error;
  } finally {
    client.release();
  }
}
~~~

Confirmar Pool/Client API e suporte transacional na versão do driver. Não separar contexto e query em chamadas HTTP independentes. Backend deriva tenant/user; nunca aceitar GUC do browser.

Testes obrigatórios: A não lê/altera/exclui B; INSERT/UPDATE tenant divergente é rejeitado; ausência de contexto nega acesso; contexto não vaza após commit/rollback e reuso da conexão; runtime não é owner/superuser/BYPASSRLS; endpoints não usam owner/migrator; nova tabela recebe RLS antes do uso. Testar com app_runtime, não apenas administrador.
