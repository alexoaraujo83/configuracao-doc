# 04 — Payloads e respostas

## DTO de projeto

~~~ts
export type ProjectDto = {
  id: string;
  name: string;
  description: string | null;
  createdAt: string;
  updatedAt: string;
};
~~~

Não serializar campos internos como tenant_id, auth0_sub, created_by, flags administrativas ou dados de auditoria sem finalidade explícita. Datas UTC ISO 8601. Campos opcionais e nullable devem ter semântica definida.

## Criar projeto

Request POST /api/v1/tenants/{tenantId}/projects:

~~~json
{
  "name": "Projeto de exemplo",
  "description": "Descrição opcional"
}
~~~

201 Created:

~~~json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000001",
    "name": "Projeto de exemplo",
    "description": "Descrição opcional",
    "createdAt": "2026-10-31T12:00:00.000Z",
    "updatedAt": "2026-10-31T12:00:00.000Z"
  }
}
~~~

Os UUIDs e timestamps acima são dados fictícios de exemplo.

## Atualizar

PATCH recebe somente campos alteráveis:

~~~json
{ "name": "Nome atualizado" }
~~~

Rejeitar body vazio, propriedades desconhecidas e campos controlados pelo servidor. Definir se null remove descrição; não confundir ausência de campo com null.

## Envelope de sucesso

~~~json
{ "data": {} }
~~~

Listagens:

~~~json
{
  "data": [],
  "page": {
    "limit": 25,
    "nextCursor": null,
    "hasMore": false
  }
}
~~~

## Envelope de erro

~~~json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Revise os campos enviados.",
    "requestId": "req_example",
    "details": [
      { "field": "name", "issue": "Obrigatório" }
    ]
  }
}
~~~

Nunca incluir stack trace, SQL, parâmetros, cookies, tokens, connection string ou detalhes que permitam inferir dados de outro tenant. Mensagens públicas devem ser estáveis e localizáveis.
