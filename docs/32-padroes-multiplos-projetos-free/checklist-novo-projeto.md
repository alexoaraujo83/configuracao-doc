# Checklist de onboarding de novo projeto

## Antes de criar recursos
- [ ] Definir finalidade, proprietário, classificação dos dados e elegibilidade do plano.
- [ ] Confirmar se Vercel Hobby é permitido para o uso pretendido (pessoal e não comercial).
- [ ] Consultar limites Free atuais dos quatro fornecedores.
- [ ] Decidir se o Auth0 tenant Free compartilhado atende sem confundir aplicações e callbacks.
- [ ] Planejar banco/branch Neon por ambiente e quotas.

## Repositório e aplicação
- [ ] Criar perfil mestre a partir de templates/projeto/PROJECT_PROFILE.md.
- [ ] Definir identificador e nomes padronizados.
- [ ] Fixar Node, Next.js, TypeScript, package manager e lockfile.
- [ ] Definir branch padrão e revisão por Pull Request.
- [ ] Criar .env.example somente com placeholders.
- [ ] Verificar .gitignore para .env*, mantendo .env.example explicitamente permitido.

## Auth0
- [ ] Application por projeto/ambiente conforme limites.
- [ ] Callback/logout/web origins mínimos e exatos.
- [ ] SDK e versão documentados.
- [ ] Client secrets diferentes por ambiente.
- [ ] Autorização de domínio validada server-side; não depender apenas de claim RBAC.

## Neon
- [ ] Projeto e branches planejados.
- [ ] Roles separadas para migration e runtime.
- [ ] Runtime sem ownership, DDL, TRUNCATE e BYPASSRLS.
- [ ] RLS habilitada e políticas testadas com dados sintéticos.
- [ ] Preview e Development nunca usam credencial de Production.
- [ ] Estratégia de recuperação e limitações de retenção registradas.

## Vercel e CI
- [ ] Elegibilidade Hobby confirmada.
- [ ] Variáveis por target/branch inventariadas.
- [ ] Secrets adicionados sem expor valores.
- [ ] Novo deployment planejado após mudanças de variáveis.
- [ ] Workflow usa permissões mínimas e não executa código não confiável com secrets.
- [ ] CI usa dados sintéticos e não executa migrations em produção.

## Aprovação
- [ ] Documentação revisada.
- [ ] Exceções aceitas por responsável.
- [ ] Critérios de rollback documentados.
- [ ] Estado final registrado como verificado, recomendado ou pendente.
