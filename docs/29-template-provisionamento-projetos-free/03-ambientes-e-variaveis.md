# Ambientes, nomenclatura e variáveis

## Convenções de nomes

Usar slug minúsculo, hífens e sem dados pessoais:
- Projeto: portal-exemplo
- Repositório: portal-exemplo
- Vercel project: portal-exemplo
- Neon project: portal-exemplo
- Auth0 application: portal-exemplo-development, portal-exemplo-preview, portal-exemplo-production, conforme limites Free e necessidade.

Não assumir que criar três applications dentro do mesmo tenant produz isolamento completo.

## Matriz de variáveis

| Variável | Uso | Exposição |
|---|---|---|
| APP_BASE_URL | URL base do ambiente para Auth0 SDK | servidor |
| AUTH0_DOMAIN | Domínio do tenant Auth0 | servidor |
| AUTH0_CLIENT_ID | Client ID da aplicação Auth0 | servidor |
| AUTH0_CLIENT_SECRET | Secret da aplicação Auth0 | servidor |
| AUTH0_SECRET | Chave forte para criptografia da sessão SDK | servidor |
| DATABASE_URL | Connection string Neon | servidor |
| NODE_ENV | Ambiente Node.js | runtime |
| NEXT_PUBLIC_* | Somente valores que podem ser públicos | browser |

Não usar prefixo NEXT_PUBLIC_ em secrets, tokens, URLs privadas com credenciais ou connection strings.

## Regras de armazenamento

- Local: arquivo .env.local, fora do Git.
- GitHub: nunca colocar secrets no repositório, issues, commits ou artefatos.
- Vercel: configurar variáveis por ambiente apenas quando uso for elegível e permitido no Hobby.
- CI: preferir validações sem acesso a produção; não imprimir variáveis.
- Auth0: usar callback/logout/web origins exatos por ambiente.
- Neon: connection string distinta por ambiente/role, quando a topologia permitir.

Mudanças em variáveis de ambiente da Vercel não alteram deployments já publicados; um novo deployment é necessário para incorporar alterações. Confirmar o comportamento atual no painel antes da operação.

## Exemplo de .env.example

~~~dotenv
APP_BASE_URL=http://localhost:3000
AUTH0_DOMAIN=example-tenant.us.auth0.com
AUTH0_CLIENT_ID=replace-with-client-id
AUTH0_CLIENT_SECRET=replace-with-local-secret
AUTH0_SECRET=replace-with-generated-random-secret
DATABASE_URL=postgresql://user:password@host/database?sslmode=require
~~~

Todos os valores são placeholders. Nunca copiar secrets reais para esse arquivo.

## Geração de AUTH0_SECRET

Gerar localmente um valor aleatório criptograficamente seguro com o mecanismo documentado pelo SDK. Não reutilizar entre projetos. Não colar o valor em tickets, documentação, chat, terminal compartilhado ou logs.

## Checklist

- [ ] Cada ambiente tem base URL correta.
- [ ] Callback, logout URL e web origin correspondem exatamente ao ambiente.
- [ ] Segredos não aparecem em NEXT_PUBLIC_*.
- [ ] .env.local está ignorado pelo Git.
- [ ] .env.example contém apenas placeholders.
- [ ] Nenhum secret foi incluído em commit, log ou artefato.
- [ ] Credenciais de produção não são usadas em Preview.
- [ ] Rotação e responsável por cada secret estão documentados.
