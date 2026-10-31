# 7. Checklist de implantação

## Antes de começar
- [ ] Projeto é pessoal/não comercial se hospedado no Hobby.
- [ ] Planos e quotas Free conferidos na data.
- [ ] Objetivo, responsável e ambientes definidos.
- [ ] Node LTS, npm, Git e versões framework definidas.

## Aplicação
- [ ] Next.js App Router + TypeScript.
- [ ] Dependências no package-lock.
- [ ] .env.example sem valores reais.
- [ ] .env.local ignorado pelo Git.
- [ ] Auth0/Neon acessíveis somente server-side.
- [ ] Validação de ambiente sem expor secrets.

## Auth0
- [ ] Regular Web Application criada.
- [ ] Credenciais armazenadas com segurança.
- [ ] AUTH0_SECRET aleatório e distinto por ambiente.
- [ ] SDK v4 e variáveis v4.
- [ ] Middleware/Proxy compatível com Next.js.
- [ ] Callback/logout URLs exatas.
- [ ] Autorização de domínio além da autenticação.

## Neon
- [ ] Projeto e branches identificados.
- [ ] DATABASE_URL por ambiente.
- [ ] Runtime role com privilégio mínimo.
- [ ] Migrations versionadas e revisadas.
- [ ] RLS revisada para tabelas multi-tenant.
- [ ] Preview isolado de produção.
- [ ] Backup/restore e quotas Free documentados.

## GitHub
- [ ] Repositório e branch padrão definidos.
- [ ] PR e revisão estabelecidos.
- [ ] CI com lint, typecheck e build.
- [ ] Actions com permissões mínimas.
- [ ] Nenhum secret de produção em workflow de PR.
- [ ] Diff revisado antes de merge.

## Vercel
- [ ] Elegibilidade Hobby confirmada.
- [ ] Repositório conectado.
- [ ] Root directory e comandos revisados.
- [ ] Variáveis Config/Secret segregadas.
- [ ] Callback Auth0 compatível com hosts.
- [ ] DATABASE_URL de preview não é produção.
- [ ] Novo deployment após mudança de variável.
- [ ] Rollback e retenção revisados.

## Gate final
- [ ] Build e testes aprovados.
- [ ] Login/logout validados.
- [ ] Autorização e isolamento multi-tenant validados.
- [ ] Logs sem tokens, cookies, secrets ou PII desnecessária.
- [ ] Smoke tests não destrutivos aprovados.
- [ ] Aprovação humana registrada.
- [ ] Pendências documentadas.

Marcar itens somente após execução e evidência verificável. A inclusão deste checklist no repositório não significa que implantação ocorreu.
