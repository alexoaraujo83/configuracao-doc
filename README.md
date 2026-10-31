# Documentação de referência — GitHub, Auth0, Vercel e Neon

> Padrões reutilizáveis para arquitetura, configuração, integração, segurança e operação.

## Política de planos

**Somente planos Free.** Não recomendar upgrades, trials, créditos promocionais, add-ons pagos ou recursos não incluídos no Free. Consulte a [matriz de compatibilidade Free](docs/09-planos-free/README.md) antes de adotar qualquer recurso. Limites, termos de uso e disponibilidade por região precisam ser revistos na data da implantação.

## Objetivo

Este repositório reúne documentação técnica versionada para padronizar GitHub, Auth0, Vercel e Neon em múltiplos projetos.

## Escopo
- GitHub: repositórios, permissões, branches, Pull Requests, Actions, APIs e segurança.
- Auth0: tenants, applications, APIs, connections, Universal Login, RBAC, tokens, sessões e proteção.
- Vercel: projetos, ambientes, deployments, domínios, variáveis, runtime, logs e rollback.
- Neon: projetos, branches, PostgreSQL, roles, pooling, migrations, backups e restore.
- Integração: GitHub → Vercel → Auth0 → Neon.
- Operação: governança, checklists, troubleshooting, mudanças e recuperação.

## Estrutura da documentação

| Diretório | Conteúdo |
|---|---|
| [docs/](docs/README.md) | Documentação principal |
| [Fontes oficiais](docs/00-fontes-oficiais/README.md) | Referências por fornecedor |
| [Matriz Free](docs/09-planos-free/README.md) | Compatibilidade Free |
| [templates/](templates/README.md) | Modelos reutilizáveis |
| [examples/](examples/README.md) | Exemplos de referência |
| [diagrams/](diagrams/README.md) | Diagramas |
| [scripts/](scripts/README.md) | Scripts auxiliares |

## Política de segurança

1. Nunca versionar secrets, tokens, senhas, connection strings ou .env reais.
2. Usar variáveis distintas por ambiente e privilégio mínimo.
3. Não compartilhar credenciais de produção com ambientes não produtivos.
4. Rotacionar credenciais após suspeita de exposição.
5. Aplicar migrations por fluxo controlado, com revisão e rollback.
6. Não executar mudanças em produção como parte da documentação sem aprovação explícita.
7. Marcar configurações como Verificada, Recomendada ou Pendente.
8. Marcar recurso indisponível no Free sem substituí-lo por trial/crédito.

## Guias técnicos
- [Next.js App Router + TypeScript](docs/10-nextjs-typescript/README.md)
- [Auth0 + Next.js SDK v4](docs/11-auth0-nextjs/README.md)
- [Neon PostgreSQL](docs/12-neon-postgresql/README.md)
- [Ambientes, CI, segurança, testes e operação](docs/13-ambientes-testes-operacao/README.md)
- [Manuais detalhados](docs/16-manuais-implementacao/README.md)
- [Testes RBAC, migrations e recuperação](docs/17-testes-rbac-migrations/README.md)
- [Provisionamento reutilizável multi-projeto](docs/18-provisionamento-multiplos-projetos/README.md)
- [Matriz de arquitetura por arquétipo](docs/19-matriz-arquitetura-free/README.md)
- [Segurança RBAC/RLS](docs/20-seguranca-rbac-rls-free/README.md)
- [Modelagem de ameaças e resposta a incidentes](docs/21-modelagem-ameacas-incidentes-free/README.md)

## Ambientes e elegibilidade

Os ambientes lógicos não são automaticamente isolados pelos fornecedores. Vercel Hobby é para uso pessoal e não comercial; não mapear SaaS comercial para esse plano. Confirmar quotas e limites dos demais planos Free antes da implantação.

## Estado da documentação

Configurações de serviço só serão declaradas verificadas com evidência do painel, API ou CLI. Documentação de padrões não comprova o estado atual de tenants/projetos. Nenhum serviço foi provisionado ou alterado por este repositório sem aprovação explícita.

Toda alteração deve indicar escopo, ambiente, pré-requisitos, impacto, validação e rollback. Mudanças estruturais seguem por Pull Request e revisão antes de incorporação à branch principal.
