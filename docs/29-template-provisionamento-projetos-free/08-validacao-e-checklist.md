# Validação e checklist de inicialização

## Gate 0 — elegibilidade Free

- [ ] Finalidade do projeto é pessoal e não comercial para uso de Vercel Hobby.
- [ ] Recursos Auth0 necessários existem no Free.
- [ ] Quotas Neon suportam topologia proposta.
- [ ] GitHub Free suporta controles planejados.
- [ ] Nenhum trial, crédito, cartão ou add-on pago é pressuposto.
- [ ] Limitações e data de consulta foram registradas.

Qualquer incompatibilidade de plano que afete requisito obrigatório resulta em NO-GO.

## Gate 1 — fundação local

- [ ] Node/npm/Git e versões registrados.
- [ ] Next.js App Router e TypeScript configurados.
- [ ] Lockfile versionado.
- [ ] .gitignore protege arquivos de ambiente.
- [ ] .env.example contém apenas placeholders.
- [ ] Build, lint e typecheck executados e evidenciados.

## Gate 2 — identidade

- [ ] Tenant/application identificados.
- [ ] URLs de callback/logout/origin revisadas.
- [ ] SDK Auth0 v4 compatível com Next.js instalado.
- [ ] Login/logout testados.
- [ ] Sessão validada no servidor.
- [ ] Autorização server-side testada com dados sintéticos.
- [ ] Secrets fora do Git.

## Gate 3 — banco

- [ ] Neon Free quotas verificadas.
- [ ] Branch/database e roles documentados.
- [ ] Runtime com privilégio mínimo.
- [ ] Connection string privada.
- [ ] Migrations revisadas e testadas fora de produção.
- [ ] RLS e isolamento avaliados.
- [ ] Recuperação e limitações Free documentadas.

## Gate 4 — GitHub e CI

- [ ] Repositório e visibilidade aprovados.
- [ ] PR review definida.
- [ ] Workflow CI sem secrets.
- [ ] Permissões mínimas.
- [ ] Limites Actions verificados.
- [ ] Nenhum merge automático.

## Gate 5 — Vercel

- [ ] Hobby elegível para uso pessoal/não comercial.
- [ ] Projeto ligado ao repositório correto.
- [ ] Variáveis por ambiente revisadas.
- [ ] Preview testado com dados sintéticos.
- [ ] Callback Auth0 e Neon correspondem ao ambiente.
- [ ] Limites e rollback documentados.

## Gate 6 — release

- [ ] Commit e versão identificados.
- [ ] Evidências críticas válidas.
- [ ] NCs e exceções revisadas.
- [ ] Plano de recuperação documentado.
- [ ] Decisão GO/GO condicionado/NO-GO registrada.
- [ ] Responsável técnico e autoridade de release identificados.

## Evidências mínimas

Para cada gate, guardar referência de commit, data/hora, ambiente, resultado, responsável e artefato sanitizado. Não armazenar tokens, connection strings, dados pessoais ou logs brutos no repositório público.

## Critério de conclusão

Um projeto está documentalmente preparado quando templates foram preenchidos e revisados. Só pode ser marcado como configurado/verificado após evidências reais obtidas no painel, CLI ou execução autorizada. A documentação por si só não comprova provisionamento, teste, deploy ou conformidade.
