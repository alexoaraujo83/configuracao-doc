# BIA, RTO, RPO e prioridades

## Conceitos
- BIA: análise de impacto de indisponibilidade por processo, usuário e dado.
- RTO: tempo máximo interno desejado para restabelecer serviço mínimo.
- RPO: perda máxima de dados, medida em tempo, aceita pelo negócio.
- MTPD/MAO: duração máxima tolerável de interrupção.
- MBCO: nível mínimo de serviço durante contingência.

RTO/RPO são objetivos internos, não SLA dos fornecedores.

## Matriz inicial proposta
| Capacidade | Prioridade | RTO alvo | RPO alvo | Ressalva |
|---|---|---:|---:|---|
| Código/pipeline | P1 | 4 h | 1 commit | Git remoto + clone verificado |
| Acesso operador | P1 | 4 h | 1 dia de inventário | email, 2FA, recovery codes |
| PostgreSQL | P1 | 8 h | 1 h | PITR Free com janela documentada de 6h; dump tem RPO do último dump |
| Auth0 | P1 | 8 h | 24 h config | export não inclui usuários nem password hashes |
| Frontend/API | P1 | 4 h | 1 commit | rollback Hobby só ao deployment de produção imediatamente anterior |
| DNS/domínio | P1 | 24 h | última zona exportada | propagação e acesso ao registrador não garantidos |
| Logs/evidências | P2 | 24 h | 24 h | retenção limitada pelo plano |

Valores ilustrativos para projeto pessoal de baixa criticidade; não usar em sistemas regulados ou críticos sem BIA formal.

## Capacidade versus objetivo
Se o RPO for menor que 6h, PITR Free sozinho não garante o objetivo. Manter dumps externos em frequência compatível. Dump diário pode perder quase 24h; semanal, quase sete dias. RPO real = instante do incidente menos horário do último backup válido.

Se RTO exigir failover automático, multi-região, suporte 24/7 ou retenção longa, Free pode não atender: registrar NO-GO.

## BIA por projeto
Registrar processo/owner, usuários afetados, classificação/volume dos dados, dependências, janela operacional, MBCO, RTO/RPO aprovados, impacto de perda, reconciliação, obrigações legais, contatos e data do último exercício.

## Degradação segura
- Banco indisponível: manutenção; não aceitar gravações sem persistência confiável.
- Auth0 indisponível: não criar bypass nem confiar em headers do browser.
- Vercel indisponível: usar status alternativo somente se previamente preparado.
- Domínio indisponível: não improvisar callback OAuth.
- GitHub indisponível: usar clone local verificado; não implantar código não revisado.
