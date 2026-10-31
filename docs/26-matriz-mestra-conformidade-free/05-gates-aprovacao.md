# Quality gates e critérios finais de aprovação

## Estados de decisão

- **GO:** todos os requisitos críticos aplicáveis estão conformes, evidências vigentes verificadas e nenhum bloqueio aberto.
- **GO condicionado:** somente pendências não críticas, com risco aceito formalmente, responsável, prazo e plano de tratamento. Nunca usar para contornar requisito contratual ou controle crítico.
- **NO-GO:** qualquer requisito crítico não conforme, evidência crítica ausente/expirada, vulnerabilidade crítica/alta sem tratamento aprovado, incompatibilidade com plano Free, exposição de segredo, falha de autorização/RLS, migration sem recuperação validada ou falha de integridade.

## Gates por fase

| Gate | Momento | Verificações mínimas | Bloqueio |
|---|---|---|---|
| G0 — Elegibilidade | Antes do projeto | Uso Hobby pessoal/não comercial; recursos Free suficientes; sem cartão/trial/add-on | Uso comercial em Hobby ou dependência paga |
| G1 — Pull Request | Cada PR | typecheck, lint, unit, integration, dependency audit, workflow permissions | Job crítico falhou ou não executou |
| G2 — Preview | Antes da promoção | E2E, Auth0 callback/session, RBAC/RLS sintético, DB migration em branch descartável, smoke | Acesso cruzado ou falha de autenticação |
| G3 — Release candidate | Antes do deploy final | CVE triage, lockfile, migration plan, backup/restore evidence, config diff, approvals | CVE crítica/alta sem tratamento ou backup não validado |
| G4 — Production validation | Após publicação autorizada | HTTPS, headers, auth, APIs, DB health, logs sanitizados, versão/SHA, consumo Free | Smoke falha, secret exposto, erro de integridade |
| G5 — Pós-release | Após janela definida | Erros, latência, quota, incidentes, integridade e rollback readiness | Degradação acima dos limites internos |

## Quality gates técnicos

### Código e dependências

- TypeScript sem erros.
- ESLint sem erros bloqueantes.
- Unit e integration tests aprovados.
- E2E dos fluxos críticos aprovados.
- npm ci reproduzível a partir do lockfile.
- npm audit e triagem das vulnerabilidades concluídos.
- Nenhuma vulnerabilidade crítica ou alta sem mitigação aprovada e prazo curto.
- Actions de terceiros fixadas em SHA e com permissões mínimas.
- Nenhum segredo no diff, logs ou artefatos.

### Identidade e autorização

- Login, callback, state/nonce, sessão e logout verificados.
- Requests sem sessão recebem 401/redirect conforme contrato.
- Usuário autenticado sem permissão recebe 403.
- Papel sem privilégio não executa ação privilegiada.
- Identidade e tenant não são aceitos do body/query como autoridade.
- Casos negativos e acesso cruzado testados com dados sintéticos.
- RLS é aplicada no banco, inclusive por role runtime não proprietária.

### Banco e migrations

- Migration testada em banco descartável/branch isolada.
- Histórico e checksum íntegros.
- Constraints, grants, índices e policies verificados.
- Restore testado e integridade pós-restore validada.
- Plano expand-contract e compatibilidade com versão anterior documentados.
- Nenhuma migration destrutiva automática no deploy de produção.

### Produção e Free

- Deployment corresponde ao SHA aprovado.
- Environment variables foram revisadas sem exibir valores.
- Auth0 callback/logout allowlists exatas.
- Vercel Hobby elegível para uso não comercial.
- Quotas Free observadas com margem operacional.
- Logs e evidências sanitizados.
- Sem recurso pago, trial, crédito ou cartão requerido.

## Política de vulnerabilidades

- **Crítica:** NO-GO imediato; bloquear release e corrigir ou remover componente afetado.
- **Alta:** NO-GO por padrão; somente exceção temporária com justificativa, compensação, owner, prazo e aprovação independente, se não houver exposição crítica.
- **Média:** corrigir no ciclo definido; avaliar exposição, reachability e compensações.
- **Baixa/Informativa:** registrar e tratar conforme política.
- Não usar somente o número CVSS: considerar versão afetada, exploitabilidade, exposição, reachability, impacto e correção disponível.
- Não usar npm audit fix --force como correção automática sem análise de breaking changes.

Os prazos de correção são políticas internas e devem ser definidos no projeto. Este documento não fornece garantia de ausência de vulnerabilidades.

## Critérios finais de aprovação

O Release Owner só pode recomendar GO quando:
1. todos os IDs críticos aplicáveis têm status Conforme;
2. evidências críticas estão Verificadas e dentro da validade;
3. testes críticos estão associados ao mesmo SHA do artefato;
4. dependências e Actions foram triadas;
5. migrations e restore foram validados fora de produção;
6. RBAC/RLS foram testados com dados sintéticos;
7. limites Free e elegibilidade foram confirmados;
8. nenhuma exceção crítica está aberta;
9. responsáveis técnicos e owner registraram aprovação;
10. plano de monitoramento, rollback e resposta está disponível.

A decisão deve nomear exatamente o projeto, commit, deployment, ambiente e escopo aprovados. Qualquer novo commit invalida a aprovação do artefato anterior.
