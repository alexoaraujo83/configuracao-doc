# Endpoints e contratos HTTP — Next.js + Auth0 + Neon (Free)

**Escopo:** especificação documental de API para a aplicação de referência. Não representa endpoints já implementados. Exemplos não executados; nenhum pacote, serviço, migration, secret ou deployment foi alterado.

## Convenções

- Next.js App Router e TypeScript; Route Handlers em src/app/api.
- Auth0 SDK v4 para sessão; identidade sempre obtida no servidor.
- Neon PostgreSQL acessado somente no servidor.
- JSON UTF-8; datas em ISO 8601 UTC; IDs UUID.
- Prefixo de API: /api/v1. Recursos tenant-scoped sempre incluem tenantId e verificam membership/permissão.
- Não aceitar do cliente auth0_sub, userId privilegiado, role, tenant_id efetivo, created_by ou flags administrativas.
- AuthN precede AuthZ; validação precede service; service chama repository em transação; RLS atua como defesa adicional.
- Somente planos Free. Não depender de recursos pagos, Organizations ou quotas não confirmadas.

## Documentos

1. [Mapa de endpoints e contratos](01-mapa-endpoints.md)
2. [Route Handlers](02-route-handlers.md)
3. [Server Actions](03-server-actions.md)
4. [Payloads e respostas](04-payloads-respostas.md)
5. [Paginação e filtros](05-paginacao.md)
6. [Erros HTTP e segurança](06-erros-seguranca.md)
7. [Integração entre módulos](07-integracao-modulos.md)
8. [Testes de contrato e critérios de aceite](08-testes-contratos.md)

## Fluxo de referência

Request → sessão Auth0 → identidade sub → perfil local ativo → membership/permission → schema de entrada → service → transação Neon com contexto tenant → repository SQL parametrizado → RLS/constraints → DTO público → resposta HTTP.

## Limites de confiança

O browser controla URL, query, headers e body. Nunca confiar neles para identidade, role ou autorização. O tenantId solicitado precisa ser confirmado contra membership ativa. Não retornar linhas de banco diretamente; mapear DTOs. Nunca expor tokens, cookies, connection strings, stack traces ou mensagens SQL.

## Versionamento

Mudanças incompatíveis exigem nova versão de API ou plano de depreciação. Mudanças aditivas devem preservar consumidores. Server Actions são interfaces internas de mutação e não substituem contratos HTTP públicos para integrações externas.
