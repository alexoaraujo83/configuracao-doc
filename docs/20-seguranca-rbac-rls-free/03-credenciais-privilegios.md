# Credenciais, roles e privilégios

## 1. Matriz de credenciais

| Credencial | Uso | Permitido | Proibido |
|---|---|---|---|
| Auth0 Client ID | identificador da aplicação | configuração pública conforme SDK | tratar como segredo |
| Auth0 Client Secret | backend Regular Web App/M2M | variável server-side protegida | browser, NEXT_PUBLIC_, Git, logs |
| AUTH0_SECRET | sessão SDK | secret server-side por ambiente | browser, Git, logs |
| Neon runtime URL | queries backend | secret server-side do ambiente | browser, NEXT_PUBLIC_, Git |
| Neon migrator URL | migrations | pipeline restrito | runtime web, browser, Git |
| GitHub token | automação necessária | secret de escopo mínimo | source code/logs |
| Vercel env var | build/runtime | painel do projeto/ambiente | repositório e logs |

## 2. Roles PostgreSQL

| Role | LOGIN | NOINHERIT | Ownership | Finalidade |
|---|---:|---:|---:|---|
| app_owner | Não | — | schema/tabelas | DDL/ownership |
| app_migrator | Sim | Sim | Não | migrations; SET ROLE app_owner controlado |
| app_runtime | Sim | Sim | Não | queries da aplicação |

Exemplo de grants para revisar e adaptar:

\`\`\`sql
REVOKE ALL ON SCHEMA app FROM PUBLIC;
GRANT USAGE ON SCHEMA app TO app_runtime;
GRANT SELECT, INSERT, UPDATE, DELETE
ON ALL TABLES IN SCHEMA app TO app_runtime;
GRANT USAGE, SELECT
ON ALL SEQUENCES IN SCHEMA app TO app_runtime;

ALTER DEFAULT PRIVILEGES FOR ROLE app_owner IN SCHEMA app
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_runtime;
ALTER DEFAULT PRIVILEGES FOR ROLE app_owner IN SCHEMA app
GRANT USAGE, SELECT ON SEQUENCES TO app_runtime;
\`\`\`

Não conceder ALL PRIVILEGES por conveniência. Não conceder TRUNCATE, TRIGGER, REFERENCES, CREATE ou ownership ao runtime sem justificativa formal. Restringir colunas sensíveis com grants específicos.

## 3. Ambientes
- Development, Preview/Staging e Production usam credenciais distintas.
- Preview nunca acessa connection string de Production.
- Auth0 Client Secret, AUTH0_SECRET e Neon runtime credentials são separados por ambiente lógico quando a topologia Free permitir.
- Auth0 Free tenant único não fornece isolamento de tenant entre ambientes; clients separados não eliminam esse risco.
- Migration não compartilha credencial com servidor web.
- Rotacionar após exposição suspeita, saída de pessoa com acesso ou mudança de privilégio.

## 4. GitHub Free e CI
- Repositórios privados Free não oferecem GitHub Environments/environment secrets.
- CI deve executar lint, typecheck, unit tests, build e testes RLS com banco efêmero/local e dados sintéticos, sem secrets de produção.
- Não executar deploy de produção a partir de PR/fork.
- Repository secrets têm escopo mais amplo; documentar consumidor, permissões e risco se indispensáveis.
- Não tornar repo público apenas para obter recurso de secrets sem avaliar exposição de código/workflows.

## 5. Vercel Hobby
O painel Vercel pode armazenar variáveis por ambiente; alteração de variável requer novo deployment para refletir no runtime. Hobby é somente pessoal/não comercial.

## 6. Rotação
1. Revogar/rotacionar no fornecedor.
2. Atualizar somente ambientes consumidores.
3. Gerar novo deployment/restart conforme necessário.
4. Validar login, queries e integração.
5. Auditar acessos no intervalo de exposição.
6. Se versionado, remover do histórico Git; rotação continua obrigatória.
7. Registrar incidente sem copiar valor secreto.

## Checklist
- [ ] Sem secrets em Git/artifacts.
- [ ] Sem variáveis sensíveis NEXT_PUBLIC_.
- [ ] Runtime e migration têm credenciais distintas.
- [ ] app_runtime não é owner e não tem BYPASSRLS.
- [ ] Sem DDL/TRUNCATE no runtime.
- [ ] Preview não usa secrets de Production.
- [ ] Logs não imprimem connection strings, tokens ou cookies.
- [ ] Rotação/revogação têm responsável e procedimento.
