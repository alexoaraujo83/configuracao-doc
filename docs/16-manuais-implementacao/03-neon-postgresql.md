# 03 — Neon PostgreSQL e driver serverless

**Estado:** Documentado; projeto, roles e migrations não executados.  
**Plano:** Neon Free.  
**Driver:** `@neondatabase/serverless`.

## 1. Pré-requisitos e limites

- Conta e projeto Neon Free.
- PostgreSQL e database definidos.
- Node.js e npm.
- Variável `DATABASE_URL` obtida no painel Connect.
- Planejamento de consumo: compute-hours, armazenamento, transferência, branches e histórico de restauração.

Não assumir cotas permanentes: confirmar valores atuais no painel e na página oficial de planos. Não criar recursos que exijam upgrade. Neon Free pode suspender compute após inatividade; prever cold start e latência na primeira consulta.

## 2. Criar projeto e conexão

No Neon Console:
1. Criar projeto Free.
2. Escolher região próxima da aplicação, considerando latência entre Vercel e Neon.
3. Criar database da aplicação.
4. Criar branch de desenvolvimento se houver quota disponível.
5. Abrir Connect e selecionar branch, database e role.
6. Copiar a connection string para o ambiente seguro.

Conexão pooled é adequada ao padrão de consultas curtas de funções serverless. Usar conexão direta quando uma ferramenta/migração exigir semântica de sessão ou conexão persistente. Não colocar connection string em código, URL pública, argumento de CLI registrado ou log.

## 3. Criar roles com privilégio mínimo

Não usar a role administrativa padrão do projeto Neon como credencial de runtime. A role de runtime não deve herdar privilégios administrativos. Criar roles SQL separadas, adaptando nomes e política de senha ao procedimento seguro de provisionamento:

```sql
CREATE ROLE app_owner NOLOGIN;
CREATE ROLE app_migrator LOGIN PASSWORD '<SECRET_FOR_MIGRATOR>';
CREATE ROLE app_runtime LOGIN PASSWORD '<SECRET_FOR_RUNTIME>';

CREATE SCHEMA app AUTHORIZATION app_owner;

GRANT CONNECT ON DATABASE appdb TO app_migrator, app_runtime;
GRANT USAGE ON SCHEMA app TO app_runtime;
GRANT USAGE ON SCHEMA app TO app_migrator;
```

Substituir `appdb` pelo nome real do database. Senhas são placeholders e devem ser definidas fora do histórico do repositório. Não executar SQL literal contendo segredo em terminal compartilhado ou ferramenta que retenha histórico.

As migrations devem operar como `app_owner` por meio da role migrator autorizada:

```sql
SET ROLE app_owner;
-- DDL da migration
RESET ROLE;
```

Conceder ao runtime apenas permissões necessárias sobre tabelas e sequences específicas. Exemplo após criar tabelas:

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON ALL TABLES IN SCHEMA app TO app_runtime;

GRANT USAGE, SELECT
ON ALL SEQUENCES IN SCHEMA app TO app_runtime;

ALTER DEFAULT PRIVILEGES FOR ROLE app_owner IN SCHEMA app
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_runtime;

ALTER DEFAULT PRIVILEGES FOR ROLE app_owner IN SCHEMA app
GRANT USAGE, SELECT ON SEQUENCES TO app_runtime;
```

Revisar permissões de cada tabela: não conceder DELETE ou UPDATE quando não necessário. Confirmar owner e memberships por consulta de catálogo. Não conceder `neon_superuser` ou privilégios equivalentes ao runtime.

## 4. Schema e migration inicial

Criar `database/migrations/001_initial.sql`:

```sql
CREATE SCHEMA IF NOT EXISTS app;

CREATE TABLE app.app_users (
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  auth0_sub TEXT NOT NULL UNIQUE,
  email TEXT NOT NULL,
  display_name TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX app_users_email_idx ON app.app_users(email);
```

Executar migrations com a credencial de migrator, assumindo `app_owner` somente para DDL. Versionar cada migration como arquivo imutável e registrar checksum/versão aplicada em tabela de controle. Não editar migration já aplicada; criar migration corretiva.

## 5. Driver Neon

Instalar:

```bash
npm install @neondatabase/serverless
```

Criar `src/lib/db.ts`:

```typescript
import "server-only";
import { neon } from "@neondatabase/serverless";
import { env } from "@/lib/env";

export const sql = neon(env.DATABASE_URL);
```

Consulta parametrizada:

```typescript
import { sql } from "@/lib/db";

export async function findUser(auth0Sub: string) {
  const rows = await sql`
    SELECT id, auth0_sub, email, display_name
    FROM app.app_users
    WHERE auth0_sub = ${auth0Sub}
    LIMIT 1
  `;

  return rows[0] ?? null;
}
```

Valores interpolados no template tag são parâmetros SQL. Não construir SQL por concatenação de strings controladas pelo usuário. Para nomes dinâmicos de tabelas/colunas, usar allowlist fixa; parâmetros SQL não substituem identificadores.

## 6. RLS e isolamento por usuário

Ativar Row-Level Security em tabelas que contenham dados por usuário:

```sql
ALTER TABLE app.app_users ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.app_users FORCE ROW LEVEL SECURITY;

CREATE POLICY app_users_select_own
ON app.app_users
FOR SELECT
TO app_runtime
USING (
  auth0_sub = current_setting('app.auth0_sub', true)
);

CREATE POLICY app_users_update_own
ON app.app_users
FOR UPDATE
TO app_runtime
USING (
  auth0_sub = current_setting('app.auth0_sub', true)
)
WITH CHECK (
  auth0_sub = current_setting('app.auth0_sub', true)
);
```

Criar policies adicionais para INSERT/DELETE somente se os casos de uso exigirem. O valor de `app.auth0_sub` deve ser definido exclusivamente a partir da sessão Auth0 validada no servidor, nunca recebido como identidade confiável do cliente.

A variável de sessão deve ser local à transação. Executar a definição do contexto e as consultas protegidas na mesma transação. Para o driver HTTP Neon, utilizar transação em lote suportada pelo driver e confirmar a API exata na versão instalada; não presumir que chamadas HTTP independentes compartilham sessão. Quando o caso exigir estado interativo de sessão, usar a modalidade WebSocket/cliente PostgreSQL apropriado, validando compatibilidade com o runtime.

Exemplo conceitual de transação HTTP (validar assinatura da API na versão instalada):

```typescript
const rows = await sql.transaction([
  sql`select set_config('app.auth0_sub', ${auth0Sub}, true)`,
  sql`select id, email from app.app_users
      where auth0_sub = current_setting('app.auth0_sub', true)`,
]);
```

Não tratar o exemplo como substituto de teste de integração: verificar que ambas as instruções são executadas na mesma transação e que o parâmetro é transaction-local. RLS é defesa adicional; manter autorização explícita na camada de aplicação.

## 7. Health check seguro

`/api/health` deve responder somente estado mínimo. Não expor hostname, database, role, SQL, exceção ou connection string. Considerar restringir endpoint, aplicar rate limit e separar liveness de readiness.

```typescript
import { NextResponse } from "next/server";
import { sql } from "@/lib/db";

export const runtime = "nodejs";

export async function GET() {
  try {
    await sql`SELECT 1`;
    return NextResponse.json({ status: "ok" });
  } catch {
    return NextResponse.json(
      { status: "unavailable" },
      { status: 503 },
    );
  }
}
```

## 8. Backup e recuperação

- Exportar backup lógico com `pg_dump` em ambiente protegido.
- Não salvar dumps no Git.
- Criptografar e limitar acesso ao armazenamento do backup.
- Testar restauração periodicamente em branch/database de teste, respeitando cotas.
- Documentar RPO/RTO esperados e limites do Free.
- Não tratar histórico de branching ou snapshot manual como backup independente.
- Guardar migrations e scripts de reconstrução fora do banco.

Exemplo (cliente PostgreSQL instalado; conexão e destino seguros):

```bash
pg_dump "$DATABASE_URL" --no-owner --no-privileges -Fc -f backup.dump
```

## 9. Testes obrigatórios

| Teste | Resultado |
|---|---|
| Conexão | `SELECT 1` responde |
| Runtime role | Não consegue executar DDL nem administrar roles |
| Migrations | Aplicadas em ordem e registradas |
| SQL injection | Entrada tratada como parâmetro |
| RLS | Usuário A não lê/altera linhas do usuário B |
| Contexto ausente | Policy nega acesso |
| Contexto trocado | Não vaza dados entre transações |
| Falha de banco | API retorna erro genérico 503 |
| Restore | Backup restaurado e validado em ambiente isolado |

## 10. Referências

- [Neon — Connect](https://neon.com/docs/get-started/connect-neon)
- [Neon — Serverless Driver](https://neon.com/docs/serverless/serverless-driver)
- [Driver GitHub](https://github.com/neondatabase/serverless)
- [PostgreSQL — Row Security Policies](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [PostgreSQL — GRANT](https://www.postgresql.org/docs/current/sql-grant.html)

