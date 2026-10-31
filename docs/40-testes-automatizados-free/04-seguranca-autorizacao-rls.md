# 04 — Segurança, autorização e RLS

| Cenário | Resultado esperado |
|---|---|
| Sem sessão | 401 ou redirect |
| Sem membership | 403/404 conforme política |
| Usuário A acessa recurso B | negado |
| Tenant adulterado | negado |
| Role viewer tenta mutação | 403 |
| Origin não permitido | rejeitado |
| Payload inválido | 400/422 |
| RLS sem tenant context | zero linhas/negação |
| Tenant A insere linha de B | policy bloqueia |
| Runtime tenta DDL/TRUNCATE | permissão negada |

Executar RLS sob role runtime real, não owner/superuser. Cobrir SELECT, INSERT, UPDATE e DELETE. Testar ausência de contexto, alternância de tenant, membership inativa e acesso cross-tenant.

Para provar sensibilidade da suíte, em ambiente descartável adicionar temporariamente policy permissiva e confirmar que testes falham; remover a policy e repetir. Nunca fazer isso em produção.

E2E Auth0 usa conta de teste dedicada. Não salvar storageState do Playwright em Git/artifacts públicos. Não executar autenticação real em PR de fork com secrets.

Fontes: https://www.postgresql.org/docs/current/ddl-rowsecurity.html ; https://playwright.dev/docs/auth ; https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html