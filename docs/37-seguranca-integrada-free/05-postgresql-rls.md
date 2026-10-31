# 05 — PostgreSQL: roles, grants e Row-Level Security

## Separação de roles

Modelo recomendado:
- app_owner: NOLOGIN, dono do schema/objetos, usado somente em migrações controladas.
- app_migrator: LOGIN, sem privilégios permanentes de runtime; pode assumir app_owner durante migration autorizada.
- app_runtime: LOGIN, NOINHERIT, NOBYPASSRLS, não owner, sem DDL, TRUNCATE ou privilégios administrativos.

A role runtime deve receber apenas SELECT/INSERT/UPDATE/DELETE nas tabelas necessárias e USAGE nas sequences pertinentes. Revogar privilégios amplos concedidos por padrão e revisar default privileges para objetos futuros.

## RLS

Habilitar RLS em cada tabela tenant-scoped e considerar FORCE ROW LEVEL SECURITY para que owner normal também esteja sujeito à policy, exceto superuser/BYPASSRLS.

~~~sql
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
ALTER TABLE projects FORCE ROW LEVEL SECURITY;

CREATE POLICY projects_tenant_isolation
ON projects
USING (
  tenant_id = current_setting('app.tenant_id', true)::uuid
)
WITH CHECK (
  tenant_id = current_setting('app.tenant_id', true)::uuid
);
~~~

A policy acima é uma camada de isolamento; ela não comprova membership. O backend deve verificar membership e permissão antes da consulta.

## Contexto transacional

O contexto não pode vazar entre requests. Definir valores com escopo local à transação (equivalente a SET LOCAL / set_config(..., true)) e executar todas as consultas protegidas na mesma transação.

~~~sql
SELECT set_config('app.tenant_id', $1, true);
SELECT set_config('app.user_sub', $2, true);
~~~

Não executar set_config em uma requisição independente e depois consultar fora da transação. O terceiro argumento true torna a configuração local à transação.

## Neon serverless driver

A API HTTP do driver é adequada para consultas serverless, mas transações interativas/sessão stateful têm limitações específicas. Não presumir que múltiplas chamadas sql independentes compartilham a mesma transação.

- Para sequência transacional compatível com HTTP, utilizar a API de transação/batch documentada pelo driver e incluir SET LOCAL/contexto e consultas protegidas na mesma unidade transacional.
- Para funcionalidades que exigem sessão PostgreSQL persistente, avaliar conexão TCP/WebSocket suportada pelo driver e ambiente Node.js.
- Migrations devem usar conexão direta e role migrator, conforme orientação Neon.
- Validar comportamento real do driver e versão escolhida antes de adotar RLS com contexto.

## Privilégios

- Runtime não deve ser owner nem BYPASSRLS.
- Não conceder CREATE no schema público sem necessidade.
- Não conceder TRUNCATE, REFERENCES, TRIGGER ou DDL à role runtime por padrão.
- Revogar acesso de PUBLIC quando apropriado.
- Auditar views, funções SECURITY DEFINER, triggers e tabelas auxiliares: RLS não é substituto de revisão desses objetos.

## Testes obrigatórios

- Sem tenant context: nenhuma linha tenant-scoped acessível.
- Tenant A não lê/atualiza/exclui dados de B.
- INSERT com tenant B enquanto contexto A falha.
- Usuário suspenso/inativo não opera mesmo com tenant correto.
- Runtime role não consegue DDL, TRUNCATE nem bypass.
- Pool/execução concorrente não carrega contexto de request anterior.

## Referências

- https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- https://www.postgresql.org/docs/current/sql-set.html
- https://www.postgresql.org/docs/current/functions-admin.html
- https://neon.com/docs/serverless/serverless-driver
- https://neon.com/docs/get-started/connect-neon
