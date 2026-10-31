# 01 — Estratégia de testes automatizados

**Estado:** especificação documental; dependências e testes não instalados/executados.
**Stack:** Next.js App Router, TypeScript, Auth0 SDK v4, Neon PostgreSQL.

## 1. Objetivos
Testar separadamente regras de domínio e autorização sem serviços externos; integração PostgreSQL com schema isolado e role runtime; fluxos de navegador com Auth0 e Next.js; build, tipos, lint e vulnerabilidades no GitHub Actions.

A suíte deve ser executável localmente sem plano pago. Testes que exigem Auth0 ou Neon reais são opt-in, usam dados de teste e não devem bloquear PR de fork por falta de secrets.

## 2. Pirâmide de testes

| Camada | Ferramenta | Dependência externa | Execução |
|---|---|---|---|
| Unitário | Vitest | Não | Todo PR |
| Componente/servidor | Vitest + mocks | Não | Todo PR |
| Integração PostgreSQL | Vitest/Node test + Neon/PG de teste | Sim, opt-in | Manual/workflow protegido |
| E2E navegador | Playwright | App local; Auth0 opcional | PR interno/release |
| Segurança | Testes negativos + npm audit | Não, exceto RLS | Todo PR/release |

Vitest e Playwright são open source; a execução local não exige plano pago. Minutos de CI dependem das cotas do GitHub Free.

## 3. Instalação proposta (quando aprovada)

~~~bash
npm install -D vitest @vitest/coverage-v8
npm install -D @playwright/test
npx playwright install chromium
~~~

Scripts a adicionar ao package.json, preservando os existentes:

~~~json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test"
  }
}
~~~

A instalação é apenas instrução documental; não foi executada. Fixar versões no lockfile e revisar avisos de segurança durante implantação.

## 4. Estrutura proposta

~~~text
src/lib/authz/permissions.ts
src/lib/authz/authorize.ts
src/lib/__tests__/permissions.test.ts
tests/integration/rls-isolation.test.ts
tests/integration/migrations.test.ts
tests/e2e/auth.spec.ts
tests/e2e/access-control.spec.ts
vitest.config.ts
playwright.config.ts
~~~

Não colocar credenciais ou dumps de produção nos testes.

## 5. Testes unitários de autorização

Exemplo conceitual de função pura:

~~~typescript
export type AppRole = "reader" | "editor" | "admin";

const grants: Record<AppRole, readonly string[]> = {
  reader: ["document:read"],
  editor: ["document:read", "document:write"],
  admin: ["document:read", "document:write", "user:manage"],
};

export function hasPermission(
  roles: readonly AppRole[],
  permission: string,
): boolean {
  return roles.some((role) => grants[role]?.includes(permission));
}
~~~

Testar: reader lê mas não escreve; editor lê/escreve mas não gerencia usuários; admin recebe somente permissões explícitas; role desconhecida e lista vazia não concedem acesso. A função recebe roles obtidas no servidor, nunca do navegador.

## 6. Testes de integração PostgreSQL
Usar projeto/branch/database Neon Free separado da produção. Casos obrigatórios:
1. Conexão com role runtime.
2. Runtime não executa DDL nem cria roles.
3. Migrations aplicadas em ordem e checksum conferido.
4. Constraints e índices existentes.
5. RLS com contexto ausente nega acesso.
6. Subjects A e B não leem nem alteram dados entre si.
7. Contexto transaction-local não vaza para transação seguinte.
8. Rollback ocorre quando instrução transacional falha.
9. Restore testado em banco isolado.

## 7. Testes negativos

| Risco | Teste | Resultado |
|---|---|---|
| IDOR | A solicita recurso de B | 403 ou 404 |
| Spoofing | Cliente envia subject B na sessão A | Subject A prevalece |
| SQL injection | Payload em parâmetro SQL | Interpretado como dado |
| Escalada | reader chama endpoint admin | 403 |
| RLS | Consulta sem contexto | Negada/zero linhas |
| Segredo no cliente | Inspecionar bundle | Nenhum secret |
| Callback inválido | Redirect URI não cadastrado | Auth0 rejeita |
| Sessão ausente | Chamar rota privada | 401/redirect |

## 8. Cobertura e evidências
Cobertura percentual é indicador, não prova de segurança. Priorizar caminhos de autorização, casos negativos, falhas, acesso multiusuário, migrations e RLS. Definir limiar após medir baseline real.

Registrar no PR versões de Node/npm/Next/Auth0/Neon, comandos, resultados, testes ignorados, ambiente, database/branch de teste e limitações Free. Nunca anexar traces com cookies, tokens ou dados pessoais.
