# Aprovações, rastreabilidade e auditoria

## Registro de mudança

Cada Issue/PR/release registra:
- identificador, solicitante e responsável;
- objetivo e justificativa;
- sistemas e ambientes afetados;
- classificação de risco;
- dados e controles de segurança afetados;
- arquivos/migrations/nomes de variáveis alterados (nunca valores secretos);
- evidências CI e revisão;
- aprovação e timestamp;
- commit SHA, tag e deployment;
- migration IDs/checksums;
- plano de rollback e resultado pós-release.

## Modelo de risco

| Nível | Exemplos | Controles |
|---|---|---|
| Baixo | docs, ajuste visual isolado | CI aplicável e revisão |
| Médio | endpoint compatível, dependência, query | testes, revisão técnica, Preview |
| Alto | auth, RBAC/RLS, schema, roles, secrets | revisão de segurança, migration isolada, rollback e aprovação explícita |
| Crítico | alteração destrutiva, credencial comprometida, exposição de dados | resposta a incidente e autorização responsável |

## Aprovações no Free

- Repositório público no GitHub Free pode usar proteção de branch, status checks e revisão conforme recursos atuais.
- Em repositório privado Free, não presumir protected branches, environment secrets ou deployment protection.
- GitHub Free oferece Environment secrets apenas em repositórios públicos; para privados são necessários planos elegíveis.
- Quando não houver aprovação técnica de ambiente no plano, usar PR review e checklist manual, sem inserir secrets em workflow.
- Aprovação registrada em comentário/PR é evidência operacional, não barreira técnica inviolável.

## Segregação de funções

Quando possível, autor, revisor e executor devem ser pessoas diferentes em mudanças de alto risco. Em projeto individual, registrar auto-revisão e controles compensatórios: CI, checklist, diff review, espera antes da release e evidência de smoke test. Não alegar revisão independente.

## Evidência auditável

Preservar Issue/PR, comentários, checks, logs de workflow, commit SHA, tag, deployment URL/ID, migration ID/checksum, smoke tests e decisões de rollback/incidente.

Não arquivar secrets, cookies, tokens, dados pessoais ou logs brutos desnecessários. Definir retenção proporcional e acesso restrito.

## Mudança emergencial

1. Registrar incidente e justificativa.
2. Limitar escopo à mitigação necessária.
3. Obter autorização do responsável, salvo delegação formal prévia.
4. Criar PR tão cedo quanto possível, inclusive após mitigação.
5. Registrar evidência, risco residual e revisão retrospectiva.
6. Abrir follow-up para correção definitiva.

## Referências

- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches
- https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments
- https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication
