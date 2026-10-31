# 4. Triagem e correção de vulnerabilidades e CVEs

## 4.1 Fontes e cobertura

Usar em conjunto:
- npm audit e avisos do registry;
- Dependabot alerts / GitHub Advisory Database;
- avisos oficiais do fornecedor (Next.js, React, Auth0 SDK, Neon driver, Node.js);
- avisos de GitHub Actions usadas;
- análise manual de dependência direta/transitiva e superfície de execução.

Nenhum scanner cobre todas as falhas. A ausência de alerta não equivale a ausência de vulnerabilidade.

## 4.2 Registro de triagem

Para cada achado, registrar:
- identificador (CVE/GHSA/advisory ou referência do fornecedor);
- pacote/action e versão afetada;
- caminho da dependência (direta/transitiva);
- severidade informada pela fonte e data;
- componente afetado e se está presente em produção;
- reachability/exposição e pré-condições de exploração descritas pelo fornecedor;
- versão corrigida e estratégia;
- impacto potencial na aplicação;
- responsável, prazo, PR e evidência pós-correção.

Não reclassificar severidade sem justificar. CVSS é um sinal; contexto de exposição, reachability, privilégio e dados acessíveis também entram na priorização.

## 4.3 Política de bloqueio proposta

| Condição | Decisão de release |
|---|---|
| CVE crítica confirmada em componente implantado/exposto, sem mitigação validada | bloquear |
| CVE alta com caminho alcançável ou fronteira sensível afetada | bloquear até corrigir ou aprovar exceção formal |
| CVE alta sem reachability demonstrável | análise documentada; não dispensar automaticamente |
| CVE moderada/baixa | corrigir no ciclo definido; bloquear se requisito regulatório/contratual ou risco contextual exigir |
| falso positivo demonstrado | exceção com evidência, escopo, expiração e reavaliação |
| versão corrigida indisponível | bloquear ou aceitar formalmente o risco com mitigação concreta e prazo; nunca ocultar alerta |

A severidade isolada não substitui análise contextual. O gate deve ser configurado conforme política de risco aprovada.

## 4.4 Fluxo de correção

1. Confirmar advisory e versões afetadas em fonte primária.
2. Determinar dependência direta/transitiva e ambiente em que é usada.
3. Avaliar reachability, superfície exposta, privilégio e dados acessíveis.
4. Identificar versão corrigida e changelog oficial.
5. Criar branch de correção isolada.
6. Atualizar package.json/lockfile ou SHA da Action; não alterar produção diretamente.
7. Executar lint, typecheck, unit, integração, E2E, build e auditoria novamente.
8. Revisar diff, API changes, Node engine e compatibilidade.
9. Abrir PR com advisory, justificativa, testes e plano de rollback.
10. Após aprovação e release autorizada, verificar versão efetivamente implantada e encerrar o achado com evidência.

## 4.5 Regras para npm

- Começar por atualização patch/minor dentro da faixa compatível, quando existir.
- Se a correção exigir major, planejar migração e revisar breaking changes.
- Usar npm audit fix apenas em branch e após avaliar o diff; não usar --force sem plano explícito.
- Atualizar lockfile de modo determinístico e revisar alterações transitivas.
- Não adicionar overrides/resolutions para esconder incompatibilidade sem justificar e testar.

## 4.6 Exceções

Toda exceção deve incluir: advisory, componente, justificativa, impacto, controles compensatórios, responsável, aprovador, data de expiração e gatilho de reabertura. Exceção vencida volta a bloquear. Não registrar “aceito” sem responsável identificável e prazo.

## 4.7 Resposta a comprometimento de pacote/action

Se houver suspeita de pacote malicioso, typosquatting, maintainer comprometido ou Action adulterada:
- suspender a adoção/execução do componente no fluxo;
- preservar logs e hashes sem executar novamente o artefato suspeito;
- identificar commits, builds e deployments potencialmente afetados;
- seguir o runbook de incidentes já documentado no projeto;
- rotacionar credenciais apenas sob autorização e com procedimento operacional aprovado;
- não declarar incidente resolvido antes de verificar a cadeia completa.

Este documento não realiza varredura do repositório nem declara ausência de CVEs.
