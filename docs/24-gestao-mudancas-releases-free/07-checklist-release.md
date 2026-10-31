# Checklist de release — somente Free

## Antes do PR

- [ ] Issue e critérios de aceite definidos.
- [ ] Branch criada a partir de main atualizada.
- [ ] Mudança coesa e commits compreensíveis.
- [ ] Sem secrets, dados reais ou arquivos .env.
- [ ] API/contratos e impacto SemVer avaliados.
- [ ] CHANGELOG atualizado.
- [ ] Migrations numeradas, imutáveis e revisadas.
- [ ] Compatibilidade com versão anterior avaliada.
- [ ] Plano de rollout e rollback escrito.

## CI e Pull Request

- [ ] npm ci reproduzível com lockfile.
- [ ] lint e typecheck aprovados.
- [ ] testes unitários/integrados aprovados.
- [ ] build Next.js concluído.
- [ ] dependências e Actions revisadas.
- [ ] permissions mínimas nos workflows.
- [ ] nenhum secret acessível a código de PR não confiável.
- [ ] Preview deployment validado quando disponível.
- [ ] Auth0 callback/URLs e variáveis Preview revisados.
- [ ] Neon branch/test database isolado com dados sintéticos.

## Aprovação e release

- [ ] revisão técnica registrada.
- [ ] mudanças de auth/RLS/schema receberam revisão de segurança.
- [ ] quota e elegibilidade Free conferidas.
- [ ] Vercel Hobby usado apenas para projeto pessoal/não comercial.
- [ ] merge aprovado conforme política.
- [ ] deployment de produção identificado após merge.
- [ ] tag anotada e release notes/changelog publicados, se aplicável.

## Pós-release

- [ ] smoke test de páginas e rotas críticas.
- [ ] login/logout/callback testados.
- [ ] leitura/escrita e isolamento multi-tenant verificados.
- [ ] migration version/checksum confirmados.
- [ ] logs e métricas revisados.
- [ ] erros e latência dentro da meta interna.
- [ ] rollback possível e limites Hobby conhecidos.
- [ ] Issue encerrada e evidências registradas.

## Gate GO/NO-GO

GO somente quando checks críticos passaram, aprovação está registrada, migration é compatível, deployment e rollback estão identificados e não há dependência de recurso pago.

NO-GO se houver secret exposto, CI crítico falho, migration destrutiva sem recuperação validada, incompatibilidade código/schema, callbacks Auth0 incorretos, uso não elegível do Hobby ou quota insuficiente.

## Estado

Checklist documental. Nenhum release, tag, migration, deployment, aprovação de produção ou teste real foi executado nesta etapa.
