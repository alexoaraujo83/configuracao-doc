# 02 — RBAC e autorização multiusuário

**Estado:** modelo de referência; não implantado nem validado em tenant Auth0 ou banco Neon.
**Objetivo:** autorização explícita no servidor, com defesa adicional no PostgreSQL.

## 1. Princípios
- Autenticação identifica o sujeito; autorização decide o que ele pode fazer.
- O identificador estável é o claim sub da sessão validada, não o email.
- O navegador não é fonte confiável de subject, role, permission ou tenant.
- Toda Route Handler, Server Action e operação privada verifica autorização.
- RLS limita o impacto de erro na aplicação, mas não substitui autorização.
- Privilégios SQL do runtime permanecem mínimos.

## 2. Modelo de papéis

| Role | Permissões |
|---|---|
| reader | document:read |
| editor | document:read, document:write |
| admin | permissões administrativas explicitamente listadas |

Evitar role admin sem delimitar escopo. Em sistemas multi-organização, a role deve ser atribuída por organização/tenant e a consulta deve validar subject e tenant.

## 3. Modelo de dados sugerido

Executar como migration revisada, nunca diretamente em produção:

~~~sql
CREATE TABLE app.app_roles (
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  role_key TEXT NOT NULL UNIQUE,
  description TEXT NOT NULL
);

CREATE TABLE app.app_user_roles (
  auth0_sub TEXT NOT NULL,
  role_id BIGINT NOT NULL REFERENCES app.app_roles(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  PRIMARY KEY (auth0_sub, role_id)
);

CREATE TABLE app.documents (
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  owner_auth0_sub TEXT NOT NULL,
  title TEXT NOT NULL,
  body TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX documents_owner_idx
  ON app.documents(owner_auth0_sub);
~~~

O exemplo é single-tenant por usuário. Se houver organizações, acrescentar organization_id, tabela de memberships e constraints compostas; não inferir tenancy somente pelo domínio de email.

## 4. Resolver roles no servidor

Buscar roles a partir de fonte controlada no servidor, usando session.user.sub. Não aceitar roles do body, query string, localStorage ou estado React como autoridade.

Exemplo conceitual da consulta (usar o template tag parametrizado do driver Neon):

~~~sql
SELECT r.role_key
FROM app.app_user_roles ur
JOIN app.app_roles r ON r.id = ur.role_id
WHERE ur.auth0_sub = $1;
~~~

Validar subject antes da consulta e obter o valor exclusivamente da sessão Auth0 validada. A role runtime precisa de SELECT somente nas tabelas necessárias.

## 5. Guard de autorização

Pseudocódigo de uma operação de servidor:

~~~typescript
const session = await auth0.getSession();

if (!session?.user?.sub) {
  return { ok: false, status: 401 };
}

const roles = await getUserRoles(session.user.sub);

if (!hasPermission(roles, requiredPermission)) {
  return { ok: false, status: 403 };
}

return { ok: true, subject: session.user.sub };
~~~

Ajustar imports, tipos e tratamento de sessão à versão instalada do SDK. Não copiar o pseudocódigo sem compilar e testar. Cache de roles exige estratégia explícita de invalidação e revogação.

## 6. RLS para dados pertencentes ao usuário

Policy de exemplo:

~~~sql
ALTER TABLE app.documents ENABLE ROW LEVEL SECURITY;
ALTER TABLE app.documents FORCE ROW LEVEL SECURITY;

CREATE POLICY documents_select_own
ON app.documents FOR SELECT TO app_runtime
USING (owner_auth0_sub = current_setting('app.auth0_sub', true));

CREATE POLICY documents_insert_own
ON app.documents FOR INSERT TO app_runtime
WITH CHECK (owner_auth0_sub = current_setting('app.auth0_sub', true));

CREATE POLICY documents_update_own
ON app.documents FOR UPDATE TO app_runtime
USING (owner_auth0_sub = current_setting('app.auth0_sub', true))
WITH CHECK (owner_auth0_sub = current_setting('app.auth0_sub', true));
~~~

Adicionar policy DELETE somente quando necessário. Confirmar que runtime não é owner nem possui BYPASSRLS. O contexto app.auth0_sub precisa ser transaction-local e definido na mesma transação da consulta; seguir o manual Neon e testar a versão fixada do driver.

## 7. Autorização por recurso
A permissão de função não basta. Um editor pode ter document:write, mas só deve alterar documentos dentro do escopo permitido. Aplicar filtros de ownership/tenant no SQL e validar a ação na camada de domínio.

Não buscar registro somente por ID e confiar no cliente para filtrar. Não receber owner subject como campo mutável do request. Não permitir que usuário altere a própria role por endpoint genérico. Não usar email como identificador de autorização.

## 8. Matriz de testes

| Ator | Ação | Esperado |
|---|---|---|
| Anônimo | Ler documento privado | 401/login |
| Reader | Ler próprio documento | 200 |
| Reader | Alterar documento próprio | 403 |
| Editor | Alterar documento permitido | 200 |
| Editor | Gerenciar usuários | 403 |
| Usuário A | Ler documento de B | 403/404 |
| Usuário A | Alterar owner para B | 400/403 |
| Sem role | Acessar endpoint protegido | 403 |
| Runtime SQL | Executar DDL | Negado |
| Runtime SQL | Ler linha sem contexto RLS | Negado/zero linhas |

## 9. Referências
- [Auth0 — RBAC](https://auth0.com/docs/manage-users/access-control/rbac)
- [Auth0 — Authorization Core](https://auth0.com/docs/manage-users/access-control)
- [PostgreSQL — Row Security Policies](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [PostgreSQL — CREATE POLICY](https://www.postgresql.org/docs/current/sql-createpolicy.html)
