# Documentação de referência — GitHub, Auth0, Vercel e Neon

> Padrões reutilizáveis para arquitetura, configuração, integração, segurança e operação.

## Política de planos

**Somente planos Free.** Não recomendar upgrades, trials, créditos promocionais, add-ons pagos ou recursos não incluídos no Free. Consulte a [matriz de compatibilidade Free](docs/09-planos-free/README.md) antes de adotar qualquer recurso. Limites, termos de uso e disponibilidade por região precisam ser revistos na data da implantação.

## Objetivo

Este repositório reúne documentação técnica versionada para padronizar GitHub, Auth0, Vercel e Neon em múltiplos projetos, separando padrões recomendados, configurações verificadas e procedimentos de implementação.

## Estrutura principal

- [Documentação geral](docs/README.md)
- [Padrões gerais](docs/01-padroes-gerais/README.md)
- [Auth0](docs/02-auth0/README.md)
- [Vercel](docs/03-vercel/README.md)
- [Neon](docs/04-neon/README.md)
- [Integrações](docs/05-integracoes/README.md)
- [Templates por projeto](docs/06-templates-projeto/README.md)
- [Operação e governança](docs/07-operacao-governanca/README.md)
- [Matriz Free](docs/09-planos-free/README.md)
- [Next.js + TypeScript + Neon](docs/10-nextjs-typescript/README.md)
- [Auth0 + Next.js SDK v4](docs/11-auth0-nextjs/README.md)
- [Segurança, RBAC e RLS](docs/20-seguranca-rbac-rls-free/README.md)
- [Revisão técnica Next.js 16 + Auth0 SDK v4 + Neon](docs/27-revisao-tecnica-next16-auth0-neon-free/README.md)
- [Operação de conformidade por projeto — somente Free](docs/28-operacao-conformidade-projeto-free/README.md)

## Ambientes e segurança

Ambientes lógicos não garantem isolamento por si só. Separar credenciais, dados, callbacks, permissões e recursos. Nunca versionar secrets, tokens, senhas, connection strings ou arquivos .env reais. Não declarar uma configuração como verificada sem evidência obtida no painel, API ou CLI.

**Vercel Hobby:** elegível somente para uso pessoal e não comercial. Um projeto comercial não pode receber aprovação GO sob este plano.

## Estado

A documentação descreve padrões e procedimentos. Não significa que contas foram criadas, pacotes instalados, controles testados, migrations executadas, backups/restores realizados ou deployments publicados.

Toda mudança deve ser submetida por Pull Request e revisada antes de incorporação à branch principal.
