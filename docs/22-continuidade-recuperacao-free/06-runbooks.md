# Runbooks integrados de recuperação

Procedimentos para operador autorizado. Registrar incidente, escopo, owner, aprovador, UTC, impacto e rollback antes de qualquer ação. Nenhuma ação foi executada.

## DR-01 — Neon perdido/corrompido
Declarar incidente; congelar gravações se corrupção continuar; identificar PITR (janela Free documentada: 6h) e último dump válido; criar branch isolada; comparar schema/dados/roles/RLS/migrations; calcular RPO e reconciliar transações posteriores; validar integridade e smoke tests; aprovar cutover; atualizar DATABASE_URL e redeploy; monitorar e preservar branch original.

## DR-02 — Auth0 inacessível
Proteger email/admin identity; determinar se tenant existe ou se é problema de client secret; usar fluxo oficial; não criar tenant alternativo automaticamente; aplicar export revisado sem secrets/deletions; se novo tenant aprovado, recriar clients/APIs/connections, reconhecendo que usuários/password hashes podem não ser recuperáveis; atualizar issuer/audience/callbacks/env; testar login, logout, refresh, RBAC e membership; comunicar eventual reset.

## DR-03 — GitHub indisponível
Distinguir conta/repo/branch/Actions; recuperar 2FA/email; revogar tokens comprometidos; validar clone com git fsck; inspecionar histórico antes de republicar código; recriar permissões/Actions mínimas; reconectar Vercel após confirmar commit; executar CI/Preview antes de produção.

## DR-04 — Vercel/deployment indisponível
Distinguir build/runtime/env/domínio/quota/conta; preservar deployment ID/commit/logs/UTC; se atual quebrado e anterior elegível, avaliar rollback Hobby ao anterior imediato; se env/secret mudou, criar deployment novo; se rollback indisponível, commit revert revisado e redeploy; confirmar auto-assignment/domínio; validar callbacks Auth0, Neon, RLS e rotas críticas.

## DR-05 — Domínio/registrador perdido
Recuperar email/conta; verificar titularidade/status; comparar DNS com último export; restaurar registros autorizados; validar DNS/TLS; atualizar Vercel e callbacks Auth0 após propagação; testar redirects, cookies e login. Novo domínio exige aprovação e revisão de segurança.

## DR-06 — Perda total de conta
Identificar owner/conta/projetos; usar recuperação oficial; não contornar controles nem reutilizar credenciais suspeitas; recuperar email/IdP do owner, depois GitHub/Auth0/Neon/Vercel/registrador conforme dependências; recriar recursos somente com autorização; validar issuer/audience/callback/DATABASE_URL/DNS/commit; registrar perdas irrecuperáveis.

## Encerramento
Serviço mínimo restabelecido; integridade e isolamento validados; RTO/RPO reais registrados; credenciais revisadas; evidências preservadas; impacto/comunicação avaliados; risco residual aceito.
