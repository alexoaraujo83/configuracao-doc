# 6. Quality gates e critérios de bloqueio de release

## 6.1 Princípio

Um gate é uma condição objetiva que precisa ser satisfeita antes de avançar. Um workflow verde não substitui revisão humana, aprovação de risco, verificação de plano Free ou evidência de ambiente.

## 6.2 Gates propostos

| Gate | Condição mínima | Falha |
|---|---|---|
| G0 — escopo | requisitos, impacto, plano Free e ambiente identificados | bloquear |
| G1 — integridade do código | npm ci, lint, typecheck e build passam | bloquear |
| G2 — testes | unitários, integração e E2E críticos passam | bloquear |
| G3 — autorização | testes Auth0/RBAC/RLS sem violação cross-tenant | bloquear |
| G4 — dependências | npm audit, advisories e Actions triados; sem achado bloqueador | bloquear |
| G5 — banco | migrations reproduzíveis; constraints, roles e RLS aprovadas | bloquear |
| G6 — operação | logs/redaction, rollback/forward-fix, backup e monitoramento documentados | bloquear |
| G7 — produção | host permitido pelos termos Free, env/config verificados e smoke tests aprovados | bloquear |
| G8 — evidências | checklist completo, responsável e aprovador identificados | bloquear |

## 6.3 Bloqueadores absolutos

- teste crítico falhando ou flaky sem triagem;
- acesso não autorizado, fuga cross-tenant ou RLS bypass pela role runtime;
- segredo exposto em Git, logs, artifacts ou browser;
- CVE crítica confirmada sem correção/mitigação validada;
- CVE alta com caminho alcançável ou risco não aceito;
- migration destrutiva sem plano e evidência de recuperação;
- migration não testada, drift de schema ou checksum divergente;
- falta de backup/restore compatível com requisito de negócio;
- build ou typecheck falhando;
- variáveis obrigatórias ausentes/inconsistentes;
- hospedagem incompatível com finalidade ou termos do plano Free;
- tentativa de usar secrets/Environments/branch protection que não estejam disponíveis no plano/repositório;
- ausência de responsável pela release ou evidência mínima.

## 6.4 Exceções controladas

Exceção não pode dispensar teste de isolamento multi-tenant, segredo exposto, migration destrutiva sem recuperação ou violação dos termos do fornecedor. Para outros itens, documentar risco, impacto, mitigação, responsável, aprovador, prazo e expiração. Em repositório privado GitHub Free, aprovação manual deve ser processo externo documentado; não alegar que CI impõe required reviewers se o plano não oferece esse controle.

## 6.5 Exemplo de pipeline

Ordem sugerida:
1. checkout;
2. setup Node com versão fixada;
3. npm ci;
4. lint;
5. typecheck;
6. unit;
7. integração com banco efêmero;
8. Playwright E2E em build local;
9. npm audit;
10. build;
11. upload de relatório não sensível somente se necessário.

Em PR/fork, não expor secrets. A etapa de integração pode usar PostgreSQL local descartável sem credenciais de produção. Branch Neon efêmera é opcional e condicionada à quota Free.

## 6.6 Cuidado com required checks

- O CI pode reportar status, mas a disponibilidade de enforcement de branch depende do plano e da visibilidade do repositório.
- GitHub Free não deve ser tratado como oferecendo proteção de branch para repositórios privados.
- Em repositório público, branch protection pode estar disponível; avaliar exposição de código e dados antes de tornar público.
- Não publicar código confidencial para obter gate gratuito.
- Em repo privado sem enforcement, reduzir escritores, exigir revisão por procedimento, registrar checklist e restringir merge manualmente; documentar que isso é controle processual, não enforcement server-side.

## 6.7 Critério GO/NO-GO

**GO** somente quando todos os gates aplicáveis estiverem Aprovados, exceções válidas e hospedagem permitida. **NO-GO** quando qualquer bloqueador estiver aberto, evidência ausente ou requisito Free não atendido.
