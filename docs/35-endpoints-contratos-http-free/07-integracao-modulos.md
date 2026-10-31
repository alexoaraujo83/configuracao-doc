# 07 — Integração entre módulos

## Grafo de dependências

~~~text
Route Handler / Server Action
          |
          v
    HTTP contract + schema
          |
          v
   Auth0 session / identity
          |
          v
  local user + account status
          |
          v
 membership + permission
          |
          v
      domain service
          |
          v
 transaction + tenant context
          |
          v
      repository SQL
          |
          v
       Neon + RLS
          |
          v
       public DTO
~~~

## Ordem de execução por request

1. Resolver sessão Auth0 no servidor.
2. Exigir sub e localizar perfil local ativo.
3. Validar path/query/body e tamanho do payload.
4. Resolver tenant e confirmar membership/permissão.
5. Abrir transação Neon e definir contexto local tenant/user.
6. Executar repository com SQL parametrizado.
7. Aplicar constraints e RLS.
8. Converter resultado em DTO permitido.
9. Responder com status, headers e requestId.
10. Registrar telemetria redigida.

A implementação pode agrupar etapas em helpers, mas não deve remover verificações de confiança.

## Um service, múltiplos adaptadores

Route Handler e Server Action chamam o mesmo service. Service não depende de NextRequest, FormData ou NextResponse. Repository não conhece Auth0 nem HTTP. Camada de identidade não executa CRUD de domínio.

## Transações e autorização

Para operações tenant-scoped, verificar autorização antes e, quando houver concorrência de revogação, dentro da mesma transação. Definir isolamento e bloqueios apenas quando necessários. Contexto RLS e SQL protegido devem compartilhar conexão e transação; nunca depender de estado de sessão persistente no pool.

## Fluxo de criação

POST → sessão → perfil ativo → projects:create → schema → INSERT com tenantId/userId do servidor → RLS WITH CHECK → DTO → 201 Location.

## Fluxo de leitura

GET → sessão → perfil → projects:read → cursor validado → SELECT filtrado por tenant + RLS USING → DTOs → 200 private/no-store.

## Fluxo de atualização

PATCH → sessão → perfil → projects:update → schema allowlist → UPDATE WHERE id e tenant_id → RLS USING/WITH CHECK → confirmar row count → DTO → 200. Nunca atualizar tenant_id pelo payload.

## Falha parcial

Se o commit tiver resultado desconhecido por desconexão, não repetir cegamente. Para criação com retry, considerar idempotency key armazenada com escopo por tenant/usuário e constraint única. Documentar retenção e expiração da chave.
