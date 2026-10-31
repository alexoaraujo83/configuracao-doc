# 3. Configurar Neon PostgreSQL

## Criar projeto
No Neon Console:
1. Criar projeto com nome padronizado.
2. Selecionar região compatível com aplicação e latência.
3. Confirmar versão PostgreSQL.
4. Identificar branch raiz.
5. Selecionar database e roles.
6. Registrar IDs não secretos e quotas observadas.

Armazenamento, compute, branches, egress, retenção e suspensão variam por plano e podem mudar. Consultar painel e documentação Free antes de definir quantidade de ambientes.

## Estratégia de branches
- Branch raiz: produção.
- Branch dev: desenvolvimento e integração.
- Branches temporárias de preview somente quando quotas e retenção permitirem.

Nunca usar branch raiz de produção para testes/previews. Não automatizar uma branch por PR sem contabilizar quotas e política de limpeza.

## Roles
Separar owner, migrator e runtime:
~~~sql
CREATE ROLE app_owner NOLOGIN;
CREATE ROLE app_migrator LOGIN NOINHERIT;
GRANT app_owner TO app_migrator;
CREATE ROLE app_runtime LOGIN NOINHERIT NOBYPASSRLS;
~~~

Modelo para revisão de DBA. Runtime não deve ser owner e não recebe DDL, TRUNCATE, privilégios administrativos ou BYPASSRLS. Conceder somente permissões necessárias ao schema, tabelas e sequences. Migrator assume owner explicitamente durante migrations controladas.

## Connection string
No Console, selecionar branch, database e role e copiar a URL pelo fluxo Connect. Preferir pooled connection para workloads serverless comuns. Operações que exigem comportamento específico de sessão devem seguir orientação de conexão direta e transações do driver.

DATABASE_URL é segredo server-side. Não colocar no código, logs ou NEXT_PUBLIC_*.

## Cliente server-side
~~~ts
import "server-only";
import { neon } from "@neondatabase/serverless";

const url = process.env.DATABASE_URL;
if (!url) throw new Error("DATABASE_URL ausente");

export const sql = neon(url);
~~~

Consulta parametrizada:
~~~ts
const rows = await sql`
  SELECT id, name FROM project
  WHERE tenant_id = ${tenantId}
`;
~~~

Não interpolar fragmentos SQL controlados pelo usuário. API HTTP stateless e transações interativas têm limitações distintas: contexto RLS e queries protegidas devem ocorrer na mesma transação suportada pelo driver.

## Migrations
- Versionar SQL em database/migrations.
- Nomear sequencialmente ou por timestamp.
- Revisar DDL e impacto em dados.
- Aplicar primeiro em branch isolada.
- Validar compatibilidade da aplicação.
- Executar com credencial migrator em processo controlado.
- Nunca executar migration no startup de Functions.
- Registrar migration, commit, ambiente, responsável e resultado.

## RLS
Habilitar RLS, avaliar FORCE ROW LEVEL SECURITY, criar policies USING/WITH CHECK, validar membership e aplicar contexto tenant transacionalmente. Contexto definido pelo backend é defesa em profundidade, não proteção independente contra backend comprometido.

## Recuperação
Branching/PITR não substituem automaticamente backup externo. Documentar export lógico, armazenamento seguro, retenção, criptografia, acesso e teste de restore. Não prometer RPO/RTO sem validar quotas e retenção Free atuais.

## Troubleshooting
- Timeout: região, compute, host e rede.
- Muitas conexões: pooling e concorrência.
- SSL: URL oficial e sslmode.
- Permission denied: grants, role efetiva e schema.
- RLS vazio: tenant context, policy e membership.
- Migration falha: role, locks, ordem e schema.

Referências: https://neon.com/docs, https://neon.com/docs/connect/connect-from-any-app, https://neon.com/docs/serverless/serverless-driver, https://neon.com/branching
