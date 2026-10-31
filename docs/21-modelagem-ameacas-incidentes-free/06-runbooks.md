# Runbooks por cenário

Procedimentos documentais para operador autorizado. Cada execução requer registro, aprovador, horário UTC, escopo e validação. Nenhuma ação abaixo foi executada.

## RB-01 — Auth0 client secret ou token comprometido
1. Identificar client/tenant, ambiente, exposição e aplicações consumidoras sem copiar segredo ao ticket.
2. Preservar eventos/configuração disponíveis.
3. Preparar rotação do secret pelo mecanismo suportado; coordenar atualização server-side em todos os ambientes.
4. Invalidar secret antigo e confirmar qual valor permanece válido.
5. Avaliar refresh tokens/grants afetados e revogar pelo client emissor. Revogação pode invalidar outros tokens do mesmo grant.
6. Encerrar sessões da aplicação/Auth0 quando necessário e disponível. JWT access tokens já emitidos geralmente não são revogáveis e permanecem válidos até expirar.
7. Revisar callbacks, origins, audience, scopes, Actions e administradores.
8. Publicar deployment com configuração nova; testar login, logout, callback e autorização.
9. Monitorar uso de credencial antiga.

## RB-02 — Neon DATABASE_URL, role ou API key exposta
1. Identificar projeto, branch, role, tipo de credencial, consumers e privilégios.
2. Preservar metadados/logs disponíveis e inventário de roles/grants.
3. Para senha: reset imediato (cutover simultâneo) ou role paralela com privilégios mínimos para migração gradual; confirmar escopo por branch.
4. Atualizar Vercel env, GitHub Actions secrets, hosts e arquivos locais sem expor valores em tickets.
5. Criar novo deployment/restart para carregar variável atualizada.
6. Invalidar senha antiga; conexões já abertas podem continuar ativas.
7. Revogar API key exposta e emitir outra; revisar projetos/branches e keys desconhecidas.
8. Revisar roles com login, ownership, BYPASSRLS, grants e conexões.
9. Testar runtime, negar operações administrativas e executar testes RLS cross-tenant.
10. Monitorar erros de autenticação e conexões não migradas.

## RB-03 — GitHub token, repositório ou Actions comprometidos
1. Identificar token/ator, permissões, repos, workflows, commits, PRs e horário.
2. Preservar audit log e metadados; procurar eventos do token e atores/IPs desconhecidos quando disponíveis.
3. Revogar token; revisar PATs, deploy keys, GitHub Apps e permissões GITHUB_TOKEN.
4. Inspecionar commits, branches, tags, rulesets, workflows, Actions e artifacts.
5. Verificar exfiltração por logs, cache, artifacts, outputs ou PR comments; invalidar secrets acessíveis.
6. Reverter alterações maliciosas por commit revisado; apagar linha de secret não invalida credencial exposta.
7. Pin Actions a SHA revisado, restringir permissions, remover shell interpolation insegura e bloquear secrets para código não confiável.
8. Executar CI limpo, revisão independente e deploy de commit confiável.
9. Monitorar chaves, webhooks, deploy keys, workflows e acessos novos.

## RB-04 — Vercel deployment, env, domínio ou conta comprometidos
1. Identificar projeto/team, ambiente, deployment, commit, domínio, variável e horário.
2. Preservar deployment IDs, autor, commit e logs disponíveis.
3. Revogar tokens/acessos Vercel comprometidos; revisar membros, integrações, domínio e DNS.
4. Rotacionar credenciais nos serviços emissores; atualizar variáveis Vercel em todos ambientes.
5. Criar deployment novo: env atualizada não altera deployments anteriores.
6. Inspecionar build output, logs, NEXT_PUBLIC_, redirects, rewrites, cron e dependências.
7. Hobby permite rollback apenas ao deployment de produção imediatamente anterior; verificar se código/configuração não reintroduz risco. Preferir redeploy de commit confiável quando necessário.
8. Validar domínio, TLS, callbacks Auth0, API e smoke tests.
9. Monitorar deployments e tráfego. Standard Protection Hobby não protege domínio de produção.

## RB-05 — Cross-tenant ou vazamento de dados
1. Classificar provisoriamente SEV-1; identificar tabelas, tenants, usuários, intervalo e tipos de dados.
2. Suspender endpoint/ação/feature vulnerável com aprovação.
3. Preservar logs, query metadata, request IDs, commits e deployments.
4. Revogar sessões/credenciais comprometidas quando aplicável; JWT emitido pode continuar válido até expirar.
5. Reproduzir em ambiente controlado com fixtures sintéticas.
6. Corrigir DAL, authorization, policy RLS, contexto transacional, grants ou query.
7. Executar testes cross-tenant para leitura, escrita, delete, export, busca e jobs.
8. Avaliar integridade, escopo, titulares e obrigações de notificação.
9. Reabrir somente após revisão independente e gate aprovado.

## RB-06 — Suspeita de SQL injection
1. Preservar request IDs, path, parâmetros redigidos e UTC; não executar payload em produção.
2. Identificar endpoint, query, tabela, role e tenant context.
3. Restringir feature se exploração plausível.
4. Revisar concatenação SQL, raw queries, filtros/ordenação dinâmicos e parametrizar valores.
5. Rotacionar runtime credential se houver possibilidade de extração; revisar grants e dados acessíveis.
6. Avaliar exfiltração/alteração, RLS e atividade anômala.
7. Testar em ambiente controlado com fixtures e casos de bypass.
8. Deploy corrigido, regressão e monitoramento.

## Validação comum
Credencial antiga inválida onde aplicável; consumers atualizados; runtime sem privilégios administrativos; testes Auth/RBAC/RLS aprovados; logs sem segredo; persistência investigada; timeline, escopo, evidências e risco residual registrados.
