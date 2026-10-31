# Neon — referência operacional e baseline Free

> Revisão: 31/10/2026. Fonte primária: documentação Neon. Escopo limitado ao plano Free; não pressupor disponibilidade de recursos Launch, Scale ou add-ons.

## 1. Modelo de recursos

- **Organization:** escopo de propriedade e billing; o plano Free é aplicado no âmbito da organização.
- **Project:** unidade de região, compute, storage, transfer e branches. A região é fixa após a criação.
- **Branch:** cópia copy-on-write do estado do banco, usada para desenvolvimento, testes, migrations e previews.
- **Compute/Endpoint:** capacidade de execução PostgreSQL associada a branch.
- **Database:** database PostgreSQL dentro do projeto/branch.
- **Role:** identidade PostgreSQL com privilégios. Roles criados pelo Console/CLI/API herdam membership de neon_superuser; criar roles SQL de privilégio mínimo para runtime e migrations.

## 2. Inventário funcional do Console

Os rótulos e a posição dos menus podem mudar. A lista abaixo é um inventário funcional para auditoria, não uma afirmação de que todos os itens aparecem exatamente nesta ordem em cada conta Free.

| Área / menu funcional | Subáreas a inspecionar | Evidência mínima |
|---|---|---|
| Organization | nome, membros, owner, plano, billing e uso agregado | plano Free, membros e quotas |
| Projects | nome, região, PostgreSQL version, criação, estado e projeto padrão | ID não secreto, região e versão |
| Project Overview | region, compute, endpoints, uso, status e consumo | quotas e estado do projeto |
| Branches | branch default, parent, timestamp, schema, estado e proteção | lista, finalidade e lifecycle |
| Databases | databases, schemas, tables, extensions e owners | inventário e privilégios |
| Roles | roles, memberships, passwords, role attributes e permissões | grants efetivos; nunca copiar senha |
| SQL Editor | conexão selecionada, branch/database, query history e execução | branch/database confirmados antes de SQL |
| Connect | connection string direta/pooled, driver, SSL e pooling | apenas tipo de conexão; não registrar segredo |
| Computes / Endpoints | compute size, autosuspend, autoscaling e endpoint state | CU máximo e timeout |
| Monitoring | compute, storage, connections, transfer, query performance e logs | retenção e data da leitura |
| Settings | region, IP allow, connection restrictions, logical replication e maintenance window | estado real, limitações Free |
| Integrations | Vercel, GitHub, ORM, drivers, CLI, API e outros conectores | escopos e permissões concedidas |
| Billing / Usage | plano, quotas, consumption period, usage e alertas existentes | confirmar ausência de cobrança e limites |
| Snapshots / Restore | histórico, instant restore, snapshots e operações de recuperação | janela e limite de histórico |

Para cada submenu registrar: caminho observado no Console, projeto/branch selecionado, plano, data, configuração não secreta, evidência e classificação (confirmado, recomendado ou pendente).

## 3. Limites confirmados do Neon Free

A FAQ oficial consultada em 31/10/2026 lista:

| Recurso | Limite Free | Consequência operacional |
|---|---:|---|
| Projects | 100 por organização | criar projetos por ambiente apenas quando necessário |
| Branches | 10 por projeto | remover branches efêmeras obsoletas; não contar com branches extras |
| Compute | 100 CU-hours por projeto/mês | ao esgotar, o compute é suspenso até a renovação |
| Autoscaling | até 2 CU (aprox. 8 GB RAM) | workloads intensivos devem ser medidos |
| Scale to zero | após 5 minutos de inatividade; não pode ser desativado | primeira conexão após pausa pode ter latência de retomada |
| PostgreSQL storage | 1 GB por projeto e 20 GB no agregado da organização | operações que aumentariam storage falham ao atingir o limite |
| Public network transfer | 5 GB por projeto/mês | esgotamento suspende compute até a renovação |
| Instant restore | 6 horas ou 1 GB de change history, o que ocorrer primeiro | não substitui backup externo de longo prazo |
| Manual snapshots | 1 por projeto | não assumir múltiplas cópias manuais retidas |
| Monitoring history | 1 dia | exportar evidências necessárias fora da janela |
| Support | comunidade | não depender de suporte contratado |

Compute e transfer renovam mensalmente; projetos, branches e storage são limites contínuos. No Free, atingir storage máximo bloqueia inserts/updates/deletes que aumentariam o espaço; não há cobrança automática de overage segundo a FAQ consultada.

## 4. Branches e ambientes no Free

- Manter branches persistentes estritamente necessárias: produção, desenvolvimento e preview compartilhado, quando a arquitetura justificar.
- Criar branches temporárias para migration tests e PRs apenas dentro da cota de 10.
- Definir owner, propósito, parent, data de expiração e condição de exclusão para cada branch temporária.
- Automatizar limpeza após validar que branch não é default, não contém trabalho pendente e não é dependência de deployment.
- Não tratar Preview como ambiente de produção nem usar dados reais em branch de desenvolvimento sem política de mascaramento.
- Para isolamento mais forte, usar projetos separados por ambiente, respeitando 100 projetos e o limite agregado de 20 GB.
- **Protected branches nativas são NÃO DISPONÍVEIS NO FREE:** a documentação Neon identifica proteção de branch como recurso dos planos Launch/Scale. No Free, usar processo de mudança, credenciais separadas, migrations revisadas e grants PostgreSQL de privilégio mínimo; isso não equivale a proteção nativa contra exclusão/reset.

## 5. Roles, privilégios e RLS

**Regra crítica:** roles criados pelo Console, CLI ou API são membros de neon_superuser. A role neon_superuser possui privilégios administrativos e BYPASSRLS em projetos criados após 15/08/2023. Não utilizar uma role administrativa como usuário de runtime da aplicação.

Recomendação:
1. Manter role de owner/admin apenas para administração controlada.
2. Criar roles de runtime e migration via SQL, sem membership em neon_superuser.
3. Conceder apenas CONNECT, USAGE, DML e execução de funções realmente necessários.
4. Não conceder BYPASSRLS à role runtime.
5. Ativar RLS nas tabelas multi-tenant, criar políticas restritivas por tenant e testar com a role real de runtime.
6. Verificar rolsuper, rolbypassrls, memberships, grants efetivos e comportamento com tenant ausente/inválido.
7. Executar migrations com identidade separada da aplicação; não colocar senha de owner em variáveis do frontend.

Exemplo conceitual (substituir nomes e grants após revisar o schema; não executar cegamente):

~~~sql
CREATE ROLE app_runtime LOGIN PASSWORD '<senha-gerada-em-cofre>';
CREATE ROLE app_migrator LOGIN PASSWORD '<senha-gerada-em-cofre>';

-- Conceder somente privilégios necessários no database/schema/tabelas.
-- Não adicionar app_runtime a neon_superuser.
-- Habilitar e testar RLS com app_runtime.
~~~

Não colocar senhas reais neste repositório. O exemplo é um padrão de referência, não uma migration pronta.

## 6. Conexões e aplicação serverless

- Preferir a connection string pooled para aplicações com muitas conexões curtas/serverless, se driver e ORM suportarem pooling.
- Usar TLS e guardar DATABASE_URL exclusivamente em ambiente server-side.
- Separar connection strings de Development, Preview e Production.
- Nunca permitir que deployment Preview consuma credenciais/branch de Production.
- Controlar tamanho do pool, tempo de conexão e retry; escala para zero pode afetar cold-start.
- Não expor variáveis de banco com prefixos públicos, como NEXT_PUBLIC_.

## 7. Backups, restore e recuperação

- Tratar o histórico instant restore do Free (6 horas/1 GB) como mecanismo operacional limitado, não como backup independente.
- Manter exportações PostgreSQL periódicas (pg_dump) criptografadas, em destino independente e com controle de acesso.
- Preservar ao menos uma cópia offline ou em domínio administrativo separado; testar restore em projeto/branch não produtivo.
- Registrar RPO/RTO pretendidos, frequência, retenção, responsável e data do último teste.
- Não depender de uma única snapshot manual.
- Testar compatibilidade de schema com a versão da aplicação antes de rollback de deployment.

O destino externo pode gerar custo ou ter quota própria; só considerar uma alternativa como Free após validar os seus limites e termos. Não presumir armazenamento ilimitado e sem custo.

## 8. Região e produtos disponíveis

A região é definida na criação do projeto e não pode ser alterada depois. O projeto observado da organização está em aws-sa-east-1 (São Paulo), PostgreSQL 18.

Na matriz oficial consultada, São Paulo disponibiliza PostgreSQL e Managed Better Auth. Object Storage, Neon Functions, AI Gateway e Realtime aparecem indisponíveis nessa região; além disso, o AI Gateway exige plano pago. Portanto, esses recursos são **NÃO DISPONÍVEIS NA COMBINAÇÃO FREE + aws-sa-east-1**. Não migrar de região apenas para habilitar uma função sem aprovar residência de dados, latência e plano de migração.

## 9. Integração GitHub / Vercel / Neon

- GitHub mantém migrations versionadas; CI testa a migration em branch temporária quando possível.
- O workflow deve usar permissões mínimas e secrets não produtivos.
- Para Vercel Hobby, não assumir que deployments comerciais sejam permitidos; conferir a restrição de uso pessoal/não comercial antes de qualquer integração.
- Usar variável de conexão própria por ambiente e branch; conferir o destino antes de executar migration.
- Não executar migration em Production como parte automática de PR/Preview.
- Controlar o número de branches Neon para não ultrapassar 10 por projeto.
- Guardar evidências de schema, grants e testes RLS vinculadas ao SHA do código e ao identificador da migration, sem credenciais.

## 10. Observação do projeto conectado (não é baseline)

Leitura somente de metadados em 31/10/2026: organização Neon em Free; projeto na região AWS São Paulo; PostgreSQL 18; escala configurada entre 0,25 e 2 CU; janela de histórico de 6 horas; branches observadas: production, development, preview e branches de teste/migration. As seis branches retornadas pela API estavam marcadas como não protegidas. Esta observação não significa que todas as configurações tenham sido auditadas, nem autoriza alterações.

## 11. Checklist operacional Free

- [ ] Organização e plano Free confirmados.
- [ ] Project, região e PostgreSQL version registrados.
- [ ] Storage agregado e transfer verificados.
- [ ] Compute e autosuspend compatíveis com o workload.
- [ ] Número de branches abaixo de 10, com lifecycle documentado.
- [ ] Branch de produção reconhecida e protegida por processo/controles compensatórios.
- [ ] Roles runtime/migrator sem membership administrativa indevida.
- [ ] RLS testado com role real de runtime.
- [ ] Connection pooling e TLS testados.
- [ ] Credenciais separadas por ambiente.
- [ ] Backup externo criptografado e restore testado.
- [ ] Limite de 6h/1GB de instant restore conhecido.
- [ ] Região e disponibilidade de produtos adicionais confirmadas.
- [ ] Quotas e logs revalidados periodicamente.

## Fontes oficiais

- [Neon Plans](https://neon.com/docs/introduction/plans)
- [Free plan limits and quotas](https://neon.com/faqs/free-plan-limits-and-quotas)
- [Regions and product availability](https://neon.com/docs/introduction/regions)
- [Protected branches](https://neon.com/docs/guides/protected-branches)
- [Manage branches](https://neon.com/docs/manage/branches)
- [Manage roles](https://neon.com/docs/manage/roles)
- [Manage database access](https://neon.com/docs/manage/database-access)
- [Connection pooling](https://neon.com/docs/connect/connection-pooling)
- [Neon CLI](https://neon.com/docs/reference/neon-cli)
- [Neon API](https://neon.com/docs/reference/api)
