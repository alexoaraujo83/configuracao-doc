# 03 — Next.js: APIs, Server Actions e CSRF

## Padrão de execução

- Server Components por padrão.
- Client Components somente para interação; não recebem secrets nem fazem SQL direto.
- Route Handlers e Server Actions são superfícies públicas: validar sessão, autorização, schema e origem em cada operação.
- Centralizar acesso a dados em módulos server-only.
- Adicionar import server-only aos módulos de acesso a segredos e banco quando compatível com a estrutura.

## Proteção de rotas

Middleware/proxy é uma primeira barreira de navegação, não autorização final. A autorização deve estar próxima da operação que lê ou altera o recurso, inclusive em funções chamadas por páginas, handlers e actions.

Exemplo conceitual:

~~~typescript
import { auth0 } from "@/lib/auth0";
import { NextResponse } from "next/server";

export async function GET() {
  const session = await auth0.getSession();
  if (!session) {
    return NextResponse.json({ error: "unauthenticated" }, { status: 401 });
  }

  // Consultar serviço que verifica membership e permissão.
  return NextResponse.json({ data: "authorized response" }, {
    headers: { "Cache-Control": "private, no-store" }
  });
}
~~~

O exemplo não implementa autorização completa; não copiar como endpoint final sem service layer, validação e tratamento de erros.

## CSRF

Cookies são enviados automaticamente pelo navegador; por isso, operações mutáveis precisam de defesa CSRF apropriada ao modelo de sessão.

- Manter SameSite seguro configurado pelo SDK; não tratar SameSite como controle único.
- Em endpoints mutáveis que usam cookie, validar Origin contra allowlist exata de origins.
- Se Origin estiver ausente em contexto onde isso é esperado, definir política explícita; não aceitar qualquer origem.
- Usar token CSRF sincronizado/double-submit conforme arquitetura quando a operação ou cliente não puder depender apenas das proteções do framework.
- Não habilitar CORS com wildcard e credentials.
- Não usar GET para mutação.
- Validar Content-Type e schema do payload.
- Para webhooks, validar assinatura e timestamp do provedor; CSRF não substitui autenticação de webhook.

Server Actions também devem validar autorização e entrada. A proteção de origem do framework não substitui a autorização do usuário nem validação de negócio.

## CORS

Para API consumida pelo próprio Next.js same-origin, não habilitar CORS sem necessidade. Se houver consumidor externo:
- Allowlist exata de origins.
- Métodos e headers mínimos.
- Credentials somente se indispensável.
- Nunca combinar wildcard origin com credentials.
- Separar autenticação bearer de sessão cookie.

## Cache

Para qualquer resposta que leia sessão, contenha dado pessoal ou dependa de autorização:
- Cache-Control: private, no-store.
- Não usar revalidate/cache público.
- Não compartilhar cache por URL quando a resposta varia por usuário/tenant.
- Revisar cache de fetch, Route Handler, CDN e componentes.

## Validação e erros

- Validar com schema (ex.: Zod) e rejeitar campos desconhecidos quando adequado.
- Limitar tamanho do body, arrays, paginação e filtros.
- Usar allowlist para nomes de coluna/ordenação; parâmetros SQL protegem valores, não identificadores.
- Respostas 401/403/404/409/422/429/503 sem stack trace ou segredo.
- Não distinguir existência de recurso protegido se isso permitir enumeração; considerar 404 conforme política.

## Referências

- https://nextjs.org/docs/app/guides/authentication
- https://nextjs.org/docs/app/guides/data-security
- https://nextjs.org/docs/app/building-your-application/routing/route-handlers
- https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
