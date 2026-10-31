# 06 — Auditoria de permissões, secrets e quotas Free

## 1. Objetivo

Executar auditoria recorrente por projeto, ambiente e fornecedor, identificando deriva de configuração, exposição de segredos, excesso de privilégios e risco de exceder cotas gratuitas.

## 2. Auditoria de identidades e permissões

### GitHub
- [ ] Acesso de colaboradores limitado ao necessário.
- [ ] Revisar admins, maintainers, deploy keys, GitHub Apps e tokens.
- [ ] Workflow permissions são read-only por padrão quando possível.
- [ ] Actions de terceiros revisadas e fixadas em SHA.
- [ ] CI de PR não recebe secrets de produção.
- [ ] Repositório privado não depende de GitHub Environments/secrets indisponíveis no Free.
- [ ] Histórico e artifacts não contêm credenciais.

### Auth0
- [ ] Tenant único identificado e responsável definido.
- [ ] Revisar membros administrativos e roles do Dashboard.
- [ ] Revisar Applications, APIs, Connections e callback URLs.
- [ ] Remover URLs antigas e wildcards amplos.
- [ ] Client credentials distintas por ambiente quando viável.
- [ ] Access Token Lifetime e políticas coerentes com os requisitos do projeto.
- [ ] Revisar usuários de teste e privilégios.
- [ ] Export/backup de configuração exclui recursos pagos não disponíveis no Free.
- [ ] Lembrar que export tenant não é backup de usuários nem de password hashes.

### Vercel Hobby
- [ ] Confirmar uso pessoal e não comercial.
- [ ] Revisar membros e permissões do projeto.
- [ ] Revisar Git integration, Production Branch e deployment URLs.
- [ ] Variáveis escopadas por Development/Preview/Production.
- [ ] Preview Staging limitado à branch esperada.
- [ ] Nenhuma variável sensível exposta via NEXT_PUBLIC_.
- [ ] Novo deployment após mudança de variável.
- [ ] Revisar logs e acessos a deployment.

### Neon
- [ ] Revisar membros da organização e projetos.
- [ ] Revisar roles SQL, memberships, grants e privilégios.
- [ ] Runtime não é owner, não possui BYPASSRLS e não executa DDL.
- [ ] Migrator separado do runtime.
- [ ] Connection strings por ambiente e armazenadas como secrets.
- [ ] SSL/TLS e pooling/direct conforme workload.
- [ ] RLS habilitada e testada para tabelas sensíveis.
- [ ] Branches temporárias removidas após uso.
- [ ] Backups lógicos e ensaios de restore registrados.

## 3. Quotas Free — painel de controle

| Serviço | Itens para medir | Gatilho de revisão |
|---|---|---|
| GitHub Free | Minutos Actions, storage artifacts/cache, runners e limites de API | Consumo próximo do teto ou workflow falhando por quota |
| Auth0 Free | MAU, aplicações, conexões, APIs, tokens, rate limits e recursos habilitados | Qualquer limite do tenant se aproximando ou recurso indisponível |
| Vercel Hobby | Deploys, builds, funções, transfer, bandwidth, logs e limites de projeto | Uso acima do necessário ou workload incompatível com Hobby |
| Neon Free | Projects, branches, CU-hours, storage por projeto/conta, transfer e restore window | Branch/compute/storage/transfer próximo da quota |

Não fixar números em documentos operacionais sem conferir a página de planos e o painel na data da auditoria. Os valores mudam e podem variar por conta/região.

## 4. Snapshot de quotas consultadas (31-10-2026)

**Neon Free:** a página oficial consultada em outubro de 2026 lista 100 projetos, até 10 branches por projeto, 100 CU-hours por projeto/mês, 1 GB por projeto, 20 GB agregados e 5 GB de transferência pública por projeto/mês. Confirmar se a conta e região seguem a mesma tabela antes de provisionar.

**Auth0 Free:** a referência de suporte consultada informa um tenant Free por conta. Confirmar os limites atuais de MAU, aplicações, conexões, APIs e recursos no painel/planos.

**Vercel Hobby:** uso pessoal e não comercial; ambientes padrão Local/Preview/Production. Custom Environments não devem ser assumidos no Hobby.

**GitHub Free:** environments/secrets para deployment em repositórios privados não estão disponíveis no Free; revalidar a matriz de planos e visibilidade do repositório.

## 5. Frequência recomendada

| Auditoria | Frequência |
|---|---|
| Permissões e membros | Mensal e após mudança de equipe |
| Secrets e variáveis | Mensal e após incidente/rotação |
| Quotas e consumo | Mensal e antes de novo projeto/branch |
| Callback URLs/domínios | A cada release e mudança de URL |
| Roles SQL/RLS | A cada migration e revisão trimestral |
| CI/workflows | A cada alteração de workflow ou Action |
| Termos Free/eligibilidade | Antes do deploy e trimestral |

## 6. Registro da auditoria

Registrar:
- projeto e ambiente;
- data e auditor;
- fornecedor e plano confirmado;
- evidências sem dados secretos;
- quotas observadas e unidade;
- acessos removidos/ajustados;
- riscos aceitos e responsável;
- pendências, prazo e próxima revisão.

## 7. Critérios de bloqueio

Bloquear novo provisionamento ou release quando:
- não houver quota suficiente;
- o uso não for elegível no plano;
- credencial estiver compartilhada indevidamente;
- secret tiver sido exposto;
- callback estiver amplo/inseguro;
- role runtime tiver privilégios excessivos;
- RLS multiusuário não tiver teste;
- não existir plano de recuperação compatível com o risco.
