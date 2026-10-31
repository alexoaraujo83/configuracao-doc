# Persistência e banco de dados — Next.js + Neon Free

**Status:** documentação de referência; não executada.  
**Escopo:** exclusivamente planos Free, PostgreSQL no Neon e aplicação Next.js + TypeScript.  
**Objetivo:** padronizar modelagem, constraints, migrations, índices, transações, recuperação e evolução do schema.

> Este guia não cria banco, roles, tabelas, branches, migrations ou backups. Os exemplos são modelos que exigem revisão e adaptação. Confirme quotas, recursos e retenção diretamente no painel e na documentação oficial antes de qualquer operação.

## 1. Princípios e arquitetura de persistência

- O PostgreSQL é a fonte de verdade dos dados transacionais.
- A aplicação acessa o banco somente no servidor Next.js; nunca exponha DATABASE_URL ao browser.
- Use conexão pooled para tráfego serverless quando apropriado e mantenha a connection string em variável server-only.
- Separe identidade externa Auth0 de entidades de domínio locais. Use o claim estável `sub` como identificador externo; não use e-mail como chave primária.
- A autorização ocorre no servidor por operação, recurso e tenant. RLS é defesa adicional, não substituto da autorização da aplicação.
- O runtime não deve possuir privilégios de DDL, ser owner das tabelas ou ter BYPASSRLS.
- Toda mudança de schema é versionada, revisada e aplicada por fluxo controlado.
- Não misture migrations de produção com inicialização automática em cada cold start.

### 1.1 Camadas

| Camada | Responsabilidade |
|---|---|
| Route Handler / Server Action | autenticação, validação de entrada, autorização e DTO |
| Service | regra de negócio e limite transacional |
| Repository | SQL parametrizado e mapeamento de linhas |
| PostgreSQL | constraints, integridade, transações e RLS |
| Migration | evolução reproduzível do schema |

### 1.2 Configuração de conexão

Exemplo ilustrativo de variável:

~~~dotenv
DATABASE_URL=postgresql://<role>:<password>@<endpoint>/<database>?sslmode=require
~~~

Regras:
- Armazenar o valor real somente no ambiente seguro do servidor.
- Não prefixar com `NEXT_PUBLIC_`.
- Não imprimir a connection string em logs, traces ou mensagens de erro.
- Preferir pooling Neon para aplicações serverless; validar limites de conexões e comportamento do driver.
- Não compartilhar credenciais entre runtime e migrator.
- A rotação de credenciais deve seguir procedimento separado e aprovado.

## 2. Modelo entidade-relacionamento de referência

O modelo abaixo representa uma aplicação multi-tenant com usuários locais, memberships, projetos e convites. É uma proposta de domínio, não uma implementação existente.

~~~mermaid
erDiagram
  APP_USERS ||--o{ TENANT_MEMBERS : participates
  TENANTS ||--o{ TENANT_MEMBERS : contains
  TENANTS ||--o{ PROJECTS : owns
  TENANTS ||--o{ INVITATIONS : issues
  APP_USERS ||--o{ PROJECTS : creates
  APP_USERS ||--o{ INVITATIONS : invites

  APP_USERS {
    uuid id PK
    varchar auth0_sub UK
    timestamptz created_at
    timestamptz updated_at
  }
  TENANTS {
    uuid id PK
    varchar slug UK
    varchar name
    timestamptz created_at
    timestamptz updated_at
  }
  TENANT_MEMBERS {
    uuid id PK
    uuid tenant_id FK
    uuid user_id FK
    varchar role
    timestamptz created_at
  }
  PROJECTS {
    uuid id PK
    uuid tenant_id FK
    uuid created_by FK
    varchar name
    varchar status
    timestamptz created_at
    timestamptz updated_at
  }
  INVITATIONS {
    uuid id PK
    uuid tenant_id FK
    varchar email
    varchar token_hash
    varchar status
    timestamptz expires_at
    uuid invited_by FK
    timestamptz created_at
  }
~~~

### 2.1 Regras do modelo

1. `app_users.auth0_sub` identifica a identidade Auth0 de forma estável e única.
2. `tenant_members` representa o vínculo usuário-tenant; uma pessoa pode pertencer a vários tenants.
3. `UNIQUE (tenant_id, user_id)` impede membership duplicada.
4. `projects.tenant_id` é obrigatório; projeto sempre pertence a um tenant.
5. `projects.created_by` referencia o usuário local que criou o projeto.
6. `invitations.token_hash` guarda somente hash do token de convite; nunca o token em claro.
7. Convites devem ter expiração, status e consumo atômico.
8. Remoções devem preferir soft delete apenas quando houver requisito de auditoria/recuperação; definir política de retenção antes de adicionar colunas de exclusão.
9. IDs UUID são identificadores internos; não substituem autorização.
10. Dados de tenant devem sempre ser filtrados por tenant e protegidos por RLS quando a arquitetura exigir.

## 3. Dicionário de dados e constraints

| Tabela | Coluna | Tipo sugerido | Nulável | Regra |
|---|---|---|---|---|
| app_users | id | uuid | não | PK, gerado no servidor/banco |
| app_users | auth0_sub | text | não | UNIQUE; identificador Auth0 |
| app_users | created_at | timestamptz | não | DEFAULT now() |
| app_users | updated_at | timestamptz | não | atualizado pela aplicação ou trigger documentada |
| tenants | id | uuid | não | PK |
| tenants | slug | text | não | UNIQUE; formato validado |
| tenants | name | text | não | comprimento limitado |
| tenant_members | tenant_id | uuid | não | FK tenants(id) |
| tenant_members | user_id | uuid | não | FK app_users(id) |
| tenant_members | role | text | não | CHECK allowlist de papéis |
| projects | id | uuid | não | PK |
| projects | tenant_id | uuid | não | FK tenants(id) |
| projects | created_by | uuid | não | FK app_users(id) |
| projects | name | text | não | comprimento limitado |
| projects | status | text | não | CHECK allowlist de estados |
| invitations | token_hash | text | não | UNIQUE; hash, nunca segredo original |
| invitations | expires_at | timestamptz | não | expiração obrigatória |
| invitations | status | text | não | CHECK de estados permitidos |

### 3.1 Exemplo SQL de schema inicial

O exemplo é deliberadamente enxuto. Ajuste tipos, nomes, extensões UUID e política de exclusão ao domínio real antes de adotá-lo.

~~~sql
CREATE TABLE app_users (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  auth0_sub text NOT NULL UNIQUE,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE tenants (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  slug text NOT NULL UNIQUE,
  name text NOT NULL CHECK (length(name) BETWEEN 1 AND 120),
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE tenant_members (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  user_id uuid NOT NULL REFERENCES app_users(id) ON DELETE RESTRICT,
  role text NOT NULL CHECK (role IN ('owner', 'admin', 'member', 'viewer')),
  created_at timestamptz NOT NULL DEFAULT now(),
  CONSTRAINT tenant_members_tenant_user_uk UNIQUE (tenant_id, user_id)
);

CREATE TABLE projects (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  created_by uuid NOT NULL REFERENCES app_users(id) ON DELETE RESTRICT,
  name text NOT NULL CHECK (length(name) BETWEEN 1 AND 160),
  status text NOT NULL DEFAULT 'active'
    CHECK (status IN ('active', 'archived')),
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE invitations (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  email text NOT NULL,
  token_hash text NOT NULL UNIQUE,
  status text NOT NULL DEFAULT 'pending'
    CHECK (status IN ('pending', 'accepted', 'revoked', 'expired')),
  expires_at timestamptz NOT NULL,
  invited_by uuid NOT NULL REFERENCES app_users(id) ON DELETE RESTRICT,
  created_at timestamptz NOT NULL DEFAULT now()
);
~~~

### 3.2 Constraints e integridade

- PK em toda tabela; FK para relações de domínio.
- UNIQUE para identidade externa, slug e chave composta de membership.
- NOT NULL para campos necessários ao ciclo de vida.
- CHECK para enums simples e invariantes locais.
- Não usar CHECK para regras que dependem de outras linhas/tabelas; usar FK, UNIQUE, EXCLUDE, transação ou regra de domínio.
- Definir explicitamente `ON DELETE`: CASCADE somente quando a remoção do pai deve remover os filhos; RESTRICT quando preservar histórico é necessário.
- E-mail deve ser normalizado para comparação de convites; decidir política de case-folding.
- Não armazenar tokens de sessão Auth0 nem credenciais de terceiros no modelo de domínio.
- Campos monetários devem usar NUMERIC com escala definida, nunca ponto flutuante.
- Datas de negócio devem distinguir instante (`timestamptz`) de data civil (`date`).

## 4. Migrations e versionamento do schema

### 4.1 Organização recomendada

~~~text
database/
  migrations/
    0001_create_app_users.sql
    0002_create_tenants.sql
    0003_create_memberships.sql
    0004_create_projects.sql
    0005_create_invitations.sql
  seeds/
    README.md
  README.md
~~~

A ferramenta de migrations deve registrar versão aplicada, checksum, instante, duração e resultado. Escolha uma única ferramenta compatível com o stack e documente sua versão; não mantenha dois mecanismos concorrentes.

### 4.2 Regras de migration

1. Uma migration representa uma mudança lógica pequena e revisável.
2. Arquivos aplicados não são editados retroativamente; correções geram nova migration.
3. Nome e versão são imutáveis e ordenáveis.
4. Executar primeiro em branch de banco não produtiva, quando disponível na quota Free.
5. Revisar SQL, locks potenciais, volume de dados, índices e compatibilidade com a versão anterior da aplicação.
6. Aplicar migrations com credencial migrator, nunca com runtime.
7. Registrar resultado sem registrar valores de dados pessoais ou secrets.
8. Evitar seed com dados reais; fixtures devem ser fictícias e reproduzíveis.
9. Em falha, interromper o pipeline e seguir runbook; não ignorar erro nem marcar migration como aplicada manualmente sem diagnóstico.
10. A aplicação deve tolerar janela de deploy em que versões antiga e nova coexistam.

### 4.3 Expand-and-contract

Para mudanças incompatíveis:
1. **Expand:** adicionar coluna/tabela opcional ou estrutura compatível.
2. **Deploy compatível:** publicar código capaz de ler/escrever formato antigo e novo.
3. **Backfill:** preencher dados em lotes controlados, com checkpoint e idempotência.
4. **Switch:** alterar leitura para o novo formato.
5. **Contract:** remover estrutura antiga somente após confirmar que nenhum código ativo depende dela.

Não combinar rename/drop de coluna e deploy de código dependente numa única etapa sem compatibilidade intermediária.

### 4.4 Pipeline documental

~~~text
Pull Request
  -> revisão do SQL e impacto
  -> migration em branch de desenvolvimento
  -> validação automatizada em banco descartável/isolado
  -> aprovação
  -> aplicação controlada no ambiente-alvo
  -> verificação pós-migration
  -> registro de versão e evidências
~~~

O fluxo acima é uma proposta. Nenhum workflow, migration ou banco foi executado neste PR.

## 5. Índices e otimização de consultas

### 5.1 Índices iniciais propostos

~~~sql
CREATE INDEX projects_tenant_created_id_idx
  ON projects (tenant_id, created_at DESC, id DESC);

CREATE INDEX tenant_members_user_tenant_idx
  ON tenant_members (user_id, tenant_id);

CREATE INDEX invitations_tenant_status_expires_idx
  ON invitations (tenant_id, status, expires_at);
~~~

A constraint UNIQUE em `tenant_members(tenant_id, user_id)` já cria índice único correspondente; não duplicar índice com mesmas colunas e ordem.

### 5.2 Diretrizes

- Indexar colunas usadas frequentemente em WHERE, JOIN, ORDER BY e constraints, com base em consultas observadas.
- Índices compostos devem refletir prefixo de filtros e ordenação usados na query.
- Não indexar indiscriminadamente: índices aumentam armazenamento e custo de escrita.
- Usar `EXPLAIN (ANALYZE, BUFFERS)` somente em ambiente controlado; ANALYZE executa a consulta.
- Comparar estimativas e planos com dados representativos, sem expor dados pessoais.
- Evitar SELECT *; selecionar apenas colunas necessárias.
- Paginação de grandes conjuntos deve preferir keyset/cursor em vez de OFFSET elevado.
- Para busca textual, avaliar necessidade real antes de adotar GIN/trigramas e confirmar custo.
- Rever índices não utilizados com estatísticas disponíveis, observando período e reinícios.
- Não alegar performance sem benchmark reproduzível.

### 5.3 Paginação keyset

~~~sql
SELECT id, name, status, created_at
FROM projects
WHERE tenant_id = $1
  AND (created_at, id) < ($2, $3)
ORDER BY created_at DESC, id DESC
LIMIT $4;
~~~

O cursor deve ser opaco, validado e vinculado ao tenant, filtros e ordenação. O backend não deve aceitar tenant do cursor sem validar membership e autorização em cada requisição.

## 6. Transações e integridade operacional

- Agrupar operações que formam uma única unidade de negócio na mesma transação.
- Definir nível de isolamento conscientemente; começar pelo padrão PostgreSQL e elevar somente quando a invariância exigir.
- Tratar deadlocks e serialization failures com retry limitado, idempotente e com backoff.
- Nunca manter transação aberta durante chamadas HTTP externas, envio de e-mail ou interação do usuário.
- Para efeitos externos, usar outbox transacional ou processo idempotente, quando necessário.
- Usar parâmetros SQL; nunca concatenar entrada do usuário.
- Evitar transações longas, que retêm locks e versões antigas de linhas.
- Não presumir que várias chamadas HTTP ao driver compartilham a mesma sessão/transação. Verificar a API do driver escolhido.
- Com driver HTTP/serverless, utilizar mecanismo documentado de transação interativa quando necessário; uma sequência de chamadas independentes não equivale a uma transação.
- Em operações críticas, usar chave de idempotência persistida e constraint UNIQUE.
- Respeitar timeout e limite de conexão do runtime serverless.

### 6.1 Exemplo conceitual de atomicidade

~~~sql
BEGIN;

INSERT INTO tenant_members (tenant_id, user_id, role)
VALUES ($1, $2, 'member')
ON CONFLICT (tenant_id, user_id) DO NOTHING;

UPDATE invitations
SET status = 'accepted'
WHERE id = $3
  AND tenant_id = $1
  AND status = 'pending'
  AND expires_at > now();

COMMIT;
~~~

Este fragmento é conceitual: a aplicação deve verificar que o convite foi consumido exatamente uma vez e validar token, usuário, tenant e autorização antes do commit. Em caso de resultado inválido, efetuar ROLLBACK.

## 7. Roles PostgreSQL e privilégio mínimo

Separar proprietário, migrator e runtime. O desenho abaixo é referência; confirme permissões e sintaxe no banco-alvo antes de executar.

~~~sql
-- Papel sem login para propriedade do schema/objetos.
CREATE ROLE app_owner NOLOGIN;

-- Credencial exclusiva para migrations.
CREATE ROLE app_migrator LOGIN NOINHERIT;
GRANT app_owner TO app_migrator;

-- Credencial de runtime sem privilégios de owner ou bypass RLS.
CREATE ROLE app_runtime LOGIN NOINHERIT NOBYPASSRLS;

GRANT CONNECT ON DATABASE appdb TO app_runtime;
GRANT USAGE ON SCHEMA public TO app_runtime;
GRANT SELECT, INSERT, UPDATE, DELETE
  ON ALL TABLES IN SCHEMA public TO app_runtime;
GRANT USAGE, SELECT
  ON ALL SEQUENCES IN SCHEMA public TO app_runtime;
~~~

Configurar default privileges para objetos futuros com o owner correto. Conceder apenas operações necessárias por tabela; não conceder CREATE, ALTER, DROP, TRUNCATE, SUPERUSER ou BYPASSRLS ao runtime. A senha de migrator não deve existir no ambiente de execução público.

## 8. Isolamento multi-tenant com RLS

RLS deve complementar a autorização server-side. Exemplo ilustrativo para projetos:

~~~sql
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
ALTER TABLE projects FORCE ROW LEVEL SECURITY;

CREATE POLICY projects_tenant_isolation
ON projects
USING (
  tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid
)
WITH CHECK (
  tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::uuid
);
~~~

Requisitos:
- Runtime não é owner, superuser nem BYPASSRLS.
- Definir tenant context dentro da mesma transação que executa a consulta.
- Preferir `set_config('app.tenant_id', value, true)` com escopo local à transação.
- Validar membership e permissões antes de definir contexto.
- Não confiar em tenant_id enviado pelo browser como autorização.
- Testar ausência de contexto, tenant inexistente, troca de tenant, operações INSERT/UPDATE e acesso cruzado.
- Contexto definido pela própria aplicação não é fronteira de segurança contra backend comprometido; proteção em camadas é obrigatória.
- Aplicar políticas equivalentes às tabelas tenant-scoped, incluindo tabelas auxiliares.
- Rever caminhos de execução por views, functions e jobs para impedir bypass acidental.

## 9. Backup, retenção e recuperação no Neon Free

### 9.1 Limites e premissas

- O Neon oferece histórico de dados/PITR conforme retenção configurada e elegibilidade do plano.
- O histórico PITR não deve ser tratado como backup externo independente nem como garantia de recuperação de longo prazo.
- A janela de histórico e quotas do Free podem mudar; validar no painel e na página oficial de planos antes de definir RPO/RTO.
- Não presumir que o Free oferece retenção longa, backup cross-region, export automático agendado ou suporte operacional dedicado.
- Branches são úteis para isolamento e testes, mas não substituem cópia externa de backup.
- Uma branch de recuperação pode consumir recursos/quotas do plano.

### 9.2 Estratégia documental mínima

1. Definir RPO e RTO aceitáveis para o projeto, sem prometer valores não confirmados.
2. Identificar o período de histórico PITR visível no projeto e registrar data da verificação.
3. Manter migrations versionadas no Git como registro reproduzível do schema.
4. Planejar export lógico periódico para armazenamento independente, se necessário, usando recursos gratuitos e respeitando limites/termos.
5. Proteger exports com controle de acesso, criptografia e política de retenção.
6. Documentar responsável, frequência, local, formato, integridade e procedimento de recuperação.
7. Fazer restore somente em ambiente isolado e com autorização; validar schema, contagens, constraints e consistência da aplicação.
8. Definir critérios para apontar a aplicação à base recuperada e plano de retorno.
9. Registrar evidência, duração e divergências após exercício autorizado.
10. Não guardar backup no mesmo projeto/conta como única cópia.

### 9.3 PITR e recuperação

PITR permite recuperar estado para um ponto no tempo dentro da janela disponível, sujeito aos recursos e limites do plano. Antes de qualquer restore:
- Confirmar projeto, branch e timestamp de destino.
- Avaliar perda de dados posterior ao ponto escolhido.
- Criar destino separado quando possível e preservar a origem.
- Validar integridade e compatibilidade da aplicação.
- Planejar atualização de DATABASE_URL e novo deployment somente após aprovação.
- Nunca sobrescrever produção sem plano de retorno e autorização explícita.

### 9.4 RPO/RTO

| Item | Definição |
|---|---|
| RPO | perda máxima de dados tolerável, definida pelo responsável |
| RTO | tempo máximo tolerável para restabelecimento |
| PITR disponível | janela exibida no projeto/plano no momento da verificação |
| Backup externo | export independente, se adotado |
| Restore testado | somente marcar após exercício real autorizado e evidenciado |

Não preencher RPO/RTO com valores arbitrários nem declarar recuperação testada sem evidência.

## 10. Evolução e rollback do schema

### 10.1 Checklist de revisão

- [ ] Objetivo e impacto da migration descritos.
- [ ] Compatibilidade com código atual e anterior avaliada.
- [ ] Constraints e índices revisados.
- [ ] Lock e duração potencial estimados.
- [ ] Alterações grandes divididas em etapas.
- [ ] Backfill idempotente, em lotes e retomável.
- [ ] Plano de observabilidade e validação definido.
- [ ] Plano de rollback ou roll-forward documentado.
- [ ] Dados pessoais e segredos ausentes dos logs.
- [ ] Aprovação do responsável registrada.

### 10.2 Rollback

- Preferir migration corretiva (roll-forward) quando a reversão implicar perda de dados.
- Migration down só é segura quando a operação inversa é realmente reversível e não descarta dados relevantes.
- Antes de DROP, truncamento ou transformação destrutiva, exigir backup/restore point adequado e aprovação explícita.
- Não apagar arquivo de migration já aplicado.
- Em falha parcial, consultar tabela de controle da ferramenta de migrations e estado real do schema; não repetir cegamente.
- Código deve suportar schema anterior durante a janela de rollback.
- Restaurar backup/PITR é procedimento de recuperação, não substituto de rollback de código.

### 10.3 Matriz de decisão

| Mudança | Estratégia preferencial |
|---|---|
| Adicionar coluna nullable | expand e deploy compatível |
| Adicionar coluna NOT NULL | adicionar nullable, backfill, validar, depois constraint |
| Renomear coluna | expand, dual read/write, migração, contract |
| Remover coluna | parar uso, observar, remover em migration posterior |
| Alterar tipo | coluna paralela, backfill, troca gradual |
| Criar índice em tabela grande | avaliar operação concorrente e suporte do ambiente |
| Alterar regra de negócio | validação na aplicação + constraint quando aplicável |

## 11. Segurança e privacidade dos dados

- Classificar dados por sensibilidade e coletar apenas o necessário.
- Definir retenção, exclusão e anonimização conforme requisitos legais e de produto.
- Não persistir access tokens, refresh tokens ou client secrets em tabelas genéricas.
- Hash de tokens de convite e tokens de recuperação; armazenar somente metadados necessários.
- Parametrizar SQL e validar entradas com schema runtime.
- Não enviar dados pessoais para logs de debug ou analytics sem necessidade.
- Restringir acesso ao painel e credenciais do banco.
- Separar credenciais de development, preview e production.
- Revisar privilégios de roles periodicamente.
- Usar TLS/SSL na conexão.
- Tratar dumps e exports como dados sensíveis.

## 12. Testes de persistência e critérios de aceite

| Área | Teste esperado |
|---|---|
| Constraints | duplicidade e valores inválidos são rejeitados |
| FK | referência inexistente não é persistida |
| Transação | falha intermediária não deixa estado parcial |
| Idempotência | repetição não duplica efeito |
| RLS | usuário de um tenant não lê/escreve dados de outro |
| Runtime role | DDL e operações privilegiadas são negadas |
| Migrations | banco vazio chega ao schema esperado |
| Upgrade | schema anterior migra sem perda não planejada |
| Rollback | procedimento documentado e validado em ambiente isolado |
| Paginação | cursores não pulam nem duplicam registros em ordenação estável |
| Performance | plano de execução avaliado com dados representativos |
| Recuperação | restore isolado validado, se o projeto exigir backup |

Critérios de aceite:
- Schema versionado e reproduzível.
- Todas as tabelas possuem PK e constraints apropriadas.
- Runtime usa role de privilégio mínimo.
- Queries são parametrizadas.
- Transações são compatíveis com o driver e o limite de conexão.
- RLS está aplicada e testada em tabelas tenant-scoped.
- Índices possuem justificativa baseada em consultas.
- Procedimentos de backup/restore refletem recursos Free realmente disponíveis.
- Nenhuma operação real é declarada sem evidência.

## 13. Limitações e decisões pendentes

| Tema | Situação |
|---|---|
| Ferramenta de migrations | Pendente de escolha e validação com stack/repositório |
| Modelo ER | Referência proposta; validar domínio real |
| Extensão/geração UUID | Confirmar disponibilidade/configuração |
| RLS por tabela | Implementação e testes pendentes |
| Driver/transações interativas | Confirmar versão e API suportada |
| PITR/retention Free | Conferir painel e documentação atual na data da operação |
| Backup externo | Definir destino gratuito, frequência e retenção |
| RPO/RTO | Definir pelo responsável do sistema |
| Quotas Neon Free | Conferir valores atuais; não fixar números neste documento |

## 14. Fontes oficiais

- Neon Docs: https://neon.com/docs
- Neon Serverless Driver: https://neon.com/docs/serverless/serverless-driver
- Neon branching: https://neon.com/docs/introduction/branching
- Neon plans: https://neon.com/docs/introduction/plans
- Neon CLI: https://neon.com/docs/reference/neon-cli
- PostgreSQL constraints: https://www.postgresql.org/docs/current/ddl-constraints.html
- PostgreSQL indexes: https://www.postgresql.org/docs/current/indexes.html
- PostgreSQL transactions: https://www.postgresql.org/docs/current/tutorial-transactions.html
- PostgreSQL RLS: https://www.postgresql.org/docs/current/ddl-rowsecurity.html

**Revisão necessária:** confirmar URLs, disponibilidade e limites Free antes da implantação. Esta seção não substitui documentação oficial nem constitui confirmação de recursos ativos na conta.

## 15. Histórico e estado

- Documento criado em branch de documentação.
- Exemplos SQL e Mermaid são referências não executadas.
- Nenhum pacote foi instalado.
- Nenhuma tabela, role, migration, branch Neon ou backup foi criado.
- Nenhum restore, teste, deployment ou alteração de credencial foi executado.
