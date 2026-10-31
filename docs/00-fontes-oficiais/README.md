# Fontes oficiais — índice

> Revisão documental realizada em 31/10/2026. Links apontam para documentação oficial, preços e termos. A interface do Dashboard pode mudar; registrar sempre o rótulo observado, a data, o plano e a evidência de cada conta.

## Política obrigatória de planos

Este repositório adota exclusivamente planos Free. Não recomendar upgrades, trials, créditos promocionais, add-ons pagos, recursos pagos ou funcionalidades que exijam cobrança. Quando um recurso não estiver incluído, usar a classificação **NÃO DISPONÍVEL NO FREE** e descrever uma alternativa gratuita viável, sem presumir infraestrutura gratuita ilimitada.

A classificação Free não elimina custos indiretos: hardware próprio, energia, conectividade, manutenção, domínio externo, armazenamento independente e operação de backups podem ter custos.

## Guias por fornecedor

| Fornecedor | Referência operacional | Foco |
|---|---|---|
| GitHub | [github.md](github.md) | Repositórios, permissões, rulesets, Actions, APIs e segurança |
| Auth0 | [auth0.md](auth0.md) | Tenants, Applications, APIs, Connections, Organizations, tokens e autenticação |
| Vercel | [vercel.md](vercel.md) | Teams, Projects, Git, ambientes, deployments, Functions e observabilidade |
| Neon | [neon.md](neon.md) | PostgreSQL, projetos, branches, roles, compute, quotas, restore e região |

## Guias de implementação da stack de referência

| Tema | Guia técnico |
|---|---|
| Next.js App Router + TypeScript | [docs/10-nextjs-typescript](../10-nextjs-typescript/README.md) |
| Auth0 SDK v4 + Next.js | [docs/11-auth0-nextjs](../11-auth0-nextjs/README.md) |
| Neon PostgreSQL + RLS + migrations | [docs/12-neon-postgresql](../12-neon-postgresql/README.md) |
| Ambientes, CI, testes e operação | [docs/13-ambientes-testes-operacao](../13-ambientes-testes-operacao/README.md) |

## Inventário dos painéis

Consulte o [inventário de menus e submenus](../14-inventario-paineis/README.md) para caminhos de navegação, finalidade, configuração relevante, restrições Free e pendências de auditoria visual.

## Matriz consolidada

Consulte [Matriz de compatibilidade Free](../09-planos-free/README.md) antes de escolher qualquer arquitetura ou habilitar um recurso.

## Estados de evidência

- **CONFIRMADO NA DOCUMENTAÇÃO OFICIAL:** a página oficial do fornecedor descreve o recurso ou limite.
- **OBSERVADO NA CONTA CONECTADA:** leitura feita em Dashboard/API/CLI conectado; não implica que todas as opções do plano estejam habilitadas.
- **RECOMENDAÇÃO:** configuração proposta, ainda não aplicada.
- **PENDENTE DE VALIDAÇÃO:** depende de inspeção no Dashboard, versão, região, política ou teste funcional.

Não classificar uma configuração como aplicada apenas porque existe instrução para configurá-la. Não registrar valores de secrets, tokens, senhas ou connection strings.

## Registro dos planos observados

| Serviço | Evidência da consulta | Estado observado em 31/10/2026 |
|---|---|---|
| GitHub | Repositório desta documentação | Repositório público; plano de cobrança da conta não inspecionado |
| Auth0 | Sem leitura direta do tenant nesta etapa | Plano Free é requisito; plano efetivo do tenant pendente de validação |
| Vercel | Contexto de implantação conectado | Team com plano Hobby; adequação para uso comercial não presumida |
| Neon | Organização e projeto consultados via conector | Organização Free; projeto em AWS São Paulo, PostgreSQL 18 |

## Fontes oficiais principais

- [GitHub Pricing](https://github.com/pricing) · [GitHub plans](https://docs.github.com/en/get-started/learning-about-github/githubs-plans)
- [Auth0 Pricing](https://auth0.com/pt/pricing)
- [Vercel Hobby](https://vercel.com/docs/plans/hobby) · [Vercel Terms](https://vercel.com/legal/terms) · [Vercel Limits](https://vercel.com/docs/limits)
- [Neon Plans](https://neon.com/docs/introduction/plans) · [Neon Free quotas](https://neon.com/faqs/free-plan-limits-and-quotas) · [Neon Regions](https://neon.com/docs/introduction/regions)

## Controle de alteração

Ao atualizar esta seção, comparar preços, quotas e termos novamente. Guardar data da consulta, URL oficial, plano comparado e mudança em relação à revisão anterior. Não copiar limites de artigos antigos quando a FAQ ou a página oficial de pricing tiver sido atualizada mais recentemente.
