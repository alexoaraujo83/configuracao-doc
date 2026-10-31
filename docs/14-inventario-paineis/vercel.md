# Vercel Hobby — inventário do Dashboard

> Revisão documental: 31/10/2026. A navegação é dividida entre Team Settings e Project Settings, além das áreas Firewall e Observability. A disposição visual pode mudar entre sidebar e abas.

## 1. Restrição do plano Hobby

O plano Hobby é destinado a uso pessoal e não comercial.
- BLOCKED: hospedar SaaS comercial, produto comercial ou workload de produção comercial no Hobby.
- Não presumir elegibilidade porque um projeto consegue fazer deploy.
- Para projeto comercial, o Hobby não pode ser usado como solução.
- Esta documentação não autoriza upgrade ou contratação.
- Revisar Termos e privacidade antes de enviar código, conteúdo ou dados confidenciais.

## 2. Contexto

Dois níveis distintos:
- Team: equipe/conta que controla membros, projetos e políticas comuns.
- Project: aplicação individual importada de um repositório Git.

Selecionar a Team correta antes de alterar configurações. Configuração de Team não é necessariamente configuração de Project.

## 3. Dashboard da Team

| Caminho | Finalidade | Revisão |
|---|---|---|
| Team Dashboard > Projects | Listar projetos | Confirmar projeto e team scope |
| Team Dashboard > Deployments/Activity | Visão geral de deployments e atividade, conforme layout | Conferir status e histórico |
| Team Settings > General | Preferências gerais | Nome e identificadores |
| Team Settings > Members | Pessoas e papéis | Menor privilégio |
| Team Settings > Security & Privacy | Segurança e políticas | MFA e controles dependentes do plano |
| Team Settings > Billing | Plano, limites e uso | Confirmar Hobby e quotas; não contratar |
| Team Settings > Integrations | Integrações/contas | Revisar permissões OAuth |
| Team Settings > Domains | Domínios da equipe, se disponíveis | Propriedade e DNS |
| Team Settings > Usage | Consumo | Monitorar quotas incluídas |

Menus variam por tipo de conta, papel e plano. Governança avançada pode não aparecer no Hobby.

## 4. Dashboard do Project

| Caminho | Finalidade | Revisão |
|---|---|---|
| Project Overview | Visão do projeto/deployment ativo | Framework, deployment e URLs |
| Deployments | Histórico e estados | Preview/Production, commit, branch e build |
| Activity | Eventos e alterações, se visível | Auditoria e responsáveis |
| Logs / Observability | Runtime, erros, funções e requisições | Diagnóstico e retenção |
| Analytics / Web Analytics | Métricas de tráfego | Privacidade e disponibilidade Free |
| Speed Insights | Performance | Disponibilidade e limites |
| Firewall | Tráfego e regras | Limites do plano |
| Domains | Domínios associados | DNS, domínio primário e certificados |
| Settings | Configurações do projeto | Seção seguinte |

## 5. Project Settings

A documentação oficial lista configurações de projeto para domínio, Functions, drains, integrações, Git, cache, variáveis, deployment protection e segurança.

| Caminho | Finalidade | Configuração |
|---|---|---|
| Settings > General | Nome, framework, Root Directory, Project ID, Node.js | Conferir Next.js, diretório e versão Node |
| Settings > Build and Deployment | Build/install command, output e Node.js | Comandos determinísticos e lockfile |
| Settings > Environment Variables | Variáveis por Development, Preview e Production | Secrets separados; sem NEXT_PUBLIC_ para segredos |
| Settings > Domains | Domínios e aliases | Propriedade comprovada |
| Settings > Git | Repositório e branch de produção | Confirmar repositório/branch |
| Settings > Functions | Runtime, região, memória e timeout | Conferir limites Hobby |
| Settings > Integrations | Integrações de terceiros | Revisar permissões e custo |
| Settings > Deployment Protection | Proteção de deployments | Validar elegibilidade Hobby |
| Settings > Security | Attack Challenge, proteção de logs/source, retenção | Opções podem ser limitadas |
| Settings > Cron Jobs | Ativar/desativar jobs | Cron configurado no código; verificar limites |
| Settings > Webhooks | Eventos do projeto | Validar assinatura e endpoint |
| Settings > Advanced | Opções avançadas | Usar apenas com requisito |
| Settings > Microfrontends | Agrupar projetos | Fora da arquitetura inicial |

Drains são documentados como recurso Pro/Enterprise; marcar NÃO DISPONÍVEL NO FREE.

## 6. Deployments

Caminho: Project > Deployments.

Operações:
- Filtrar por branch, status, data e ambiente.
- Abrir detalhes de build/deployment.
- Redeploy de versão anterior.
- Promover deployment para produção quando elegível.
- Identificar commit e URL.

Regras:
- Não promover Preview sem revisão.
- Confirmar variáveis e callback Auth0 do ambiente alvo.
- Redeploy não é rollback de banco de dados.
- Versionar migrations e planejar compatibilidade retroativa.
- Verificar quotas e restrições Hobby.

## 7. Firewall

Caminho: Project > Firewall.

A documentação descreve regras customizadas, rate limiting, IP blocking, managed rulesets e observabilidade; disponibilidade depende do plano.

No Hobby:
- System Bypass Rules: NÃO DISPONÍVEL NO FREE.
- Não presumir WAF customizado/rate limiting sem confirmação do plano.
- Aplicar controles na aplicação Next.js: validação, autorização, rate limiting compatível e limites de payload.
- Não confundir mitigação DDoS gerenciada com regras customizadas.

## 8. Observability

Caminho: Project > Observability.

Áreas:
- Runtime Logs.
- Function invocations e erros.
- Requests e métricas CDN.
- Traces/monitoramento quando disponíveis.
- Web Analytics.
- Speed Insights.

Documentar retenção, filtros e limites. Drains externos são Pro/Enterprise; não usar no baseline Hobby.

## 9. Integração GitHub + Next.js

1. Selecionar a Team correta.
2. Importar repositório autorizado.
3. Conferir detecção de Next.js e Root Directory.
4. Conferir branch de produção e política de Preview.
5. Definir variáveis por ambiente sem expor secrets.
6. Revisar build e versão Node.
7. Fazer deployment de validação somente após confirmar elegibilidade Hobby.
8. Conferir logs e callbacks Auth0.
9. Não mapear ambiente comercial para Hobby.

## 10. Fontes oficiais

- [Vercel Plans](https://vercel.com/docs/plans)
- [Hobby Plan](https://vercel.com/docs/plans/hobby)
- [Vercel Terms](https://vercel.com/legal/terms)
- [Projects overview](https://vercel.com/docs/projects)
- [Project settings](https://vercel.com/docs/project-configuration/project-settings)
- [Managing deployments](https://vercel.com/docs/deployments/managing-deployments)
- [Tour the Dashboard](https://vercel.com/academy/optimize-your-vercel-account/tour-the-dashboard)
- [Functions configuration](https://vercel.com/docs/functions/configuring-functions)
- [Firewall](https://vercel.com/docs/vercel-firewall)
- [Observability](https://vercel.com/docs/observability)

## 11. Pendências de validação

- [ ] Confirmar Team e plano Hobby.
- [ ] Confirmar uso pessoal/não comercial.
- [ ] Conferir papel e membros.
- [ ] Conferir projeto, repositório Git e branch de produção.
- [ ] Conferir Node.js, build e Root Directory.
- [ ] Conferir variáveis por ambiente sem expor valores.
- [ ] Conferir domínios e callbacks Auth0.
- [ ] Verificar Firewall, Analytics e Observability disponíveis no Hobby.
- [ ] Confirmar retenção e quotas.
