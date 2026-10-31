# 03 — Perfis, permissões e autorização

Para multi-tenancy Free, manter autorização de domínio no PostgreSQL. Auth0 autentica; a aplicação decide acesso ao tenant. Não depender de Auth0 Organizations ou RBAC cuja elegibilidade Free não esteja confirmada.

~~~sql
CREATE TABLE tenant (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now()
);
CREATE TABLE tenant_member (
  tenant_id uuid NOT NULL REFERENCES tenant(id),
  user_id uuid NOT NULL REFERENCES app_user(id),
  role text NOT NULL CHECK (role IN ('owner','admin','editor','reader')),
  status text NOT NULL DEFAULT 'active'
    CHECK (status IN ('invited','active','suspended')),
  created_at timestamptz NOT NULL DEFAULT now(),
  PRIMARY KEY (tenant_id, user_id)
);
~~~

Catálogo ilustrativo: owner = tenant:manage, members:manage, projects:*; admin = members:manage e CRUD de projetos; editor = create/read/update; reader = read. Definir negação, herança e precedência. Proteger regra do último owner ativo.

~~~ts
export async function requireTenantPermission(tenantId: string, permission: Permission) {
  const identity = await getCurrentIdentity();
  if (!identity) throw new UnauthorizedError();
  const user = await userRepository.findByAuth0Sub(identity.sub);
  if (!user || user.status !== "active") throw new ForbiddenError();
  const membership = await membershipRepository.findActive({
    userId: user.id, tenantId,
  });
  if (!membership) throw new ForbiddenError();
  if (!rolePermissions[membership.role].includes(permission)) {
    throw new ForbiddenError();
  }
  return { identity, user, membership };
}
~~~

Cada caso de uso valida membership ativa e permissão específica; tenant vindo da URL é seletor, não prova de acesso. Revalidar dentro da transação se houver risco de alteração concorrente.

Se usar Auth0 RBAC em API, validar assinatura, issuer, audience e expiração do access token. Não usar ID token como access token. Não confiar em metadata editável para privilégios; claims não substituem membership dinâmica.

Testes: matriz positiva/negativa, acesso horizontal entre tenants, revogação, usuário suspenso, tenant inexistente e tentativa de alterar role sem permissão.
