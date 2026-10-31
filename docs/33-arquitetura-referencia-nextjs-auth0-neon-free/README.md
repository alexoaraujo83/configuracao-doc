# Arquitetura de referência — Next.js + TypeScript + Auth0 + Neon

**Status:** recomendada; aplicação real pendente de validação. **Revisão:** 31/10/2026. **Planos:** exclusivamente Free.

## Objetivo e decisões

Esta arquitetura organiza a aplicação em interface, casos de uso, domínio e infraestrutura. É documentação; não instala pacotes, cria recursos, executa testes ou publica deployments.

- Next.js App Router; Server Components por padrão e Client Components somente quando necessários.
- Route Handlers para endpoints HTTP; Server Actions para mutações da interface, ambos com validação e autorização.
- Auth0 SDK v4 para sessão e identidade; não usar exemplos legados do SDK v3.
- Neon PostgreSQL acessado somente no servidor, com driver oficial e consultas parametrizadas.
- Domínio independente de Next.js, React, Auth0 e Neon.
- Autorização de negócio no backend; RLS como defesa adicional no banco.
- Variáveis e credenciais específicas por projeto/ambiente, nunca versionadas.

## Diagrama lógico

~~~text
Browser ──HTTPS/cookie──► Next.js App Router
                           ├── app/       rotas e composição
                           ├── features/  UI e casos de uso
                           ├── domain/    entidades e contratos
                           ├── server/    Auth0, Neon, policies, logs
                           └── shared/    schemas e tipos neutros
                                  │
                     ┌────────────┴─────────────┐
                     ▼                          ▼
                Auth0 SDK v4               Neon Driver
                Auth0 Free                PostgreSQL Free
~~~

O browser nunca recebe AUTH0_CLIENT_SECRET, AUTH0_SECRET, DATABASE_URL ou credenciais PostgreSQL.

## Estrutura de diretórios

~~~text
src/
├── app/
│   ├── (public)/
│   ├── (protected)/
│   ├── api/
│   ├── layout.tsx
│   └── globals.css
├── features/
│   ├── account/
│   │   ├── components/
│   │   ├── schemas/
│   │   ├── actions/
│   │   └── queries/
│   └── projects/
│       ├── components/
│       ├── schemas/
│       ├── actions/
│       └── queries/
├── domain/
│   ├── account/
│   └── projects/
├── server/
│   ├── auth/
│   ├── db/
│   │   └── repositories/
│   ├── policies/
│   └── observability/
└── shared/
    ├── config/
    ├── schemas/
    ├── errors/
    └── types/
~~~

A árvore é adaptável: não criar camadas vazias por antecipação. Preservar sempre as fronteiras server-only, domínio independente, validação de entrada e autorização por operação.

## Direção das dependências

| Camada | Pode depender de | Não deve depender de |
|---|---|---|
| app/UI | features e schemas compartilhados | driver Neon, secrets |
| features/use cases | domain, ports e schemas | SDK de fornecedor diretamente |
| domain | tipos e regras puras | Next.js, React, Auth0, Neon, process.env |
| server/infrastructure | domain e SDKs externos | componentes UI |
| shared | tipos e validação neutra | secrets e conexões |

Fluxo: rota → caso de uso → autorização → contrato de repositório → adaptador Neon. A sessão Auth0 é convertida em identidade interna antes de entrar nas regras de negócio.

## Autenticação e autorização

Autenticação responde quem é a pessoa; autorização verifica se ela pode executar uma operação sobre um recurso/tenant.

- Validar sessão no servidor.
- Identificar usuário por issuer + subject (iss + sub), não por e-mail como chave primária.
- Resolver associação e permissões internas no banco.
- Verificar tenant e ownership em cada operação sensível.
- Não aceitar userId, role ou tenantId enviados pelo browser como prova de autorização.
- Não presumir recursos Auth0 RBAC/Organizations de planos superiores no Free.
- Não confiar exclusivamente em middleware/proxy; cada operação protegida deve autorizar no servidor.

## Acesso a dados

- Importar o driver Neon somente em módulos server-only.
- Preferir connection string pooled conforme o runtime.
- Usar consultas parametrizadas; nunca concatenar entrada de usuário em SQL.
- Separar role de migration da role runtime.
- Runtime sem ownership, DDL, TRUNCATE ou BYPASSRLS.
- Habilitar RLS e, quando aplicável, FORCE ROW LEVEL SECURITY.
- Configurar contexto tenant/user e executar consultas protegidas na mesma transação.
- Usar o mecanismo transacional suportado pelo driver; chamadas HTTP independentes não formam uma transação.
- Testar policies com runtime role e dados sintéticos.

RLS e GUCs configuradas pelo backend são defesa em profundidade, não uma fronteira independente contra backend comprometido.

## Erros e logging

Converter erros de domínio para respostas HTTP estáveis: 400 validação, 401 sessão, 403 autorização, 404 recurso não visível, 409 conflito e 500 falha inesperada. Não retornar stack trace, SQL, connection string ou tokens ao cliente.

Logs estruturados podem conter timestamp, nível, correlation ID, rota, duração e categoria do erro. Redigir cookies, Authorization headers, tokens, passwords, connection strings, dados pessoais e payloads privados.

## Configuração

Validar variáveis obrigatórias no servidor. Não importar configuração secreta em Client Components. Não usar NEXT_PUBLIC_ para DATABASE_URL, AUTH0_SECRET ou AUTH0_CLIENT_SECRET. Manter .env.example com placeholders não funcionais. Alterações em variáveis Vercel requerem novo deployment para serem incorporadas.

## Critérios de qualidade

- TypeScript strict sem any injustificado.
- Validação de entrada em Route Handlers e Server Actions.
- Nenhum módulo server-only importado por Client Components.
- Nenhum SQL concatenado.
- Autorização por operação e tenant.
- Testes unitários para domínio e casos de uso.
- Integração Neon em ambiente não produtivo com dados sintéticos.
- RLS testada com runtime role.
- CI com lint, typecheck, testes e build, conforme recursos do GitHub Free.
- Sem secrets em logs ou workflows de PR não confiável.
- Elegibilidade e limites Free revalidados.

## Sequência de implementação documentada

1. Fixar Node, Next.js, TypeScript, package manager e lockfile.
2. Criar módulos de configuração server-only.
3. Integrar Auth0 SDK v4 e restringir callback/logout URLs.
4. Criar adaptador Neon e roles distintas.
5. Definir contratos de domínio e repositórios.
6. Implementar autorização server-side e RLS.
7. Padronizar erros, logging redigido e correlation ID.
8. Documentar testes e CI.
9. Revisar variáveis, elegibilidade Free e rollback.
10. Submeter por Pull Request; sem deploy sem aprovação.

## Referências internas e oficiais

- [Next.js + TypeScript](../10-nextjs-typescript/README.md)
- [Auth0 + Next.js SDK v4](../11-auth0-nextjs/README.md)
- [Neon PostgreSQL](../12-neon-postgresql/README.md)
- [Ambientes, CI e operação](../13-ambientes-testes-operacao/README.md)
- [Padrões multi-projeto Free](../32-padroes-multiplos-projetos-free/README.md)
- https://nextjs.org/docs/app
- https://auth0.github.io/nextjs-auth0/
- https://github.com/auth0/nextjs-auth0/blob/main/V4_MIGRATION_GUIDE.md
- https://neon.com/docs/serverless/serverless-driver
- https://www.postgresql.org/docs/current/ddl-rowsecurity.html

**Aviso:** arquitetura de referência; não houve instalação, geração de código, testes, migrations, provisionamento ou deploy.
