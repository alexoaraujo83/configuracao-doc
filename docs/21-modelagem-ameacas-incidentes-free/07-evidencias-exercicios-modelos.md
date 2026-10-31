# Evidências, exercícios e modelos

## Preservação
Registrar ID, incidente, coletor, origem, timestamp UTC, período, método de export, SHA-256 quando aplicável, local restrito, classificação, redactions e cadeia de transferências.
- Preservar exports originais; analisar cópia.
- Não armazenar tokens, senhas, cookies, connection strings ou payloads pessoais integrais.
- Screenshots são complementares; registrar URL/ID, data/hora, filtros e contexto.
- Hash prova integridade desde a coleta, não autenticidade nem completude da fonte.
- Restringir acesso e definir descarte conforme política legal/privacidade.
- Plano Free pode limitar retenção/exportação; não alegar trilha imutável sem confirmação.

## Evidências por serviço
| Serviço | Evidência | Limites |
|---|---|---|
| Auth0 | eventos login, mudanças de user/app/API, sessões/revogações disponíveis | retenção, export e quotas por plano |
| Neon | logs disponíveis, roles/grants, branches, conexões, migrations | logs, backup e restore no Free |
| GitHub | commits, PRs, runs, Actions, releases, alerts, audit log | elegibilidade por plano/repo |
| Vercel | deployment ID, commit, autor, build/runtime logs, env/domínio changes | retenção/observabilidade por plano |
| Aplicação | request ID, ator pseudonimizado, endpoint, ação, tenant e status | minimizar PII e segredos |

## Tabletop exercises
Executar em ambiente de teste, sem segredos/dados reais. Medir detecção, decisões, dependências, passos indisponíveis no Free e lacunas.

| ID | Cenário | Resultado esperado |
|---|---|---|
| TT-01 | DATABASE_URL em commit público | identificar role/branch/consumers, preservar evidência, invalidar senha e validar |
| TT-02 | refresh token comprometido | identificar client emissor, revogar grant, avaliar sessões e JWT residual |
| TT-03 | PR workflow tenta imprimir env | impedir execução privilegiada, investigar run, revogar secrets acessíveis |
| TT-04 | usuário acessa registro de outro tenant | bloquear, classificar SEV-1, reproduzir com fixture, corrigir DAL/RLS |
| TT-05 | deployment altera callback/domínio | identificar commit/autor, conter e revisar Auth0/Vercel/DNS |
| TT-06 | abuso esgota quota Neon | limitar tráfego, identificar padrão e validar capacidade |

## Modelo de incidente
- ID/severidade/estado:
- Detectado em UTC / reportado por:
- Responsável / aprovador:
- Serviço, projeto, ambiente e ativos:
- Indicadores e fonte:
- Fatos confirmados / hipóteses:
- Dados, usuários e tenants afetados:
- Timeline UTC:
- Contenção aprovada/executada por:
- Credenciais afetadas (identificadores não secretos):
- Evidências e hashes:
- Erradicação/recuperação/testes:
- Avaliação de privacidade/notificação:
- Risco residual/aceitação:
- Próxima atualização/encerramento:

## Postmortem sem culpa
Resumo/impacto; timeline; detecção; causa técnica e fatores; controles que funcionaram/falharam; escopo; decisões; incertezas; ações com owner/prioridade/prazo; atualizações de STRIDE, threat register, testes e runbooks.

## Comunicação inicial
"Identificamos um evento de segurança em [serviço/ambiente] em [data/hora UTC]. A investigação está em andamento. Até o momento, [fatos confirmados]. As medidas de contenção [foram/estão sendo] aplicadas. Ainda não confirmamos [incertezas]. Próxima atualização: [horário]. Não compartilhe credenciais ou dados sensíveis neste canal."

Comunicado externo exige aprovação do responsável e avaliação jurídica/privacidade.
