# Operação de conformidade por projeto — somente Free

## Objetivo e limites

Este conjunto transforma a matriz mestra de requisitos em um processo contínuo, rastreável e específico por projeto. Aplica-se à stack Next.js + TypeScript, Auth0, Neon PostgreSQL, GitHub Free e Vercel Hobby, respeitando a elegibilidade de cada plano.

É documentação e modelo de governança. Não comprova que serviços foram configurados, controles testados ou evidências coletadas. Não executar mudanças, testes, deploys, migrations, backups ou restores por meio destes modelos sem autorização específica.

> **Restrição Vercel Hobby:** uso pessoal e não comercial. Projeto com finalidade comercial não pode receber decisão GO sob este plano. Registre NO-GO e suspenda a promoção até existir uma alternativa elegível dentro do escopo aprovado; não recomendar upgrade.

## Princípios operacionais

1. Um registro independente por projeto e por ciclo de avaliação.
2. Cada requisito recebe estado explícito, responsável, evidência e data de revisão.
3. Evidência deve ser mínima, pertinente, datada, íntegra e sanitizada.
4. Segredos, tokens, connection strings, dados pessoais e conteúdo de clientes não entram nos registros.
5. “Documentado” não significa “implementado”; “implementado” não significa “testado”.
6. Quotas, termos e elegibilidade Free são revalidados antes de cada release e quando o fornecedor alterar sua oferta.
7. Exceções têm escopo, justificativa, responsável, compensação, vencimento e aprovação.
8. A decisão de release é registrada separadamente da avaliação dos controles.

## Estados padronizados

| Estado | Significado |
|---|---|
| Não avaliado | Nenhuma avaliação válida foi registrada |
| Conforme | Evidência atual demonstra atendimento ao critério |
| Parcial | Há atendimento incompleto ou limitado |
| Não conforme | Critério não atendido |
| Não aplicável | Justificativa documentada e aprovada |
| Bloqueado | Não foi possível avaliar por dependência ou falta de acesso |
| Expirado | Evidência ultrapassou validade ou perdeu pertinência |

Não usar “Conforme” com base apenas em intenção, configuração proposta ou documentação de referência.

## Fluxo de trabalho

1. Abrir registro do projeto usando [cadastro](01-cadastro-projeto.md).
2. Definir escopo, classificação, ambiente e responsáveis.
3. Copiar os requisitos aplicáveis da matriz mestra para o registro local; preservar os IDs de origem.
4. Avaliar controles e associar evidências conforme [registro de evidências](02-requisitos-evidencias.md).
5. Abrir não conformidades e exceções quando necessário, conforme [tratamento](03-nao-conformidades-excecoes.md).
6. Programar revisões e revalidar limitações Free, conforme [calendário](04-revisoes-periodicas.md).
7. Registrar a decisão de release conforme [gate](05-decisao-release.md).
8. Arquivar o snapshot do ciclo e manter o histórico sem sobrescrever avaliações anteriores.

## Pacote de templates

- [Cadastro do projeto](templates/cadastro-projeto.md)
- [Registro de controles](templates/registro-controles.csv)
- [Manifesto de evidências](templates/manifesto-evidencias.yaml)
- [Registro de não conformidade/exceção](templates/registro-nc-excecao.md)
- [Log de decisão de release](templates/decisao-release.md)

## Indicadores mínimos

- Cobertura avaliada = requisitos com estado diferente de “Não avaliado” / requisitos aplicáveis.
- Conformidade demonstrada = requisitos “Conforme” / requisitos aplicáveis.
- Evidências vencidas = quantidade de evidências com validade expirada.
- NCs abertas por severidade e idade.
- Exceções ativas e próximas do vencimento.
- Bloqueios Free: incompatibilidade de plano, quota ou termo.
- Releases GO, GO condicionado e NO-GO por período.

Não agregar requisitos não aplicáveis ao denominador. Não transformar ausência de evidência em conformidade.

## Critérios mínimos de governança

- Dono do projeto confirma escopo e aceita o risco residual.
- Responsável técnico valida evidências técnicas.
- Revisor independente verifica controles críticos quando houver pessoa disponível.
- Quem implementa um controle não deve ser o único aprovador da própria evidência, sempre que a estrutura da equipe permitir.
- Em equipe de uma pessoa, registrar a limitação de segregação como risco e exigir revisão posterior por outra pessoa antes de uma release sensível.

## Revisão e retenção

Manter histórico por ciclo, versão da matriz, commit/release avaliado e data. A retenção deve respeitar necessidade, minimização de dados e capacidade disponível no plano Free. Não presumir logs longos, armazenamento ilimitado ou backup externo gratuito.

## Referências internas

- [Matriz mestra Free](../26-matriz-mestra-conformidade-free/README.md) — se disponível na branch principal após revisão e incorporação.
- [Revisão técnica Next.js 16 + Auth0 SDK v4 + Neon](../27-revisao-tecnica-next16-auth0-neon-free/README.md).
- [Segurança, RBAC e RLS](../20-seguranca-rbac-rls-free/README.md).
- [Gestão de releases](../24-gestao-mudancas-releases-free/README.md).

## Estado desta documentação

Padrões e templates recomendados, ainda não preenchidos por projeto. Nenhum controle está avaliado e nenhuma evidência operacional foi coletada por este documento.
