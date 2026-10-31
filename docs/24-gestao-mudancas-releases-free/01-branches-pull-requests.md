# Estratégia de branches e Pull Requests

## Modelo recomendado

Usar trunk-based development simplificado: main representa a linha integrada; branches curtas partem de main e retornam por Pull Request. Evitar branches de longa duração e GitFlow completo em equipes pequenas.

| Tipo | Convenção | Uso |
|---|---|---|
| Feature | feature/<ticket>-<slug> | funcionalidade |
| Fix | fix/<ticket>-<slug> | correção |
| Docs | docs/<slug> | documentação |
| Refactor | refactor/<slug> | mudança interna |
| Chore | chore/<slug> | manutenção |
| Hotfix | hotfix/<ticket>-<slug> | correção urgente |

Usar minúsculas, hífens, nomes curtos e sem dados sensíveis.

## Ciclo de PR

1. Abrir Issue com problema, resultado esperado e critérios de aceite.
2. Criar branch a partir de main atualizada.
3. Implementar uma mudança coesa e commits pequenos.
4. Atualizar testes, documentação, migrations e changelog quando aplicável.
5. Abrir PR com contexto, impacto, risco, validação e rollback.
6. Aguardar CI e revisão humana.
7. Resolver comentários e atualizar branch sem apagar histórico.
8. Fazer merge pela estratégia definida (squash ou merge commit).
9. Confirmar deployment e smoke tests após merge.
10. Encerrar Issue e registrar release quando aplicável.

## Template mínimo de PR

~~~markdown
## Contexto
Issue:
Problema:

## Mudança
O que muda:
Contratos/API/schema afetados:

## Validação
- [ ] lint
- [ ] typecheck
- [ ] testes
- [ ] build
- [ ] migration revisada
- [ ] secrets revisados

## Operação
Risco:
Plano de rollout:
Rollback:
Evidência pós-deploy:
~~~

## Proteção da main no GitHub Free

Protected branches estão disponíveis no GitHub Free para repositórios públicos. Para este repositório público, avaliar:
- exigir PR antes de merge;
- exigir status checks disponíveis e nomeados;
- exigir resolução de conversas;
- bloquear force-push e exclusão;
- exigir linear history se a estratégia escolhida suportar;
- exigir aprovação quando houver mais de um revisor disponível.

Limitações:
- não presumir que required reviewers seja útil com um único colaborador;
- confirmar no painel a disponibilidade exata das regras e rulesets;
- para repositórios privados Free, não afirmar que todas as proteções estão disponíveis;
- branch protection não substitui revisão de código nem controle de acesso.

## Política de commits

Recomenda-se Conventional Commits:
- feat: funcionalidade;
- fix: correção;
- docs: documentação;
- refactor: refatoração;
- test: testes;
- build/ci: pipeline e build;
- chore: manutenção.

Exemplos: feat(auth): validate membership; fix(db): close transaction on error; docs(release): document rollback.

## Regras de merge

PR não deve conter mudanças não relacionadas. Não aprovar PR próprio como revisão independente. CI verde não prova ausência de vulnerabilidade. Alterações em auth, RLS, roles, migrations e secrets exigem revisão de segurança. Não fazer merge automático como parte desta política documental.

## Referências

- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches
- https://docs.github.com/en/pull-requests/collaborating-with-pull-requests
- https://www.conventionalcommits.org/en/v1.0.0/
