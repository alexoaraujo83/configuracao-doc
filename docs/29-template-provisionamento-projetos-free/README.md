# Template de provisionamento de novos projetos — somente Free

## Objetivo

Padronizar a criação documental de novos projetos baseados em Next.js App Router + TypeScript + Node.js + Auth0 SDK v4 + Neon PostgreSQL + GitHub Free + Vercel Hobby. O pacote permite reutilizar decisões, estrutura, checklists e exemplos sem misturar credenciais ou dados entre aplicações.

Este pacote descreve como provisionar manualmente recursos quando houver autorização. Não cria contas, tenants, aplicações, bancos, repositórios, variáveis, domínios ou deployments.

## Limites obrigatórios

- Auth0: planejar dentro de tenant Free. Não pressupor RBAC gerenciado, retenção longa de logs, múltiplos tenants, Organizations avançadas ou add-ons.
- Vercel Hobby: somente uso pessoal e não comercial. Projeto comercial/SaaS não recebe GO nesse plano.
- Neon Free: quotas de compute, armazenamento, branches, egress e retenção devem ser verificadas no painel. Não assumir capacidade ilimitada nem backup externo.
- GitHub Free: verificar proteção de branch, Actions, secrets e ambientes conforme visibilidade do repositório. Não presumir controles em repositórios privados.
- Não depender de trial, crédito promocional, cartão, add-on ou funcionalidade paga.
- Não inserir secrets em Git, documentação pública, logs, screenshots ou exemplos.

## Padrão de documentação em dois níveis

1. **Configuração padrão inicial:** caminho mínimo, seguro e funcional, pré-requisitos, instalação, variáveis, integração mínima, validação e checklist.
2. **Configuração completa:** arquitetura, controles avançados, ambientes, automação, governança, observabilidade, recuperação, limites, troubleshooting e checklist de produção.

Os exemplos são fictícios. Não representam configurações existentes ou verificadas.

## Navegação

1. [Arquitetura de referência](01-arquitetura-referencia.md)
2. [Bootstrap Next.js e TypeScript](02-bootstrap-nextjs.md)
3. [Ambientes, nomes e variáveis](03-ambientes-e-variaveis.md)
4. [Auth0 Free](04-auth0-free.md)
5. [Neon PostgreSQL Free](05-neon-postgresql-free.md)
6. [GitHub Free e CI](06-github-free-ci.md)
7. [Vercel Hobby](07-vercel-hobby.md)
8. [Validação e checklist](08-validacao-e-checklist.md)
9. [Templates reutilizáveis](templates/README.md)

## Fluxo de adoção

1. Avaliar elegibilidade dos planos Free e registrar limitações.
2. Criar registro independente para o projeto.
3. Definir ambientes, classificação de dados e responsáveis.
4. Criar estrutura Next.js e TypeScript localmente.
5. Preparar Auth0, Neon e variáveis sem compartilhar credenciais.
6. Versionar no GitHub e definir CI de menor privilégio.
7. Integrar Vercel somente se o uso estiver dentro dos termos Hobby.
8. Validar localmente e em Preview com dados sintéticos.
9. Executar checklist e gate de release.
10. Registrar evidências e decisões no processo de conformidade.

## Estados da documentação

- **Recomendado:** padrão proposto.
- **Exemplo:** conteúdo fictício a adaptar.
- **Pendente de validação:** depende de painel, versão ou quota.
- **Verificado:** somente após observação real, com data, escopo e evidência.

## Referências oficiais

- [Next.js — instalação](https://nextjs.org/docs/app/getting-started/installation)
- [Auth0 — documentação](https://auth0.com/docs)
- [Auth0 — Next.js SDK](https://auth0.com/docs/quickstart/webapp/nextjs)
- [Neon — documentação](https://neon.com/docs)
- [GitHub Docs](https://docs.github.com/pt)
- [Vercel — documentação](https://vercel.com/docs)
- [Vercel — planos](https://vercel.com/docs/plans)
