# 2. Configurar Auth0

## Criar aplicação
No Auth0 Dashboard:
1. Applications → Create Application.
2. Selecionar Regular Web Application.
3. Nomear por projeto e ambiente.
4. Obter Domain, Client ID e Client Secret.
5. Armazenar os valores em gerenciador seguro, fora do repositório.

Confirmar no painel a elegibilidade dos recursos necessários no Free. Não depender de Organizations, RBAC avançado ou add-ons sem confirmação de disponibilidade.

## Variáveis SDK v4
~~~dotenv
AUTH0_DOMAIN=tenant-region.auth0.com
AUTH0_CLIENT_ID=placeholder
AUTH0_CLIENT_SECRET=placeholder
AUTH0_SECRET=placeholder-aleatorio
APP_BASE_URL=http://localhost:3000
~~~

Gerar AUTH0_SECRET:
~~~bash
openssl rand -hex 32
~~~

Usar segredo distinto por ambiente. SDK v4 usa AUTH0_DOMAIN e APP_BASE_URL; não copiar AUTH0_ISSUER_BASE_URL nem AUTH0_BASE_URL de tutoriais v3. APP_BASE_URL pode ser omitido em certos previews dinâmicos, mas é útil defini-lo localmente.

## URLs locais
Cadastrar:
- Allowed Callback URLs: http://localhost:3000/auth/callback
- Allowed Logout URLs: http://localhost:3000
- Allowed Web Origins: http://localhost:3000

SDK v4 monta rotas /auth/login, /auth/logout e /auth/callback. Não criar rota legada /api/auth/[...auth0].

## Cliente
Arquivo src/lib/auth0.ts:
~~~ts
import { Auth0Client } from "@auth0/nextjs-auth0/server";

export const auth0 = new Auth0Client();
~~~

## Middleware / Proxy
Next.js 15: src/middleware.ts:
~~~ts
import type { NextRequest } from "next/server";
import { auth0 } from "@/lib/auth0";

export async function middleware(request: NextRequest) {
  return await auth0.middleware(request);
}

export const config = {
  matcher: ["/((?!_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)"],
};
~~~

Next.js 16: src/proxy.ts:
~~~ts
import { auth0 } from "@/lib/auth0";

export async function proxy(request: Request) {
  return await auth0.middleware(request);
}

export const config = {
  matcher: ["/((?!_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)"],
};
~~~

Usar somente o arquivo correspondente à versão efetiva. Não manter middleware e proxy duplicados sem justificativa.

## Proteger página no servidor
~~~tsx
import { redirect } from "next/navigation";
import { auth0 } from "@/lib/auth0";

export default async function DashboardPage() {
  const session = await auth0.getSession();
  if (!session) redirect("/auth/login");
  return <main>Área autenticada</main>;
}
~~~

A página, cada Route Handler e cada Server Action devem validar sessão e autorização. Subject vem da sessão server-side; membership e permissões vêm do domínio.

## Preview dinâmico
SDK v4 pode inferir base URL do host recebido quando APP_BASE_URL não está definido. A segurança depende de Allowed Callback URLs restritas. Não usar wildcard amplo nem confiar cegamente no Host header. Se o plano/configuração não permitir allowlist segura de previews, não habilitar autenticação nesses hosts; usar host estável aprovado ou manter testes autenticados localmente.

## Validação e troubleshooting
- Callback inválido: conferir protocolo, porta, host e path /auth/callback.
- Client inválido: conferir tipo Regular Web Application e credenciais.
- SDK não inicializa: conferir variáveis v4 e AUTH0_SECRET.
- Login em loop: cookies, base URL, proxy/middleware e host.
- Preview falha: callback/logout URLs e allowlist.
- Logout não conclui: Allowed Logout URLs e rota /auth/logout.

Referências: https://auth0.github.io/nextjs-auth0/ e https://github.com/auth0/nextjs-auth0/blob/main/V4_MIGRATION_GUIDE.md
