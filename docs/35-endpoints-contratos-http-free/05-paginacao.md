# 05 — Paginação, filtros e ordenação

## Cursor pagination

Para tabelas que crescem, preferir cursor estável a OFFSET. Ordenação recomendada: created_at DESC, id DESC; usar ambos para desempate determinístico.

SQL conceitual:

~~~sql
SELECT id, name, description, created_at
FROM project
WHERE tenant_id = $1
  AND (created_at, id) < ($2::timestamptz, $3::uuid)
ORDER BY created_at DESC, id DESC
LIMIT $4;
~~~

O cursor representa o último par (created_at,id). Buscar limit + 1 registros para calcular hasMore; emitir nextCursor somente se houver próxima página. Validar limit como inteiro entre 1 e 100, com default 25.

## Cursor opaco

Codificar cursor como payload versionado e assinado por HMAC no servidor, ou usar mecanismo equivalente; nunca confiar em tenantId, filtros ou SQL codificados pelo cliente. Vincular cursor à ordenação/filtros e ao tenant solicitado; rejeitar cursor inválido, expirado ou incompatível. Não incluir PII.

~~~ts
type CursorPayload = {
  v: 1;
  createdAt: string;
  id: string;
  tenantId: string;
  filterHash: string;
};
~~~

A chave de assinatura é segredo server-only, exclusiva por ambiente e rotacionável. Alternativamente, cursor pode ser opaco armazenado em servidor, considerando custo operacional.

## Query contract

GET /api/v1/tenants/{tenantId}/projects?limit=25&cursor=...

Filtros devem ser allowlisted e validados (por exemplo status, prefixo de nome se definido). Escape de curingas LIKE quando aplicável. Não concatenar ordenação recebida; mapear enum de sort para expressão SQL fixa.

## Consistência

Cursor reduz problemas de deslocamento causados por inserções antes do offset, mas não cria snapshot consistente entre requisições. Documentar comportamento quando registros mudam durante navegação. Para exportações consistentes, definir fluxo separado e limites operacionais.

## Testes

Primeira página, última página, cursor adulterado, cursor de outro tenant, filtro alterado, limite zero/negativo/alto, registros com mesmo timestamp, remoção entre páginas e ausência de duplicação inesperada.
