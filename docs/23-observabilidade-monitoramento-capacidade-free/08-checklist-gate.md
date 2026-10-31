# Checklist e gate de observabilidade

## Checklist de implementação

### Next.js
- [ ] Logs estruturados JSON e schema estável.
- [ ] requestId e traceId propagados sem confiar em input não validado.
- [ ] Rotas normalizadas; query strings e payloads não registrados.
- [ ] Tokens, cookies, senhas e PII removidos.
- [ ] Logs apenas em código server-side.

### Auth0
- [ ] Acesso Monitoring > Logs validado com role apropriada.
- [ ] Eventos de sucesso, falha, bloqueio e alterações administrativas revisados.
- [ ] Retenção Free documentada com data de verificação.
- [ ] Sem dependência de Log Streams não incluído no Free.
- [ ] Nenhum token/sessão gravado em logs.

### Neon
- [ ] Usage/Monitoring revisado por projeto.
- [ ] CU-hours, storage e conexões registrados.
- [ ] Pooler e estratégia de conexão documentados.
- [ ] Queries de diagnóstico usam privilégio mínimo.
- [ ] Sem SQL com valores pessoais em logs ou relatórios.

### Vercel Hobby
- [ ] Build Logs revisados por deployment.
- [ ] Runtime Logs e retenção Hobby conhecidos.
- [ ] Observability base diferenciada de Observability Plus.
- [ ] Uso pessoal/não comercial confirmado.
- [ ] Sem dependência de Log Drains pagos.

### GitHub
- [ ] Workflows principais possuem logs e status monitorados.
- [ ] Permissões mínimas e actions revisadas.
- [ ] Uso/armazenamento de artifacts e cache observado.
- [ ] Elegibilidade de secret scanning/Dependabot confirmada para o tipo de repositório.
- [ ] Nenhum secret em logs.

## Gate GO/NO-GO

**GO documental** somente quando:
- schema de logs e política de redaction estiverem revisados;
- responsáveis e periodicidade de revisão estiverem definidos;
- limites Free tiverem sido conferidos nos painéis;
- runbooks tiverem owner e critérios de escalonamento;
- retenção e limitações tiverem sido aceitas formalmente.

**NO-GO** quando:
- tokens/secrets/PII aparecerem em logs;
- não houver requestId ou release para correlacionar falha;
- quotas estiverem sem acompanhamento;
- a arquitetura depender de recurso pago ou trial;
- o uso pretendido violar as condições do Vercel Hobby.

## Evidências necessárias para marcar como Verificada

Registrar fonte, data/hora UTC, plano, projeto/tenant/repositório, tela ou saída relevante e responsável. Remover dados sensíveis antes de anexar evidência.

## Estado desta entrega

Esta seção é documentação recomendada. Nenhum SDK foi instalado, arquivo de aplicação foi alterado, dashboard foi configurado, alerta foi criado, métrica real foi coletada, quota foi consultada em conta autenticada ou teste foi executado nesta etapa.
