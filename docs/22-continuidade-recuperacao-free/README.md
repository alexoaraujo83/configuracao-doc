# Continuidade de negócio e recuperação de desastres — Free-only

Revisão documental: 2026-10-31. Stack: Next.js + TypeScript + Auth0 + Neon PostgreSQL + GitHub Free + Vercel Hobby.

Estado: proposta documental. Nenhum backup, restore, failover, recuperação de conta, DNS ou deployment foi executado.

## Escopo
- Somente recursos incluídos nos planos Free; sem trials, créditos promocionais ou add-ons.
- Vercel Hobby é exclusivamente pessoal/não comercial.
- RTO/RPO são metas internas, não SLA ou garantia de fornecedor.
- A documentação Neon consultada limita o history window do Free a 6 horas. Confirmar o valor efetivo em Settings → Instant restore.
- Backup gerado não é válido até checksum e restore de teste.
- Não guardar dumps, tokens, passwords ou dados pessoais em Git, issues, Actions artifacts ou logs.

## Índice
1. [BIA, RTO/RPO](./01-bia-rto-rpo.md)
2. [Neon backup e restore](./02-neon-backup-restore.md)
3. [Auth0 recuperação](./03-auth0-recuperacao.md)
4. [GitHub recuperação](./04-github-recuperacao.md)
5. [Vercel, domínio e deploy](./05-vercel-dominio-deploy.md)
6. [Runbooks](./06-runbooks.md)
7. [Testes periódicos](./07-testes-periodicos.md)
8. [Gate de continuidade](./08-gate-continuidade.md)

## Princípios
Preparar inventário, contatos, recovery codes, exportações e cópias offline. Restaurar em ambiente isolado, reconciliar gravações posteriores, validar Auth0, DATABASE_URL, migrations, RLS, DNS e TLS e medir tempos reais em exercícios.

## Referências
- [Neon history window](https://neon.com/docs/introduction/branching#history-window)
- [Neon restore a branch](https://neon.com/docs/guides/branching-neon#restore-a-branch-to-a-previous-point-in-time)
- [Neon pg_dump/pg_restore](https://neon.com/docs/import/migrate-from-postgres)
- [Auth0 backup/export](https://support.auth0.com/center/s/article/Backup-Auth0-Data)
- [Auth0 Deploy CLI](https://github.com/auth0/auth0-deploy-cli)
- [GitHub 2FA recovery](https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/recovering-your-account-if-you-lose-your-2fa-credentials)
- [Vercel rollback](https://vercel.com/docs/deployments/rollback-production-deployment)
- [Vercel environment variables](https://vercel.com/docs/environment-variables)
- [Vercel Hobby retention](https://vercel.com/changelog/hobby-projects-now-retain-fewer-deployments-to-free-up-storage)
- [NIST SP 800-34 Rev. 1](https://csrc.nist.gov/pubs/sp/800/34/r1/upd1/final)
