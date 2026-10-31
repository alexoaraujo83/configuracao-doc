# 05 — Rotina operacional e checklist

## Diário/semanal
- [ ] Revisar erros recentes de runtime dentro da janela disponível.
- [ ] Verificar falhas de login/callback no Auth0.
- [ ] Conferir compute, storage, egress e branches Neon.
- [ ] Conferir status dos deployments e build failures Vercel.
- [ ] Revisar workflows falhos e minutos GitHub Actions.
- [ ] Verificar alerts de dependências e lockfile.

## Mensal
- [ ] Rever limites e termos dos planos Free.
- [ ] Rever secrets e acessos administrativos.
- [ ] Confirmar branches Neon obsoletas e capacidade.
- [ ] Rever tamanho de banco e egress.
- [ ] Revisar migrations e compatibilidade de rollback.
- [ ] Verificar backup lógico e evidência do último restore testado.
- [ ] Atualizar matriz de riscos e documentação.

## Indicadores mínimos
- taxa de erro HTTP;
- p95 de latência;
- falhas de autenticação;
- compute/storage/egress Neon;
- builds/deploys e minutos Actions;
- idade do backup validado;
- incidentes abertos e tempo de resolução.

## Critério de encerramento
Operação Free documentada, limites confirmados, runbooks aprovados e procedimentos de backup/restore validados. Este projeto permanece no estado documental até execução autorizada.