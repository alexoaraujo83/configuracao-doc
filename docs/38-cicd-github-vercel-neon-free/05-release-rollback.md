# 05 — Release, promoção e rollback

## Release checklist
- [ ] PR revisado e CI aprovado.
- [ ] Lockfile atualizado e npm ci reproduzível.
- [ ] Migration revisada e compatível com versão anterior.
- [ ] Variáveis corretas no ambiente de destino.
- [ ] Auth0 callback/logout URLs adequadas.
- [ ] Preview smoke tests concluídos.
- [ ] Plano de rollback da aplicação e banco registrado.
- [ ] Cotas Free confirmadas.
- [ ] Vercel Hobby elegível para uso e repositório.
- [ ] Evidência e commit SHA registrados.

## Rollback de aplicação
Preferir revert do commit/PR e novo deployment pela integração Git, preservando trilha de auditoria. O rollback da Vercel pode ter limitações específicas no Hobby; confirmar documentação atual e o deployment alvo. Não assumir que rollback de aplicação reverte schema ou dados.

## Rollback de banco
- Migration corretiva forward quando possível.
- Restaurar backup/PITR somente após estimar perda de dados, janela e efeitos sobre writes.
- Não executar restore automaticamente após rollback de código.
- Verificar compatibilidade entre código antigo e schema atual.
- Registrar RPO/RTO realistas para o plano Free, sem prometer SLA.

## Proibição de release automático neste projeto documental
Os exemplos não autorizam merge, deploy, migration, promoção ou rollback. A operação requer autorização explícita, contexto de projeto, commit e ambiente definidos.

## Referências
- https://vercel.com/docs/deployments
- https://vercel.com/docs/git
- https://neon.com/docs/introduction/branching
- https://neon.com/docs/guides/schema-migrations
