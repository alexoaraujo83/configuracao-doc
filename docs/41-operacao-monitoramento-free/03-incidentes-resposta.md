# 03 — Incidentes e resposta

## Severidade operacional
- S1: suspeita de vazamento de credencial, acesso cross-tenant, exposição de dados ou indisponibilidade total.
- S2: falha relevante de login, banco ou deploy com impacto amplo.
- S3: degradação parcial, erro isolado ou consumo elevado sem indisponibilidade.
- S4: alerta preventivo ou erro sem impacto percebido.

## Runbook
1. Detectar e registrar horário, serviço, deployment/commit e escopo afetado.
2. Preservar logs disponíveis sem copiar tokens, cookies ou dados pessoais.
3. Conter com ação aprovada: revogar credencial, desabilitar fluxo, reverter release ou restringir acesso.
4. Rotacionar secrets comprometidos nos painéis Auth0/Vercel/Neon/GitHub.
5. Avaliar invalidação de sessões e tokens, conexão de banco e execução de workflows.
6. Identificar usuários, tenants, tabelas e janela afetados.
7. Recuperar dados somente após decisão sobre RPO/RTO e consistência.
8. Validar correção com testes negativos e revisão de logs.
9. Registrar causa raiz, impacto, timeline e ações preventivas.

## Casos específicos
- Auth0: callback mismatch, falha de login, segredo exposto, sessão suspeita.
- Neon: compute suspenso por quota, egress/storage excedido, conexão ou policy RLS incorreta.
- Vercel: build failed, runtime error, deploy incorreto, variável errada.
- GitHub: workflow comprometido, action supply-chain, token com permissões excessivas.

## Comunicação
Definir responsável, canal e obrigações legais/contratuais antes do incidente. Este documento não substitui avaliação jurídica/regulatória.

Nenhuma resposta real foi executada. Não rotacionar ou revogar recursos sem autorização explícita.