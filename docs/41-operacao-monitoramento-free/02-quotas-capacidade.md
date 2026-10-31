# 02 — Quotas Free e capacidade

## Neon Free — valores de referência consultados em 31/10/2026
- 100 projetos.
- 10 branches por projeto.
- 100 CU-hours por projeto/mês.
- 1 GB PostgreSQL por projeto e até 20 GB no total da conta.
- 5 GB de public network transfer por projeto/mês.
- Scale-to-zero após 5 minutos de inatividade.
- Instant restore: janela de 6 horas, limitada a 1 GB de change history.
- Monitoring history: 1 dia.

**Divergência documental:** FAQ e anúncio de 02/10/2026 informam 1 GB/projeto, enquanto a página Plans ainda apresenta 0,5 GB em partes do conteúdo. Até confirmar o limite exibido no Console do projeto, adotar 0,5 GB como teto operacional conservador e registrar evidência da data.

Ao esgotar CU-hours ou egress, compute pode ser suspenso até o reset mensal; ultrapassar storage impede operações que aumentem armazenamento; atingir limite de branches impede criar novas. Não presumir cobrança de excedente no Free.

## Vercel Hobby
- 100 deployments/dia e até 32 builds/hora, conforme página de limites.
- 1 build simultâneo.
- 45 minutos por build.
- 4 CPU-hours ativos.
- 1 hora de runtime logs e até 4.000 linhas.
- Limites de functions, analytics, transfer e edge variam; verificar página atual.
- Uso pessoal e não comercial.

## GitHub Free
- 2.000 minutos mensais Actions em repositórios privados; repositórios públicos seguem política própria de minutos.
- Controlar consumo, concurrency, workers e artifacts.

## Auth0 Free
Não fixar quotas mutáveis neste documento. Consultar página de pricing e tenant dashboard antes de depender de MAU, retenção de logs, MFA, RBAC, Organizations ou custom domains.

## Política de capacidade
Definir limiares internos conservadores, revisar semanalmente, reduzir consultas/egress e remover branches de teste obsoletas. Não ativar recursos pagos ou trials para contornar limite Free.

Fontes: https://github.com/neondatabase/website/blob/main/content/faqs/free-plan-limits-and-quotas.md ; https://neon.com/docs/introduction/plans ; https://vercel.com/docs/limits ; https://vercel.com/docs/plans/hobby ; https://github.com/pricing