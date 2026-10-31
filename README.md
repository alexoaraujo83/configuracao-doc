# Documentação de referência — GitHub, Auth0, Vercel e Neon

> Padrões reutilizáveis para arquitetura, configuração, integração, segurança e operação.

## Política de planos

**Somente planos Free.** Não recomendar upgrades, trials, créditos promocionais, add-ons pagos ou recursos não incluídos no Free. Consulte a [matriz de compatibilidade Free](docs/09-planos-free/README.md) antes de adotar qualquer recurso. Limites, termos de uso e disponibilidade por região precisam ser revistos na data da implantação.

## Objetivo

Este repositório reúne uma documentação técnica versionada para padronizar o uso de **GitHub**, **Auth0**, **Vercel** e **Neon** em múltiplos projetos. O conteúdo separa padrões recomendados, configurações efetivamente verificadas e procedimentos de implementação.

## Escopo

- **GitHub:** repositórios, permissões, branches, Pull Requests, Actions, APIs e segurança.
- **Auth0:** tenants, applications, APIs, connections, Universal Login, Actions, RBAC, tokens, sessões e proteção.
- **Vercel:** projetos, equipes, ambientes, deployments, domínios, variáveis, build/runtime, logs e rollback.
- **Neon:** projetos, branches, PostgreSQL, roles, databases, pooling, migrations, backups, restore e observabilidade.
- **Integração:** GitHub → Vercel → Auth0 → Neon, incluindo autenticação, autorização e acesso a dados.
- **Operação:** governança, checklist, troubleshooting, gestão de mudanças e recuperação.

## Estrutura da documentação

| Diretório | Conteúdo |
|---|---|
| [docs/](docs/README.md) | Documentação principal |
| [Fontes oficiais](docs/00-fontes-oficiais/README.md) | Referências por fornecedor e data de verificação |
| [Matriz Free](docs/09-planos-free/README.md) | Compatibilidade e arquitetura restrita a planos Free |
| [templates/](templates/README.md) | Modelos reutilizáveis por projeto |
| [examples/](examples/README.md) | Exemplos de referência |
| [diagrams/](diagrams/README.md) | Diagramas de arquitetura e fluxos |
| [scripts/](scripts/README.md) | Scripts auxiliares e validações |

## Ambientes de referência

| Ambiente lógico | GitHub | Hospedagem | Auth0 | Neon |
|---|---|---|---|---|
| Development | branch de desenvolvimento | local/Development | client de desenvolvimento no tenant Free compartilhado | branch de desenvolvimento |
| Preview | Pull Request | Preview | client/callback de Preview no tenant compartilhado | branch de teste, conforme quota |
| Staging | branch staging | Preview com override por branch no Hobby | client/callback de Staging no tenant compartilhado | branch staging, se houver quota |
| Production | fluxo de release aprovado | Production se elegível no Hobby | client de produção no tenant compartilhado | branch root de produção |

Os ambientes lógicos não são automaticamente isolados pelos fornecedores. A Vercel Hobby não permite uso comercial; não mapear um SaaS comercial para esse plano.

## Regras de segurança

1. Nunca versionar secrets, tokens, senhas, connection strings ou arquivos .env reais.
2. Usar variáveis distintas por ambiente e privilégio mínimo.
3. Não compartilhar credenciais de produção com ambientes não produtivos.
4. Rotacionar credenciais após suspeita de exposição.
5. Aplicar migrations por fluxo controlado, com revisão e rollback.
6. Não executar mudanças em produção como parte da documentação sem aprovação explícita.
7. Marcar cada configuração como **Verificada**, **Recomendada** ou **Pendente de validação**.
8. Marcar recurso não incluído no Free como **NÃO DISPONÍVEL NO FREE**; não substituir por trial ou crédito temporário.

## Estado da documentação

As configurações atuais dos serviços só serão declaradas como verificadas quando houver evidência obtida no painel, API ou CLI correspondente. Documentos de aplicações existentes podem demonstrar padrões implementados, mas não comprovam o estado atual dos tenants ou projetos dos fornecedores.

## Guias de implementação Next.js + TypeScript + Neon

- [Next.js App Router + TypeScript](docs/10-nextjs-typescript/README.md)
- [Auth0 + Next.js SDK v4](docs/11-auth0-nextjs/README.md)
- [Neon PostgreSQL: driver, roles, RLS e migrations](docs/12-neon-postgresql/README.md)
- [Ambientes, CI, segurança, testes e operação](docs/13-ambientes-testes-operacao/README.md)

## Manuais detalhados de implementação

- [Manuais de implementação Free: Next.js, Auth0, Neon, GitHub Actions, Vercel, segurança e validação](docs/16-manuais-implementacao/README.md)

## Testes, RBAC, migrations e recuperação

- [Estratégia de testes automatizados, autorização multiusuário, migrations, CI/E2E e backup/restore](docs/17-testes-rbac-migrations/README.md)

## Referências técnicas oficiais

- [Matriz de referências oficiais: Next.js, Auth0 v4, Neon Driver, GitHub Actions e Vercel Hobby](docs/15-referencias-tecnicas/README.md)

## Inventário dos painéis

- [Inventário de menus e submenus GitHub, Auth0, Vercel e Neon](docs/14-inventario-paineis/README.md)

## Navegação

- [Índice geral](docs/README.md)
- [Fontes oficiais](docs/00-fontes-oficiais/README.md)
- [Matriz Free](docs/09-planos-free/README.md)
- [Padrões gerais](docs/01-padroes-gerais/README.md)
- [Auth0](docs/02-auth0/README.md)
- [Vercel](docs/03-vercel/README.md)
- [Neon](docs/04-neon/README.md)
- [Integrações](docs/05-integracoes/README.md)
- [Templates por projeto](docs/06-templates-projeto/README.md)
- [Operação e governança](docs/07-operacao-governanca/README.md)
- [Referências e histórico](docs/08-referencias-historico/README.md)

## Contribuição

Toda alteração deve indicar escopo, ambiente afetado, pré-requisitos, impacto, validação e rollback. Mudanças estruturais devem ser submetidas por Pull Request e revisadas antes de serem incorporadas à branch principal.


## Provisionamento reutilizável para múltiplos projetos

- [Guia de provisionamento Free](docs/18-provisionamento-multiplos-projetos/README.md)
- [Templates de projeto](templates/provisionamento/README.md)

Este conjunto padroniza nomes, ambientes, variáveis, provisionamento manual e auditoria. Considera as limitações de tenant único do Auth0 Free, Preview branch-scoped no Vercel Hobby e cotas do Neon Free.


## Matriz de arquitetura por tipo de projeto

- [Matriz Free por arquétipo](docs/19-matriz-arquitetura-free/README.md): aplicação simples, SaaS multiusuário, API/backend e múltiplas aplicações; inclui isolamento, reutilização e critérios GO/NO-GO.


## Segurança e segregação de dados

- [Matriz de segurança RBAC/RLS — somente Free](docs/20-seguranca-rbac-rls-free/README.md): autorização, isolamento PostgreSQL, credenciais, testes e gate de produção.
