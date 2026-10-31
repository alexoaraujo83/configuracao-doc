# 03 — Migrations e evolução do schema PostgreSQL

**Estado:** procedimento documentado; não executado.
**Executor de referência:** cliente psql e migrations SQL versionadas.
**Credencial:** role migrator separada; nunca runtime.

## 1. Regras
1. Cada alteração de schema é um arquivo SQL novo e imutável.
2. Não editar migration já aplicada em ambiente compartilhado.
3. Revisar SQL em Pull Request antes de executar.
4. Executar primeiro em ambiente descartável/de desenvolvimento.
5. Fazer backup antes de alterações destrutivas.
6. Usar role migrator, que pode assumir owner explicitamente; nunca usar credencial administrativa em runtime.
7. Não colocar senha ou connection string em arquivo versionado.
8. Registrar versão, checksum, data, ambiente e resultado sem registrar segredo.
9. Avaliar lock, tempo, tamanho de tabela e compatibilidade com deploy.
10. Mudança incompatível exige estratégia expand/contract em etapas.

## 2. Estrutura proposta

~~~text
database/
  migrations/
    0001_create_app_schema.sql
    0002_create_users.sql
    0003_add_roles.sql
    0004_create_documents.sql
  README.md
~~~

Usar sequência monotônica e nomes descritivos. Uma migration deve ter escopo pequeno e previsível.

## 3. Tabela de controle

~~~sql
CREATE TABLE IF NOT EXISTS app.schema_migrations (
  version TEXT PRIMARY KEY,
  checksum TEXT NOT NULL,
  applied_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
~~~

O runner deve:
- Ler arquivos ordenados.
- Calcular SHA-256 do conteúdo.
- Verificar se a versão aplicada existe.
- Comparar checksum e interromper se arquivo aplicado foi alterado.
- Executar migration e inserir registro na mesma transação, quando o DDL permitir.
- Parar no primeiro erro.
- Não executar duas instâncias concorrentes sem lock/coordenação.
- Produzir log sem connection string.

PostgreSQL suporta transações para grande parte do DDL, mas alguns comandos têm restrições transacionais. Consultar a documentação de cada comando e separar esses passos quando necessário.

## 4. Execução manual com psql

Pré-requisitos: cliente PostgreSQL instalado, connection string direta/compatível e segredo fornecido por canal seguro.

~~~bash
psql "$DATABASE_URL_MIGRATOR" \
  -v ON_ERROR_STOP=1 \
  -f database/migrations/0001_create_app_schema.sql
~~~

Se o arquivo usa SET ROLE app_owner, validar que o migrator é membro autorizado e que o comando retorna ao papel anterior ao final. Em falha, revisar estado antes de tentar novamente; não assumir que qualquer migration parcialmente executada foi revertida.

Não imprimir connection string em logs. Não fornecer segredo em argumentos visíveis do processo quando o ambiente permitir mecanismo mais seguro, como variável protegida ou arquivo de serviço com permissões restritas.

## 5. Estratégia expand/contract
Para renomear/remover coluna consumida pela versão atual:
1. **Expand:** adicionar coluna/tabela nova mantendo a antiga.
2. Deploy de código que grava compativelmente nos dois formatos, se necessário.
3. Backfill controlado e verificável.
4. Deploy de código que lê o novo formato.
5. **Contract:** remover coluna antiga em migration posterior, após confirmar que nenhuma versão ativa a usa.

Não combinar deploy incompatível e remoção imediata em uma única etapa.

## 6. Rollback
Rollback de banco não é sempre o inverso automático do SQL. Preferir migration corretiva forward-only, backup validado, plano de reversão explícito e testado e deploy compatível com schema anterior durante a janela de release.

Para cada migration documentar objetivo, pré-condições, SQL, impacto em dados/locks, compatibilidade, rollback/forward-fix e evidência em ambiente de teste.

## 7. Checklist
- [ ] Nome e número únicos.
- [ ] SQL revisado e sem segredos.
- [ ] Testado em banco descartável.
- [ ] Permissões verificadas com migrator/runtime.
- [ ] Impacto de lock/volume avaliado.
- [ ] Backup/restore considerado.
- [ ] Compatibilidade de deploy verificada.
- [ ] Checksum registrado.
- [ ] Plano de correção documentado.

## 8. Referências
- [PostgreSQL — SQL Commands](https://www.postgresql.org/docs/current/sql-commands.html)
- [PostgreSQL — Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html)
- [PostgreSQL — psql](https://www.postgresql.org/docs/current/app-psql.html)
- [Neon — Branching](https://neon.com/docs/introduction/branching)
