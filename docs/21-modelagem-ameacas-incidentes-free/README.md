# Modelagem de ameaças e resposta a incidentes — somente Free

**Estado:** documentação proposta; não representa auditoria, implantação ou resposta executada. **Revisão:** 2026-10-31. **Stack:** Next.js App Router + TypeScript + Auth0 + Neon PostgreSQL + GitHub Free + Vercel Hobby.

## Objetivo e limites
Estabelecer processo repetível para identificar ameaças, priorizar riscos, prevenir/detectar incidentes e orientar contenção, erradicação e recuperação. Aplicar por aplicação, ambiente, tenant e mudança arquitetural relevante.

- Escopo somente documental: nenhuma credencial, sessão, conta, banco, deployment ou configuração foi alterada.
- Somente recursos incluídos nos planos Free. Não usar trial, crédito promocional, add-on ou recurso pago como pressuposto.
- Vercel Hobby é para uso pessoal e não comercial. Projeto comercial não deve entrar em produção sob esta premissa.
- Confirmar no painel quotas, logs, retenção e controles antes de depender deles.
- Não registrar segredos, tokens, dados pessoais desnecessários ou payloads integrais em evidências.

## Índice
1. [Arquitetura, ativos e limites de confiança](./01-arquitetura-ativos-limites.md)
2. [STRIDE por componente](./02-stride-por-componente.md)
3. [Registro de ameaças](./03-registro-ameacas.md)
4. [Controles Free-only](./04-controles-free.md)
5. [Resposta a incidentes](./05-resposta-incidentes.md)
6. [Runbooks](./06-runbooks.md)
7. [Evidências e exercícios](./07-evidencias-exercicios-modelos.md)
8. [Gate de produção](./08-gate-producao.md)

## Método
- STRIDE aplicado a componentes, fluxos e fronteiras de confiança; misuse cases para abuso de regras de negócio.
- Probabilidade e impacto qualitativos de 1–5; risco inerente = P × I.
- Resposta alinhada ao NIST SP 800-61 Rev. 3 e CSF 2.0.
- Cada risco deve ter responsável, evidência, prazo e decisão explícita.

## Estados de evidência
| Estado | Significado |
|---|---|
| Proposto | descrito, ainda não aplicado |
| Implementado | código/configuração revisado e identificado |
| Testado | teste executado com resultado e data |
| Verificado | observado diretamente no fornecedor |
| Pendente | requer validação |

Não converter procedimento escrito em afirmação de controle existente.

## Referências
- [NIST SP 800-61 Rev. 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final)
- [NIST Incident Response](https://csrc.nist.gov/projects/incident-response)
- [OWASP Threat Modeling](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html)
- [OWASP Authorization](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [GitHub — investigação de incidentes](https://docs.github.com/en/code-security/reference/security-incident-response/investigation-areas)
- [Auth0 — revogar Refresh Tokens](https://auth0.com/docs/secure/tokens/refresh-tokens/revoke-refresh-tokens)
- [Neon — segurança](https://neon.com/docs/security/security-overview)
- [Vercel — rotação de variáveis](https://vercel.com/docs/environment-variables/rotating-secrets)
