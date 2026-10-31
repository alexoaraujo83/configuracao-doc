# 03 — Server Actions

## Uso apropriado

Server Actions são funções assíncronas executadas no servidor, úteis para mutações iniciadas pela UI Next.js. Não são endpoints REST públicos nem substituem Route Handlers para consumidores externos, webhooks ou contratos independentes do frontend.

~~~ts
// src/app/tenants/[tenantId]/projects/actions.ts
"use server";

export async function createProjectAction(
  tenantId: string,
  formData: FormData,
) {
  const identity = await getCurrentIdentity();
  if (!identity) return { ok: false, error: { code: "UNAUTHENTICATED" } };

  const parsed = createProjectSchema.safeParse({
    name: formData.get("name"),
    description: formData.get("description") || undefined,
  });
  if (!parsed.success) {
    return {
      ok: false,
      error: {
        code: "VALIDATION_FAILED",
        fields: parsed.error.flatten().fieldErrors,
      },
    };
  }

  try {
    const project = await projectService.create({
      identity, tenantId, input: parsed.data,
    });
    return { ok: true, data: toProjectDto(project) };
  } catch (error) {
    return mapDomainErrorToActionResult(error);
  }
}
~~~

Exemplo conceitual; validar serialização e tipos de retorno na versão do Next.js escolhida.

## Segurança obrigatória

- Tratar cada Server Action como endpoint público alcançável por request; executar AuthN/AuthZ dentro da action.
- Não confiar em tenantId, IDs ou campos escondidos do formulário.
- Validar e normalizar FormData no servidor.
- Não retornar secrets, stack trace ou dados de outros tenants.
- Não exportar helper comum como action pública sem necessidade.
- Preservar proteção de origem/CSRF do framework; configurar allowed origins somente com necessidade explícita.
- Revalidar membership/permissão no momento da mutação.

## UI e estado

Consumir resultado discriminado (ok/data ou ok/error) e apresentar mensagens sem detalhes internos. useActionState pode ligar action e estado da interface; não substitui validação nem autorização. Após sucesso, revalidar caminho/tag no servidor quando apropriado.

## Quando preferir Route Handler

Integração com aplicações externas, clientes mobile, API versionada, contratos OpenAPI, webhooks e operações que precisam de semântica HTTP explícita. Não duplicar regras de negócio: ambos devem chamar o mesmo service.
