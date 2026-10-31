# 5. Erros, observabilidade e privacidade

## Taxonomia

| Falha | Resposta externa | Tratamento |
|---|---|---|
| Sessão ausente/expirada | login ou 401 | SDK oficial |
| Sem permissão | 403 genérico | registrar evento mínimo |
| Entrada inválida | 400/422 | validar no servidor |
| Conflito | 409 | tratar idempotência |
| Neon indisponível | 503 genérico | timeout e política segura |
| Rate limit | 429/503 conforme origem | backoff com jitter |
| Build/deploy inválido | não publicar | revisar logs sem secrets |

Não expor stack trace, SQL, nomes internos, tokens ou configuração na resposta ao usuário.

## Mapeamento de erro

~~~ts
type PublicErrorCode =
  | "UNAUTHENTICATED"
  | "FORBIDDEN"
  | "INVALID_INPUT"
  | "CONFLICT"
  | "TEMPORARILY_UNAVAILABLE";

export function toHttpStatus(code: PublicErrorCode): number {
  switch (code) {
    case "UNAUTHENTICATED": return 401;
    case "FORBIDDEN": return 403;
    case "INVALID_INPUT": return 400;
    case "CONFLICT": return 409;
    case "TEMPORARILY_UNAVAILABLE": return 503;
  }
}
~~~

Não converter exceção desconhecida em sucesso. Não repetir operações mutáveis sem idempotência.

## Logs estruturados
Campos mínimos possíveis: timestamp, nível, ambiente, operação lógica, código de erro, correlation ID aleatório, duração e status HTTP.

Nunca registrar tokens, cookies, headers de autorização, AUTH0_CLIENT_SECRET, AUTH0_SECRET, DATABASE_URL, payload completo ou dados pessoais sem necessidade demonstrada.

## Privacidade
- Definir finalidade e necessidade de cada dado.
- Minimizar perfil armazenado.
- Estabelecer retenção, exclusão e desativação.
- Restringir acesso a logs e dados.
- Usar dados sintéticos em CI e preview.
- Documentar solicitações de acesso, correção e exclusão.
- Não afirmar conformidade legal apenas pela adoção destes controles.

## Resiliência
Aplicar timeouts. Retry somente para falhas transitórias e operações idempotentes, com backoff exponencial e jitter. Não repetir login/callback ou mutações cegamente. Indisponibilidade de Auth0 ou Neon nunca deve causar bypass de autenticação ou autorização. Usar somente recursos de observabilidade gratuitos efetivamente disponíveis; não depender de add-ons pagos.
