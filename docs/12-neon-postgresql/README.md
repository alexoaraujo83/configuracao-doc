# Neon PostgreSQL — acesso, roles, migrations e isolamento

> Revisão documental: 31/10/2026. Escopo: Neon Free e PostgreSQL. Não executar comandos desta página em produção sem revisão, backup e autorização.

## 1. Arquitetura de conexão

Next.js acessa Neon somente no servidor. O driver `@neondatabase/serverless` executa SQL por HTTP ou WebSockets e oferece tagged templates com parâmetros separados. Para consultas curtas em ambientes serverless, usar o driver HTTP e a connection string pooled do Neon.

```text
Browser ──HTTPS──► Next.js Server / Route Handler
                       │
                       ├── Auth0 session / authorization
                       │
                       └── @neondatabase/serverless
                                  │ TLS/HTTPS
                                  ▼
                             Neon PostgreSQL
```

Instalação documentada:

```bash
npm install @neondatabase/serverless
```

## 2. Plano Free e limites operacionais

A página oficial Neon Plans consultada em 31/10/2026 informa:

| Recurso | Free |
|---|---:|
| Projects | 100 por organização |
| Branches | 10 por projeto |
| Compute | 100 CU-hours por projeto/mês |
| Autoscaling | até 2 CU / 8 GB RAM |
| Scale to zero | após 5 minutos; não pode ser desativado |
| PostgreSQL storage | 1 GB por projeto; 20 GB por organização |
| Public network transfer | 5 GB por projeto/mês |
| Monitoring history | 1 dia |
| Instant restore | não incluído como recurso pago de restore; histórico Free de 6 horas e até 1 GB |
| Manual snapshots | 1 por projeto |
| Protected branches | não incluído no Free |

Confirmar quotas na página [Neon Plans](https://neon.com/docs/introduction/plans) e [Free Plan Limits](https://neon.com/faqs/free-plan-limits-and-quotas) antes de implantar. O plano Free não tem garantia de compute sempre ativo. O primeiro acesso após scale-to-zero pode ter latência de retomada.

## 3. Projeto, região e branches

- Região de um projeto é uma decisão estrutural; escolher próxima à aplicação e aos usuários, avaliando latência e residência dos dados.
- Branches são úteis para desenvolvimento e testes, mas contam para o limite de 10 por projeto.
- Definir owner, finalidade, branch pai e política de limpeza para cada branch.
- Não usar dados pessoais de produção em Preview/Development sem minimização e mascaramento.
- Branches não substituem backup externo.
- No Free, branch protection nativa não está incluída. Migrations revisadas, roles separadas e backups são controles compensatórios, não bloqueio equivalente contra reset/exclusão.

## 4. Connection strings: pooled e direct

No Neon Console → Project → Branch → Connect:

- **Pooled:** hostname contém `-pooler`; usar no runtime Next.js serverless.
- **Direct:** conexão direta ao endpoint; usar para operações administrativas/migrations que precisem de sessão PostgreSQL direta.
- Manter SSL habilitado.
- Não registrar URL completa em logs, tickets, screenshots ou documentação.
- Nunca enviar connection string ao browser.
- Não compartilhar a role administrativa/owner com runtime.

Variáveis:

```dotenv
DATABASE_URL=
DATABASE_URL_MIGRATION=
```

`DATABASE_URL` é a conexão pooled da role de runtime. `DATABASE_URL_MIGRATION` é a conexão direta de uma role com permissões de migration, mantida fora do ambiente público.

## 5. Cliente Neon

Arquivo `src/lib/db.ts`:

```typescript
import "server-only";
import { neon } from "@neondatabase/serverless";
import { env } from "@/lib/env";

export const sql = neon(env.DATABASE_URL);
```

Marcar o módulo como server-only ajuda a impedir import acidental em Client Components. Route Handlers que acessam banco devem validar sessão e autorização antes de consultar dados.

## 6. Consultas parametrizadas

```typescript
const rows = await sql`
  SELECT id, email
  FROM app.app_users
  WHERE auth0_sub = ${auth0Sub}
  LIMIT 1
`;
```

Os valores interpolados no tagged template são enviados como parâmetros; não concatenar input em SQL. Identificadores dinâmicos (nome de tabela/coluna, ORDER BY) não devem ser interpolados como valor: usar allowlist fixa e mapear para SQL estático.

## 7. Roles e privilégios

**Regra crítica do Neon:** roles criadas pelo Console, API ou CLI recebem membership de `neon_superuser`. Roles criadas por SQL recebem apenas privilégios básicos e grants explícitos. A role administrativa do Neon não deve ser usada pelo runtime da aplicação. Consultar [Manage roles](https://neon.com/docs/manage/roles) e [neon_superuser](https://neon.com/docs/manage/roles#the-neon-superuser-role).

Baseline de roles:

| Role | Login | Finalidade |
|---|---|---|
| `app_owner` | NOLOGIN | owner lógico de schema/tabelas |
| `app_migrator` | LOGIN | executa migrations aprovadas |
| `app_runtime` | LOGIN | consultas de aplicação com privilégio mínimo |
| role administrativa do projeto | conforme Neon | administração manual; nunca runtime |

Exemplo conceitual para executar por conexão administrativa controlada:

```sql
CREATE ROLE app_owner NOLOGIN;
CREATE ROLE app_migrator LOGIN;
CREATE ROLE app_runtime LOGIN;

GRANT app_owner TO app_migrator;
```

Definir senhas de `app_migrator` e `app_runtime` por canal seguro (por exemplo, comando interativo `\\password` no psql). Não escrever senha literal em migration versionada. O migrator deve usar `SET ROLE app_owner` durante criação/alteração de objetos. O runtime não deve ser membro de owner, migrator ou neon_superuser.

Conceder somente permissões necessárias, ajustando database/schema reais:

```sql
GRANT CONNECT ON DATABASE appdb TO app_runtime;
SET ROLE app_owner;
CREATE SCHEMA app AUTHORIZATION app_owner;
RESET ROLE;
GRANT USAGE ON SCHEMA app TO app_runtime;
GRANT SELECT, INSERT, UPDATE, DELETE
  ON ALL TABLES IN SCHEMA app TO app_runtime;
GRANT USAGE, SELECT
  ON ALL SEQUENCES IN SCHEMA app TO app_runtime;

ALTER DEFAULT PRIVILEGES FOR ROLE app_owner IN SCHEMA app
  GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_runtime;
ALTER DEFAULT PRIVILEGES FOR ROLE app_owner IN SCHEMA app
  GRANT USAGE, SELECT ON SEQUENCES TO app_runtime;
```

Revisar privilégios de UPDATE/DELETE por tabela; não conceder globalmente se o domínio exigir somente leitura. Os comandos são exemplos, não devem ser executados sem conferir database, schema e role alvo.

## 8. Modelagem mínima de identidade

Migration ilustrativa (executar com `app_migrator` e assumir a role `app_owner` antes de criar objetos):

```sql
SET ROLE app_owner;
CREATE TABLE app.app_users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  auth0_sub TEXT NOT NULL UNIQUE,
  email TEXT NOT NULL,
  display_name TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX app_users_email_idx ON app.app_users(email);
```

Usar `auth0_sub` como identidade externa; não usar email como chave primária. Em sistemas multi-tenant, modelar `organizations`, `memberships` e `tenant_id` com constraints e índices adequados. A membership deve ser verificada pelo backend em cada operação.

## 9. Row-Level Security (RLS)

RLS é defesa adicional contra leitura/escrita fora do escopo. Não substitui autenticação, autorização de aplicação, nem controle de privilégios SQL. Habilitar RLS e criar políticas específicas para cada tabela que contenha dados isolados.

Exemplo de isolamento por usuário:

```sql
ALTER TABLE app.app_users ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.app_users FORCE ROW LEVEL SECURITY;

CREATE POLICY app_users_self
ON app.app_users
FOR ALL
TO app_runtime
USING (
  auth0_sub = current_setting('app.auth0_sub', true)
)
WITH CHECK (
  auth0_sub = current_setting('app.auth0_sub', true)
);
```

A aplicação define o contexto **dentro da mesma transação** em que executa a consulta:

```typescript
import "server-only";
import { sql } from "@/lib/db";

export async function findCurrentUser(auth0Sub: string) {
  const [context, rows] = await sql.transaction([
    sql`SELECT set_config('app.auth0_sub', ${auth0Sub}, true)`,
    sql`SELECT id, auth0_sub, email
        FROM app.app_users
        WHERE auth0_sub = ${auth0Sub}
        LIMIT 1`,
  ]);

  return rows[0] ?? null;
}
```

A opção `true` de `set_config` limita o valor à transação; não deixar contexto de usuário em estado de sessão persistente. Verificar a API transacional da versão do driver instalada antes de adotar o exemplo.

**Limitações importantes:**

- O valor `auth0Sub` deve vir da sessão validada no servidor, nunca do body/query/header controlado pelo browser.
- A role de runtime não pode ter `BYPASSRLS), superuser ou ser owner sem FORCE RLS.
- Roles criadas via Console/CLI/API podem herdar `neon_superuser`; verificar membership efetiva.
- Um atacante que obtenha execução SQL como a role de runtime pode tentar alterar custom GUC. RLS não protege contra comprometimento total do backend/credencial; manter SQL parametrizado, privilégios mínimos e segredos isolados.
- Testar cada policy com a role real de runtime, não somente com a role administrativa.
- Em multi-tenancy, validar membership no backend e incluir `tenant_id` na policy; não confiar em tenant escolhido pelo cliente.

Fontes: [Neon RLS](https://neon.com/docs/guides/row-level-security), [Manage roles](https://neon.com/docs/manage/roles).

## 10. Migrations versionadas

Princípios:

1. Cada mudança de schema gera arquivo SQL numerado e imutável.
2. Migration é revisada por PR antes da aplicação.
3. Testar em branch Neon não produtiva.
4. Aplicar com role migrator e conexão direta.
5. Registrar versão aplicada e resultado.
6. Não editar migration já aplicada; criar migration corretiva.
7. Preparar rollback lógico ou migration compensatória; nem toda mudança destrutiva é reversível.
8. Fazer backup/export independente antes de mudança de alto risco.

Exemplo de execução documentada:

```bash
psql "$DATABASE_URL_MIGRATION" \
  -v ON_ERROR_STOP=1 \
  -f database/migrations/002_users_and_rls.sql
```

A migration não deve ser executada automaticamente durante cada build Preview/Production. Separar release da aplicação e mudança de schema, confirmar branch/database de destino e exigir aprovação operacional antes de aplicar em produção.

## 11. Transações e driver

O driver HTTP é adequado para consultas curtas e transações suportadas pelo protocolo do driver. Para transações com estado de sessão, prepared statements persistentes, cursores ou operações que dependam de conexão TCP longa, avaliar WebSockets ou cliente PostgreSQL compatível. Não assumir que uma sequência de chamadas HTTP separadas compartilha a mesma sessão PostgreSQL.

Evitar:

- abrir uma conexão por módulo de browser;
- executar queries em Client Components;
- usar pool próprio em cima do pooler sem necessidade;
- deixar transações abertas durante chamadas externas;
- executar operações de longa duração no caminho de request interativo.

## 12. Backup e restore

O histórico Free do Neon é limitado (6 horas e até 1 GB-month de change history conforme plano consultado); não é backup de longo prazo. Manter export lógico independente sob controle do operador e armazenar cópias fora do projeto Neon, com criptografia e acesso restrito.

Exemplo de export:

```bash
pg_dump "$DATABASE_URL_MIGRATION" \
  --no-owner --no-privileges -Fc \
  -f backup.dump
```

Exemplo de restauração para banco/branch de teste:

```bash
pg_restore --no-owner --no-privileges \
  --clean --if-exists \
  -d "$RESTORE_DATABASE_URL" backup.dump
```

Não executar `--clean` em produção sem procedimento aprovado. Testar restore periodicamente, medir duração, validar contagem e integridade. Guardar evidência de data, versão PostgreSQL, tamanho, hash e resultado do teste sem expor connection strings.

## 13. Monitoramento e troubleshooting

| Sintoma | Verificação |
|---|---|
| conexão lenta após inatividade | scale-to-zero e cold start |
| erro de limite de compute | CU-hours consumidas por projeto |
| storage esgotado | tamanho de tabelas/índices e retenção |
| transfer esgotada | tráfego de saída e consultas |
| permissão negada | role atual, grants, schema e membership |
| RLS bloqueia tudo | contexto transaction-local e policy |
| RLS permite dados indevidos | role BYPASSRLS/owner, policy e origem do GUC |
| conexão excedida | pooled connection, concorrência e duração |
| migration falha | branch/database/role, locks e migration parcialmente aplicada |

No Free, o histórico de monitoramento é curto. Exportar métricas e evidências que precisem ser mantidas além da janela oficial.

## 14. Checklist

- [ ] projeto e região aprovados;
- [ ] quotas Free documentadas;
- [ ] branch strategy dentro do limite de 10;
- [ ] pooled URL para runtime e direct URL para migrations;
- [ ] role runtime criada via SQL e sem membership administrativa;
- [ ] grants mínimos por tabela;
- [ ] migrations versionadas e revisadas;
- [ ] RLS ativada onde necessário;
- [ ] teste RLS executado com role real;
- [ ] backup externo e restore testado;
- [ ] logs sem dados pessoais ou secrets;
- [ ] consumo de compute, storage e transfer monitorado.

## 15. Fontes oficiais

- [Neon Plans](https://neon.com/docs/introduction/plans)
- [Neon Free quotas](https://neon.com/faqs/free-plan-limits-and-quotas)
- [Neon serverless driver](https://neon.com/docs/serverless/serverless-driver)
- [Neon connection pooling](https://neon.com/docs/connect/connection-pooling)
- [Neon manage roles](https://neon.com/docs/manage/roles)
- [Neon row-level security](https://neon.com/docs/guides/row-level-security)
- [PostgreSQL Row Security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [PostgreSQL pg_dump](https://www.postgresql.org/docs/current/app-pgdump.html)
