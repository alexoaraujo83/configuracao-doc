# 06 — Erros HTTP, segurança e observabilidade

## Mapeamento

| Situação | Status | Código |
|---|---:|---|
| Sessão ausente/expirada | 401 | UNAUTHENTICATED |
| Sem permissão | 403 | FORBIDDEN |
| Recurso inexistente ou oculto | 404 | NOT_FOUND |
| JSON/schema inválido | 400 | VALIDATION_FAILED |
| Conflito de estado/unique | 409 | CONFLICT |
| Body excede limite | 413 | PAYLOAD_TOO_LARGE |
| Rate limit | 429 | RATE_LIMITED |
| Dependência temporariamente indisponível | 503 | DEPENDENCY_UNAVAILABLE |
| Falha não classificada | 500 | INTERNAL_ERROR |

Adotar regra consistente para 403 versus 404 em recursos de outros tenants. Recomenda-se 404 quando revelar a existência do recurso constituir vazamento.

## Cabeçalhos

- Cache-Control: private, no-store para sessão e dados de tenant.
- X-Request-Id: identificador de correlação gerado/confiável pelo servidor.
- Retry-After: somente quando houver política de retry definida.
- Não enviar Access-Control-Allow-Origin wildcard com cookies.

## CSRF, CORS e redirects

Cookie de sessão exige atenção a CSRF em mutações. Manter verificações de origem do Next.js/Auth0 e não criar bypass customizado. CORS deve ser fechado por padrão; se necessário, allowlist explícita e credenciais configuradas corretamente. returnTo e URLs de callback/logout devem ser allowlisted para evitar open redirect.

## Rate limiting

Definir limites por usuário e/ou IP em endpoints sensíveis (login auxiliar, convites, mutações caras). Não presumir que plano Free fornece rate limiting gerenciado suficiente. Se não houver storage/serviço de rate limit confiável, documentar a limitação e aplicar limites de aplicação com cuidado; não usar memória efêmera como garantia distribuída.

## Logging

Registrar requestId, route, método, status, duração, código interno e identificador pseudonimizado quando necessário. Redigir Authorization, cookies, tokens, e-mails e body sensível. Nunca logar DATABASE_URL, SQL parametrizado com valores sensíveis ou stack trace em resposta pública.

## Erros de banco

Traduzir constraint conhecida para conflito/validação quando apropriado. Timeout e falha de rede não significam necessariamente rollback se a conexão caiu após commit. Operações não idempotentes precisam de chave idempotente ou consulta de reconciliação antes de retry.
