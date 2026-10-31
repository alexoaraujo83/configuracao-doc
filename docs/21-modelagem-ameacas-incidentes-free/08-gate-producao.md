# Gate de segurança e aprovação de produção

Aprovar somente com evidência vinculada e owner. Marcar NO-GO quando requisito crítico falhar, estiver desconhecido ou depender de recurso pago não contratado.

| Controle | Evidência exigida | Estado |
|---|---|---|
| Inventário e fronteiras atualizados | inventário revisado e owner | Pendente |
| STRIDE revisado | ameaças, risco residual e decisões | Pendente |
| OIDC validado | testes issuer/audience/exp/assinatura/callback | Pendente |
| Autorização server-side | testes DAL/Action/Handler | Pendente |
| Isolamento multi-tenant | leitura/escrita/export e RLS cross-tenant | Pendente |
| Role runtime mínima | grants e atributos verificados | Pendente |
| Segredos protegidos | varredura repo/log/build/browser e rotação comprovável | Pendente |
| GitHub Actions endurecido | SHA pin, permissions, fork safety, review | Pendente |
| Vercel íntegro | commit/autor/env/domínio/TLS e smoke tests | Pendente |
| Runbooks testados | tabletop executado e lacunas registradas | Pendente |
| Recuperação validada | backup/restore testado, RPO/RTO medidos | Pendente |
| Evidências adequadas | redaction, acesso, retenção e export confirmados | Pendente |
| Plano Free confirmado | quotas/limites atuais, sem dependência paga | Pendente |
| Elegibilidade Vercel | Hobby apenas pessoal/não comercial | Pendente |

## Bloqueios absolutos
- Qualquer leitura/escrita cross-tenant ou IDOR não corrigido.
- Runtime role owner, migrator, superuser ou BYPASSRLS.
- RLS ausente em tabela sensível ou teste negativo falhando.
- Autorização apenas na UI ou token não validado.
- Segredo de produção em browser, repo, logs, artifacts ou histórico sem rotação.
- Workflow/Action com permissão excessiva ou código não confiável com secrets.
- Deployment/domínio/callback sem owner verificável.
- Restore não testado quando os dados exigem recuperação.
- Incidente crítico aberto ou risco crítico sem decisão formal.
- Dependência de recurso pago, trial ou limite não confirmado.
- Uso comercial em Vercel Hobby.

## Aprovação formal
Projeto/versão/commit; ambiente; responsável técnico; responsável segurança; data UTC; evidências; riscos aceitos e justificativa; pendências/owner/prazo; decisão GO/NO-GO; aprovadores.

## Pós-release
Monitorar autenticação, autorização, erros DB, quotas, deployments e mudanças de configuração. Definir janela e critérios de rollback antes do deploy. Hobby limita rollback ao deployment de produção imediatamente anterior; rollback pode restaurar configuração antiga e não substitui rotação de secrets.

## Declaração
Este gate é documental. Nenhum teste, auditoria, backup, restore, deploy, rollback, rotação ou verificação de painel é declarado como realizado.
