# 2. Auth0 + Next.js

## Instalação

~~~bash
npm install @auth0/nextjs-auth0
~~~

Variáveis ilustrativas, em arquivo local ignorado pelo Git e no ambiente de hospedagem:

~~~dotenv
APP_BASE_URL=http://localhost:3000
AUTH0_DOMAIN=seu-tenant.us.auth0.com
AUTH0_CLIENT_ID=placeholder
AUTH0_CLIENT_SECRET=placeholder
AUTH0_SECRET=segredo-aleatorio-forte
~~~

Confirmar nomes e requisitos no guia do SDK v4. AUTH0_SECRET deve ser forte, aleatório, estável dentro do mesmo ambiente e diferente entre ambientes. Nunca versionar valores reais.

## Inicialização server-side

~~~ts
import { Auth0Client } from "@auth0/nextjs-auth0/server";

export const auth0 = new Auth0Client();
~~~

A API exata, rotas integradas e convenções devem seguir a versão instalada. Não copiar exemplos v3 como handleAuth ou /api/auth/[...auth0].

## Middleware e Proxy
- Next.js 15: convenção middleware.ts.
- Next.js 16: convenção proxy.ts.
- Usar somente a convenção da versão real e a integração indicada pelo SDK v4.
- Middleware/Proxy pode iniciar ou atualizar o fluxo de sessão, mas não substitui autorização dentro de cada Route Handler, Server Action e acesso a dados.

Pseudocódigo conceitual:

~~~ts
// Pseudocódigo: integrar o mecanismo oficial do SDK conforme versão instalada.
export async function middlewareOrProxy(request: Request) {
  // Delegar ao SDK Auth0; não validar tokens manualmente.
}
~~~

## Identidade no servidor

~~~ts
import { auth0 } from "@/lib/auth0";

export async function requireIdentity() {
  const session = await auth0.getSession();

  if (!session?.user?.sub) {
    throw new Error("UNAUTHENTICATED");
  }

  return {
    subject: session.user.sub,
    email: typeof session.user.email === "string"
      ? session.user.email
      : null,
  };
}
~~~

Confirmar assinatura e contexto de getSession na documentação da versão instalada. Nunca aceitar subject, email, tenant ou role enviados pelo navegador como identidade confiável.

## Autorização de domínio

~~~ts
type Permission = "project:read" | "project:update" | "member:invite";

type Principal = {
  subject: string;
  tenantId: string;
  permissions: ReadonlySet<Permission>;
};

export function assertPermission(
  principal: Principal,
  permission: Permission,
): void {
  if (!principal.permissions.has(permission)) {
    throw new Error("FORBIDDEN");
  }
}
~~~

O servidor carrega membership e permissões de fonte confiável, verifica estado ativo e vínculo do recurso ao tenant. O exemplo não implementa o carregamento do banco.

## Callback, logout e sessão
Cadastrar callback, logout URL e Allowed Web Origins explicitamente para cada ambiente. Não usar curingas amplos em produção. Confirmar endpoint real exposto pelo SDK v4. Preferir sessão gerenciada pelo SDK; não expor access/refresh tokens ao cliente. Usar HTTPS, cookies seguros, HttpOnly e SameSite conforme SDK, proteção CSRF e scopes mínimos.
