# 02 — Auth0 com Next.js App Router

**Estado:** Documentado; configuração no tenant não executada.  
**SDK de referência:** `@auth0/nextjs-auth0` v4.  
**Plano:** Auth0 Free, respeitando cotas e recursos habilitados na conta.

## 1. Pré-requisitos e decisões

- Aplicação Next.js criada conforme o manual 01.
- Tenant Auth0 Free existente ou a ser criado pelo responsável.
- Criar uma aplicação do tipo **Regular Web Application**.
- Usar a conexão de banco `Username-Password-Authentication` quando o fluxo de senha for requerido pelo projeto.
- Não configurar Google OAuth2 nesta arquitetura.
- Manter **Allow Skipping User Consent** desabilitado.
- Limitar **Maximum Access Token Lifetime** a 900 segundos conforme a política deste projeto.
- Não habilitar funcionalidades que exijam plano pago, trial, cartão ou add-on. Domínio padrão do tenant é suficiente para a primeira versão.

A presença de um item no Dashboard não comprova disponibilidade no plano Free. Verificar a elegibilidade no tenant e registrar limites antes de depender do recurso.

## 2. Criar a aplicação

No Auth0 Dashboard:
1. Abrir Applications e criar uma aplicação.
2. Selecionar Regular Web Application.
3. Registrar nome e descrição sem dados sensíveis.
4. Copiar Domain, Client ID e Client Secret para o cofre local/deploy, nunca para o repositório.
5. Configurar URLs por ambiente.

Development:
- Callback: `http://localhost:3000/auth/callback`
- Logout: `http://localhost:3000`
- Web Origins: `http://localhost:3000`

O SDK v4 monta rotas em `/auth/*`, não no caminho legado `/api/auth/*`. Não misturar configuração de callback de SDK v3 com SDK v4.

Para Production, adicionar o callback e logout exatos do domínio final. Para Preview, preferir uma aplicação Auth0 separada ou uma lista explícita e controlada de URLs. Não liberar curingas amplos para domínios de Preview.

## 3. Configuração do SDK v4

Instalar:

```bash
npm install @auth0/nextjs-auth0
```

Criar `src/lib/auth0.ts`:

```typescript
import "server-only";
import { Auth0Client } from "@auth0/nextjs-auth0/server";

export const auth0 = new Auth0Client();
```

A configuração padrão lê variáveis de ambiente do SDK, incluindo `AUTH0_DOMAIN`, `AUTH0_CLIENT_ID`, `AUTH0_CLIENT_SECRET`, `AUTH0_SECRET` e `APP_BASE_URL`. Caso a aplicação adote configuração explícita no construtor, manter os nomes e propriedades compatíveis com a versão efetivamente instalada. Não reutilizar variáveis legadas v3 sem confirmar a migração.

Gerar segredo independente por ambiente:

```bash
node -e "console.log(require('node:crypto').randomBytes(32).toString('hex'))"
```

## 4. Proxy conforme a versão do Next.js

Next.js 16: criar `src/proxy.ts`:

```typescript
import { auth0 } from "@/lib/auth0";

export async function proxy(request: Request) {
  return auth0.middleware(request);
}

export const config = {
  matcher: [
    "/((?!_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)",
  ],
};
```

Next.js 15: usar `src/middleware.ts` e exportar a função `middleware`:

```typescript
import type { NextRequest } from "next/server";
import { auth0 } from "@/lib/auth0";

export async function middleware(request: NextRequest) {
  return auth0.middleware(request);
}

export const config = {
  matcher: [
    "/((?!_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)",
  ],
};
```

A partir do Next.js 16, Middleware foi renomeado/depreciado em favor de Proxy. Não misturar nomes de arquivo e export entre versões. O matcher amplo permite ao SDK processar endpoints de autenticação e gerenciamento de sessão; não significa que todas as rotas estejam protegidas.

## 5. Rotas de autenticação e sessão

Com SDK v4 e middleware/proxy montado, as rotas são gerenciadas pelo SDK, incluindo:
- `/auth/login`
- `/auth/logout`
- `/auth/callback`
- `/auth/profile`

Página privada de exemplo:

```tsx
import { redirect } from "next/navigation";
import { auth0 } from "@/lib/auth0";

export default async function DashboardPage() {
  const session = await auth0.getSession();

  if (!session) {
    redirect("/auth/login?returnTo=/dashboard");
  }

  return <main><h1>Área autenticada</h1><p>{session.user.name}</p></main>;
}
```

Route Handler deve verificar sessão novamente:

```typescript
import { NextResponse } from "next/server";
import { auth0 } from "@/lib/auth0";

export async function GET() {
  const session = await auth0.getSession();

  if (!session) {
    return NextResponse.json({ error: "Não autenticado" }, { status: 401 });
  }

  return NextResponse.json({
    user: {
      sub: session.user.sub,
      name: session.user.name,
    },
  });
}
```

Não retornar o objeto de sessão completo, tokens, claims desnecessárias ou dados pessoais sem necessidade.

## 6. Autenticação não é autorização

O Proxy do SDK monta endpoints e mantém o ciclo de sessão, mas não protege páginas ou APIs por padrão. Em Next.js 16, Proxy é executado no runtime Node.js; use-o para verificações otimistas/redirecionamentos, nunca como fronteira exclusiva de autorização. Em cada Server Component que exponha dados, Route Handler, Server Action e operação de banco:
1. Obter `await auth0.getSession()` no servidor e rejeitar `null`.
2. Validar entrada e identificar o recurso solicitado.
3. Consultar membership/estado atual no banco e verificar a permissão exigida.
4. Executar a operação com escopo de tenant/recurso aplicado na própria consulta/transação.
5. Retornar 401 para ausência de autenticação e 403 (ou 404 quando a política de não divulgação exigir) para falta de autorização.

Server Actions são invocáveis por requisições diretas; não presumir que a action só pode ser chamada pela UI. Repetir a autorização dentro de cada action, inclusive quando a página já verificou a sessão. Não usar valores `sub`, `tenant_id`, `role` ou `permissions` recebidos no body/query como identidade/autoridade. `sub` vem da sessão validada; o tenant solicitado é apenas um seletor e deve ser confirmado por membership ativa.

Nunca aceitar `sub`, role ou permission como prova quando enviado pelo navegador. A identidade deve vir da sessão validada pelo SDK. Para autorização baseada em token de API, validar assinatura, issuer, audience, expiração e scopes no serviço destinatário.

## 7. Tokens e política do projeto

- ID Token identifica a autenticação para o cliente OIDC; não usar como access token de API.
- Access Token destina-se ao recurso/audience correspondente.
- Não persistir tokens em localStorage.
- Não registrar tokens nos logs.
- Solicitar somente scopes necessários.
- Política do projeto: Maximum Access Token Lifetime = 900 segundos.
- Refresh Token só deve ser habilitado quando o caso de uso justificar e o plano/configuração suportar com segurança; documentar rotação, revogação e armazenamento antes de ativar.

## 8. Validações

| Teste | Resultado esperado |
|---|---|
| Abrir `/auth/login` | Redirecionamento ao Universal Login |
| Login bem-sucedido | Retorno ao callback cadastrado e sessão criada |
| Callback não cadastrado | Falha de redirect URI, sem relaxar validações |
| Abrir dashboard sem sessão | Redirecionamento para login |
| Chamar API privada sem sessão | 401 |
| Usuário autenticado sem permissão | 403 |
| Logout | Sessão local encerrada e fluxo do provedor concluído |
| Cookie em produção | Secure, HttpOnly e política SameSite compatível |
| Inspecionar bundle do navegador | Nenhum Client Secret ou AUTH0_SECRET |

## 9. Troubleshooting

- **Callback 404:** conferir que o domínio de callback é o domínio da aplicação, não o domínio do tenant; confirmar rota SDK v4 e URL exata.
- **Invalid state/nonce:** revisar cookies, domínio, HTTPS, callback e configurações de ambiente; não desabilitar validações OIDC.
- **Login em loop:** verificar se Proxy/Middleware está montado no caminho correto e se o matcher não exclui rotas `/auth`.
- **Sessão inválida após deploy:** confirmar segredo consistente para o mesmo ambiente e não compartilhado com outros ambientes.
- **Preview não autentica:** cadastrar URL exata no Auth0 e revisar `APP_BASE_URL`; não abrir callback wildcard.

## 10. Referências

- [Auth0 Next.js SDK v4 API](https://auth0.github.io/nextjs-auth0/)
- [Guia de migração v4](https://github.com/auth0/nextjs-auth0/blob/main/V4_MIGRATION_GUIDE.md)
- [Quickstart Auth0 Next.js](https://auth0.com/docs/quickstart/webapp/nextjs)
- [Next.js Proxy (v16)](https://nextjs.org/docs/app/api-reference/file-conventions/proxy)
- [OAuth 2.0 e OIDC — Auth0 Docs](https://auth0.com/docs)

