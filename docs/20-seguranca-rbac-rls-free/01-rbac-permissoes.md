# RBAC e matriz de permissões

## Modelo
1. **Identidade:** Auth0 `sub` validado no servidor.
2. **Membership:** vínculo ativo do usuário com o tenant no banco.
3. **Role:** função atribuída dentro do tenant.
4. **Permission:** ação específica sobre um recurso.

Não usar e-mail como chave de identidade. Não aceitar `user_id`, `role` ou `tenant_id` enviados pelo browser como prova de autorização.

## Papéis de referência

| Role | Permissões exemplificativas | Limites |
|---|---|---|
| tenant_owner | tenant:read/update, members:invite/role:assign, projects:* | somente seu tenant; ações destrutivas com fluxo próprio |
| tenant_admin | tenant:read, members:invite/read, projects:create/update | sem transferir ownership ou alterar política global |
| editor | projects:read/create/update, documents:read/create/update | sem gestão de membros |
| viewer | projects:read, documents:read | somente leitura |
| service_worker | permissões técnicas mínimas | identidade técnica separada; sem privilégios humanos |

## Matriz ação × papel

| Ação | Owner | Admin | Editor | Viewer |
|---|---:|---:|---:|---:|
| Ler dados do tenant | Sim | Sim | conforme recurso | conforme recurso |
| Atualizar tenant | Sim | restrito | Não | Não |
| Convidar membros | Sim | Sim | Não | Não |
| Alterar role | Sim | restrito | Não | Não |
| Criar/editar projeto | Sim | Sim | Sim | Não |
| Excluir projeto | Sim, auditado | política explícita | Não | Não |
| Ler documentos | Sim | Sim | Sim | Sim |
| Editar documentos | Sim | Sim | Sim | Não |
| Exportar dados | explícito/auditado | não por padrão | Não | Não |

Todo "Sim" depende de membership ativo e tenant correto. Nenhum papel concede acesso cross-tenant automaticamente.

## Tabelas locais de autorização

```sql
CREATE TABLE app.tenants (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  slug text NOT NULL UNIQUE
);

CREATE TABLE app.tenant_memberships (
  tenant_id uuid NOT NULL REFERENCES app.tenants(id),
  auth0_sub text NOT NULL,
  role text NOT NULL CHECK (role IN ('tenant_owner','tenant_admin','editor','viewer')),
  status text NOT NULL CHECK (status IN ('active','invited','suspended')),
  PRIMARY KEY (tenant_id, auth0_sub)
);

CREATE TABLE app.projects (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES app.tenants(id),
  name text NOT NULL,
  created_by_sub text NOT NULL,
  UNIQUE (tenant_id, id)
);
```

Atribuição de roles deve ser server-side, autorizada e auditada. Nunca permitir autoelevação por campo `role` enviado no formulário.

## Auth0 RBAC
- O tenant Free é único por conta e tem limite de 10 Applications no total, compartilhado entre projetos; não pressupor um client por ambiente. Inventariar slots antes de configurar API e aplicações.
- Definir API e permissões funcionais (ex.: `projects:read`, `projects:write`).
- Ativar RBAC e a claim `permissions` se a configuração estiver disponível e elegível no Free.
- Validar assinatura, issuer, audience, expiração e algoritmo esperado.
- ID token não é access token de API.
- A claim autoriza capacidades da API; ainda é necessário validar membership, tenant, ownership e estado atual no banco.
- Confirmar limites/eligibilidade Free no painel. Não presumir recurso pago nem usar Organizations como pré-requisito.
- Preferir role → permissions → usuário; atribuição direta de permissions a usuários não é padrão recomendado.

## DAL TypeScript (exemplo conceitual)

```ts
import "server-only";

type Permission =
  | "projects:read"
  | "projects:create"
  | "projects:update"
  | "documents:read"
  | "documents:update"
  | "members:invite";

const permissionsByRole: Record<string, Permission[]> = {
  tenant_owner: ["projects:read","projects:create","projects:update","documents:read","documents:update","members:invite"],
  tenant_admin: ["projects:read","projects:create","projects:update","documents:read","documents:update","members:invite"],
  editor: ["projects:read","projects:create","projects:update","documents:read","documents:update"],
  viewer: ["projects:read","documents:read"],
};

export async function requireTenantPermission(
  auth0Sub: string,
  tenantId: string,
  permission: Permission,
) {
  const membership = await findActiveMembership(tenantId, auth0Sub);
  if (!membership) throw new Error("FORBIDDEN");
  if (!permissionsByRole[membership.role]?.includes(permission)) {
    throw new Error("FORBIDDEN");
  }
  return membership;
}
```

`findActiveMembership` consulta o banco no servidor. Em operações críticas, verificar autorização junto à operação/transação para reduzir TOCTOU.

## Regras
- DAL marcada com `server-only`.
- Checagem em cada Server Action e Route Handler, não apenas layout/Proxy.
- Consultar recurso por ID + tenant_id, nunca por ID isolado.
- Falha ao consultar membership/política deve negar.
- DTO retorna somente campos necessários.
- Auditoria registra ator, ação, recurso, tenant, horário e resultado, sem tokens ou payload sensível.
- Revogação/suspensão precisa surtir efeito no backend; claims antigas podem continuar válidas até expirarem.
