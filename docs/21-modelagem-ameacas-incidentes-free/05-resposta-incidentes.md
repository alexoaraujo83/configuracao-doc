# Plano de resposta a incidentes

## Acionamento e comando
Aplicável a suspeita/comprometimento de identidade, dados, credenciais, código, pipeline, deployment, domínio ou disponibilidade. Qualquer operador reporta; responsável designado coordena, define severidade e registra decisões. Sem responsável disponível, suspender ações de alto impacto e escalar ao owner.

Acionar imediatamente diante de credencial privilegiada exposta, cross-tenant, exfiltração, alteração não autorizada, CI/deploy comprometido, token administrativo desconhecido ou DNS/callback alterado.

## Ciclo de resposta
1. **Preparação contínua:** inventário, contatos, acesso emergencial, backups/recovery testados, runbooks, logs e exercícios.
2. **Detecção/análise:** registrar UTC, origem, indicador, ativos, contas/tenants, ambientes, escopo conhecido e incertezas. Preservar evidências sem prolongar exposição crítica.
3. **Contenção:** limitar blast radius; bloquear endpoint/feature, suspender workflow/deploy ou desabilitar credencial comprometida com autorização.
4. **Erradicação:** remover persistência, corrigir vulnerabilidade, substituir segredo, limpar workflow/dependência e revisar permissões.
5. **Recuperação:** restaurar de fonte confiável, atualizar env e emitir deployment limpo; validar Auth, autorização, RLS e observar erros.
6. **Pós-incidente:** timeline, causa raiz, impacto, notificações, risco residual e ações com owner/prazo.

O ciclo é iterativo; repetir análise e contenção se surgirem indicadores novos.

## Severidade
| Nível | Definição | Ação |
|---|---|---|
| SEV-1 crítico | dados expostos, cross-tenant, privilégio admin/owner, CI/deploy controlado | resposta imediata; interromper operação afetada |
| SEV-2 alto | segredo produção, sessão/token, workflow/deploy suspeito | triagem imediata e contenção prioritária |
| SEV-3 moderado | tentativa suspeita sem evidência de acesso | investigar, registrar e mitigar |
| SEV-4 baixo | falso positivo ou evento sem impacto | documentar e melhorar detecção |

Tempos são metas internas sugeridas, não SLA de fornecedor. Definir RTO/RPO por classificação e medir capacidade real Free.

## Regras de contenção
- Não apagar repo, workflow, deployment, branch, usuário ou log como primeira ação.
- Não publicar tokens, payloads ou dados pessoais em issue pública/chat/PR.
- Não reutilizar credencial suspeita para investigar.
- Logout não invalida necessariamente JWT access token emitido.
- Reset de senha PostgreSQL não encerra conexões já abertas; novas conexões com senha antiga falham.
- Alterar variável Vercel não atualiza deployments antigos; vale para novos deployments.
- Em cross-tenant, bloquear o caminho afetado e preservar evidência mínima redigida.
- Revogação, reset, rollback e bloqueio são executados pelo operador autorizado; este documento não os executa.

## Comunicação e encerramento
Centralizar comunicação; compartilhar fatos confirmados, impacto conhecido, medidas e próxima atualização. Avaliar obrigações legais/privacidade com responsável competente. Encerrar somente quando vetor/persistência investigados, credenciais tratadas, vulnerabilidade corrigida, testes negativos aprovados, integridade verificada, monitoramento ativo, evidências preservadas e risco residual aceito formalmente.
