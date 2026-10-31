# Segurança e segregação de dados — somente Free

**Estado:** padrão documental recomendado; não aplicado nem auditado em infraestrutura. **Stack:** Next.js App Router + TypeScript + Auth0 + Neon PostgreSQL + GitHub Free + Vercel Hobby. **Revisão:** 2026-10-31.

## Conteúdo
1. [RBAC e matriz de permissões](./01-rbac-permissoes.md)
2. [RLS e segregação de tenants](./02-rls-postgresql.md)
3. [Credenciais e privilégios](./03-credenciais-privilegios.md)
4. [Isolamento entre aplicações](./04-isolamento-aplicacoes.md)
5. [Testes automatizados](./05-testes-automatizados.md)
6. [Aprovação para produção](./06-aprovacao-producao.md)

## Princípios
- Deny by default e least privilege.
- Autenticação não equivale a autorização.
- Verificar autorização em toda Server Action, Route Handler, DAL e operação de dados.
- Ocultar botões no browser não protege recursos.
- RLS é defesa em profundidade; não substitui RBAC, validação de identidade ou proteção contra SQL injection.
- Runtime nunca usa owner, migrator, administrador ou superuser.
- Segredos não são versionados, enviados ao browser ou impressos em logs.
- Falhas de configuração devem negar acesso.

## Arquitetura

\`\`\`text
Browser --cookie HttpOnly/HTTPS--> Next.js + Auth0 SDK v4
  -> DAL server-only: identidade + permission + membership
  -> transação PostgreSQL: contexto tenant/user
  -> Neon: app_runtime sem ownership/BYPASSRLS
  -> tabelas com RLS e default deny
\`\`\`

## Limite de confiança importante
Quando o backend usa uma credencial de banco compartilhada, PostgreSQL não autentica cada usuário final. O backend define o contexto transacional (tenant/user). Um comprometimento total do backend/runtime pode permitir alterar esse contexto; RLS baseada em GUC é defesa em profundidade, não isolamento independente contra execução SQL arbitrária.

Auth0 Free é documentado neste projeto como tenant único por conta. Confirmar no painel a disponibilidade e quotas de RBAC/API permissions no Free. Não presumir Organizations, recursos adicionais ou add-ons pagos. Vercel Hobby é exclusivamente pessoal/não comercial.

## Estados de evidência
- **Recomendado:** padrão proposto.
- **Implementado:** código/configuração existe.
- **Testado:** teste executado e resultado registrado.
- **Verificado:** observado no painel/serviço com data.
- **Pendente:** requer validação.

Nenhum item deve ser declarado implementado/testado/verificado apenas por estar documentado.

## Referências
- [Next.js Authentication — DAL, DTO, Server Actions e Route Handlers](https://nextjs.org/docs/app/guides/authentication)
- [Next.js Data Security](https://nextjs.org/docs/15/app/guides/data-security)
- [Auth0 RBAC permissions claim](https://support.auth0.com/center/s/article/Adding-RBAC-Permissions-to-Access-Tokens)
- [PostgreSQL Row Security](https://www.postgresql.org/docs/17/ddl-rowsecurity.html)
- [PostgreSQL CREATE POLICY](https://www.postgresql.org/docs/16/sql-createpolicy.html)
- [OWASP Authorization](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [OWASP Authorization Regression Testing](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Regression_Testing_Cheat_Sheet.html)
