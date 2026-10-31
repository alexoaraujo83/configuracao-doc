# 07 — Checklist de segurança e resposta a incidentes

## Checklist de implementação

### Identidade e sessão
- [ ] SDK Auth0 v4 compatível com versão Next.js.
- [ ] Callback/logout allowlisted por ambiente.
- [ ] AUTH0_SECRET distinto e protegido por ambiente.
- [ ] Cookies HttpOnly, Secure em produção e SameSite apropriado.
- [ ] Tokens não expostos a Client Components.
- [ ] Redirects validados contra open redirect.

### Aplicação e API
- [ ] Cada Route Handler e Server Action verifica sessão.
- [ ] Cada operação verifica permissão e ownership/membership.
- [ ] CSRF/Origin tratados para mutações baseadas em cookie.
- [ ] CORS mínimo e sem wildcard + credentials.
- [ ] Schemas e limites de payload aplicados.
- [ ] Respostas privadas usam private, no-store.
- [ ] Erros não expõem stack trace nem segredos.

### Banco
- [ ] Runtime role não é owner nem BYPASSRLS.
- [ ] Grants mínimos e default privileges revisados.
- [ ] RLS + FORCE em todas as tabelas tenant-scoped.
- [ ] Contexto tenant/user definido dentro da mesma transação.
- [ ] Consultas parametrizadas e identificadores via allowlist.
- [ ] Testes negativos cross-tenant aprovados.
- [ ] Migrations usam role e conexão apropriadas.

### Segredos e CI
- [ ] Secrets ausentes de Git, logs e artifacts.
- [ ] Actions permissions mínimas.
- [ ] Actions pinadas em SHA completo revisado.
- [ ] Não usar pull_request_target para executar código não confiável de fork.
- [ ] Workflow Preview não recebe secrets de produção.
- [ ] Dependências e lockfile revisados.

## Casos de teste negativos

| Caso | Resultado esperado |
|---|---|
| Sem sessão | 401 ou redirect de login |
| Sessão válida sem membership | 403/404 conforme política |
| Tenant adulterado no body | rejeitado |
| ID de recurso de outro tenant | sem acesso |
| Role insuficiente | 403 |
| Origin não permitido em mutação | rejeitado |
| Payload inválido | 400/422 |
| SQL injection em valor | tratado como dado |
| RLS sem tenant context | zero linhas / negação |
| Preview tentando usar DB production | bloqueado por configuração |
| Secret ausente | falha controlada sem revelar valor |

## Resposta a incidente

1. Classificar o incidente: sessão, credencial, autorização, banco, pipeline ou exposição de dados.
2. Preservar evidências sem copiar tokens, cookies ou dados pessoais para tickets.
3. Conter: desabilitar credencial comprometida, restringir deployment ou suspender operação conforme autoridade aprovada.
4. Rotacionar segredos no provedor correspondente.
5. Avaliar invalidação de sessões, conexões e tokens.
6. Determinar escopo: usuários, tenants, registros, deployments e período.
7. Recuperar a partir de fonte validada; backup/PITR não substitui análise de integridade.
8. Testar correção e revisar logs.
9. Registrar causa raiz, impacto, ações e prevenção.
10. Comunicar responsáveis conforme obrigações aplicáveis.

Este runbook é orientação. Nenhuma contenção, rotação, recuperação ou notificação foi executada.

## Gate de produção

Não considerar pronto até que:
- todos os casos negativos críticos tenham evidência de teste;
- RLS e autorização tenham sido verificados com role runtime;
- ambientes estejam isolados;
- secrets estejam configurados e revisados;
- logs e cache não exponham dados;
- rollback e recuperação estejam documentados;
- restrições Free e elegibilidade comercial da Vercel estejam confirmadas.

## Fontes

- https://docs.github.com/en/actions/reference/security/secure-use
- https://docs.github.com/en/actions/reference/security/securely-using-pull_request_target
- https://owasp.org/www-project-top-ten/
- https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
