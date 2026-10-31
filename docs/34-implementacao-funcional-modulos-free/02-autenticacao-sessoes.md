# 02 — Login, logout e sessões Auth0

Adotar @auth0/nextjs-auth0 v4, sem misturar API legada v3. A documentação v4.33 informa Node.js 20 LTS ou mais recente, aplicação Regular Web Application e variáveis AUTH0_DOMAIN, AUTH0_CLIENT_ID, AUTH0_CLIENT_SECRET e AUTH0_SECRET. APP_BASE_URL pode ser definida para URL estável.

~~~ts
import { Auth0Client } from "@auth0/nextjs-auth0/server";
export const auth0 = new Auth0Client();
~~~

Registrar callback local http://localhost:3000/auth/callback e logout URL http://localhost:3000, além dos hosts reais. Não aceitar redirects arbitrários.

Next.js 15: middleware.ts com NextRequest. Next.js 16: proxy.ts com Request. Respeitar diretório src/ e matcher recomendado pelo SDK.

~~~ts
import { auth0 } from "./src/lib/auth0";
export async function proxy(request: Request) {
  return auth0.middleware(request);
}
export const config = {
  matcher: ["/((?!_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)"],
};
~~~

O SDK monta /auth/login, /auth/logout e /auth/callback. Cadastro pode usar /auth/login?screen_hint=signup quando conexão permite auto cadastro. Evitar prefetch completo da rota de login.

~~~ts
import { auth0 } from "@/lib/auth0";
export async function getCurrentIdentity() {
  const session = await auth0.getSession();
  if (!session?.user?.sub) return null;
  return {
    sub: session.user.sub,
    email: session.user.email,
    name: session.user.name,
  };
}
~~~

Tratar sessão ausente/expirada como anônima. Middleware não substitui autorização por recurso. Logout encerra sessão da aplicação e inicia fluxo configurado; não presumir que encerra sessões de outros apps.

AUTH0_SECRET deve ser forte, exclusivo por ambiente e nunca versionado. Manter cookies HttpOnly, Secure em HTTPS e SameSite adequado. Não armazenar tokens em localStorage nem expor tokens/client secret ao browser. returnTo deve ser caminho local ou allowlist. Preview exige callbacks/logout URLs controladas.

Testar callback válido, state/nonce inválido, expiração, logout, redirect externo e cookie não acessível por JavaScript. Free: não depender de custom domains, Organizations, MFA avançado ou add-ons sem confirmar elegibilidade; Universal Login padrão é baseline.
