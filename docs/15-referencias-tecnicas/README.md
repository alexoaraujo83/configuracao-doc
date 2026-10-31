# Referências técnicas oficiais — stack Next.js, Auth0, Neon, GitHub e Vercel

> Revisão dos links: 31/10/2026. Este índice prioriza documentação primária dos fabricantes. Os links não significam que uma funcionalidade esteja incluída em um plano Free, nem que tenha sido configurada no ambiente.

## Matriz de referências solicitadas

| Plataforma | Tema | Documentação oficial | Uso nesta documentação |
|---|---|---|---|
| Next.js | Instalação e configuração | [Installation — App Router](https://nextjs.org/docs/app/getting-started/installation) | Requisitos, create-next-app, scripts, TypeScript, ESLint, estrutura inicial |
| Next.js App Router | App Router | [App Router](https://nextjs.org/docs/app) · [Getting Started](https://nextjs.org/docs/app/getting-started) | Layouts, pages, navegação, Server/Client Components e convenções |
| Auth0 | SDK Next.js | [Quickstart — Next.js](https://auth0.com/docs/quickstart/webapp/nextjs) | Integração oficial, criação da aplicação Regular Web Application, callbacks e login |
| Auth0 SDK | Referência técnica v4 | [Auth0 Next.js SDK v4.33.0](https://auth0.github.io/nextjs-auth0/) · [API Reference](https://auth0.github.io/nextjs-auth0/) | Classes, métodos, tipos, configuração, sessões e middleware/proxy |
| Neon | Conexão PostgreSQL | [Connecting Neon to your stack](https://neon.com/docs/get-started/connect-neon) | Connection string, seleção de branch/database/role, pooled e direct |
| Neon Driver | Driver TypeScript | [Neon serverless driver](https://neon.com/docs/serverless/serverless-driver) · [Configuração](https://neon.com/docs/serverless/serverless-driver#configuration) · [Repositório oficial](https://github.com/neondatabase/serverless) | `@neondatabase/serverless`, TypeScript, HTTP, WebSockets e transações |
| GitHub | Documentação GitHub | [GitHub Docs — Português](https://docs.github.com/pt) | Repositórios, branches, Pull Requests, segurança, governança e API |
| GitHub Actions | Cotas gratuitas | [Uso incluído por plano](https://docs.github.com/pt/billing/reference/product-usage-included) · [Cobrança do Actions](https://docs.github.com/pt/billing/concepts/product-billing/github-actions) · [Limites](https://docs.github.com/pt/actions/reference/limits) | Minutos/compute incluído, armazenamento, runners, limites e cobrança |
| Vercel | Documentação Vercel | [Vercel Docs](https://vercel.com/docs) | Projetos, builds, deployments, Functions, domínios e observabilidade |
| Vercel Hobby | Plano gratuito | [Hobby Plan](https://vercel.com/docs/plans/hobby) · [Limits](https://vercel.com/docs/limits) | Elegibilidade, franquias, limites e restrição de uso pessoal/não comercial |

## Observações técnicas importantes

### Next.js

- O guia de instalação App Router informa Node.js mínimo 20.9.
- O instalador recomendado pode configurar TypeScript, ESLint, Tailwind CSS e App Router.
- Para Next.js 16, verificar as convenções atuais do framework em vez de copiar exemplos antigos sem revisão.

### Auth0 SDK v4

- O Quickstart atual é direcionado a Next.js 16 e SDK v4.
- A referência da API publicada identifica a versão v4.33.0 nesta revisão; conferir a versão mais recente antes de atualizar dependências.
- SDK v4 utiliza `Auth0Client` e o boundary `proxy.ts` no Next.js 16; no Next.js 15 a documentação apresenta `middleware.ts`.
- Não misturar instruções de SDK v3 (por exemplo, handlers legados) com a implementação v4.

### Neon PostgreSQL e driver

- O Console fornece connection strings padrão PostgreSQL; escolher explicitamente branch, database e role.
- Usar conexão pooled para workloads com muitas conexões concorrentes/serverless, quando compatível com o fluxo.
- Usar conexão direta quando a ferramenta ou operação exigir semântica de sessão/conexão direta, como determinados processos de migration.
- O driver serverless oferece transporte HTTP e WebSockets. HTTP é adequado a consultas e transações suportadas pelo driver; WebSockets são necessários para fluxos que dependem de conexão persistente/interactive transactions.
- Não publicar `DATABASE_URL`, senha ou connection string em código, logs ou documentação.

### GitHub Actions — interpretar cotas com cuidado

- A página de planos do GitHub informa 2.000 minutos mensais para GitHub Free pessoal; já a tabela atual de uso incluído expressa o crédito de compute de contas Free em US$ 12/mês e o armazenamento de Actions em 500 MB. Não tratar 2.000 minutos como franquia universal para qualquer conta, runner ou sistema operacional.
- A documentação de cobrança descreve uso de computação incluído em valor monetário para runners padrão em repositórios privados/internos, e cobrança após consumir a franquia aplicável; a página de planos e a tabela de uso incluído devem ser lidas em conjunto e conforme o tipo de conta.
- Runners padrão em repositórios públicos e runners auto-hospedados são tratados de forma diferente da franquia de runners hospedados para repositórios privados.
- Multiplicadores de sistema operacional, tipo de runner, armazenamento de artifacts/cache e regras de billing podem afetar consumo e custo. Não interpretar “2.000 minutos” como 2.000 minutos universais em qualquer runner.
- Para operação sem cobrança, configurar budgets e a opção de interromper uso quando o limite de orçamento for atingido, quando disponível; validar o comportamento na conta antes de depender dele.

### Vercel Hobby

- Hobby é gratuito e restrito a uso pessoal/não comercial.
- Não usar Hobby como base de hospedagem de SaaS comercial ou aplicação comercial.
- Quotas e limites podem mudar; conferir as páginas Hobby e Limits na data da implantação.
- Não ativar trial Pro, add-ons ou funcionalidades pagas: este repositório permanece Free-only.

## Política de atualização das fontes

1. Priorizar documentação oficial do fabricante e páginas específicas da versão em uso.
2. Registrar data de consulta, versão do SDK/framework e plano analisado.
3. Para limites e preços, consultar também pricing, billing, fair use e termos legais.
4. Se duas páginas oficiais apresentarem métricas diferentes (por exemplo, minutos versus valor monetário de computação), registrar a diferença e não converter uma unidade na outra sem fórmula oficial.
5. Não substituir documentação de versão por blog antigo quando houver referência técnica atual.
6. Não considerar recurso elegível no Free apenas porque existe uma página ou opção no Dashboard.

## Manuais detalhados de implementação

- [Manuais de implementação Free](../16-manuais-implementacao/README.md)

## Relação com os guias do repositório

- [Next.js + TypeScript](../10-nextjs-typescript/README.md)
- [Auth0 + Next.js SDK v4](../11-auth0-nextjs/README.md)
- [Neon PostgreSQL](../12-neon-postgresql/README.md)
- [Ambientes, CI, testes e operação](../13-ambientes-testes-operacao/README.md)
- [Inventário dos painéis](../14-inventario-paineis/README.md)
- [Fontes oficiais por fornecedor](../00-fontes-oficiais/README.md)

## Critérios de aceite

- [ ] Todos os dez temas solicitados possuem link oficial.
- [ ] SDK Auth0 v4 é distinguido das versões anteriores.
- [ ] Cotas Actions são apresentadas com unidade e contexto de cobrança.
- [ ] Hobby é identificado como pessoal/não comercial.
- [ ] Neon pooled/direct e HTTP/WebSockets são diferenciados.
- [ ] Nenhum recurso pago, trial ou upgrade é proposto como parte da solução Free.
