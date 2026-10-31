# 01 — Cadastro e gerenciamento de usuários

Auth0 mantém credenciais e autentica a identidade; Neon mantém perfil de domínio, preferências, status e memberships. Nunca armazenar senha/hash ou credenciais Auth0 localmente. Cadastro ocorre no Universal Login; após sessão válida, criar/atualizar perfil de forma idempotente.

~~~sql
CREATE TABLE app_user (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  auth0_sub text NOT NULL UNIQUE,
  email text,
  display_name text,
  avatar_url text,
  status text NOT NULL DEFAULT 'active'
    CHECK (status IN ('active','suspended','pending_deletion')),
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);
~~~

Tratar sub como identificador opaco e imutável. E-mail é mutável; não usar como chave primária nem para vincular contas automaticamente. Verificar indicadores de verificação do e-mail conforme conexão e SDK.

~~~ts
type Identity = { sub: string; email?: string; name?: string; picture?: string };

export async function ensureLocalUser(identity: Identity) {
  if (!identity.sub) throw new Error("IDENTITY_SUB_REQUIRED");
  return userRepository.upsertFromIdentity({
    auth0Sub: identity.sub,
    email: identity.email ?? null,
    displayName: identity.name ?? null,
    avatarUrl: identity.picture ?? null,
  });
}
~~~

Repository usa INSERT ... ON CONFLICT por auth0_sub. Atualizar apenas atributos permitidos; não reativar conta suspensa automaticamente; não aceitar sub/status/role do browser; definir precedência entre atributos do IdP e preferências locais.

Gerenciamento: leitura do próprio perfil; atualização por allowlist; suspensão local bloqueia acesso mesmo que Auth0 autentique; exclusão define exclusão lógica/anonimização e eventual exclusão no Auth0; account linking exige fluxo suportado e consentido.

Testes: login inicial cria registro; repetição não duplica; mudança de e-mail não cria identidade nova; suspensão não é revertida; cliente não altera sub/status; ausência de sub é rejeitada.

Free: evitar sincronização administrativa em massa via Management API. Se necessária, revisar escopos, quotas e elegibilidade.
