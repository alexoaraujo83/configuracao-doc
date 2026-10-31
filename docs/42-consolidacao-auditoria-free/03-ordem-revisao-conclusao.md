# 03 — Ordem de revisão e conclusão

## Ordem sugerida para revisão técnica
1. Arquitetura e implementação modular (#27 e #28, já incorporados à main).
2. Contratos HTTP e persistência (#29 e #30).
3. Segurança integrada (#31).
4. CI/CD e ambientes (#32 e #33).
5. Testes e operação (#34 e #35).
6. Consolidação e critérios de aceite (#36).

Os conteúdos #29–#36 foram reunidos no PR #37 para evitar conflitos repetidos nos arquivos README.md e docs/README.md. Os PRs originais serão encerrados como superseded somente após confirmação de que todos os arquivos foram copiados e o PR #37 estiver incorporado.

## Gate documental
- [ ] Revisar todos os documentos integrados e comparar com main.
- [ ] Confirmar que todos os links internos resolvem após merge.
- [ ] Revalidar versões, quotas Free e termos de uso.
- [ ] Confirmar que nenhuma instrução exige recurso pago.
- [ ] Verificar placeholders, secrets e comandos perigosos.
- [ ] Atualizar changelog e fontes oficiais.

## Gate operacional — etapa posterior autorizada separadamente
- [ ] Instalação limpa reproduzível.
- [ ] Typecheck, lint e build.
- [ ] Testes unitários e de componentes.
- [ ] Integração com PostgreSQL e migrations em banco não produtivo.
- [ ] Testes RLS com role runtime e cenários cross-tenant.
- [ ] Login/logout Auth0 em cada ambiente.
- [ ] E2E Playwright.
- [ ] Deploy Preview/Production elegível no Hobby.
- [ ] Backup/restore comprovado em ambiente isolado.
- [ ] Revisão de segurança e quotas.

## Conclusão
A incorporação documental será concluída após revisão e merge do PR #37. A validação operacional permanece pendente e será executada em etapa posterior, com instalação, credenciais, ambientes e dados de teste apropriados. Nenhum provisionamento, migration, teste, backup, restore ou deployment foi executado nesta etapa.
