# 04 — Neon

## Configuração padrão inicial

1. Criar projeto na região mais próxima da aplicação e compatível com requisitos de residência.
2. Definir branch principal de produção e branches não produtivas com ciclo de vida explícito.
3. Separar roles de aplicação, migration e administração; evitar usar owner em runtime.
4. Usar pooling quando adequado ao padrão de conexões serverless e limites do plano.
5. Armazenar connection strings apenas em variáveis seguras por ambiente.
6. Versionar migrations no Git e executar mudanças via pipeline controlado.
7. Definir política de backup, restore e retenção compatível com plano e RPO/RTO.

## Configuração completa — inventário do console

| Área | Itens a registrar |
|---|---|
| Projects | nome, região, Postgres version, owner e finalidade |
| Branches | parent, estado, proteção, TTL, reset e promoção |
| Databases | nomes, schemas, extensões e ownership |
| Roles | privilégios, grants, rotação e separação por função |
| Compute | autoscaling, suspend, sizing e limites do plano |
| Connections | direct vs pooled, SSL, hostname e parâmetros (sem segredo) |
| Migrations | ferramenta, ordem, lock, compatibilidade e rollback |
| Backups / Restore | retenção, restore points, teste de recuperação e RPO/RTO |
| Monitoring | métricas, logs, consumo, alertas e limites |
| Security | IP restrictions, secrets, roles e auditoria disponíveis |

## Branching e ambientes

- Production: branch protegida, mudanças aprovadas e backups/restauração compreendidos.
- Staging: branch persistente derivada da base apropriada, com dados sintéticos/mascarados.
- Development: branch de desenvolvimento com dados descartáveis.
- Preview: branches efêmeras quando o fluxo e o plano permitirem; aplicar TTL e limpeza.

Não presumir que branch é backup independente. Documentar dependência do parent, comportamento de reset, retenção e custos.

## Conexões e migrations

Usar pooled connection para workloads serverless que abrem muitas conexões curtas, validando compatibilidade do driver e de prepared statements. Reservar conexão direta para tarefas que exigem esse comportamento. Aplicar TLS e não registrar connection strings.

Migrations devem ser idempotentes quando possível, revisadas, executadas por role específica e compatíveis com rollout gradual. Preferir expansão/contração (expand-contract) para alterações incompatíveis.

## Referências

- [Neon Documentation](https://neon.com/docs)
- [Neon branching](https://neon.com/branching)
- [Neon CLI](https://neon.com/docs/cli/install)

## Restrições Neon Free — baseline

No plano Free, considerar 10 branches por projeto, 100 CU-hours por projeto/mês, autoscaling até 2 CU, 1 GB de storage por projeto, 20 GB agregados na organização, 5 GB de transferência pública por projeto/mês e instant restore limitado a 6 horas/1 GB de histórico. Scale-to-zero após cinco minutos é obrigatório.

Protected branches nativas são **NÃO DISPONÍVEIS NO FREE**. Não apresentar branch de produção como protegida sem evidência. Usar controles compensatórios: migrations versionadas e revisadas, role administrativa separada, roles SQL de privilégio mínimo, backup externo criptografado e restore testado.

Atenção a RLS: roles criadas pelo Console/CLI/API herdam membership de neon_superuser, que possui BYPASSRLS em projetos criados após 15/08/2023. Criar roles de runtime/migration via SQL com grants explícitos e validar RLS com a role real da aplicação. Não usar role administrativa no runtime.
