# 03 — Matriz de variáveis de ambiente

## 1. Regras

- `.env.example` contém nomes e placeholders não secretos, nunca valores reais.
- `.env.local` permanece local e ignorado pelo Git.
- Segredos só existem em runtime/build environment protegido e no cofre operacional aprovado.
- Nunca usar `NEXT_PUBLIC_` para credenciais, tokens, connection strings ou valores exclusivos do servidor.
- Cada projeto e ambiente recebe credenciais próprias quando o fornecedor permitir.
- Não copiar valores entre projetos para facilitar provisionamento.
- Alterar variáveis na Vercel exige novo deployment para que o deployment passe a usá-las.

## 2. Matriz canônica

| Variável | Tipo | Development | Preview | Staging (Hobby Preview branch) | Production |
|---|---|---|---|---|---|
| `APP_BASE_URL` | URL pública | localhost | URL preview específica | URL da branch staging | domínio de produção |
| `NODE_ENV` | runtime | development | production no deployment | production no deployment | production |
| `AUTH0_DOMAIN` | identificador não secreto | tenant Free | mesmo tenant Free | mesmo tenant Free | mesmo tenant Free |
| `AUTH0_CLIENT_ID` | identificador | client dev | client preview ou dev dedicado | client staging se existir | client prod |
| `AUTH0_CLIENT_SECRET` | segredo | segredo dev | segredo preview | segredo staging | segredo prod |
| `AUTH0_SECRET` | segredo de sessão | segredo local | segredo próprio | segredo próprio | segredo próprio |
| `DATABASE_URL` | segredo | branch dev | branch test | branch staging | branch root prod |
| `LOG_LEVEL` | configuração | debug | info | info | warn/info |
| `APP_ENV` | configuração | development | preview | staging | production |

Os nomes exatos dependem da versão do SDK e da implementação. Para o Auth0 Next.js SDK v4, conferir também os nomes exigidos pela versão instalada; não misturar variáveis antigas de SDK v3 com v4.

## 3. Template `.env.example`

```dotenv
# Public application configuration (not secrets)
APP_BASE_URL=http://localhost:3000
APP_ENV=development
LOG_LEVEL=debug

# Auth0 — values below are placeholders only
AUTH0_DOMAIN=your-tenant-region.auth0.com
AUTH0_CLIENT_ID=replace-with-development-client-id
AUTH0_CLIENT_SECRET=replace-with-development-client-secret
AUTH0_SECRET=replace-with-random-session-secret

# Neon PostgreSQL — never commit a real connection string
DATABASE_URL=postgresql://USER:PASSWORD@HOST/DB?sslmode=require
```

O template acima é genérico. Ajustar às variáveis efetivamente exigidas pelo SDK e ao validador Zod do projeto. Não incluir uma connection string utilizável.

## 4. Onde armazenar

| Local | Uso permitido | Proibido |
|---|---|---|
| GitHub repository | Código e templates não secretos | .env real, client secret, DATABASE_URL |
| GitHub Actions Free | CI sem segredos de produção; tokens mínimos quando inevitáveis e de escopo restrito | Credenciais de produção em workflow de PR |
| Vercel Project Settings | Variáveis escopadas a Development/Preview/Production | Reutilizar secret entre projetos sem justificativa |
| Máquina local | `.env.local` com permissões restritas | Commit ou sincronização pública |
| Neon | Roles e credenciais PostgreSQL por ambiente | Usar owner/admin como role runtime |
| Auth0 Dashboard | Client credentials e configurações do tenant | Publicar Client Secret em documentação |

## 5. Rotação e alteração

1. Gerar novo segredo por ambiente.
2. Cadastrar novo valor no destino correto.
3. Implantar/reiniciar workload para consumir o novo valor.
4. Validar autenticação/conexão.
5. Revogar o segredo antigo após confirmar a troca.
6. Registrar data, responsável, ambiente e resultado sem registrar o valor.

## 6. Auditoria de exposição

Pesquisar no histórico Git, logs e artifacts por nomes de variáveis e padrões de token sem imprimir o valor encontrado. Se um secret for encontrado:
- considerar comprometido;
- revogar/rotacionar no provedor;
- revisar acessos e logs;
- remover o valor do histórico Git conforme procedimento próprio;
- registrar incidente.

Apagar o arquivo no commit mais recente não invalida o segredo nem remove cópias do histórico.
