# Neon Free — inventário do Console

> Revisão documental: 31/10/2026. A navegação é contextual: selecionar organização, projeto e branch. A lista de itens pode variar com permissões e evolução do Console.

## 1. Escopo Free e limites

Baseline Free registrada neste repositório; confirmar novamente na página de planos antes da implantação:
- 100 CU-hours por projeto/mês.
- 10 branches por projeto.
- 1 GB de armazenamento por projeto.
- 20 GB agregados por conta.
- 5 GB de transferência por projeto/mês.
- Histórico de restore de 6 horas, sujeito a limites.
- Um snapshot manual.
- Protected branches e Instant Restore: NÃO DISPONÍVEL NO FREE.

Não tratar valores como ilimitados nem manter branches temporárias sem política de limpeza.

## 2. Contexto de navegação

Sequência:
1. Neon Console.
2. Selecionar Organization.
3. Selecionar Project.
4. Selecionar Branch no seletor de projeto/branch.
5. Abrir a seção desejada no menu lateral.

A branch selecionada afeta database, roles, SQL Editor e connection string. Confirmar branch antes de executar SQL.

## 3. Organização e projetos

| Caminho | Finalidade | Revisão |
|---|---|---|
| Organization switcher | Alternar organização | Confirmar organização e permissões |
| Organization > Members / Access | Membros e papéis | Menor privilégio |
| Organization > Projects | Projetos PostgreSQL | Nome, região, versão e proprietário |
| Project > Overview | Resumo do projeto | Status, região, compute e branch padrão |
| Project > Settings | Configurações gerais | Região, versão PostgreSQL e compute |
| Project > Integrations | Integrações | Serviço e elegibilidade Free |
| Project > Monitoring | Métricas e atividade | Compute, conexões, armazenamento e limites |

Papéis organizacionais e permissões por projeto controlam o que cada membro pode ver. Usuário de leitura pode não ver connection strings nem operações de escrita.

## 4. Branches

Caminho: Project > branch selector / Branches.

Operações:
- Listar branches.
- Criar branch a partir de parent branch.
- Renomear, restaurar ou excluir conforme permissões.
- Alternar branch ativa.
- Comparar schema/data quando disponível.

Padrão:
- Branch principal para referência estável.
- Branches de desenvolvimento/temporárias somente dentro da quota.
- Branch não é backup permanente.
- Protected branches não estão incluídas no Free.
- Registrar parent, objetivo, data de criação e descarte.
- Não apagar branch com dados não replicados.

## 5. Databases

Caminho: selecionar branch > Postgres database > Databases.

Revisar nome, database padrão/adicionais, schema associado, role proprietária e databases presentes em cada branch.

Database não é branch nem role. A branch contém estado PostgreSQL; database é unidade lógica dentro desse estado.

## 6. Roles

Caminho: selecionar branch > Postgres database > Roles.

Consideração crítica:
- Roles criadas pelo Console, CLI ou API recebem associação a neon_superuser.
- neon_superuser inclui privilégios elevados como CREATEDB, CREATEROLE e BYPASSRLS.
- Para roles de privilégio mínimo, criá-las via SQL e conceder privilégios explicitamente.
- Não usar role administrativa do projeto como credencial runtime.
- Separar owner, migrator e runtime.
- Restringir runtime ao schema e operações necessários.

O modelo SQL está em [Neon PostgreSQL](../12-neon-postgresql/README.md).

## 7. SQL Editor

Caminho: selecionar branch > SQL Editor.

Uso:
- Consultar schema e dados de teste.
- Executar migrations em ambiente controlado.
- Inspecionar permissões e RLS.
- Validar consultas e índices.

Regras:
- Confirmar projeto, branch, database e role.
- Não colar segredos ou dados pessoais.
- Não executar DROP, TRUNCATE ou DELETE amplo sem plano e autorização.
- Para migrations de produção, usar pipeline controlado e role de migration.
- Salvar SQL versionado no Git; não deixar fonte da verdade apenas no editor web.

## 8. Connection Details e pooling

Caminho: Project/Branch > Connect ou Connection Details, conforme a interface.

Selecionar branch, database e role corretos:
- Connection string pooled para runtime serverless.
- Connection string direct para migrations e operações que exigem conexão direta.

Não copiar connection string para documentação, tickets ou screenshots. Armazenar em variáveis de ambiente seguras. Não reutilizar credenciais runtime e migration.

## 9. Monitoring

Caminho: Project > Monitoring.

Revisar compute/CU-hours, storage, transferência, conexões, pool, queries lentas e erros conforme métricas disponíveis. O Free limita retenção e observabilidade; registrar a janela exibida no Console e compará-la com os limites oficiais.

## 10. Restore, snapshots e backups

Caminho: Project/Branch > Restore / Snapshots, conforme a interface.

Free:
- Histórico de restore limitado a 6 horas, sujeito às quotas.
- Um snapshot manual.
- Instant Restore e protected branches não incluídos no baseline.

Regras:
- Restore não substitui backup independente.
- Exportar backups lógicos em rotina externa e protegê-los.
- Testar restore em branch/projeto não produtivo.
- Registrar RPO/RTO esperado e limitações Free.
- Não presumir que snapshot único e histórico curto atendam a recuperação de desastre.

## 11. Integrações e automação

Caminho: Project > Integrations; CLI e API são superfícies complementares.

- Integrar Vercel somente após validar elegibilidade e escopo.
- Restringir tokens de API e rotacionar quando expostos.
- A CLI pode administrar branches/variáveis; não executar provisionamento nesta etapa documental.
- Manter scripts e migrations no Git.
- Não ativar Neon Auth, Data API, Functions, Object Storage ou AI Gateway como dependência sem requisito explícito e confirmação de elegibilidade Free.
- A aplicação de referência usa PostgreSQL e @neondatabase/serverless; não exige Data API.

## 12. Segurança e permissões

- Menor privilégio para papéis organizacionais e permissões por projeto.
- Não compartilhar API keys.
- Criar roles runtime via SQL com privilégios mínimos.
- Separar roles por função e branch.
- Ativar RLS em tabelas multi-tenant quando adequado.
- Usar consultas parametrizadas.
- Não expor DATABASE_URL no browser.
- Não usar role com BYPASSRLS como runtime.
- Não executar migrations com role runtime.

## 13. Fontes oficiais

- [Neon Plans](https://neon.com/docs/introduction/plans)
- [Free plan limits and quotas](https://neon.com/faqs/free-plan-limits-and-quotas)
- [Manage projects](https://neon.com/docs/manage/projects)
- [Manage branches](https://neon.com/docs/manage/branches)
- [Manage roles](https://neon.com/docs/manage/roles)
- [Connect to a Neon database](https://neon.com/docs/connect/connect-from-any-app)
- [Neon Serverless Driver](https://neon.com/docs/serverless/serverless-driver)
- [Restore data](https://neon.com/docs/introduction/restore)
- [Neon regions](https://neon.com/docs/introduction/regions)

## 14. Pendências de validação

- [ ] Confirmar organização e plano Free.
- [ ] Confirmar projeto, região, PostgreSQL version e branch padrão.
- [ ] Registrar papéis e permissões por projeto.
- [ ] Confirmar branches e quota restante.
- [ ] Confirmar databases, schemas e owners.
- [ ] Auditar roles e associação a neon_superuser.
- [ ] Confirmar role runtime sem BYPASSRLS.
- [ ] Validar pooled/direct connection sem expor strings.
- [ ] Conferir CU-hours, storage, egress e retenção.
- [ ] Validar backup externo e teste de restore.
