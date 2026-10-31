# Documentação de referência — GitHub, Auth0, Vercel e Neon

**Política: somente planos Free.** Consulte docs/09-planos-free/README.md antes de implementar qualquer recurso.

> Padrões reutilizáveis para arquitetura, configuração, integração, segurança e operação de aplicações.

## Objetivo

Este repositório reúne uma documentação técnica versionada para padronizar o uso de **Auth0**, **Vercel** e **Neon** em múltiplos projetos. O conteúdo separa padrões recomendados, configurações efetivamente verificadas e procedimentos de implementação.

## Escopo

- **Auth0:** tenants, applications, APIs, connections, Universal Login, Actions, RBAC, tokens, sessões, segurança e observabilidade.
- **Vercel:** projetos, equipes, ambientes, deployments, domínios, variáveis, build/runtime, integrações, logs e rollback.
- **Neon:** projetos, branches, PostgreSQL, roles, databases, pooling, conexão, migrations, backups, restore e observabilidade.
- **Integração:** GitHub → Vercel → Auth0 → Neon, incluindo fluxos de autenticação, autorização e acesso a dados.
- **Operação:** governança, checklist, troubleshooting, gestão de mudanças e recuperação.

## Estrutura da documentação

| Diretório | Conteúdo |
|---|---|
| [docs/](docs/README.md) | Documentação principal |
| [templates/](templates/README.md) | Modelos reutilizáveis por projeto |
| [examples/](examples/README.md) | Exemplos de referência |
| [diagrams/](diagrams/README.md) | Diagramas de arquitetura e fluxos |
| [scripts/](scripts/README.md) | Scripts auxiliares e validações |

## Ambientes de referência

| Ambiente lógico | Vercel | Auth0 | Neon |
|---|---|---|---|
| Development | Development | Development / tenant isolado quando disponível | branch de desenvolvimento |
| Preview | Preview por deployment | configuração de teste controlada | branch efêmera ou de preview |
| Staging | projeto/ambiente de staging | configuração de staging | branch de staging |
| Production | Production | tenant/aplicação de produção | branch de produção |

A disponibilidade de ambientes e recursos depende do plano contratado. Não se deve presumir que Development, Preview e Staging sejam ambientes isolados automaticamente em todos os três fornecedores.

## Regras de segurança

1. Nunca versionar secrets, tokens, senhas, connection strings ou arquivos .env reais.
2. Usar variáveis distintas por ambiente e privilégio mínimo.
3. Não compartilhar credenciais de produção com ambientes não produtivos.
4. Rotacionar credenciais conforme política e após qualquer exposição suspeita.
5. Aplicar migrations de banco por fluxo controlado, com revisão e plano de rollback.
6. Não executar mudanças em produção como parte da documentação sem aprovação explícita.
7. Marcar cada configuração como **Verificada**, **Recomendada** ou **Pendente de validação**.

## Estado da documentação

Este repositório começa como uma base de documentação. As configurações atuais dos serviços só serão declaradas como verificadas quando houver evidência obtida no painel, API ou CLI correspondente. Documentos de aplicações existentes podem demonstrar padrões implementados, mas não comprovam, por si só, o estado atual dos tenants ou projetos nos provedores.

## Navegação

- [Índice geral](docs/README.md)
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
