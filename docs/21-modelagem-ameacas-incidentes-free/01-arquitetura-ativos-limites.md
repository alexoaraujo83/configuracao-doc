# Arquitetura, ativos e limites de confiança

## Diagrama lógico

    Browser não confiável
       | HTTPS, cookie de sessão
       v
    Next.js + Auth0 SDK v4 / Vercel Function
       | identidade validada + autorização server-side
       v
    DAL / Route Handlers / Server Actions
       | transação PostgreSQL + contexto tenant/user
       v
    Neon PostgreSQL: app_runtime + RLS

    Auth0: tenant, clients, API, sessões e tokens
       | OIDC/OAuth
       +----------------------> Next.js

    GitHub: código, PRs, Actions, tokens
       | pipeline e artefato
       v
    Vercel: build, env vars, deployment e domínio

Diagrama conceitual; fluxos reais, endpoints, domínios e integrações devem ser inventariados por projeto.

## Ativos críticos

| ID | Ativo | C | I | D |
|---|---|---|---|---|
| A-01 | Dados pessoais e de negócio no Neon | Alta | Alta | Alta |
| A-02 | DATABASE_URL e credenciais PostgreSQL | Crítica | Crítica | Alta |
| A-03 | Auth0 client secret, refresh tokens e sessão | Crítica | Alta | Alta |
| A-04 | Segredos de ambiente Vercel | Crítica | Crítica | Alta |
| A-05 | Tokens GitHub e permissões Actions | Crítica | Crítica | Alta |
| A-06 | Código, workflows e dependências | Média/Alta | Crítica | Alta |
| A-07 | Artefatos e deployments | Alta | Crítica | Alta |
| A-08 | Domínio, DNS e TLS | Alta | Crítica | Alta |
| A-09 | Logs e evidências | Alta | Alta | Média/Alta |
| A-10 | Identidade, membership, tenant e papéis | Alta | Crítica | Alta |

C = confidencialidade; I = integridade; D = disponibilidade.

## Fronteiras de confiança
1. Browser → aplicação: input, cookie e header são não confiáveis; validar origem, sessão, esquema, tamanho e autorização.
2. Aplicação → Auth0: validar issuer, audience, assinatura, expiração, state/nonce e PKCE conforme o fluxo.
3. Aplicação → Neon: TLS, credencial runtime mínima, queries parametrizadas, RLS por tabela e contexto transacional.
4. GitHub → pipeline: workflows, Actions, dependências, PRs e artefatos são superfícies da cadeia de suprimentos.
5. Pipeline → Vercel: credenciais de deploy e variáveis com escopo mínimo; PR não confiável não recebe segredo de produção.
6. Vercel → usuários: previews não são automaticamente confiáveis; produção pública recebe tráfego hostil.
7. Administradores → painéis: contas individuais, privilégio mínimo, MFA quando incluído no plano e revisão de acesso.

## Premissas e limitações
- Auth0 autentica; a aplicação autoriza membership, tenant e domínio.
- Com credencial PostgreSQL compartilhada pelo backend, o banco não identifica diretamente o usuário final. GUCs de tenant/user são contexto definido pelo backend; comprometimento total do runtime pode adulterá-lo. RLS é defesa em profundidade, não isolamento independente contra SQL arbitrário.
- Auth0 Free: um tenant por conta e limite de 10 Applications por tenant, compartilhado entre projetos; vários clients não equivalem a isolamento administrativo entre tenants. Inventariar aplicações antes de provisionar; não presumir Organizations ou add-ons.
- GitHub Free em repositórios privados tem limitações de Environments/secrets; não depender de controles indisponíveis.
- Vercel Hobby é pessoal/não comercial. Standard Protection não protege o domínio de produção.
- Logs e retenção variam por plano; não pressupor SIEM ou retenção premium.

## Inventário por aplicação
Registrar nome lógico, owner, ambiente, domínio, Auth0 tenant/client/API, projeto/branch Neon, role runtime, repositório, workflow, projeto Vercel, nomes/escopos de variáveis (sem valores), dados tratados, integrações, responsável e data da revisão.
