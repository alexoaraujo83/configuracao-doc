# Matriz de arquitetura por tipo de projeto — somente Free

**Estado:** padrão recomendado, não provisionado. **Stack:** Next.js App Router + TypeScript + Auth0 + Neon PostgreSQL + GitHub Free + Vercel Hobby. **Revisão:** 2026-10-31.

## Objetivo
Escolher uma topologia por arquétipo, identificando reutilização, isolamento lógico/físico, quotas, blast radius e critérios go/no-go.

## Documentos
1. [Matriz dos arquétipos](./01-matriz-arquetipos.md)
2. [Isolamento e reutilização](./02-isolamento-e-reutilizacao.md)
3. [Decisão go/no-go](./03-decisao-go-no-go.md)
4. [Ficha de arquitetura](./04-ficha-arquitetura.md)

## Resumo executivo

| Arquétipo | Reutilização sugerida | Limite determinante |
|---|---|---|
| Aplicação simples pessoal | Um repo, projeto Vercel, client Auth0 e projeto Neon | Hobby apenas pessoal/não comercial |
| SaaS multiusuário | Auth0 tenant único; Neon com RBAC/RLS para protótipo | SaaS comercial bloqueado no Vercel Hobby |
| API/backend | Repo/API e projeto Neon por domínio | API comercial bloqueada no Hobby |
| Múltiplas aplicações | Auth0 tenant compartilhado com clients separados; Neon separado por app | Tenant compartilhado não isola identidade; quotas e blast radius |

## Restrições oficiais verificadas
- **Auth0 Free:** somente um tenant por conta e até 10 Applications por tenant. A quota é compartilhada entre projetos e clients; clients separados e callbacks distintos são separação lógica, não tenant isolation. [Tenant](https://support.auth0.com/center/s/article/Second-Tenant-on-a-Free-Plan) · [10 Applications](https://support.auth0.com/center/s/article/Error-Problem-creating-clients-Forbidden-You-reached-the-limit-of-entities-of-this-type-for-this-tenant)
- **Vercel Hobby:** uso pessoal e não comercial. Não hospedar SaaS, API ou workload comercial. [Plano](https://vercel.com/docs/plans/hobby) · [Fair Use](https://vercel.com/docs/limits/fair-use-guidelines)
- **Neon Free:** 100 projetos, 10 branches/projeto, 100 CU-hours/projeto/mês, 1 GB por projeto, 20 GB agregados e 5 GB de transferência pública/projeto/mês conforme docs atuais. Revalidar antes de provisionar. [Quotas](https://neon.com/docs/introduction/plans)
- **GitHub Free:** environments e environment secrets em repositórios públicos; indisponíveis em repositórios privados/internos. A página de planos pode indicar 2.000 minutos/mês, enquanto a tabela de uso incluído expressa compute Free em US$ 12/mês para runners padrão privados/internos; conferir tipo de runner/SO e billing. Runners padrão em repositórios públicos são gratuitos em minutos. [Ambientes](https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments) · [Preços](https://github.com/pricing) · [Uso incluído](https://docs.github.com/en/billing/reference/product-usage-included) · [Cobrança Actions](https://docs.github.com/en/actions/concepts/billing-and-usage)

## Estados
- **RECOMENDADO:** compatível com restrições documentadas, sujeito à validação de quotas.
- **CONDICIONAL:** exige evidência e aceitação explícita dos riscos.
- **PENDENTE:** faltam dados ou validação.
- **BLOQUEADO NO FREE:** requisito incompatível; não contornar com trial, crédito ou redução de controles.

**Regra:** se for necessário tenant Auth0 independente, environment secrets GitHub em repo privado Free, isolamento físico não disponível ou hospedagem Vercel comercial, marcar BLOQUEADO NO FREE.
