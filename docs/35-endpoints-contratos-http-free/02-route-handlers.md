# 02 — Route Handlers

## Localização e responsabilidades

App Router define handlers em route.ts dentro do segmento. GET, POST, PATCH e DELETE são funções exportadas. Route Handler é fronteira HTTP: extrai parâmetros, resolve sessão, valida schema, chama service e serializa DTO. Não implementar SQL ou autorização apenas na UI.

~~~ts
// src/app/api/v1/tenants/[tenantId]/projects/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function GET(
  request: NextRequest,
  context: { params: Promise<{ tenantId: string }> },
) {
  const { tenantId } = await context.params;
  const identity = await getCurrentIdentity();
  if (!identity) {
    return NextResponse.json(
      { error: { code: "UNAUTHENTICATED", message: "Autenticação necessária" } },
      { status: 401 },
    );
  }

  const query = listProjectsQuerySchema.safeParse(
    Object.fromEntries(request.nextUrl.searchParams),
  );
  if (!query.success) {
    return validationError(query.error);
  }

  const result = await projectService.list({
    identity, tenantId, query: query.data,
  });
  return NextResponse.json(result, {
    headers: { "Cache-Control": "private, no-store" },
  });
}
~~~

Este exemplo é ilustrativo. Assinatura de params deve corresponder à versão do Next.js instalada; versões recentes tipam params como Promise. Centralizar conversão de erros em wrapper compartilhado para evitar respostas inconsistentes.

## Mutação

~~~ts
export async function POST(
  request: Request,
  context: { params: Promise<{ tenantId: string }> },
) {
  const { tenantId } = await context.params;
  const identity = await getCurrentIdentity();
  if (!identity) return apiError(401, "UNAUTHENTICATED");

  const raw = await readJsonBody(request);
  const parsed = createProjectSchema.safeParse(raw);
  if (!parsed.success) return validationError(parsed.error);

  const project = await projectService.create({
    identity, tenantId, input: parsed.data,
  });
  return NextResponse.json(
    { data: toProjectDto(project) },
    {
      status: 201,
      headers: {
        Location: `/api/v1/tenants/${tenantId}/projects/${project.id}`,
        "Cache-Control": "private, no-store",
      },
    },
  );
}
~~~

## Regras

- Validar UUID e formato dos path params.
- Limitar tamanho do body antes de parsear; não confiar apenas em Content-Length.
- Rejeitar content type inesperado.
- Verificar AuthN, status local, membership e permission no service.
- SQL parametrizado e transação; DTO de saída explícito.
- Não retornar objeto Error nem rows diretamente.
- Para endpoints mutáveis baseados em cookie, aplicar proteção CSRF conforme orientação do Next.js/Auth0; não desabilitar verificações de origem.
- Não adicionar CORS permissivo por padrão.
- Para DELETE, exigir permissão e verificar tenant/resource na mesma operação protegida.

## Runtime

Escolher runtime Node.js quando driver/pool ou dependência exigir Node. Declarar runtime explicitamente se necessário e validar compatibilidade. Não mover credenciais ou código DB para Client Components.
