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
CREATE TABLE app_user (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  auth_issuer text NOT NULL,
  auth_subject text NOT NULL,
  email text,
  created_at timestamptz NOT NULL DEFAULT now(),
  disabled_at timestamptz,
  UNIQUE (auth_issuer, auth_subject)
);

CREATE TABLE tenant (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE tenant_membership (
  tenant_id uuid NOT NULL REFERENCES tenant(id),
  user_id uuid NOT NULL REFERENCES app_user(id),
  role text NOT NULL CHECK (role IN ('owner','admin','member','viewer')),
  active boolean NOT NULL DEFAULT true,
  PRIMARY KEY (tenant_id, user_id)
);

CREATE TABLE project (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenant(id),
  name text NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now()
);
CREATE INDEX project_tenant_id_idx ON project(tenant_id);
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

~~~sql
ALTER TABLE project ENABLE ROW LEVEL SECURITY;
ALTER TABLE project FORCE ROW LEVEL SECURITY;

CREATE POLICY project_tenant_isolation
ON project
USING (
 tenant_id = nullif(current_setting('app.tenant_id', true), '')::uuid
)
WITH CHECK (
 tenant_id = nullif(current_setting('app.tenant_id', true), '')::uuid
);
~~~

Adicionar validação de membership ativa e políticas apropriadas às demais tabelas. O contexto definido pelo backend é defesa em profundidade, não fronteira independente contra backend comprometido.

## Contexto transacional

~~~sql
BEGIN;
SELECT set_config('app.tenant_id', '00000000-0000-0000-0000-000000000001', true);
SELECT id, name FROM project WHERE id = $1;
COMMIT;
~~~

É essencial que contexto e consultas protegidas ocorram na mesma transação/conexão. Usar API transacional suportada pelo driver; não definir contexto numa chamada separada sob pooling. Não concatenar entrada em SQL.

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
