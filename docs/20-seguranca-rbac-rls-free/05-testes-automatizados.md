# Testes automatizados de segurança

## 1. Estratégia

| Camada | Ferramenta sugerida | Escopo |
|---|---|---|
| Unitário | Vitest | permission matrix, membership, deny-by-default |
| Integração | Vitest + PostgreSQL efêmero | roles, GRANT, RLS e contexto |
| E2E | Playwright | login, APIs, IDOR e cross-tenant |
| Estático | ESLint + TypeScript | imports server-only, tipos, padrões |
| Dependências | npm audit + revisão lockfile | vulnerabilidades conhecidas; não prova segurança |

Usar ferramentas open-source e execução compatível com GitHub Free. Nunca incluir credenciais de produção.

## 2. Fixtures sintéticas
Tenant A e B; Alice owner de A; Bruno viewer de A; Carla editor de B; usuário sem membership; usuário suspenso; projetos/documentos de ambos tenants. Não copiar dados reais.

## 3. Matriz mínima

| ID | Cenário | Resultado |
|---|---|---|
| SEC-001 | Sem sessão | 401/login |
| SEC-002 | Sessão sem permissão | 403 |
| SEC-003 | A lê recurso B por ID | 404/403 sem revelar existência |
| SEC-004 | Viewer edita | 403 |
| SEC-005 | Editor administra membros | 403 |
| SEC-006 | Usuário suspenso | 403 |
| SEC-007 | INSERT tenant B sob contexto A | RLS rejeita |
| SEC-008 | UPDATE A para B | RLS rejeita |
| SEC-009 | Contexto ausente | default deny |
| SEC-010 | Contexto A seguido de B | sem contaminação |
| SEC-011 | Runtime executa DDL/CREATE ROLE | permissão negada |
| SEC-012 | Runtime executa TRUNCATE/bypass | permissão negada |
| SEC-013 | Token audience errada/expirado/inválido | rejeitado |
| SEC-014 | Token da App B na API A | rejeitado |
| SEC-015 | Callback/returnTo não permitido | rejeitado |
| SEC-016 | Server Action sem permissão | 403/erro seguro |
| SEC-017 | Route Handler sem sessão | 401 |
| SEC-018 | DTO contém segredo/campo interno | teste falha |
| SEC-019 | Logs de erro | sem token/cookie/connection string |
| SEC-020 | Membership removido | acesso revogado |

## 4. Teste unitário de política

```ts
import { describe, expect, it } from "vitest";
import { can } from "@/lib/authz/permissions";

describe("authorization policy", () => {
  it("denies viewer from updating a project", () => {
    expect(can("viewer", "projects:update")).toBe(false);
  });
  it("allows editor to update a project", () => {
    expect(can("editor", "projects:update")).toBe(true);
  });
  it("denies unknown roles by default", () => {
    expect(can("unknown-role", "projects:read")).toBe(false);
  });
});
```

Role unit tests não provam isolamento de tenant. Integração deve executar queries como app_runtime (não owner).

## 5. Testes RLS de integração
1. Criar PostgreSQL efêmero para CI.
2. Aplicar migrations como owner/migrator de teste.
3. Inserir fixtures sintéticas.
4. Conectar como app_runtime.
5. Executar cenários com contexto na mesma transação.
6. Testar SELECT/INSERT/UPDATE/DELETE entre tenants.
7. Contexto ausente deve negar.
8. Confirmar que runtime não altera policies, cria roles, assume owner ou executa TRUNCATE.
9. Descartar banco ao final.

Não usar connection string de produção. Testar como owner mascara falhas de RLS.

## 6. CI no GitHub Free
- PR: lint, typecheck, unit, integração RLS com PostgreSQL efêmero e build.
- Não entregar secrets de produção a PR/fork.
- Repo privado Free: não depender de GitHub Environments/environment secrets.
- Sem deploy de produção a partir de PR.
- Guardar relatórios sem dados sensíveis.
- Conferir quotas de Actions.

## 7. Regressão
Mudança de rota, Server Action, tabela, policy, role, permission ou endpoint exige atualizar matriz e testes negativos. Falha de isolamento bloqueia merge.
