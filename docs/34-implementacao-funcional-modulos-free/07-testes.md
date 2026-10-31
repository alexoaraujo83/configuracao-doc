# 07 — Testes unitários e de integração

| Camada | Escopo | Dependências |
|---|---|---|
| Unitário | schemas, policies, services puros, erros | mocks/fakes |
| Integração | repositories, constraints, transações, RLS | Neon isolado |
| Contrato | Route Handlers, payloads, status | Auth0/DB simulados |
| E2E | login, sessão, CRUD, tenant boundaries | ambiente dedicado |

Nunca executar contra produção. Usar dados sintéticos e branch/banco separado, respeitando quotas Free.

Unitários: validação e campos extras; matriz role→permission; membership ativa/suspensa; mapeamento HTTP; tenant e identificadores. Mockar dependências externas, não a lógica sob teste.

Integração PostgreSQL com app_runtime, nunca somente owner: CRUD, constraints, rollback, RLS SELECT/INSERT/UPDATE/DELETE, contexto local, isolamento entre transações, ausência de contexto, reuso de conexão sem vazamento e migrations em banco descartável. Fixtures devem ter ao menos dois tenants e usuários distintos.

E2E Auth0: preferir tenant/app de teste separado quando quota permitir. Não incluir credenciais em código/artefatos públicos. Se automação de login não for viável no Free, separar smoke tests com sessão simulada de validação manual controlada do Universal Login; não contornar controles do provedor.

CI GitHub Actions Free: checkout, Node fixado, instalação por lockfile, lint, typecheck, unit tests, build e integração apenas com DB de teste e secrets disponíveis. Em repositório privado Free, não presumir environments protegidos ou quotas ilimitadas. Permissões mínimas no GITHUB_TOKEN; nunca imprimir secrets; revisar actions.

Critérios: permissões com testes positivos/negativos; endpoint não depende da UI; RLS testada com role runtime; tenant A não acessa B; campos privilegiados rejeitados; sessão ausente não acessa dados; logs não expõem secrets/PII; migrations/testes repetíveis; build/typecheck/testes passam em ambiente autorizado.

Registrar commit, branch, data, versões, comandos, ambiente, resultado e limitações. Este guia não afirma que testes foram executados.
