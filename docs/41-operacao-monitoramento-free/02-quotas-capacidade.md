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

**Nota de atualização (31/10/2026):** a FAQ específica de quotas, atualizada em 01/10/2026, e o anúncio de 02/10/2026 confirmam 1 GB/projeto e 20 GB agregados. Alguns trechos da página geral Plans ainda mostram 0,5 GB e devem ser tratados como desatualizados. Usar a FAQ específica como fonte primária; conferir o Console na data de provisionamento.

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
- A tabela oficial atual de uso incluído registra 500 MB de armazenamento de Actions e US$ 12/mês de compute incluído para GitHub Free (incluindo runners padrão privados/internos); não tratar 2.000 minutos como franquia universal. O valor efetivo depende da conta, runner e sistema operacional; verificar Billing. Runners padrão em repositórios públicos têm política distinta.
- Controlar consumo, concurrency, workers e artifacts.

## Auth0 Free
Não fixar quotas mutáveis neste documento. Consultar página de pricing e tenant dashboard antes de depender de MAU, retenção de logs, MFA, RBAC, Organizations ou custom domains.

## Política de capacidade
Definir limiares internos conservadores, revisar semanalmente, reduzir consultas/egress e remover branches de teste obsoletas. Não ativar recursos pagos ou trials para contornar limite Free.

Fontes: https://github.com/neondatabase/website/blob/main/content/faqs/free-plan-limits-and-quotas.md ; https://neon.com/docs/introduction/plans ; https://vercel.com/docs/limits ; https://vercel.com/docs/plans/hobby ; https://github.com/pricing