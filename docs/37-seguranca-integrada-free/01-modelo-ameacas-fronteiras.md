# 01 — Modelo de ameaças e fronteiras de confiança

## Ativos

- Identidade Auth0: issuer, subject, claims e tokens.
- Cookie de sessão web e chave de criptografia.
- Segredos OAuth: client secret e AUTH0_SECRET.
- DATABASE_URL e credenciais de roles PostgreSQL.
- Dados pessoais, registros e relações tenant/user.
- Código, workflows, artifacts e tokens de automação GitHub.
- Variáveis e deployments Vercel.

## Fronteiras

| Fronteira | Dados que cruzam | Controle requerido |
|---|---|---|
| Browser → Next.js | Cookies, parâmetros e payloads | HTTPS, validação, CSRF, limites |
| Next.js → Auth0 | OAuth/OIDC | SDK oficial, redirect allowlist |
| Next.js → Neon | SQL e contexto | TLS, role mínima, parâmetros, transação |
| GitHub → CI | Código e eventos PR | permissões mínimas, isolamento de forks |
| Vercel Preview → serviços | Configuração de preview | credenciais e dados isolados |

## Ameaças e controles

| Ameaça | Impacto | Controle primário |
|---|---|---|
| Session theft / XSS | Uso indevido da sessão | HttpOnly, CSP, escaping, dependências |
| CSRF | Ação involuntária com cookie | SameSite, validação Origin e token quando necessário |
| IDOR/BOLA | Acesso a recurso de outro usuário | Autorização por recurso no servidor |
| Tenant escape | Leitura/escrita entre tenants | membership + RLS + contexto transacional |
| SQL injection | Leitura/alteração indevida | SQL parametrizado e allowlists para identificadores |
| Open redirect | Redirecionamento malicioso | destinos relativos/allowlist |
| Secret leakage | Comprometimento de contas/DB | secret store, logs redigidos, rotação |
| CI supply-chain | Execução de código hostil com secrets | SHA pinning, least privilege, não executar fork com privilégio |
| Cache leakage | Resposta de usuário servida a outro | private, no-store e sem cache compartilhado |
| Abuse/rate exhaustion | indisponibilidade ou esgotamento de cota | limites por identidade/IP e quotas |

## Suposições explícitas

- O browser e todos os dados recebidos dele são não confiáveis.
- O tenant_id enviado na URL ou body é apenas um seletor; não concede acesso.
- RLS não protege contra um backend comprometido que possa escolher livremente qualquer contexto de tenant. A autorização server-side continua obrigatória.
- GUCs como app.tenant_id definidos pelo backend ajudam a aplicar isolamento, mas não são uma fronteira independente contra SQL arbitrário executado pela mesma role.
- Vercel Hobby é para uso pessoal/não comercial; não utilizar para SaaS comercial.
- Limites e elegibilidade dos recursos Free precisam ser confirmados antes de cada implantação.

## Critério de aceite

Para cada endpoint protegido deve existir uma descrição de: ator, recurso, tenant, permissão, dados de entrada, validação, transação, policy RLS, resposta e trilha de auditoria.
