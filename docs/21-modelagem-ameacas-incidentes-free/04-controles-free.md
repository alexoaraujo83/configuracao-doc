# Controles preventivos, detectivos e corretivos — Free-only

| Domínio | Preventivo | Detectivo | Corretivo | Limite a confirmar |
|---|---|---|---|---|
| Next.js | DAL server-only, validação, cookies seguros, autorização por operação | logs sem PII/segredo, métricas, testes | bloquear feature, corrigir e redeploy | não pressupor WAF/SIEM pago |
| Auth0 | OIDC SDK, callback allowlist, audience, scopes mínimos | eventos/logs disponíveis, revisão de apps | rotacionar secret, revogar refresh grants | JWT access token emitido geralmente só expira; validar logs/quotas Free |
| Neon | TLS, role runtime mínima, parâmetros, RLS + FORCE | logs disponíveis, revisar roles/conexões | reset role password, role paralela, revogar API key | confirmar backups, restore, logs e quotas Free |
| GitHub | permissions mínimo, Actions pinadas SHA, PR review, lockfile | alertas elegíveis, histórico/audit log disponível | revogar token, corrigir workflow e secrets | recursos variam por repo/plano |
| Vercel | env por ambiente, segredo fora de NEXT_PUBLIC_, deploy Git | deployments e logs disponíveis | atualizar env, redeploy ou rollback permitido | Hobby pessoal/não comercial; rollback limitado |
| Pessoas | contas individuais, MFA quando disponível, aprovação dupla para ações críticas | revisão de acesso e eventos | revogar acesso e credenciais | não presumir governança premium |

## Controles técnicos obrigatórios
- Nunca enviar AUTH0_CLIENT_SECRET, DATABASE_URL, PAT ou tokens para browser, NEXT_PUBLIC_, commits, issues, artifacts ou logs.
- Nunca usar owner/migrator/superuser no request path. Runtime sem ownership, BYPASSRLS ou DDL.
- Parametrizar SQL; RLS não corrige SQL injection.
- Validar JWT: assinatura, issuer, audience, expiração e algoritmo; validar scopes e membership no domínio.
- Autorização server-side em toda DAL, Server Action e Route Handler; deny by default.
- Ações mutáveis validam origem/CSRF conforme sessão/framework.
- Actions GitHub pinadas a SHA revisado; permissions mínimos; não interpolar input não confiável em shell; não expor secrets a fork não confiável.
- Separar secrets por ambiente/projeto. Mudança de env Vercel exige deployment novo; deployments antigos podem reter valores antigos.
- Limitar tamanho, frequência, concorrência, paginação e timeout.
- Logs: timestamp UTC, request ID, ator pseudonimizado, ação/resultado/objeto, sem token, senha, cookie, connection string ou payload desnecessário.

## Revisão periódica
1. Revisar membros, papéis, tokens, apps Auth0 e permissões GitHub.
2. Conferir workflows, Actions, dependências e lockfile.
3. Conferir envs por ambiente, deployments e domínios.
4. Conferir roles, grants, RLS e policies em tabelas sensíveis.
5. Executar testes negativos de autorização, RLS e segredo.
6. Conferir limites Free antes de release relevante.
7. Registrar evidência, data, responsável, resultado e exceções.

Não pressupor SIEM, retenção estendida, auditoria avançada, proteção de domínio de produção, password protection paga, private deployments, GitHub Environments em repo privado Free, suporte premium, HA ou PITR se não estiver explicitamente incluído.
