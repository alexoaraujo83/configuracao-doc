# Migrations PostgreSQL com Neon

## Princípios

Migration é artefato de código: arquivo SQL versionado, imutável após aplicado e revisado junto com a mudança de aplicação. A sequência e o checksum detectam drift e divergência entre ambientes.

## Estrutura recomendada

~~~text
db/
  migrations/
    0001_create_accounts.sql
    0002_add_account_status.sql
    0003_add_membership_index.sql
~~~

Cada migration deve ser única, ordenada, pequena, revisada por impacto, lock, duração e volume; ter pré-condição, pós-condição e rollback/mitigação. Não editar migration já aplicada em ambiente compartilhado; criar migration corretiva.

## Estratégia expand-contract

1. Expand: adicionar coluna/tabela/índice ou aceitar formato antigo e novo.
2. Deploy compatível: código tolera schema anterior e novo.
3. Backfill idempotente e controlado.
4. Contract: remover estrutura antiga somente após migrar todos os consumidores.

Isso permite rollback do código sem exigir rollback destrutivo do schema.

## Fluxo por ambiente

- Local: banco local ou branch Neon não produtiva.
- PR/Preview: branch Neon isolada quando houver quota; caso contrário Postgres de teste controlado. Dados sintéticos.
- Staging lógico: branch dedicada somente se houver capacidade e política de retenção.
- Production: execução aprovada, coordenada e após revisar recuperação.

Neon branching cria ambiente isolado a partir de branch pai; branches podem conter dados do pai. Não usar cópia de produção em Preview sem anonimização e aprovação. A quota Free limita quantidade/uso; exclusão de branches temporárias é destrutiva e exige revisão.

## Exemplo SQL

~~~sql
BEGIN;

ALTER TABLE public.accounts
  ADD COLUMN status text NOT NULL DEFAULT 'active';

ALTER TABLE public.accounts
  ADD CONSTRAINT accounts_status_check
  CHECK (status IN ('active', 'suspended')) NOT VALID;

ALTER TABLE public.accounts
  VALIDATE CONSTRAINT accounts_status_check;

COMMIT;
~~~

Revisar locks e tempo de validação em tabelas grandes. Defaults e constraints podem ter custo; não presumir operação sem lock.

## Histórico de migrations

Manter tabela de controle (por exemplo schema_migrations) com version, checksum, applied_at e metadata mínima. Executor deve adquirir lock contra concorrência, verificar versão/checksum, aplicar em ordem, gravar sucesso e parar na primeira falha. Não ignorar migration quebrada automaticamente.

Se ORM for usado, adotar o mecanismo oficial dele e não misturar gerenciadores no mesmo schema.

## Roles e segurança

- Executor de migration usa role dedicada com DDL necessário, nunca credencial runtime.
- Runtime permanece sem ownership, CREATEDB, CREATEROLE, SUPERUSER ou BYPASSRLS.
- Revisar políticas RLS após alterações.
- Connection string de migration não aparece em logs nem artifacts.
- Nunca rodar migration em workflow de PR contra produção.

## Rollback

Preferir forward-fix para schema. Rollback SQL somente se operação inversa for segura, reversível e testada em ambiente isolado. DROP, perda de dados, reescrita e downgrade de enum podem não ser reversíveis. Restore/PITR pode perder alterações posteriores e exige aprovação.

## Referências

- https://neon.com/docs/get-started/workflow-primer
- https://neon.com/docs/introduction/branching
- https://www.postgresql.org/docs/current/ddl-alter.html
- https://www.postgresql.org/docs/current/explicit-locking.html
