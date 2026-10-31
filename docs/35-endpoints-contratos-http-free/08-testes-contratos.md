# 08 — Testes de contrato e critérios de aceite

## Contratos por endpoint

Para cada rota documentar: método, path, auth requerida, permissão, schema, status de sucesso, status de erro, headers, DTO, cache e efeitos colaterais. Versionar fixtures JSON de request/response sem dados reais.

## Matriz de testes

- Sessão ausente/expirada: 401.
- Usuário local suspenso: 403.
- Membership ausente/inativa: negar acesso.
- Permissão insuficiente: 403 ou 404 conforme política de ocultação.
- Path UUID inválido, query inválida e body malformado: 400.
- Campo privilegiado extra: rejeitado.
- Criar: 201, Location e DTO sem campos internos.
- Atualizar: somente allowlist; recurso de outro tenant não é alterado.
- Excluir: status definido e idempotência documentada.
- Paginação: cursor inválido/adulterado/de outro tenant rejeitado.
- RLS: consultas cruzadas negadas com role app_runtime.
- Erros: sem stack trace, tokens, SQL ou PII indevida.
- Cache: respostas autenticadas/tenant-scoped não são compartilhadas.

## Unitários e integração

Unitários testam schemas, mapeadores e services com dependências controladas. Integração testa Route Handlers, sessão simulada e banco Neon isolado. Executar RLS com credencial app_runtime real; não validar somente como owner. E2E usa dados sintéticos e ambiente dedicado, nunca produção.

## OpenAPI

Quando houver consumidores externos, manter OpenAPI versionado com schemas, security schemes, exemplos e respostas. Gerar/validar especificação em CI, se ferramentas e quotas Free permitirem. Não declarar endpoints implementados até existirem arquivos e testes correspondentes.

## Critérios de aceite

- Todos os endpoints da matriz têm contrato e autorização definida.
- DTOs são explícitos e não vazam colunas internas.
- Route Handlers e Server Actions compartilham services.
- Toda mutação revalida sessão, membership e permissão.
- Queries tenant-scoped usam transação/contexto RLS.
- Paginação é determinística e cursor validado.
- Erros e headers seguem convenção.
- Testes positivos e negativos cobrem tenant boundary.
- CI não imprime secrets e usa apenas dados sintéticos.
- Versões reais de Next.js, Auth0 SDK, driver Neon e bibliotecas são registradas.

Este material é especificação documental; nenhum endpoint, contrato OpenAPI ou teste foi implementado/executado nesta etapa.
