# 3. Neon, persistência e autorização

## Driver e conexão

~~~bash
npm install @neondatabase/serverless
~~~

~~~dotenv
DATABASE_URL=postgresql://usuario:senha@host/banco?sslmode=require
~~~

~~~ts
import "server-only";
import { neon } from "@neondatabase/serverless";

const url = process.env.DATABASE_URL;
if (!url) throw new Error("DATABASE_URL não configurada");

export const sql = neon(url);
~~~

Manter DATABASE_URL exclusivamente no servidor. Ajustar pooling, timeouts e transações conforme documentação do driver e do Neon.

## Modelo de domínio inicial

~~~sql
CREATE SCHEMA IF NOT EXISTS app;\n\nCREATE TABLE app.app_user (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  auth_issuer text NOT NULL,
  auth_subject text NOT NULL,
  email text,
  created_at timestamptz NOT NULL DEFAULT now(),
  disabled_at timestamptz,
  UNIQUE (auth_issuer, auth_subject)
);

CREATE TABLE app.tenant (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE app.tenant_membership (
  tenant_id uuid NOT NULL REFERENCES app.tenant(id),
  user_id uuid NOT NULL REFERENCES app.app_user(id),
  role text NOT NULL CHECK (role IN ('owner','admin','member','viewer')),
  active boolean NOT NULL DEFAULT true,
  PRIMARY KEY (tenant_id, user_id)
);

CREATE TABLE app.project (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenant(id),
  name text NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now()
);
CREATE INDEX project_tenant_id_idx ON app.project(tenant_id);
~~~

Adaptar constraints, retenção, índices e exclusão ao domínio real.

## Papéis PostgreSQL

~~~sql
CREATE ROLE app_owner NOLOGIN;
CREATE ROLE app_migrator LOGIN NOINHERIT;
GRANT app_owner TO app_migrator;
CREATE ROLE app_runtime LOGIN NOINHERIT NOBYPASSRLS;
~~~

O migrator assume explicitamente o owner durante migrations aprovadas. Runtime não é owner e não recebe DDL, TRUNCATE, privilégios administrativos ou BYPASSRLS. Conceder apenas permissões necessárias. Segredos fora do repositório.

## Row-Level Security

O exemplo de schema anterior usa `app_user`, `tenant_membership` e `project`. Para manter consistência com o padrão de segurança já documentado, qualificar objetos no schema `app`, conceder privilégios mínimos e aplicar RLS também à tabela de membership.

```sql
ALTER TABLE app.app_user ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.app_user FORCE ROW LEVEL SECURITY;
ALTER TABLE app.tenant_membership ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.tenant_membership FORCE ROW LEVEL SECURITY;
ALTER TABLE app.project ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.project FORCE ROW LEVEL SECURITY;

GRANT SELECT (auth_issuer, auth_subject, disabled_at)
  ON app.app_user TO app_runtime;
GRANT SELECT ON app.tenant_membership TO app_runtime;
GRANT SELECT, INSERT, UPDATE, DELETE ON app.project TO app_runtime;

CREATE POLICY app_user_self_read ON app.app_user
FOR SELECT TO app_runtime
USING (
  auth_issuer = current_setting('app.auth_issuer', true)
  AND auth_subject = current_setting('app.user_sub', true)
  AND disabled_at IS NULL
);

CREATE POLICY membership_self_read ON app.tenant_membership
FOR SELECT TO app_runtime
USING (
  EXISTS (
    SELECT 1 FROM app.app_user u
    WHERE u.auth_issuer = current_setting('app.auth_issuer', true)
      AND u.auth_subject = current_setting('app.user_sub', true)
      AND u.disabled_at IS NULL
      AND u.id = tenant_membership.user_id
  )
);

CREATE POLICY project_tenant_select ON app.project
FOR SELECT TO app_runtime
USING (
  tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid
  AND EXISTS (
    SELECT 1
    FROM app.tenant_membership m
    JOIN app.app_user u ON u.id = m.user_id
    WHERE m.tenant_id = project.tenant_id
      AND m.active = true
      AND u.auth_issuer = current_setting('app.auth_issuer', true)
      AND u.auth_subject = current_setting('app.user_sub', true)
      AND u.disabled_at IS NULL
  )
);

CREATE POLICY project_tenant_insert ON app.project
FOR INSERT TO app_runtime
WITH CHECK (
  tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid
  AND EXISTS (
    SELECT 1
    FROM app.tenant_membership m
    JOIN app.app_user u ON u.id = m.user_id
    WHERE m.tenant_id = project.tenant_id
      AND m.active = true
      AND u.auth_issuer = current_setting('app.auth_issuer', true)
      AND u.auth_subject = current_setting('app.user_sub', true)
      AND u.disabled_at IS NULL
  )
);

CREATE POLICY project_tenant_update ON app.project
FOR UPDATE TO app_runtime
USING (
  tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid
  AND EXISTS (
    SELECT 1 FROM app.tenant_membership m
    JOIN app.app_user u ON u.id = m.user_id
    WHERE m.tenant_id = project.tenant_id AND m.active = true
      AND u.auth_issuer = current_setting('app.auth_issuer', true)
      AND u.auth_subject = current_setting('app.user_sub', true)
      AND u.disabled_at IS NULL
  )
)
WITH CHECK (
  tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid
  AND EXISTS (
    SELECT 1 FROM app.tenant_membership m
    JOIN app.app_user u ON u.id = m.user_id
    WHERE m.tenant_id = project.tenant_id AND m.active = true
      AND u.auth_issuer = current_setting('app.auth_issuer', true)
      AND u.auth_subject = current_setting('app.user_sub', true)
      AND u.disabled_at IS NULL
  )
);

CREATE POLICY project_tenant_delete ON app.project
FOR DELETE TO app_runtime
USING (
  tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid
  AND EXISTS (
    SELECT 1 FROM app.tenant_membership m
    JOIN app.app_user u ON u.id = m.user_id
    WHERE m.tenant_id = project.tenant_id AND m.active = true
      AND u.auth_issuer = current_setting('app.auth_issuer', true)
      AND u.auth_subject = current_setting('app.user_sub', true)
      AND u.disabled_at IS NULL
  )
);
```

O SQL é ilustrativo e pressupõe que a migration tenha criado objetos no schema `app`. Policies de membership consultam o registro de usuário e devem ser avaliadas para evitar recursão; mantenha os grants apenas nas colunas necessárias e nunca conceda ownership ou BYPASSRLS ao runtime. Para políticas mais complexas, considerar funções cuidadosamente revisadas, sem `SECURITY DEFINER` insegura.

GUCs customizadas são contexto confiável somente na medida em que o backend valida a sessão e define os valores; não são fronteira criptográfica contra comprometimento da própria credencial runtime. A autorização da aplicação deve verificar permissão de domínio além da membership.

## Contexto transacional

Não enviar `BEGIN`/ `COMMIT` em chamadas HTTP independentes do helper `neon()`. Para lote de consultas conhecido antecipadamente, usar `sql.transaction([...])`; todas as instruções do lote executam na mesma transação HTTP e o resultado preserva a ordem das consultas.

```ts
import { auth0 } from "@/lib/auth0";

export async function listProjects(requestedTenantId: string) {
  const session = await auth0.getSession();
  if (!session?.user?.sub) throw new Error("UNAUTHENTICATED");

  const issuer = process.env.AUTH0_ISSUER; // issuer OIDC configurado e validado pelo servidor
  if (!issuer) throw new Error("AUTH0_ISSUER ausente");

  const [, rows] = await sql.transaction([
    sql`SELECT
      set_config('app.tenant_id', ${requestedTenantId}, true),
      set_config('app.auth_issuer', ${issuer}, true),
      set_config('app.user_sub', ${session.user.sub}, true)`,
    sql`SELECT id, name FROM app.project ORDER BY created_at DESC`,
  ]);

  return rows;
}
```

`requestedTenantId` é apenas um seletor de tenant; RLS confirma membership ativa para issuer + subject da sessão. A permissão de domínio (por exemplo, `project:read`) também deve ser verificada na camada de aplicação. Para operações que dependem de resultados intermediários/decisões condicionais, usar conexão interativa `Pool`/`Client` via WebSocket e garantir BEGIN/COMMIT/ROLLBACK e liberação em `finally`.

## Query parametrizada e upsert

~~~ts
const rows = await sql`
  SELECT id, name FROM project
  WHERE tenant_id = ${tenantId}
  ORDER BY created_at DESC
`;
~~~

~~~sql
INSERT INTO app_user (auth_issuer, auth_subject, email)
VALUES ($1, $2, $3)
ON CONFLICT (auth_issuer, auth_subject)
DO UPDATE SET email = EXCLUDED.email
RETURNING id;
~~~

O issuer vem de configuração confiável e subject da sessão validada. Primeiro login não concede papel ou membership privilegiada.

## Migrations e recuperação
Versionar migrations, revisar alterações destrutivas e executá-las com identidade de migração, nunca runtime. Não executar migration automaticamente em cada inicialização serverless. Planejar backup externo e restauração testada; PITR não substitui backup externo. Conferir limites e retenção Free no painel Neon na data de uso.
