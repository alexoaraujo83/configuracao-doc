# Checklist de novo projeto

## Planejamento
- [ ] Finalidade e classificação dos dados definidos.
- [ ] Elegibilidade Vercel Hobby pessoal/não comercial confirmada.
- [ ] Quotas/recursos Free Auth0, Neon e GitHub verificados.
- [ ] Owners e responsáveis definidos.
- [ ] Ambientes e estratégia de isolamento aprovados.

## Fundação
- [ ] Repositório criado e visibilidade aprovada.
- [ ] Next.js App Router + TypeScript scaffold documentado.
- [ ] Node.js/npm e versões fixados.
- [ ] Lockfile e .gitignore revisados.
- [ ] Estrutura de diretórios aprovada.

## Auth0
- [ ] Tenant/application e owners registrados.
- [ ] Callback/logout/origin configurados por ambiente.
- [ ] SDK v4 compatível com a versão Next.js.
- [ ] Sessão e autorização server-side verificadas.

## Neon
- [ ] Projeto/database/branches documentados.
- [ ] Roles runtime/migration separadas.
- [ ] RLS avaliada e policies revisadas.
- [ ] Connection strings privadas.
- [ ] Migrations versionadas.

## GitHub e Vercel
- [ ] CI sem secrets e com permissões mínimas.
- [ ] PR review definida.
- [ ] Vercel Hobby elegível.
- [ ] Variáveis por ambiente revisadas.
- [ ] Preview validado com dados sintéticos.

## Aceite
- [ ] Typecheck, lint e build executados e evidenciados.
- [ ] Testes de autenticação/autorização e banco registrados.
- [ ] NCs/exceções revisadas.
- [ ] Decisão de release registrada.
- [ ] Nenhum segredo ou dado pessoal em artefatos públicos.
