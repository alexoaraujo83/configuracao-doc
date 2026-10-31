# 03 — Integração, contratos e banco

Testar service + repository com PostgreSQL isolado. Não usar produção. Criar dados por teste e limpar deterministicamente.

Para cada endpoint testar método/rota, autenticação, autorização por recurso/tenant, request schema, DTO, status, cache headers, paginação, idempotência e conflitos.

Neon: usar PostgreSQL local para testes rápidos ou branch isolada quando quotas Free permitirem. Não criar branch por PR automaticamente sem orçamento/expiração. CI de PR não recebe DATABASE_URL de produção.

Testar rollback, atomicidade, concorrência e contexto local de tenant. Não presumir que chamadas HTTP independentes do driver compartilham sessão/transação; usar API transacional oficialmente suportada.

Migration: aplicar do zero, atualizar schema anterior, validar constraints/índices e compatibilidade expand-contract.