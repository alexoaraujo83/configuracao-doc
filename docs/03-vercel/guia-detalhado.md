# Vercel — guia detalhado de configuração e operação

Este complemento expande o [guia principal](README.md), com foco no fluxo GitHub → Vercel e na operação por ambiente.

## 1. Modelo de recursos e escopo

Um Project Vercel representa uma aplicação implantada a partir de um repositório Git. Um projeto pode produzir um deployment Production e diversos deployments pré-produção. Um repositório monorepo pode alimentar vários projetos Vercel, cada um com Root Directory e configuração próprios.

Diferenciar:
- **Team:** organização, membros, billing e configurações compartilhadas.
- **Project:** configuração da aplicação, Git, builds, domains, env vars e proteção.
- **Deployment:** build imutável de um commit/branch com URL e estado próprios.
- **Environment:** conjunto de variáveis e comportamento de deployment (Local, Preview, Production e Custom quando habilitado).
- **Domain:** hostname associado a projeto/deployment/ambiente.

## 2. Configuração inicial recomendada

1. Selecionar a Team correta e importar o repositório GitHub.
2. Confirmar Framework Preset, Root Directory e se o projeto usa monorepo.
3. Definir Production Branch explicitamente.
4. Confirmar install/build command, output e versão Node compatível com o projeto.
5. Definir env vars em cada target sem reutilizar secrets de produção em Preview.
6. Configurar domínios e TLS somente depois de validar deployment.
7. Habilitar Deployment Protection para previews sensíveis, conforme plano e necessidade.
8. Definir região das Functions próxima do banco e dos usuários, considerando latência e residência de dados.
9. Documentar logs, retenção, limites e procedimento de rollback.

## 3. Settings — inventário detalhado

| Seção | Configuração e perguntas de auditoria |
|---|---|
| General | project name, framework, root directory, build/install/dev commands, output directory, Node.js version |
| Build and Development Settings | overrides, include files, ignored build step, monorepo build behavior |
| Git | provider, repository, production branch, deploy hooks, branch deploy behavior |
| Environments | Local, Preview, Production, custom environments, branch tracking |
| Environment Variables | key, target, sensitive/non-sensitive, branch-specific values, redeploy behavior |
| Domains | domain, target environment, DNS records, TLS, redirects, verification |
| Deployment Protection | Vercel Authentication, trusted IPs, password protection (plano aplicável), bypass rules |
| Functions | runtime, region, duration, memory, fluid compute/other options where available |
| Networking | firewall, secure compute/private networking/static IPs when available |
| Caching | CDN, cache headers, revalidation, skew protection and framework behavior |
| Observability | runtime/build logs, Web Analytics, Speed Insights, traces, drains, retention |
| Integrations | Git provider, Marketplace integrations, storage, tokens and granted scopes |
| Team | members, roles, SSO and access policy available on plan |

A interface e os nomes dos menus podem mudar. Registrar a data da verificação e o plano da Team; não tratar esta lista como reprodução literal e imutável do Dashboard.

## 4. Git Integration e ciclo de deployment

Com Git Integration, Vercel pode criar deployments para pushes em branches e merges na production branch. Um PR pode receber Preview deployment; o merge na production branch pode gerar Production deployment. Validar a configuração real de branch e deployment em cada projeto.

Fluxo recomendado:
1. Abrir PR no GitHub.
2. CI valida lint, types, tests, build e segurança.
3. Vercel gera Preview e adiciona URL ao contexto do PR.
4. Revisores testam o Preview com dados não produtivos.
5. Aprovar e mergear conforme regras do GitHub.
6. Vercel gera Production deployment.
7. Validar health, autenticação, banco, logs e domínio.
8. Em falha, promover deployment anterior/rollback e avaliar migrations.

Evitar deploy duplicado por GitHub Actions e integração nativa sem definir qual sistema é autoridade para build e promoção.

## 5. Ambientes: Local, Preview, Production e Staging

Vercel fornece três ambientes padrão: Local Development, Preview e Production. Custom Environments (por exemplo, staging/QA) dependem de plano e limites; validar elegibilidade atual da Team antes de desenhar arquitetura baseada neles.

| Ambiente lógico | Mapeamento Vercel | Uso recomendado |
|---|---|---|
| Development | Local Development | desenvolvimento local e variáveis locais |
| Preview | Preview | validação de branch/PR |
| Staging | Custom Environment ou Project separado | ambiente persistente de homologação |
| Production | Production | tráfego real |

Se custom environment não estiver disponível, staging pode exigir projeto separado, com custo, domains, variáveis e integração Git próprios. Não chamar um Preview fixo de staging sem documentar sua natureza efêmera, branch tracking e proteção.

### CLI para ambientes

```bash
# Associar diretório local a um projeto existente
vercel link

# Baixar variáveis de desenvolvimento para arquivo local
vercel env pull .env.local

# Executar um comando com variáveis do projeto sem materializá-las em arquivo
vercel env run -- npm run dev

# Criar Preview deployment
vercel deploy

# Consultar variáveis do ambiente staging (se custom environment existir)
vercel pull --environment=staging

# Deploy em custom environment
vercel deploy --target=staging

# Deploy de produção — executar somente após aprovação
vercel deploy --prod
```

Não commitar `.env.local`. Proteger arquivos gerados e remover cópias temporárias de secrets ao terminar.

## 6. Environment Variables

Variáveis podem existir em escopo Team ou Project. Variáveis de Team podem ser compartilhadas com projetos selecionados; Project variables ficam limitadas ao projeto. Mudanças de variáveis não alteram deployments existentes: um novo deployment é necessário para consumir a configuração atualizada.

Padrão:
- Classificar cada variável como pública, configuração interna ou segredo.
- Prefixos públicos (por exemplo, `NEXT_PUBLIC_`) são incorporados/expostos ao bundle cliente conforme framework; nunca colocar credenciais neles.
- Separar valores de Development, Preview, Staging e Production.
- Usar branch-specific values quando apropriado e suportado.
- Documentar apenas nomes, propósito, consumidor e ambientes; não documentar valores.
- Rotacionar credenciais e fazer redeploy.
- Revisar quem tem acesso ao projeto, pois membros autorizados podem ter acesso às configurações/valores conforme o tipo e permissão.

Exemplo de catálogo:

| Nome | Finalidade | Target | Classificação |
|---|---|---|---|
| AUTH0_ISSUER_BASE_URL | issuer Auth0 | server | configuração |
| AUTH0_CLIENT_ID | identificador cliente | server | identificador |
| AUTH0_CLIENT_SECRET | segredo cliente | server | segredo |
| DATABASE_URL | conexão Neon | server | segredo |
| NEXT_PUBLIC_APP_URL | URL pública | browser/server | público |

Validar nomes exatos exigidos pelo framework/SDK instalado. Não assumir que todas as variáveis listadas são necessárias a todas as versões.

## 7. Domains, DNS e TLS

- Adicionar domínio ao projeto e selecionar ambiente correto.
- Verificar instruções DNS geradas pela Vercel; não inventar registros.
- Confirmar ownership, status de verificação e certificado TLS.
- Definir domínio canônico e redirects HTTP/HTTPS e www/apex.
- Separar domains de Preview/Staging/Production quando a aplicação exigir.
- Atualizar callback/logout URLs no Auth0 após mudança de domínio.
- Não apontar domínio de produção para Preview deployment.
- Registrar TTL, DNS provider, owner e plano de reversão.

## 8. Functions, runtime e região

Documentar runtime (Node.js/Edge quando aplicável), versão Node, região, duração máxima, memória, limites de payload e comportamento de streaming. Configurar região de execução próxima do Neon para reduzir round trips, mas validar latência de usuários e requisitos de residência. Evitar assumir que toda Function executa na região do banco automaticamente.

Para apps Next.js, registrar se rotas são server-rendered, static, route handlers, middleware/proxy e quais operações dependem de runtime Node. Confirmar compatibilidade das dependências com o runtime escolhido.

## 9. Segurança, proteção e rede

- Usar Deployment Protection para ambientes pré-produção que contenham informação sensível.
- Configurar Firewall/WAF e regras de rate limiting conforme tráfego e plano.
- Revisar acesso de membros e integrações.
- Preferir tokens de curta duração e escopo mínimo para automações.
- Não registrar headers Authorization, cookies, connection strings ou dados pessoais.
- Avaliar static IP/private networking quando houver requisito de allowlist ou conectividade privada e o plano suportar.
- Tratar bypass URLs/tokens de proteção como segredos.
- Validar segurança de previews originados de forks e PRs externos.

## 10. Observabilidade e incidentes

Runtime Logs permitem filtrar e inspecionar logs de invocações de Functions. Retenção, traces, drains e recursos de observabilidade variam por plano/add-on. Não assumir retenção longa sem confirmar plano atual.

Campos úteis: deployment, branch, environment, region, function, duration, memory, request ID, cache status, outgoing requests e erros. Adotar logs estruturados com correlation/request ID e redaction de dados sensíveis.

Runbook:
1. Identificar deployment, commit, região e request ID.
2. Separar falha de build, runtime, edge, rede, autenticação ou banco.
3. Inspecionar logs e traces disponíveis.
4. Comparar variáveis e config com último deployment saudável sem revelar secrets.
5. Verificar Auth0 issuer/audience/callback e Neon endpoint/branch/role.
6. Reverter deployment se necessário; avaliar compatibilidade de schema antes de rollback.
7. Registrar impacto, causa e ação preventiva.

## 11. CLI e API

A CLI suporta autenticação, link do projeto, pull de variáveis, deploy, logs, domains, DNS, env e inspeção. Para automação, fixar versão da CLI no pipeline e usar tokens em secrets com escopo mínimo. Nunca colocar token em argumento visível, commit ou log.

Comandos úteis:

```bash
vercel --version
vercel whoami
vercel project ls
vercel project inspect <project>
vercel env ls
vercel logs <deployment-url>
vercel domains ls
vercel domains inspect <domain>
```

Confirmar sintaxe na versão instalada antes de executar. Para produção, usar gates de aprovação e identidade apropriada.

## 12. Checklist de validação

- [ ] Team e Project corretos.
- [ ] GitHub repository e production branch confirmados.
- [ ] Root Directory e comandos de build conferidos.
- [ ] Node/runtime e região documentados.
- [ ] Local/Preview/Production mapeados; Staging explicitamente resolvido.
- [ ] Variáveis segregadas e sem secrets públicos.
- [ ] Domains, DNS, TLS e Auth0 callbacks alinhados.
- [ ] Preview protegido e com dados não produtivos.
- [ ] Logs e limites de retenção conhecidos.
- [ ] Rollback testado e compatível com migrations.
- [ ] Plano/limites/custos revisados.

## 13. Fontes oficiais

- [Vercel Documentation](https://vercel.com/docs)
- [Projects overview](https://vercel.com/docs/projects)
- [Project settings](https://vercel.com/docs/project-configuration/project-settings)
- [Git deployments](https://vercel.com/docs/git)
- [Deployment environments](https://vercel.com/docs/deployments/environments)
- [Environment variables](https://vercel.com/docs/environment-variables)
- [Vercel CLI](https://vercel.com/docs/cli)
- [Deploy from CLI](https://vercel.com/docs/projects/deploy-from-cli)
- [Runtime logs](https://vercel.com/docs/logs/runtime)
- [Limits](https://vercel.com/docs/limits)

## 14. Estado e evidências

Este documento descreve recursos e procedimentos com base na documentação oficial consultada em 2026-10-31. Para declarar configuração como **Verificada**, registrar projeto, team, plano, data, ambiente e evidência observada. O guia não afirma que todas as opções estejam habilitadas nos projetos atuais.


## Restrição Free e decisão de Staging

A conta conectada foi observada em Hobby. Esse plano é limitado a uso pessoal/não comercial; não usar para implantar ou operar produto SaaS comercial.

Custom Environments, como staging/QA, não devem ser pressupostos no Hobby. Usar Local Development, Preview e Production apenas dentro do uso permitido e das quotas do plano. Um Project separado não remove a restrição contratual de uso comercial.

Se o produto for comercial, classificar a hospedagem Vercel Hobby como **NÃO DISPONÍVEL PARA ESTE USO**. A alternativa Free-only é self-host ou outro provedor com termos explícitos de uso comercial no plano gratuito; validar a alternativa antes de adotá-la. Não recomendar upgrade, trial nem crédito promocional.
