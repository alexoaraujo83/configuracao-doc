# Manuais de implementação — stack Free

**Estado:** documentação de referência; procedimentos ainda não executados neste repositório.
**Stack:** Next.js App Router + TypeScript + Auth0 + Neon PostgreSQL + GitHub Free + Vercel Hobby.
**Política:** não depender de planos pagos, trials, créditos promocionais ou add-ons.

## Objetivo

Converter as referências oficiais em procedimentos reproduzíveis de instalação, configuração, integração, validação, segurança e operação. Estes manuais complementam os documentos de arquitetura, inventário de painéis e referências oficiais.

## Manuais

| Documento | Conteúdo |
|---|---|
| [01 — Next.js e TypeScript](./01-nextjs-typescript.md) | Pré-requisitos, criação, estrutura, scripts, configuração e build |
| [02 — Auth0 + Next.js](./02-auth0-nextjs.md) | Aplicação Regular Web, SDK v4, callback, sessão, autenticação e autorização |
| [03 — Neon PostgreSQL](./03-neon-postgresql.md) | Projeto Free, roles, schema, migrations, driver, RLS e backup |
| [04 — GitHub Free e CI](./04-github-ci.md) | Repositório, branches, Actions, permissões mínimas e limites |
| [05 — Vercel Hobby e deploy](./05-vercel-deploy.md) | Importação, ambientes, variáveis, deploy e restrições de uso |
| [06 — Segurança e aceite](./06-seguranca-validacao.md) | Modelo de ameaça, checklist, testes e resposta a incidentes |

## Próxima camada: testes e operação verificável

Consulte [docs/17 — Testes, RBAC, migrations e recuperação](../17-testes-rbac-migrations/README.md), que detalha:
- Estratégia unitária, integração PostgreSQL e E2E com ferramentas open source.
- RBAC no servidor e isolamento de dados por usuário.
- Migrations versionadas, checksums e estratégia expand/contract.
- GitHub Actions com permissões mínimas e sem secrets em PRs não confiáveis.
- Backup lógico e ensaio de restore compatível com as cotas Free.

## Convenções obrigatórias

- Comandos são executados na raiz do projeto, salvo indicação contrária.
- Valores sensíveis são representados por placeholders; nunca copiar segredos para documentação, issues ou commits.
- O arquivo .env.example contém nomes e exemplos não secretos; valores reais ficam em .env.local ou no cofre de variáveis do provedor.
- Não usar prefixo NEXT_PUBLIC_ em segredos, connection strings ou configurações exclusivamente de servidor.
- Toda rota, Server Action e consulta privada verifica sessão e autorização no servidor. Proxy não substitui autorização.
- SQL parametrizado; migrations versionadas; privilégios mínimos; isolamento de dados por usuário.
- Separar credenciais e bancos/branches por ambiente sempre que possível dentro das cotas Free.
- Não usar Vercel Hobby para SaaS, atividade comercial ou workloads comerciais: o plano é limitado a uso pessoal e não comercial.
- Não assumir que menus ou recursos visíveis no painel estão habilitados no plano Free. Confirmar elegibilidade e cotas no painel e na documentação oficial antes de depender do recurso.

## Fluxo de adoção

1. Ler este índice e as decisões de arquitetura em [docs/10–13](../10-nextjs-typescript/README.md).
2. Seguir os manuais 01 a 05 na ordem.
3. Aplicar o manual 06 antes de publicar qualquer ambiente.
4. Registrar versões efetivamente instaladas, data da validação, resultados e limitações encontradas.
5. Atualizar a matriz Free quando qualquer fornecedor alterar cotas ou condições.

## Critério de evidência

Cada procedimento deve receber um estado explícito:
- **Documentado:** instruções e código revisados, sem execução.
- **Executado:** comando ou configuração realizado, com data e ambiente.
- **Validado:** teste reproduzível aprovado, com evidência não sensível.
- **Pendente/Bloqueado:** depende de acesso, quota, decisão ou autorização.

## Fontes oficiais

- [Next.js — Installation](https://nextjs.org/docs/app/getting-started/installation)
- [Next.js — Proxy](https://nextjs.org/docs/app/api-reference/file-conventions/proxy)
- [Auth0 — Next.js SDK](https://auth0.github.io/nextjs-auth0/)
- [Auth0 — SDK v4 migration guide](https://github.com/auth0/nextjs-auth0/blob/main/V4_MIGRATION_GUIDE.md)
- [Neon — Serverless Driver](https://neon.com/docs/serverless/serverless-driver)
- [Neon — Connect](https://neon.com/docs/get-started/connect-neon)
- [GitHub — Actions usage limits](https://docs.github.com/pt/actions/reference/limits)
- [GitHub — Included usage](https://docs.github.com/pt/billing/reference/product-usage-included)
- [Vercel — Hobby](https://vercel.com/docs/plans/hobby)
- [Vercel — Limits](https://vercel.com/docs/limits)


## Provisionamento reutilizável para múltiplos projetos

Consulte [docs/18 — Provisionamento reproduzível para múltiplos projetos](../18-provisionamento-multiplos-projetos/README.md) para convenções de nomenclatura, ambientes Development/Preview/Staging/Production compatíveis com Free, matriz de variáveis, templates não secretos, procedimento manual de provisionamento e auditoria de permissões/secrets/quotas.
