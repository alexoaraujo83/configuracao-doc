# 06 — Validação de dados e tratamento de erros

Validar no browser para UX, no Route Handler/Server Action para segurança, no service para invariantes e no PostgreSQL para constraints. Client-side nunca substitui validação server-side. Zod é opção; confirmar versão/compatibilidade.

~~~ts
import { z } from "zod";
export const createProjectSchema = z.object({
  name: z.string().trim().min(1).max(120),
  description: z.string().trim().max(2000).optional(),
}).strict();
~~~

strict rejeita propriedades não declaradas. Nunca aceitar tenant_id, created_by, role, status administrativo ou auth0_sub em payload público. Validar UUID, limites, enums, paginação, ordenação por allowlist, redirects e uploads.

Route Handler converte erros conhecidos em HTTP; service lança erros de domínio; repository traduz falhas PostgreSQL sem expor detalhes. Logs usam correlation ID e códigos internos; nunca tokens, cookies, SQL com parâmetros sensíveis ou DATABASE_URL.

| Condição | HTTP | Código |
|---|---:|---|
| Não autenticado | 401 | UNAUTHENTICATED |
| Sem permissão | 403 | FORBIDDEN |
| Não encontrado/oculto | 404 | NOT_FOUND |
| Payload inválido | 400/422 | VALIDATION_FAILED |
| Conflito | 409 | CONFLICT |
| Payload grande | 413 | PAYLOAD_TOO_LARGE |
| Limite excedido | 429 | RATE_LIMITED |
| Falha inesperada | 500 | INTERNAL_ERROR |
| Dependência indisponível | 503 | DEPENDENCY_UNAVAILABLE |

Escolher 400 ou 422 consistentemente. Não revelar existência de registros de outro tenant. Respostas de erro não incluem stack trace, SQL, nomes internos de tabelas, hostname, connection string ou mensagem bruta do driver.

Retry apenas para falhas transitórias reconhecidas; não repetir operação não idempotente sem chave idempotente. Timeout após commit pode deixar resultado desconhecido. Não converter qualquer exceção Auth0 em 401. Em Server Components, não capturar redirect/notFound do Next.js e convertê-los em 500; propagar controle de fluxo.

Testar schema, campos extras, payload grande, Unicode, UUID inválido, constraint, indisponibilidade, timeout e ausência de vazamento.
