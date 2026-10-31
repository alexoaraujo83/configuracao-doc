# 01 — Convenções de nomenclatura

## 1. Objetivo

Nomes consistentes permitem identificar a qual projeto e ambiente um recurso pertence, facilitam auditoria e reduzem risco de usar credenciais no ambiente errado.

## 2. Regras gerais

- Usar minúsculas e kebab-case para nomes de recursos que aceitam hífen.
- Usar slug curto, estável, sem dados pessoais, nome de cliente confidencial ou segredo.
- Não colocar tokens, emails, URLs privadas ou connection strings em nomes/tags.
- Reservar o slug do projeto no manifesto antes de criar qualquer recurso.
- Evitar renomear recursos depois que integrações e URLs estiverem ativas.
- Usar sufixos de ambiente somente quando o recurso realmente for isolado por ambiente.

## 3. Identificadores canônicos

| Entidade | Padrão | Exemplo |
|---|---|---|
| Projeto lógico | `<produto>` | `portal-equipe` |
| GitHub repository | `<produto>` | `portal-equipe` |
| Vercel Project | `web-<produto>` | `web-portal-equipe` |
| Neon Project | `db-<produto>` | `db-portal-equipe` |
| Neon branch | `<ambiente>` ou `pr-<numero>` | `development`, `staging`, `pr-142` |
| PostgreSQL database | `appdb` | `appdb` |
| Schema de aplicação | `app` | `app` |
| Auth0 application | `<produto>-<ambiente>` | `portal-equipe-development` |
| Auth0 API identifier | URI estável do produto | `https://api.example.invalid/portal-equipe` |
| Git branch | `feature/<assunto>`, `fix/<assunto>`, `docs/<assunto>` | `feature/invite-users` |
| Vercel Preview branch Staging | `staging` | `staging` |
| Environment variable | `UPPER_SNAKE_CASE` | `AUTH0_CLIENT_ID` |

Os exemplos de domínio são ilustrativos e usam o TLD reservado `.invalid`; não copiar como URL de produção.

## 4. Auth0 com um tenant Free

O plano Free permite apenas um tenant segundo o artigo de suporte consultado. Assim:
- Nomear o tenant existente como recurso compartilhado da conta; não criar tenants `-dev`, `-staging` e `-prod` como requisito do padrão.
- Se quotas e permissões permitirem, criar aplicações/clientes distintos por projeto e por ambiente.
- Registrar explicitamente que todas essas aplicações residem no mesmo tenant e compartilham a fronteira administrativa do tenant.
- Separar Allowed Callback URLs, Logout URLs e Web Origins por ambiente; evitar curingas amplos.
- Não tratar client IDs diferentes como isolamento equivalente a tenants separados.
- Nunca reutilizar client secret entre ambientes se puder ser evitado.

Se a conta Free não permitir a quantidade de aplicações necessária, documentar a limitação e escolher conscientemente um modelo de aplicação única com URLs restritas; não contornar limites por trial ou plano pago.

## 5. Neon: projeto e branches

Padrão inicial sugerido:
- Um Neon Project por aplicação/domínio de isolamento.
- Branch root reservada a Production.
- Uma branch persistente `development`.
- Uma branch `staging` somente se houver necessidade operacional e quota.
- Branches efêmeras `pr-<numero>` apenas para PRs selecionados, com política de limpeza e quota disponível.

O limite Free publicado é por projeto e não deve ser interpretado como autorização para criar branches ilimitadas. Confirmar também quotas de compute/storage/transferência.

## 6. IDs e inventário

Registrar no manifesto:
- project_slug;
- repository_url;
- vercel_project_name;
- auth0_tenant_domain (sem segredo);
- auth0_application_name por ambiente;
- neon_project_name e branch por ambiente;
- responsáveis e classificação dos dados.

Não registrar Client Secret, Auth0 Secret, DATABASE_URL, tokens ou credenciais de API no manifesto.

## 7. Checklist

- [ ] Slug reservado e único.
- [ ] Nomes dentro das regras dos fornecedores.
- [ ] Sufixo de ambiente aplicado somente a recursos isolados.
- [ ] Auth0 tenant único explicitamente identificado.
- [ ] URLs de callback específicas e sem wildcard amplo.
- [ ] Branches Neon compatíveis com quota.
- [ ] Nenhum dado sensível no nome ou manifesto.
