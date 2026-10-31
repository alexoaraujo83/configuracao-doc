# 04 — Autorização e multi-tenancy

## Modelo de autorização

Separar:
1. **Autenticação:** sessão válida e user sub.
2. **Membership:** associação do usuário ao tenant.
3. **Permissão:** ação permitida sobre um recurso.
4. **Isolamento:** consultas limitadas ao tenant autorizado.

RBAC pode definir papéis gerais, mas autorização efetiva deve considerar tenant, recurso, estado e relação do usuário com o objeto (RBAC + checks relacionais/ABAC quando necessário).

## Modelo de dados de referência

~~~sql
CREATE TABLE tenants (
  id uuid PRIMARY KEY,
  name text NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE app_users (
  id uuid PRIMARY KEY,
  auth0_sub text NOT NULL UNIQUE,
  email text,
  created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE tenant_memberships (
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  user_id uuid NOT NULL REFERENCES app_users(id) ON DELETE CASCADE,
  role text NOT NULL CHECK (role IN ('owner','admin','member','viewer')),
  status text NOT NULL CHECK (status IN ('active','suspended','invited')),
  created_at timestamptz NOT NULL DEFAULT now(),
  PRIMARY KEY (tenant_id, user_id)
);
~~~

O schema é ilustrativo e precisa ser reconciliado com o modelo ER do projeto antes de uso.

## Fluxo de autorização

1. Obter sub da sessão Auth0 validada.
2. Resolver user interno por auth0_sub.
3. Receber tenant_id como seletor.
4. Verificar membership ativa.
5. Verificar permissão exigida para a ação e recurso.
6. Executar a operação com tenant context na mesma transação.
7. RLS aplica uma segunda barreira no banco.
8. Retornar DTO mínimo, sem expor campos internos.

## Regras

- Não confiar em user_id, auth0_sub, role, tenant_id ou owner enviados no body.
- Não inferir acesso por email ou domínio de email.
- Não permitir que usuário altere o próprio role sem fluxo privilegiado.
- Alterações de membership devem exigir permissão administrativa e auditoria.
- Toda operação de leitura, escrita, exportação e exclusão deve ter regra explícita.
- Revisar BOLA/IDOR em cada endpoint que recebe identificador de objeto.

## Matriz de permissão exemplo

| Ação | owner | admin | member | viewer |
|---|---:|---:|---:|---:|
| Ler projeto | Sim | Sim | Conforme membership | Sim |
| Criar projeto | Sim | Sim | Conforme política | Não |
| Atualizar projeto | Sim | Sim | Próprio/permitido | Não |
| Excluir projeto | Sim | Conforme política | Não | Não |
| Gerir membros | Sim | Conforme política | Não | Não |
| Alterar owner | Fluxo especial | Não | Não | Não |

A matriz deve ser adaptada por recurso. Nunca implementar permissões apenas por esconder botões na interface.

## Free-only

Não exigir Auth0 Organizations. A tabela tenant_memberships é o registro autoritativo do domínio; o tenant Auth0 é provedor de identidade compartilhado, não tenant de negócio.
