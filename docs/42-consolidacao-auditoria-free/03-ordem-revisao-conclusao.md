# 03 — Ordem de revisão e conclusão

## Ordem sugerida para revisão técnica
1. Arquitetura e implementação modular (#27 e #28, já incorporados à main).
2. Contratos HTTP e persistência (#29 e #30).
3. Segurança integrada (#31).
4. CI/CD e ambientes (#32 e #33).
5. Testes e operação (#34 e #35).
6. Consolidação e critérios de aceite (#36).

Os conteúdos #29–#36 foram reunidos no PR #37 para evitar conflitos repetidos nos arquivos README.md e docs/README.md. O PR #37 foi incorporado à `main` em 31/10/2026 (merge commit `a315eddc465d87ecf1852a5b5d1a82806f5d508d`); os PRs originais #29–#36 foram encerrados como substituídos.

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
A incorporação documental foi concluída pelo merge do PR #37 na `main`. A inspeção realizada é estática e documental: o repositório não contém uma aplicação Next.js instalável na raiz. Typecheck, lint, build, testes e E2E não foram executados e não se aplicam ao conteúdo Markdown isolado. Integrações Auth0/Neon, migrations, backup/restore e deployment estão fora do escopo desta etapa documental e não foram executados.
