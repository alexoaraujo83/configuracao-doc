# 04 — CRUD com Neon PostgreSQL

Camadas: Route Handler (HTTP/sessão), schema (entrada), service (autorização/regras), repository (SQL parametrizado), PostgreSQL (constraints/grants/RLS). Não colocar SQL em componentes React. tenant_id, created_by, role e status administrativo são derivados no servidor.

~~~ts
import "server-only";
import { Pool } from "@neondatabase/serverless";
export const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});
~~~

Exemplo para runtime Node.js. Revisar ciclo de vida do pool, limite de conexões e runtime; confirmar suporte a transações interativas e API exata da versão do driver.

~~~sql
CREATE TABLE project (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenant(id),
  name text NOT NULL CHECK (length(trim(name)) BETWEEN 1 AND 120),
  description text,
  created_by uuid NOT NULL REFERENCES app_user(id),
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);
CREATE INDEX project_tenant_created_idx
  ON project (tenant_id, created_at DESC);
~~~

~~~ts
export async function listProjects(client, tenantId: string) {
  const result = await client.query(
    "SELECT id, name, description, created_at FROM project WHERE tenant_id = $1 ORDER BY created_at DESC LIMIT 100",
    [tenantId],
  );
  return result.rows;
}
~~~

O exemplo pressupõe client.query compatível. Nunca concatenar valores SQL; identificadores dinâmicos exigem allowlist.

CRUD: Create deriva tenant/usuário, valida e insere; Read filtra tenant e pagina; Update usa allowlist e verifica linhas afetadas; Delete define exclusão lógica/física e valida dependências. Retornar 404/resposta neutra para não revelar recursos de outro tenant.

~~~ts
export async function createProject(tenantId: string, input: unknown) {
  const actor = await requireTenantPermission(tenantId, "projects:create");
  const parsed = createProjectSchema.parse(input);
  return withTenantContext(
    { tenantId, userSub: actor.identity.sub },
    (client) => projectRepository.create(client, {
      tenantId, createdBy: actor.user.id,
      name: parsed.name, description: parsed.description ?? null,
    }),
  );
}
~~~

Preferir cursor pagination; limitar payload; indexar filtros; evitar SELECT *; retry apenas para falhas transitórias. Testar CRUD por tenant, payload inválido, ID inexistente, concorrência, constraints, SQL parametrizado e paginação.
