# SLI/SLO, alertas operacionais e quotas

## Conceitos

- SLI: medida observável de um comportamento (ex.: proporção de requests sem erro 5xx).
- SLO: meta interna definida pelo responsável do sistema.
- SLA: compromisso contratual do fornecedor. Não inferir SLA a partir de SLI/SLO internos nem de disponibilidade percebida.

As metas abaixo são propostas iniciais, não garantias nem resultados medidos. Ajustar com dados reais e criticidade.

## SLI propostos

| Domínio | SLI | Janela sugerida |
|---|---|---|
| Aplicação | requests 2xx/3xx sobre requests elegíveis | 7 e 30 dias |
| Aplicação | taxa 5xx | 1 hora e 24 horas |
| Latência | p95 de rotas críticas | 1 hora e 7 dias |
| Auth0 | taxa de autenticação concluída versus tentativas elegíveis | 1 hora |
| Auth0 | erros de callback/refresh por client | 1 hora |
| Neon | conexões em uso e erros de conexão | 15 min e 24 horas |
| Neon | crescimento de storage e CU-hours | diário/semanal |
| CI | sucesso de workflows principais | 7 dias |
| Deploy | build success rate e tempo até Ready | 7 dias |

Definir denominadores, excluir health checks e tráfego automatizado conhecido, e não mascarar indisponibilidade por exclusões amplas.

## SLO internos iniciais

Como ponto de partida para um projeto pessoal de baixa criticidade:
- disponibilidade percebida mensal >= 99,0%;
- taxa de 5xx <= 1% em janela móvel de 7 dias;
- p95 das rotas críticas <= 1 segundo, desde que a amostra seja suficiente;
- CI principal concluído com sucesso antes de cada release.

Esses valores são metas ilustrativas e devem ser substituídos por metas acordadas e sustentadas por medições. Vercel Hobby não oferece SLA comercial e seu uso é não comercial.

## Alertas sem SaaS pago

O plano operacional Free não pressupõe um motor de alertas centralizado. Usar:
- notificações nativas de falha de workflow e atividade GitHub;
- observação do painel Vercel e dos estados de deployment;
- painel Neon Usage/Monitoring revisado em calendário;
- Auth0 Monitoring > Logs revisado em janela curta, respeitando retenção Free;
- checklist periódico com responsável, data, evidência e ação.

Se uma notificação não estiver disponível no plano ou configuração, registrar como revisão manual. Não simular alerta automático com polling agressivo ou credenciais privilegiadas.

## Limites e orçamento operacional

Manter uma planilha com colunas:
- fornecedor/recurso;
- plano efetivo;
- quota publicada;
- consumo atual;
- data/hora da leitura;
- janela de medição;
- margem interna;
- tendência;
- ação preventiva;
- fonte oficial.

Adotar limites internos conservadores, por exemplo revisão humana ao atingir 70% e plano de mitigação a 85%, somente como política local. Não confundir esses percentuais com alertas do fornecedor.

## Resposta a quota

1. Confirmar se o limite é compute, storage, requests, build, logs, artifacts ou API rate limit.
2. Verificar se o limite é por projeto, conta, repositório ou janela de tempo.
3. Identificar o recurso que consome a quota.
4. Aplicar redução segura no código ou no fluxo operacional.
5. Não contornar limite por múltiplas contas, abuso ou uso de plano incompatível.
6. Registrar impacto e data de reavaliação.

## Revisão periódica

- Semanal: CI, deployments, Auth0 failures, Neon usage e logs recentes.
- Mensal: tendência de storage/compute, dependências, retenção, quotas e SLO.
- Antes de release: verificar orçamento de capacidade, logs, migrações e plano de rollback.
