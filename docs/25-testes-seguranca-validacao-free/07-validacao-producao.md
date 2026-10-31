# 7. Validação final de produção

## 7.1 Antes do deploy

- Confirmar finalidade: pessoal/não comercial ou comercial.
- Validar os termos do host. Vercel Hobby só pode ser considerado para uso pessoal/não comercial; para SaaS comercial, marcar **NO-GO**.
- Identificar commit exato, tag, PR e changelog.
- Confirmar compatibilidade Next.js/Node/npm/Auth0 SDK/Neon driver.
- Verificar variáveis obrigatórias por ambiente sem copiar valores para evidência.
- Conferir Auth0 issuer, audience, callback/logout URLs e domínio; não compartilhar client secret.
- Confirmar DATABASE_URL de runtime e role efetiva; runtime não pode usar owner/migrator.
- Confirmar migrations já testadas e plano de recuperação.
- Confirmar quotas Free e comportamento ao atingir limite.
- Confirmar logs com redaction e sem tokens/cookies/PII desnecessários.

## 7.2 Pós-deploy smoke tests

Somente após deploy autorizado em ambiente permitido:
1. health/readiness sem revelar detalhes internos;
2. página pública e rota protegida;
3. login/logout real;
4. sessão expirada e callback;
5. usuário autorizado e usuário sem permissão;
6. isolamento tenant A/B;
7. leitura/escrita mínima com dado sintético;
8. conexão Neon e comportamento de scale-to-zero;
9. logs sem segredos;
10. alertas/limites operacionais observáveis;
11. confirmar commit e deployment correspondentes.

Não executar carga, fuzzing, destructive test ou teste de recuperação em produção como parte do smoke test.

## 7.3 Verificação de configuração

- Vercel: deployment e variáveis associadas ao ambiente correto; alteração de env var normalmente requer novo deployment para surtir efeito no deployment existente.
- Auth0: callback URLs estritamente necessárias, logout URLs, Web Origins, sessão/cookie e configurações de API.
- Neon: connection pooling quando adequado, TLS, role runtime, migrations e RLS.
- GitHub: branch/release rastreável, workflows com permissões mínimas e evidências de CI.

Não afirmar configuração verificada sem evidência obtida no painel/API/CLI e data de observação.

## 7.4 Rollback e recuperação

- Aplicação: identificar deployment anterior conhecido e compatível com schema atual.
- Banco: preferir forward-fix quando migration destrutiva ou reversão não for segura; rollback de aplicação não desfaz schema.
- Configuração Auth0: registrar estado anterior e procedimento de retorno; não presumir backup/restore gerenciado de tenant.
- Secrets: rotação apenas se autorizada; atualizar variáveis e gerar novo deployment quando necessário.
- Confirmar que o procedimento cabe nos limites e retenção do plano Free.
- Não executar rollback em produção sem autorização específica.

## 7.5 Critério de aceite

Registrar data/hora, URL/ambiente, commit, deployment ID, versões, resultados dos smoke tests, alertas, limitações Free, exceções, aprovador e decisão GO/NO-GO. O resultado só comprova o ambiente e instante observados; não constitui SLA ou garantia contínua.
