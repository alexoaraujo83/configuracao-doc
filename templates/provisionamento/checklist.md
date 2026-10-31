# Checklist de provisionamento por projeto

## Registro
- [ ] Project slug reservado.
- [ ] Proprietário e classificação de dados definidos.
- [ ] Manifesto preenchido sem segredos.
- [ ] Uso comercial/pessoal avaliado.
- [ ] Elegibilidade dos planos Free confirmada.

## GitHub
- [ ] Repositório e visibilidade definidos.
- [ ] Branch padrão e fluxo de PR definidos.
- [ ] CI funciona sem secrets de produção.
- [ ] Permissões mínimas e Actions revisadas.
- [ ] Nenhum secret no Git, logs ou artifacts.

## Auth0
- [ ] Tenant Free existente identificado.
- [ ] Limite de tenant único considerado no desenho.
- [ ] Applications/clients definidos conforme quotas.
- [ ] Callback, logout e Web Origins específicos.
- [ ] Client secrets armazenados fora do Git.
- [ ] Login, callback e logout testados.

## Neon
- [ ] Quotas Free verificadas.
- [ ] Projeto e branches dentro do limite.
- [ ] Roles owner, migrator e runtime separadas.
- [ ] Connection strings fora do Git.
- [ ] Migrations revisadas e aplicadas apenas com aprovação.
- [ ] RLS e isolamento multiusuário testados.

## Vercel Hobby
- [ ] Elegibilidade pessoal/não comercial confirmada.
- [ ] GitHub conectado ao projeto.
- [ ] Production Branch definida.
- [ ] Development, Preview e Production variables separadas.
- [ ] Staging modelado como Preview branch-scoped.
- [ ] URLs/callbacks Auth0 atualizados.
- [ ] Deploy e rollback testados.

## Aceite
- [ ] Health check sem exposição de detalhes internos.
- [ ] Testes unitários, integração e E2E adequados executados.
- [ ] Secrets e permissões auditados.
- [ ] Quotas e limites documentados.
- [ ] Evidências registradas sem dados sensíveis.
- [ ] Estado final classificado: Documentado / Executado / Validado / Bloqueado.
