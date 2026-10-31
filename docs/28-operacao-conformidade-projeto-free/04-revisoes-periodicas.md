# Revisões periódicas e manutenção do registro

## Eventos que exigem revisão extraordinária

- Alteração de código, dependência, versão do framework ou SDK.
- Mudança de tenant, aplicação, callback, domínio, projeto, branch ou role.
- Mudança de variável de ambiente, segredo ou política.
- Alteração de quota, preço, termos ou elegibilidade de plano Free.
- Nova integração, tipo de dado ou finalidade de tratamento.
- Incidente, exposição, indisponibilidade ou suspeita de comprometimento.
- Alteração relevante na matriz mestra ou requisito legal/contratual.
- Antes de release de produção.

## Cadência recomendada

A cadência é uma política local, não uma garantia oferecida pelos fornecedores:

| Atividade | Frequência de referência |
|---|---|
| Elegibilidade e quotas Free | Antes de cada release e mensalmente |
| Evidências de controles críticos | Antes de cada release |
| NCs e exceções abertas | Semanalmente |
| Inventário de ambientes e acessos | Trimestralmente e após mudanças |
| Revisão completa da matriz do projeto | Trimestralmente |
| Links e documentação oficial | Trimestralmente |
| Revisão de retenção e minimização | Semestralmente |

Aumentar frequência conforme criticidade, ritmo de mudança e obrigações do projeto.

## Agenda de revisão

Cada reunião/revisão registra:
- Ciclo e período.
- Participantes e funções.
- Mudanças desde a revisão anterior.
- Evidências vencidas ou invalidadas.
- Requisitos alterados e controles reavaliados.
- NCs e exceções, inclusive vencimentos.
- Quotas/termos Free verificados e fontes.
- Decisões, responsáveis e prazos.
- Próxima data.

## Controle de versões

Não sobrescrever avaliações anteriores. Criar novo snapshot por ciclo e registrar:
- commit do repositório;
- versão da matriz de referência;
- data de corte;
- lista de requisitos alterados;
- responsável pela atualização;
- decisão resultante.

## Painel de acompanhamento

Exibir contagens e tendências, sem expor conteúdo sensível:
- requisitos aplicáveis por estado;
- percentual avaliado e conforme;
- evidências expiradas;
- NCs abertas por severidade/idade;
- exceções ativas/próximas do vencimento;
- bloqueios de elegibilidade Free;
- data da última revisão e próxima revisão.

O painel pode ser mantido em Markdown/CSV no GitHub Free; não pressupor ferramenta de BI, automação ou armazenamento adicional gratuito.
