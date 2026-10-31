# Auth0 + Next.js — autenticação e autorização

> Revisão documental: 31/10/2026. Referência do SDK: `@auth0/nextjs-auth0` v4.33.0. Confirmar a versão instalada antes de copiar exemplos. Este guia não comprova configuração efetiva do tenant.

## 1. Arquitetura

O Auth0 é o Identity Provider (IdP). Next.js atua como Backend for Frontend (BFF), mantém a sessão em cookies seguros e executa operações protegidas no servidor. Neon armazena o perfil e as permissões internas da aplicação.

```text
Browser
  │ HTTPS + cookie HttpOnly
  ▼
Next.js App Router / BFF
  ├── Auth0 SDK v4 ── OIDC ──► Auth0 Universal Login
  └── Neon Driver ── TLS/HTTPS ──► Neon PostgreSQL
```

Princípios:

- Senhas são tratadas pelo Auth0, não armazenadas no Neon.
- Sessão autentica a pessoa; autorização é verificada pelo backend.
- O browser nunca recebe Client Secret, Auth0 Secret ou DATABASE_URL.
- A aplicação usa `iss + sub` como identidade externa estável; não usa email como chave primária.
- Organizations e permissões de negócio podem ser modeladas no PostgreSQL, pois RBAC por Organization do Auth0 não está incluído no Free.

## 2. Compatibilidade Free

Segundo a tabela oficial Auth0 consultada em 31/10/2026:

| Recurso | Free | Decisão |
|---|---|---|
| External Active Users | até 25.000 MAU | monitorar uso mensal |
| Organizations | até 5 | não desenhar mais de cinco |
| Machine-to-Machine Authentication | até 1.000 autenticações/tokens por mês | usar somente se necessário, com cache/reutilização |
| M2M Add-on | não incluído | não pressupor ampliação |
| Auth0 Database Connection | incluída | usar Username-Password-Authentication quando aplicável |
| Custom Database Connections | não incluídas | não depender de banco externo como conexão Auth0 |
| MFA OTP/Duo | Essentials | NÃO DISPONÍVEL NO FREE |
| RBAC por Organization | Essentials | NÃO DISPONÍVEL NO FREE |
| Custom Domain | exige verificação de cartão | baseline usa domínio padrão do tenant |

Passwordless, passkeys e conexões sociais aparecem na tabela Free. Não equiparar automaticamente passwordless a MFA; documentar o método e o fator efetivamente exigido.

Fonte: [Auth0 Pricing](https://auth0.com/pt/pricing). Os limites podem mudar; revisar antes da implantação.

## 3. Criar e configurar a aplicação Auth0

No Auth0 Dashboard:

1. Selecionar o tenant destinado ao ambiente.
2. Abrir Applications → Applications.
3. Criar uma aplicação do tipo **Regular Web Application**.
4. Nomear com padrão `<produto>-<ambiente>-web`.
5. Registrar callback e logout URLs exatos.
6. Obter Domain, Client ID e Client Secret por canal seguro.
7. Não adicionar Google OAuth2 sem decisão formal; requisito atual é não adicionar esse provedor.
8. Usar a Database Connection integrada `Username-Password-Authentication`, quando login local do Auth0 for o método escolhido.

Configuração local:

| Campo | Valor |
|---|---|
| Allowed Callback URLs | `http://localhost:3000/auth/callback` |
| Allowed Logout URLs | `http://localhost:3000` |
| Allowed Web Origins | preencher somente quando o fluxo/SDK exigir; não liberar curingas |

Para produção, substituir por origem HTTPS estável aprovada. Não usar curingas amplos nem cadastrar domínio de preview indiscriminadamente. A callback URL deve corresponder exatamente ao host autorizado.

## 4. Configurar SDK v4

Instalar (procedimento para execução posterior):

```bash
npm install @auth0/nextjs-auth0
```

Variáveis:

```dotenv
AUTH0_DOMAIN=tenant-region.auth0.com
AUTH0_CLIENT_ID=
AUTH0_CLIENT_SECRET=
AUTH0_SECRET=
APP_BASE_URL=http://localhost:3000
```

Gerar segredo local:

```bash
openssl rand -hex 32
```

Criar `src/lib/auth0.ts`:

```typescript
import "server-only";
import { Auth0Client } from "@auth0/nextjs-auth0/server";

export const auth0 = new Auth0Client();
```

A inicialização padrão lê `AUTH0_DOMAIN`, `AUTH0_CLIENT_ID`, `AUTH0_CLIENT_SECRET` e `AUTH0_SECRET`. `APP_BASE_URL` é opcional: pode ser fixada para uma origem estável ou inferida pelo SDK a partir da requisição. Em produção com host dinâmico, a lista de Allowed Callback URLs no Auth0 é a barreira de segurança; não confiar cegamente no header Host.

## 5. Interceptação de requisições por versão Next.js

**Next.js 16:** usar `src/proxy.ts`:

```typescript
import type { NextRequest } from "next/server";
import { auth0 } from "@/lib/auth0";

export async function proxy(request: NextRequest) {
  return await auth0.middleware(request);
}

export const config = {
  matcher: [
    "/((?!_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)",
  ],
};
```

**Next.js 15:** usar `src/middleware.ts` com o mesmo handler exportado como `middleware`:

```typescript
import type { NextRequest } from "next/server";
import { auth0 } from "@/lib/auth0";

export async function middleware(request: NextRequest) {
  return await auth0.middleware(request);
}

export const config = {
  matcher: [
    "/((?!_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)",
  ],
};
```

Não criar ambos os arquivos sem uma razão de migração. A convenção `proxy.ts` substitui `middleware.ts` no Next.js 16. O matcher amplo permite ao SDK executar tratamento de sessão e endpoints de autenticação. Ajustar somente após avaliar rolling sessions e as rotas excluídas.

## 6. Login, sessão e logout

Página inicial Server Component:

```tsx
import Link from "next/link";
import { auth0 } from "@/lib/auth0";

export default async function HomePage() {
  const session = await auth0.getSession();

  return (
    <main>
      <h1>Sistema Web</h1>
      {session ? (
        <>
          <p>Olá, {session.user.name ?? "usuário"}.</p>
          <Link href="/dashboard">Acessar painel</Link>
          <a href="/auth/logout">Sair</a>
        </>
      ) : (
        <a href="/auth/login">Entrar</a>
      )}
    </main>
  );
}
```

A SDK v4 monta rotas de autenticação sob `/auth` por padrão. Não implementar callback/login manualmente sem necessidade. Para evitar iniciar login por prefetch total, preferir um link de navegação simples ou `prefetch={false}` conforme a documentação do SDK.

Página privada:

```tsx
import { redirect } from "next/navigation";
import { auth0 } from "@/lib/auth0";

export default async function DashboardPage() {
  const session = await auth0.getSession();

  if (!session) {
    redirect("/auth/login?returnTo=/dashboard");
  }

  return (
    <main>
      <h1>Painel privado</h1>
      <p>Identidade: {session.user.sub}</p>
    </main>
  );
}
```

Regras:

- Verificar `getSession()` em cada página/handler que exige autenticação.
- Não usar middleware/proxy como único controle de autorização.
- Tratar sessão ausente/expirada como estado normal.
- Nunca imprimir objeto completo de sessão ou tokens nos logs.
- Não passar access token para Client Components.

## 7. Access Token e API própria

Quando Next.js chamar uma API protegida, configurar no Auth0 uma API com Identifier (audience) estável e solicitar scopes mínimos. A aplicação obtém o token no servidor pelo SDK; o token deve ser destinado à API e validado no serviço receptor.

Validar no backend receptor:

1. Assinatura e algoritmo permitido via JWKS.
2. `iss` esperado.
3. `aud` igual ao identificador da API.
4. `exp` e `nbf`.
5. `scope`/permissions exigidos.
6. Regras de negócio e membership atual no banco.

ID Token serve à autenticação do cliente OIDC; não usar como bearer token de API. A sessão Auth0 também não substitui validação de autorização por recurso.

## 8. Requisito de validade de token

Manter como requisito do tenant/API:

- **Maximum Access Token Lifetime = 900 segundos.**
- **Allow Skipping User Consent = desabilitado.**

Esses valores são decisões de configuração solicitadas, não evidência de que foram salvos. Confirmar no Dashboard antes de marcar como verificado. Não confundir a duração do Access Token com a duração da sessão web/cookie.

## 9. M2M dentro da quota Free

O Free inclui até 1.000 autenticações/tokens M2M por mês; M2M Add-on não está incluído. Para fluxos Client Credentials:

- Solicitar token quando necessário, não em cada chamada.
- Reutilizar o token até próximo da expiração, considerando margem de renovação.
- Guardar o token somente em memória/cache server-side com TTL menor que `exp`.
- Não armazenar em localStorage, cookie de navegador ou log.
- Monitorar o consumo mensal e definir limite operacional abaixo da quota.
- Ao atingir o limite, falhar de forma segura e registrar incidente de quota; não criar fluxo alternativo inseguro.

Não usar ID Token, sessão de usuário compartilhada ou segredo estático no frontend como substituto de M2M.

## 10. Autorização Free com PostgreSQL

Como RBAC por Organization do Auth0 não está no Free, armazenar papéis e memberships internos no PostgreSQL:

- `users`: identidade externa `iss + sub`.
- `organizations`: tenants internos.
- `memberships`: relação usuário/tenant.
- `roles` e `permissions`: catálogo de autorização do produto.
- `resource.owner_user_id` ou `tenant_id`: vínculo de propriedade.

A autorização deve acontecer no servidor em toda operação. Não aceitar `role`, `tenant_id`, `user_id` ou `auth0_sub` como prova de identidade quando vierem do browser. Obter identidade da sessão e verificar membership no banco.

Passkeys/passwordless podem melhorar autenticação, mas não concedem autorização nem substituem controles de acesso a dados.

## 11. Sincronização de usuário

Usar `session.user.sub` como chave externa imutável. Email pode mudar e não é chave de identidade. Se o negócio exigir email verificado, exigir claim `email_verified === true`; presença de email não significa verificação.

Exemplo de upsert para executar somente em serviço server-side autenticado:

```typescript
import "server-only";
import { sql } from "@/lib/db";

type Auth0Profile = {
  sub: string;
  email?: string;
  email_verified?: boolean;
  name?: string;
};

export async function synchronizeUser(profile: Auth0Profile) {
  if (!profile.sub) throw new Error("Missing subject");
  if (!profile.email || profile.email_verified !== true) {
    throw new Error("Verified email required");
  }

  const rows = await sql`
    INSERT INTO app_users (auth0_sub, email, display_name)
    VALUES (${profile.sub}, ${profile.email}, ${profile.name ?? null})
    ON CONFLICT (auth0_sub)
    DO UPDATE SET
      email = EXCLUDED.email,
      display_name = EXCLUDED.display_name,
      updated_at = now()
    RETURNING id, auth0_sub, email, display_name
  `;

  return rows[0] ?? null;
}
```

A função acima não deve ser exposta diretamente como endpoint público sem autenticação, proteção CSRF/origin, validação e rate limit. Prefira executar a sincronização a partir de fluxo server-side controlado após callback ou ação autenticada.

## 12. Cookies, sessão e segurança

Usar os defaults seguros do SDK e não customizar cookies sem análise. Em produção:

- HTTPS obrigatório.
- Cookies HttpOnly e Secure.
- SameSite compatível com o fluxo OIDC.
- Segredo de sessão distinto por ambiente.
- Callback/logout URLs restritas.
- Rotação do Client Secret após suspeita de exposição.
- Não serializar tokens em props de componentes cliente.
- Não armazenar dados pessoais desnecessários na sessão.
- Tratar redirecionamentos `returnTo` com allowlist/paths relativos para evitar open redirect.

## 13. Testes

| Teste | Resultado esperado |
|---|---|
| Acessar login | redirecionamento ao Universal Login |
| Callback válido | sessão criada |
| Callback com redirect não autorizado | rejeitado pelo Auth0 |
| Acessar dashboard sem sessão | redirecionamento para login |
| Acessar dashboard com sessão | conteúdo privado disponível |
| Logout | sessão local encerrada e fluxo de logout concluído |
| Sessão expirada | acesso negado/renovação conforme SDK |
| Access Token com audience errada | API retorna 401 |
| Token expirado ou assinatura inválida | API retorna 401 |
| Usuário sem membership | API retorna 403 |
| M2M quota excedida | operação falha de forma segura |

## 14. Troubleshooting

| Sintoma | Verificar |
|---|---|
| `Callback URL mismatch` | callback exata, scheme, host e path |
| `Invalid state` | cookies, domínio, proxy, múltiplas transações |
| Loop de login | matcher, base URL, cookie e redirects |
| Sessão sempre nula | segredo, domain, client id e cookie Secure |
| 431 Request Header Fields Too Large | tamanho de cookie e fluxo de login; não ampliar cabeçalhos sem diagnóstico |
| Token sem audience | configuração de authorization parameters e API Identifier |
| 403 após login | autenticação não equivale a membership/permission |
| Login falha em Preview | host dinâmico não registrado; restringir a preview estável ou desabilitar login |

## 15. Fontes oficiais

- [Auth0 Next.js SDK v4.33.0](https://auth0.github.io/nextjs-auth0/)
- [Auth0 Next.js Quickstart](https://auth0.com/docs/quickstart/webapp/nextjs)
- [Auth0 Pricing](https://auth0.com/pt/pricing)
- [Auth0 OIDC](https://auth0.com/docs/authenticate/protocols/openid-connect-protocol)
- [Auth0 Access Tokens](https://auth0.com/docs/secure/tokens/access-tokens)
- [Next.js Proxy](https://nextjs.org/docs/app/api-reference/file-conventions/proxy)
