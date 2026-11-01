# Limitações e elegibilidade dos planos Free

Data de consulta: 2026-10-31. Confirmar novamente no painel e na documentação oficial antes de cada implantação/release. As páginas podem mudar sem aviso e quotas podem depender de região, produto, idade da conta ou configuração.

## Resumo por fornecedor

| Fornecedor | Premissa Free usada | Limitação que afeta conformidade | Tratamento obrigatório |
|---|---|---|---|
| GitHub Free | Repositório e Actions dentro das cotas publicadas | Proteção de branches disponível em repositórios públicos no Free; não assumir equivalência em privados. Quotas de Actions e artefatos limitam CI | Registrar visibilidade, minutos, armazenamento e alternativa de gate |
| Auth0 Free | Tenant único e até 25.000 MAU conforme página de preços consultada | Role Management não incluído na tabela Free; retenção de logs indicada como 1 dia; domínio customizado exige verificação de cartão | Implementar autorização de domínio na aplicação; usar domínio padrão sem cartão; exportar/registrar configuração sem alegar backup completo |
| Vercel Hobby | Deploy e recursos base para uso pessoal | Termos limitam uso a pessoal/não comercial; cotas de runtime/build/logs e colaboração limitadas | NO-GO se o serviço for comercial; não sugerir upgrade ou trial |
| Neon Free | Banco e branches dentro de cotas do plano | Compute, storage, egress, branches e PITR limitados; compute pode suspender ao esgotar cotas | Monitorar consumo; manter backup lógico externo e restore testado |

## Pontos que exigem cautela

### GitHub

- Branch protection está documentada como disponível no GitHub Free para repositórios públicos. Não assumir que a mesma regra possa ser aplicada em repositório privado Free.
- Não depender de GitHub Environments, environment secrets ou regras de aprovação específicas sem confirmar elegibilidade do repositório/plano.
- CI deve funcionar com permissões mínimas, sem secrets de produção em pull requests não confiáveis.
- Ao atingir quotas de Actions, o pipeline pode deixar de executar ou consumir recursos de forma limitada. Planejar jobs curtos, cache controlado e execução seletiva.

### Auth0

- A página de preços consultada indica 25.000 MAU no Free, 1 tenant, 3 administradores/contribuidores, retenção de logs de 1 dia e ausência de Role Management.
- A página também informa até cinco Organizations no Free, mas isso não significa RBAC por organização incluído. Não basear autorização de domínio em recurso não listado como incluído.
- Domínio personalizado exige verificação de cartão segundo a página de preços. Para política sem cartão, usar domínio padrão do tenant.
- Não tratar exportação de configuração como backup de usuários, credenciais, conexões completas ou recuperação integral do tenant.

### Vercel Hobby

- Termos oficiais restringem Hobby a uso pessoal ou não comercial. Isso é gate de elegibilidade, não uma mera cota técnica.
- Limites publicados incluem, entre outros, builds simultâneos e duração de build, deploys diários, CPU ativa, memória provisionada, invocações e retenção curta de logs.
- Não apresentar Hobby como hospedagem gratuita para SaaS, negócio, cliente pagante ou operação comercial.
- Mudança de variável de ambiente requer novo deployment para que a aplicação publicada use o valor atualizado.

### Neon

- A FAQ de limites Free consultada em 2026-10-31 informa 100 projetos, 10 branches/projeto, 100 CU-hours/projeto/mês, 1 GB por projeto, 20 GB totais, 5 GB de transferência pública/projeto/mês e janela de restore de 6 horas limitada a 1 GB de histórico de alterações.
- A FAQ específica de quotas, atualizada em 01/10/2026, e o anúncio de 02/10/2026 confirmam 1 GB por projeto e 20 GB agregados. A página geral Plans contém trechos de 0,5 GB que parecem desatualizados; usar a FAQ específica como fonte primária e revalidar o valor exibido no Console antes de provisionar.
- Esgotar compute ou transferência pode suspender compute até o próximo período; atingir armazenamento pode impedir operações que aumentem dados.
- PITR não substitui backup lógico independente nem garante RPO inferior à janela de histórico.

## Política de decisão Free-only

1. Se o requisito depende de recurso pago, marcar **NÃO DISPONÍVEL NO FREE**.
2. Não propor trial, crédito promocional, add-on, cartão ou upgrade como solução.
3. Registrar alternativa gratuita apenas se ela satisfizer o requisito e os termos aplicáveis.
4. Se não existir alternativa gratuita aceitável, marcar **NO-GO para o escopo** ou reduzir formalmente o escopo.
5. Revalidar cotas no início de cada release e após mudanças de fornecedor.

## Fontes oficiais consultadas

- [Auth0 — preços](https://auth0.com/pt/pricing)
- [Vercel — Hobby Plan](https://vercel.com/docs/plans/hobby)
- [Vercel — Terms of Service, seção Hobby](https://vercel-docs.vercel.sh/legal/terms)
- [Vercel — limites](https://vercel.com/docs/limits)
- [Neon — FAQ de limites Free](https://github.com/neondatabase/website/blob/main/content/faqs/free-plan-limits-and-quotas.md)
- [Neon — planos](https://github.com/neondatabase/website/blob/main/content/docs/introduction/plans.md)
- [GitHub — protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches)
- [GitHub — billing usage included](https://docs.github.com/pt/billing/reference/product-usage-included)

## Aviso de precisão

Valores e recursos acima são uma fotografia documental na data de consulta, não uma promessa de disponibilidade. Onde as fontes do próprio fornecedor divergem, a matriz registra a divergência e adota a postura conservadora. A verificação final deve ser feita no painel e na documentação oficial no dia da decisão.
