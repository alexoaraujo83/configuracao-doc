# 1. Estratégia de testes unitários, integração e E2E

## 1.1 Princípios

- Testar comportamento observável e invariantes de segurança; não perseguir 100% de cobertura como objetivo isolado.
- Pirâmide: muitos testes unitários rápidos, integração nas fronteiras relevantes e um conjunto menor de E2E críticos.
- Testes determinísticos, repetíveis, sem dependência de dados pessoais ou serviços de produção.
- Falha de autorização deve ser testada como resultado esperado, não apenas o caminho feliz.
- Cada teste deve ter identificador, requisito associado, resultado esperado e evidência.
- Componentes Server assíncronos e fluxos completos devem ser validados por E2E quando a ferramenta unitária não os suportar integralmente. A documentação Next.js registra essa limitação do Vitest para Async Server Components.

## 1.2 Camadas e escopo

| Camada | Ferramenta de referência | O que validar | Dependências |
|---|---|---|---|
| Unitário | Vitest | funções puras, schemas, normalização, regras de negócio, policy helpers | mocks determinísticos |
| Componente | Vitest + Testing Library | renderização, acessibilidade, estados e interações | DOM simulado |
| Integração | Vitest + PostgreSQL isolado | DAL, Route Handlers, Server Actions, transações, persistência e erros | DB de teste |
| Contrato HTTP | Vitest/fetch | status, headers, schema de request/response, erros | dependências controladas |
| E2E | Playwright | login, logout, navegação, formulários, fluxos críticos e permissões | servidor de teste e browser |
| Build | Next.js build | compilação, imports server/client, variáveis obrigatórias e rotas | configuração de teste |
| Banco | PostgreSQL/Neon branch isolada | migrations, constraints, grants, RLS e integridade | role de runtime |

## 1.3 Organização sugerida

~~~text
src/
  app/
  lib/
    auth/
    authorization/
    db/
    validation/
tests/
  unit/
  integration/
  e2e/
  fixtures/
  contracts/
~~~ 

A estrutura é ilustrativa; adaptar à organização real do repositório.

## 1.4 Scripts de projeto

Definir scripts equivalentes no package.json, ajustados às dependências realmente instaladas:

~~~json
{
  "scripts": {
    "lint": "eslint .",
    "typecheck": "tsc --noEmit",
    "test:unit": "vitest run tests/unit",
    "test:integration": "vitest run tests/integration",
    "test:e2e": "playwright test",
    "build": "next build"
  }
}
~~~

Não declarar estes scripts como existentes antes de implementá-los. Fixar versões compatíveis no lockfile e usar npm ci no CI.

## 1.5 Casos mínimos unitários

- Validação de payload: campos ausentes, tipos inválidos, limites e valores inesperados.
- Mapeamento de claims Auth0 para principal interno: issuer e subject obrigatórios; nunca confiar em e-mail como chave imutável.
- RBAC: papel permitido, papel insuficiente, permissão desconhecida, usuário sem membership e tenant inativo.
- DAL: ausência de sessão, sessão expirada, identidade válida sem permissão.
- Tratamento de erros: não vazar stack trace, token, cookie, SQL ou connection string.
- Segurança: entrada não confiável não altera tenantId/userId definido pelo servidor.
- Serialização: DTO não inclui campos internos, tokens ou segredos.

## 1.6 Integração

Executar contra banco descartável e credenciais de privilégio mínimo. Validar:
- commit/rollback e atomicidade;
- constraints NOT NULL, UNIQUE, CHECK e foreign keys;
- comportamento de timeout, conexão interrompida e transação abortada;
- operações DAL e Route Handler com identidade autenticada e sem identidade;
- idempotência quando aplicável;
- concorrência em operações sensíveis;
- ausência de efeitos colaterais entre testes.

Não simular PostgreSQL com SQLite quando o comportamento depende de RLS, roles, transações ou extensões específicas.

## 1.7 E2E com Playwright

Cobrir, no mínimo:
1. usuário anônimo abre rota protegida e é redirecionado ao login;
2. login válido retorna ao callback permitido;
3. logout encerra sessão e impede reuso;
4. sessão expirada exige autenticação novamente;
5. usuário autenticado sem permissão recebe 403/negação equivalente;
6. usuário de tenant A não consulta nem altera dados do tenant B;
7. formulários validam entradas e exibem erro sem expor detalhes internos;
8. navegação crítica funciona no build de produção de teste.

Preferir executar contra build de produção local (build + start), não apenas dev server, para detectar diferenças de runtime.

## 1.8 Qualidade dos testes

- Testes independentes e repetíveis; limpar fixtures dentro de transação quando possível.
- Fixtures sintéticas identificadas como TEST, nunca copiar produção.
- Evitar sleeps arbitrários; aguardar condições explícitas.
- Capturar trace/screenshot apenas em falha e remover dados sensíveis antes de armazenar.
- Quarentena de teste flaky exige issue, responsável, prazo e justificativa; não converter falha em sucesso silencioso.
- Cobertura é métrica auxiliar. Definir thresholds por módulo crítico após baseline real, sem inventar percentuais universais.

## 1.9 Critério de conclusão

A camada só pode ser marcada como Aprovada quando: comando, commit, ambiente, versão Node/npm, quantidade de testes, falhas, duração e link/artefato de evidência estiverem registrados. Testes documentados ou workflow criado não provam execução.
