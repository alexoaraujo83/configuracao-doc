# Compatibilidade de versões e rollback

## Matriz de compatibilidade

Toda release avalia:
- Next.js e Node suportados;
- lockfile e package manager;
- Auth0 SDK, sessão e callbacks;
- schema PostgreSQL antes/depois da migration;
- contratos API consumidos por frontend/clients;
- variáveis requeridas;
- versão de migration e compatibilidade com código anterior.

Manter versões fixadas no package.json/lockfile e revisar changelogs oficiais antes de upgrades major. Evitar combinar upgrade de framework, SDK de autenticação e mudança destrutiva de schema na mesma release.

## Compatibilidade durante rollout

| Camada | Compatibilidade desejada |
|---|---|
| Código | versão nova tolera schema anterior durante rollout |
| Banco | schema expandido suporta versão atual e anterior |
| Auth0 | callbacks/claims/sessões coexistem durante transição |
| Configuração | variáveis novas têm validação/fallback antes de remover antigas |
| Frontend | cliente não depende imediatamente de API removida |

## Rollback de aplicação

1. Confirmar impacto e deployment afetado.
2. Identificar último deployment de produção saudável.
3. No Vercel Hobby, rollback de produção é limitado ao deployment imediatamente anterior.
4. Após rollback, validar rotas, autenticação, leitura/escrita e logs.
5. Conferir estado de auto-assignment/domínios; rollback pode desativar atribuição automática até undo/promote.
6. Registrar commit, deployment IDs, horário, decisão e evidência.

Rollback de deployment não desfaz migration SQL, mudança Auth0, alteração de env vars nem efeitos externos.

## Rollback de banco

Preferir forward-fix, feature flag já existente, compatibilidade expand-contract e migration compensatória. Não assumir que migration down é segura. Se dados foram removidos/transformados, avaliar recuperação via backup/PITR e impacto com responsável.

## Rollback Auth0

Exportar configuração não é backup integral do tenant nem inclui necessariamente usuários/segredos. Registrar mudança anterior, valores não secretos, diff revisado e procedimento de reversão. Alterações em produção exigem aprovação e validação manual.

## Checklist pós-rollback

- [ ] deployment correto ativo;
- [ ] smoke tests concluídos;
- [ ] login/logout e callback validados;
- [ ] RLS e isolamento multi-tenant validados;
- [ ] migrations compatíveis com código restaurado;
- [ ] jobs/cron e integrações revisados;
- [ ] causa-raiz registrada;
- [ ] auto-deploy/branch tracking verificado.

## Referências

- https://vercel.com/docs/deployments/rollback-production-deployment
- https://vercel.com/docs/instant-rollback
- https://vercel.com/docs/cli/rollback
- https://neon.com/docs/introduction/branching
