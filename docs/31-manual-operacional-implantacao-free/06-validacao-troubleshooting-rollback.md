# 6. Validar, diagnosticar e reverter

## Princípio
Este capítulo define verificações futuras. Nenhum teste, migration, deploy, restore ou rollback foi executado ao criar esta documentação.

## Gate pré-execução
- [ ] Finalidade compatível com Vercel Hobby.
- [ ] Planos e quotas Free conferidos.
- [ ] Versões Node, Next.js e Auth0 SDK registradas.
- [ ] Callback/logout URLs exatas.
- [ ] Secrets distintos por ambiente.
- [ ] DATABASE_URL isolada por ambiente.
- [ ] Runtime sem DDL, owner ou BYPASSRLS.
- [ ] Migrations revisadas e reversão planejada.
- [ ] CI sem secrets de produção.
- [ ] PR/deployment submetidos a revisão humana.

## Validação local autorizada
~~~bash
npm ci
npm run lint
npx tsc --noEmit
npm run build
npm run dev
~~~

Confirmar scripts no package.json. Verificar página pública, login/callback/logout, sessão server-side, rota protegida, autorização, isolamento de tenant e health check sem dados internos.

## Diagnóstico seguro
| Camada | Registrar | Nunca registrar |
|---|---|---|
| Next.js | status e etapa | dump de env |
| Auth0 | código de erro e request ID | tokens, cookies, Client Secret |
| Neon | SQLSTATE e correlation ID | URL de conexão e dados pessoais |
| GitHub Actions | job e etapa | secrets em echo |
| Vercel | deployment ID e etapa | valores de variáveis |

## Critérios de parada
Interromper quando recurso necessário não estiver no Free; callback não estiver autorizado; preview apontar para produção; CI exigir secret de produção sem justificativa; grants/RLS não forem revisados; migration destrutiva não tiver recuperação; build/testes falharem; ou finalidade conflitar com Hobby.

## Rollback da aplicação
1. Identificar deployment ativo e versão estável anterior.
2. Confirmar compatibilidade da versão anterior com schema atual.
3. Revisar migrations desde a versão estável.
4. Conferir mecanismo de rollback disponível no plano atual.
5. Executar somente após aprovação explícita.
6. Validar smoke tests e registrar resultado.

Rollback de deployment não reverte dados nem schema. Preferir migrations expand/contract.

## Recuperação do banco
1. Interromper mutações se houver risco de corrupção.
2. Identificar branch/database e janela afetada.
3. Conferir recursos Free de branching/PITR disponíveis.
4. Restaurar em branch separada e validar.
5. Comparar schema/dados.
6. Promover somente após aprovação e estratégia de backup.
7. Registrar RPO/RTO observados; não inventar garantias.

## Rotação de segredos
Gerar segredo novo, atualizar escopo correto, criar novo deployment, validar, revogar o anterior e registrar sem reproduzir valor. Não imprimir segredos.

## Encerramento de preview
Antes de excluir branch ou deployment, confirmar PR encerrado, ausência de necessidade de retenção, branch não ser produção, impacto de quota e aprovação humana.

## Registro pós-implantação
Registrar ambiente, commit SHA, versões, deployment ID, branch Neon, migration, data, responsável, resultado e pendências. Não registrar tokens, secrets ou PII.
