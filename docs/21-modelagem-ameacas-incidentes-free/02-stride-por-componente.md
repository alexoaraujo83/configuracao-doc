# STRIDE por componente

STRIDE: Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service e Elevation of Privilege. Avaliar cenário, ativo, fronteira, impacto, controle e teste. Baseline para adaptação, não auditoria do ambiente real.

| Componente/fluxo | STRIDE | Cenário | Controles e testes mínimos |
|---|---|---|---|
| Browser → Next.js | S, T | cookie roubado, CSRF, input adulterado | cookies Secure/HttpOnly/SameSite, proteção CSRF/origin, validação server-side; testar sessão/CSRF |
| Next.js ↔ Auth0 | S, R | callback falso, state/nonce ausente, sessão sequestrada | SDK oficial, callback allowlist, state/nonce/PKCE, logs sem tokens; testes OIDC negativos |
| Auth0 client/API | S, I, E | client secret exposto, audience/scope incorreto | segredo server-only, audience explícita, validação JWT, scopes mínimos; token inválido deve falhar |
| DAL / Actions / Handlers | T, I, E | IDOR, mass assignment, bypass de autorização | autorização em cada operação, DTO allowlist, validação de membership; testes cross-user/tenant |
| Next.js → Neon | T, I, E | SQL injection, role privilegiada, tenant context alterado | queries parametrizadas, role mínima, transação única para set_config + queries, RLS FORCE |
| Neon / RLS | I, T, E | policy permissiva, tabela sem policy, owner ignora RLS | inventário de tabelas, FORCE RLS, grants mínimos; testes sem contexto/tenant errado |
| GitHub repo/PR | T, R, I | commit malicioso, branch protection fraca, segredo em histórico | revisão independente, proteção disponível, secret scanning quando elegível; fixture de segredo |
| GitHub Actions | S, T, I, E | Action comprometida, workflow injection, token amplo | pin SHA revisado, permissions mínimo, shell seguro, fork sem secrets; lint workflow |
| GitHub token | S, I, E | PAT/GITHUB_TOKEN roubado | escopo mínimo, expiração quando disponível, revogação, audit log quando disponível |
| Vercel build/deploy | T, I, E | dependência comprometida, env exposta, deploy malicioso | env por ambiente, nenhuma NEXT_PUBLIC_ secreta, revisar autor/commit; smoke test |
| Domínio/DNS | S, T, I | sequestro DNS, callback adulterado | controle de registrador/DNS, MFA, callback allowlist; validar DNS/TLS |
| Logs/evidências | R, I, T | exclusão/alteração, PII ou token em log | acesso restrito, timestamp UTC, hash de export, redaction; teste de redaction |
| Disponibilidade/quota | D | flood, abuso de login/API, esgotamento de recursos | rate limit, limite por usuário, timeout, paginação; teste de carga controlado |

## Abuse cases
- Alterar tenant_id, user_id, role, owner_id ou is_admin no body/URL.
- Reenviar Server Action, webhook ou operação idempotente em concorrência.
- Scraping, enumeração, credential stuffing ou abuso de recursos por conta legítima.
- PR de fork tenta exfiltrar secrets por logs, artifacts, cache ou rede.
- SSRF tenta alcançar IP privado ou metadata; bloquear destinos internos, validar redirects e URLs.
- Dependência npm, Action ou script de build introduz código malicioso.
- Atacante remove logs, altera workflow, cria token persistente ou muda DNS.
- DoS/quota é confundido com incidente de segurança; classificar e preservar evidências.

Revisar no início do projeto, antes de dados multi-tenant, após mudança de fronteira/privilégio, após incidente e em periodicidade definida.
