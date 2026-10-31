# Matriz mestra de requisitos

Data-base: 2026-10-31. Os requisitos abaixo são critérios de referência. O status inicial de cada requisito é **Não avaliado** até existir evidência do projeto consumidor.

| ID | Domínio | Requisito verificável | Controle esperado | Evidência mínima | Responsável primário | Limitação Free / observação | Critério de aprovação |
|---|---|---|---|---|---|---|---|
| GOV-01 | Governança | Escopo, finalidade e classificação do sistema definidos | Documento de contexto e inventário de dados | Contexto aprovado e data | Product/Service Owner | Sem recurso pago presumido | Escopo e dados identificados |
| GOV-02 | Governança | Uso permitido pela Vercel Hobby | Classificação pessoal/não comercial | Declaração de uso e revisão jurídica/negócio | Owner | Hobby apenas pessoal/não comercial | Se comercial, NO-GO para Hobby |
| GOV-03 | Governança | Planos e cotas verificados | Revisão de páginas oficiais e painel | Capturas sem segredos, URLs e data | Platform Owner | Limites mudam e podem variar | Limites vigentes documentados |
| GOV-04 | Governança | Exceções formalizadas | Registro de risco, prazo e aprovador | Registro de exceção vigente | Security/Tech Lead | Exceção não habilita recurso pago | Nenhuma exceção crítica aberta |
| APP-01 | Aplicação | TypeScript e build sem erros | Typecheck, lint e build no CI | Logs CI com SHA | Engineering | Minutos Actions limitados | Todos os jobs obrigatórios verdes |
| APP-02 | Aplicação | Configuração validada no servidor | Schema de env sem exposição cliente | Teste de env inválida e revisão | Engineering | Segredos em contexto privado limitado | Falha fechada, sem secret em bundle |
| APP-03 | Aplicação | APIs validam sessão e autorização no servidor | Guardas por rota/operação | Testes 401/403 e revisão | Engineering | Middleware não substitui autorização | Casos negativos aprovados |
| APP-04 | Aplicação | Entradas e saídas validadas | Schemas, limites e serialização segura | Testes de validação | Engineering | Sem WAF avançado pressuposto | Entradas inválidas rejeitadas |
| IAM-01 | Auth0 | Fluxos OIDC e callback restritos | Redirect URI exata, state/nonce/PKCE conforme SDK | Configuração exportada/redigida e teste browser | IAM Owner | Um tenant Free; domínio custom pode exigir cartão | Callback válido e inválido testados |
| IAM-02 | Auth0 | Segredos e sessões separados por ambiente lógico | Secrets fortes e não compartilhados | Inventário de nomes/escopos sem valores | IAM Owner | Tenant único reduz isolamento de ambiente | Sem reutilização de AUTH0_SECRET |
| IAM-03 | Auth0 | Identidade vem da sessão validada | Sub do servidor, nunca do body | Testes de spoofing e código revisado | Engineering | Não confiar em claims custom sem validação | Identidade não manipulável pelo cliente |
| IAM-04 | Auth0 | Autorização de domínio implementada na aplicação | RBAC/app permissions e membership próprios | Matriz papel-ação e testes | Product + Engineering | Role Management não incluído no Free atual | Negação por padrão e testes por papel |
| IAM-05 | Auth0 | Logout e sessão verificados | Logout SDK e expiração controlada | Evidência browser redigida | IAM Owner | Retenção de logs Free limitada | Sessão invalidada conforme desenho |
| DB-01 | Neon | Roles com privilégio mínimo | Owner/migrator/runtime separados | SQL de grants redigido e query de catálogo | DBA/Engineering | Quotas de compute, storage e branches | Runtime não é owner nem BYPASSRLS |
| DB-02 | Neon | RLS aplicada onde há isolamento por tenant | ENABLE/FORCE RLS, USING e WITH CHECK | DDL e testes multi-tenant sintéticos | DBA/Engineering | RLS não protege contra backend comprometido | Acesso cruzado negado |
| DB-03 | Neon | Contexto de tenant definido na transação | SET LOCAL/set_config transacional | Teste de isolamento e rollback | Engineering | GUC definido pelo backend não é fronteira independente | Contexto não vaza entre requests |
| DB-04 | Neon | SQL parametrizado | Driver com parâmetros e revisão | Testes de entrada hostil e code review | Engineering | Não elimina falhas de autorização | Nenhuma concatenação de input |
| DB-05 | Neon | Migrations imutáveis e reproduzíveis | SQL versionado, checksum e histórico | Logs de aplicação em DB descartável | DBA/Engineering | Branches limitadas no Free | Migração repetível e rollback documentado |
| DB-06 | Neon | Integridade e constraints verificadas | PK, FK, UNIQUE, CHECK e índices | Relatório de constraints e testes | DBA/Engineering | Storage e egress limitados | Constraints e invariantes aprovados |
| DB-07 | Neon | Backup lógico e restore ensaiado fora de produção | pg_dump, checksum, restore isolado | Manifesto e log sem dados reais | DBA/Operations | PITR Free curto; backup externo é responsabilidade do projeto | Restore validado e RPO/RTO realistas |
| SCM-01 | GitHub | Mudanças via PR e revisão | Branches curtas, PR e aprovação | Histórico de PR e commits | Maintainer | Proteção de branch Free disponível em repositório público; não presumir em privado | Revisão registrada antes da integração |
| SCM-02 | GitHub | Segredos ausentes do Git | .gitignore, secret scanning manual/automatizado | Relatório de busca e revisão | Maintainer | Recursos avançados podem variar por plano/repo | Zero segredo conhecido versionado |
| SCM-03 | GitHub Actions | Permissões mínimas no workflow | permissions explícitas, contents: read por padrão | YAML e logs CI | DevOps | Quotas de minutos e armazenamento | Permissões justificadas e job verde |
| SCM-04 | GitHub Actions | Actions de terceiros fixadas | SHA completo revisado e atualização controlada | Inventário de actions e SHA | DevOps | Dependabot alerts/config variam | Nenhuma action mutável não avaliada |
| SCM-05 | Dependências | Lockfile reproduzível e auditoria | npm ci, npm audit e triagem | package-lock e relatório datado | Engineering | npm audit não detecta todas as classes de risco | Sem CVE crítica/alta sem tratamento |
| SCM-06 | Dependências | Dependabot e atualizações governadas | Alertas, PRs e validação | Configuração e histórico de PR | Maintainer | Cotas/recursos de segurança dependem do repo | Alertas triados dentro do SLA interno |
| HOST-01 | Vercel | Deploy vinculado ao commit revisado | Git integration e preview | Deployment ID + SHA | Platform Owner | Hobby pessoal/não comercial | Deployment corresponde ao SHA aprovado |
| HOST-02 | Vercel | Env vars separadas e redigidas | Valores por contexto e redeploy após mudança | Inventário de nomes e deployment ID | Platform Owner | Hobby não equivale a ambiente enterprise | Sem segredo em logs/build output |
| HOST-03 | Vercel | Domínio e HTTPS verificados | TLS, callback allowlist e DNS | Teste HTTPS e configuração redigida | Platform Owner | Domínio/recursos podem ter restrições | Certificado válido e URLs exatas |
| HOST-04 | Vercel | Logs não expõem dados sensíveis | Redação e minimização | Amostra sanitizada | Operations | Retenção Hobby curta | Sem token, PII ou SQL sensível |
| OPS-01 | Operação | SLI/SLO internos definidos sem chamar de SLA | Indicadores, limites e runbooks | Documento e histórico sintético | Operations | Sem SLA contratado no Free | Limites e resposta definidos |
| OPS-02 | Operação | Incidente e rotação documentados | Runbook, contatos e sequência de ações | Exercício tabletop/documental | Security/Operations | Suporte comunitário/limitado | Responsáveis e decisões rastreáveis |
| OPS-03 | Operação | Rollback compatível com schema | Expand-contract e compatibilidade app/DB | Plano de rollback revisado | Tech Lead/DBA | Rollback Vercel Hobby limitado | Rollback não depende de downgrade destrutivo |
| REL-01 | Release | Quality gates obrigatórios definidos | Testes, scans, migration check, smoke test | Pipeline e checklist de release | Release Owner | Não pressupor Environments privados Free | Nenhum gate crítico ignorado |
| REL-02 | Release | Evidência ligada ao artefato | SHA, versão, data, ambiente e resultado | Manifesto de evidências | Release Owner | Retenção de logs limitada | Rastreabilidade completa |
| REL-03 | Release | Aprovação final e NO-GO explícitos | Aprovação de owner e segurança/DB | Registro de decisão | Release Owner | Aprovação manual não substitui controles | Todos os critérios de aceite satisfeitos |

## Classificação de criticidade

- **Crítico:** identidade, autorização, isolamento de tenant, segredos, integridade de dados, migrations de produção, exposição pública e elegibilidade contratual do plano.
- **Alto:** dependências, CI permissions, backups, rollback, logs e rastreabilidade.
- **Médio:** documentação, dashboards, eficiência de cotas e manutenção.
- **Baixo:** conveniência, formatação e otimizações não essenciais.

A criticidade é uma classificação interna de governança, não uma classificação oficial do fornecedor.

## Como preencher

Para cada ID, registrar: status do requisito, status da evidência, link do artefato, SHA/versão, data de coleta, responsável pela coleta, revisor, observações e risco residual. Não preencher como Conforme por inferência.
