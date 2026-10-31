# Vercel Hobby, domínio e reconstrução de deployments

## Limites
- Hobby é somente pessoal/não comercial.
- Rollback Hobby limita-se ao deployment de produção imediatamente anterior.
- Mudanças de environment variables não afetam deployments existentes; novo deployment é necessário.
- Em 16 Sep 2026, Vercel anunciou 10 GB de Deployment Storage para Hobby e retenção dos três deployments de produção mais recentes e três deployments recentes de qualquer tipo, com exceções. Histórico de deployments não é backup de código/configuração.

## Reconstrução
Manter em Git: package.json/lockfile, next.config, tsconfig, scripts, migrations/schema sem dados, nomes/escopos/origem das env vars (nunca valores), vercel.json sem secrets.

1. Recuperar conta e email Vercel.
2. Confirmar owner, projeto, Git repo, framework, root dir, build settings e Node version.
3. Recriar env vars de fonte autorizada por ambiente.
4. Conferir nomes como AUTH0_ISSUER_BASE_URL, AUTH0_CLIENT_ID, AUTH0_CLIENT_SECRET, APP_BASE_URL e DATABASE_URL conforme o projeto.
5. Reconectar Git após revisão do repo.
6. Fazer Preview deploy, smoke tests e validar Auth/RLS.
7. Publicar produção somente para projeto pessoal/não comercial após gate GO.
8. Validar domínio, TLS, redirects, callbacks e cookies.

## Rollback/redeploy
- Instant rollback Hobby: somente deployment de produção imediatamente anterior.
- Rollback pode reutilizar build/configuração antiga e não corrige secret exposto.
- Após rotação de secret/env, fazer novo deployment; deployment anterior pode continuar usando valor antigo.
- Se rollback não for elegível, corrigir por commit revert revisado e novo deploy, respeitando quotas Hobby.
- Confirmar se rollback desativou auto-assignment de domínio e restaurar fluxo conforme documentação.
- Não promover deployment antigo sem validar commit, integridade, dependências e secrets.

## Domínio/DNS
Manter inventário de registrador/titular/email recuperação/renovação/nameservers; export da zona; registros A/AAAA/CNAME/TXT/MX e TTL; subdomínios; TLS; callbacks/allowed origins Auth0; contatos.

Recuperação: recuperar email e conta do registrador; verificar titularidade/expiração/bloqueios/nameservers; comparar zona com último export; restaurar somente registros autorizados; revalidar domínio Vercel/TLS/DNS; atualizar callbacks Auth0 após confirmar URL final; testar cookies, redirects, CORS e login.

Não presumir domínio customizado, proteção ou suporte incluídos no Free. Confirmar elegibilidade no painel.

## Referências
- [Vercel Hobby](https://vercel.com/docs/plans/hobby)
- [Rollback production deployment](https://vercel.com/docs/deployments/rollback-production-deployment)
- [Instant rollback](https://vercel.com/docs/instant-rollback)
- [Environment variables](https://vercel.com/docs/environment-variables)
- [Hobby deployment retention — Sep 16 2026](https://vercel.com/changelog/hobby-projects-now-retain-fewer-deployments-to-free-up-storage)
