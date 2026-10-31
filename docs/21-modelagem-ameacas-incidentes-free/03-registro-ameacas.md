# Registro de ameaças e risco residual

## Escala
Probabilidade (P): 1 rara a 5 quase certa. Impacto (I): 1 insignificante a 5 crítico. Risco inerente = P × I. Faixas sugeridas: 1–4 baixo; 5–9 moderado; 10–16 alto; 17–25 crítico. Scores abaixo são hipóteses iniciais, não estatística observada.

| ID | Ameaça/ativo | P | I | Risco | Tratamento / teste | Residual |
|---|---|---:|---:|---:|---|---|
| TH-01 | IDOR/cross-tenant A-01/A-10 | 4 | 5 | 20 crítico | autorização DAL + RLS + testes cross-tenant | Alto até teste |
| TH-02 | DATABASE_URL exposta A-02 | 3 | 5 | 15 alto | rotacionar role, revisar histórico e consumers | Moderado após validação |
| TH-03 | Auth0 secret/token exposto A-03 | 3 | 5 | 15 alto | rotação, revogar grants afetados, revisar sessões | Moderado; JWT pode persistir até expirar |
| TH-04 | GitHub token comprometido A-05 | 3 | 5 | 15 alto | revogar token, audit log, revisar commits/workflows | Moderado |
| TH-05 | Workflow/Action malicioso A-06 | 3 | 5 | 15 alto | pin SHA, menor permissão, revisão e fork safety | Moderado |
| TH-06 | Vercel env/deploy comprometido A-04/A-07 | 3 | 5 | 15 alto | atualizar env, redeploy, revisar deployment | Moderado após invalidação |
| TH-07 | SQL injection A-01/A-02 | 3 | 5 | 15 alto | parâmetros, validação, role mínima, testes | Moderado após evidência |
| TH-08 | Sessão/cookie roubado A-03 | 3 | 4 | 12 alto | expiração/rotação e revogação adequada | Moderado |
| TH-09 | Dependência npm comprometida A-06/A-07 | 3 | 5 | 15 alto | lockfile, revisão e rollback | Moderado |
| TH-10 | SSRF A-07 | 2 | 4 | 8 moderado | allowlist, bloquear IP privado/metadata | Baixo/moderado |
| TH-11 | Exfiltração via logs A-09 | 3 | 4 | 12 alto | redaction, acesso mínimo, retenção | Moderado |
| TH-12 | DoS/esgotamento quota A-01/A-07 | 4 | 3 | 12 alto | rate limits, quotas, timeout | Moderado |
| TH-13 | Sequestro DNS/callback A-08 | 2 | 5 | 10 alto | controle DNS, MFA, callback allowlist | Moderado |
| TH-14 | Phishing de operador A-02–A-08 | 3 | 5 | 15 alto | MFA quando incluído, verificação fora de banda | Moderado |

Residual esperado é meta condicional; substituir por avaliação comprovada. Sem teste/evidência, manter alto ou não avaliado.

## Campos de gestão
Projeto/ambiente, owner, data identificada, vetor/indicador, ativo, controle e estado, evidência, impacto em pessoas/negócio, decisão (mitigar/transferir/evitar/aceitar), aprovador, prazo e próxima revisão.

## Escalonamento
- Crítico: cross-tenant, exfiltração sensível, credencial privilegiada exposta, CI/deploy comprometido, adulteração de dados.
- Alto: segredo de produção, token/sessão, SQL injection explorável, DNS/callback/workflow alterado sem autorização.
- Moderado: suspeita sem evidência de exploração ou abuso limitado.
- Baixo: falso positivo confirmado ou evento sem impacto.

Na dúvida sobre exposição de dados ou privilégio, classificar provisoriamente no nível superior.
