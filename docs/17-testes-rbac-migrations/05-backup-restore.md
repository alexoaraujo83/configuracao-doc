# 05 — Backup, restore e recuperação

**Estado:** procedimento documentado; nenhum backup ou restore executado.
**Restrição:** Neon Free; não presumir PITR, histórico, snapshots ou retenção além da quota disponível.

## 1. Objetivos
Definir por projeto:
- **RPO:** perda máxima de dados aceitável.
- **RTO:** tempo máximo aceitável para recuperar serviço.
- Escopo: database, schema, dados, roles, secrets, Auth0 config e aplicação.
- Responsáveis, dependências e ordem de recuperação.

Não declarar RPO/RTO garantidos sem ensaio real e confirmação das capacidades Free atuais.

## 2. Fontes de recuperação

| Artefato | Fonte | Observação |
|---|---|---|
| Schema e migrations | GitHub | Versionado |
| Dados | Backup lógico externo | Criptografado e restrito |
| Roles e grants | Scripts SQL sem senhas | Segredos separados |
| Configuração da aplicação | GitHub | Excluir secrets |
| Variáveis e secrets | Cofre/gerenciador de ambiente | Nunca versionar |
| Configuração Auth0 | Exportação/API quando permitida | Validar limites Free |
| Deploy | Histórico de releases | Não substitui backup |

## 3. Backup lógico

Executar em ambiente controlado com cliente PostgreSQL e credencial de leitura apropriada:

~~~bash
pg_dump "$DATABASE_URL_BACKUP" \
  --no-owner \
  --no-privileges \
  --format=custom \
  --file=database-backup.dump
~~~

Definir retenção, criptografia, acesso e destino fora do repositório. Não usar credencial administrativa se role de backup restrita puder atender.

O dump de database não é exportação completa de usuários/roles globais do cluster. Manter scripts de infraestrutura sem senhas e provisionar segredos separadamente.

## 4. Verificação do backup
1. Confirmar que o arquivo existe e tem tamanho plausível.
2. Calcular checksum.
3. Criptografar antes de transferir/armazenar.
4. Restringir permissões e acesso.
5. Registrar data, origem lógica, versão PostgreSQL e checksum.
6. Não anexar dump em PR, issue ou chat.

## 5. Restore de ensaio

Executar somente em database/branch de teste isolado:

~~~bash
pg_restore \
  --no-owner \
  --no-privileges \
  --dbname="$RESTORE_DATABASE_URL" \
  database-backup.dump
~~~

A criação de database e o uso de pg_restore dependem de privilégios e compatibilidade da conexão. Adaptar ao ambiente Neon e à role autorizada; nunca apontar para produção por engano.

Validar tabelas, constraints, índices, extensões, consistência de dados fictícios, versão do schema, grants, RLS e login/autorização com usuários de teste.

## 6. Recuperação de incidente
1. Identificar incidente e congelar alterações não essenciais.
2. Avaliar integridade e escolher ponto de recuperação disponível.
3. Criar ambiente isolado para restore.
4. Restaurar e validar schema/dados.
5. Reaplicar roles, grants, RLS e secrets por canal seguro.
6. Validar autenticação, autorização e fluxos críticos.
7. Planejar troca de conexão/cutover com aprovação.
8. Monitorar erros e consistência após recuperação.
9. Registrar causa, impacto, tempo real e melhorias.

Não sobrescrever ou excluir banco original durante diagnóstico sem autorização explícita e plano de preservação.

## 7. Ensaio periódico
- [ ] Backup lógico criado.
- [ ] Checksum calculado.
- [ ] Backup criptografado.
- [ ] Restore concluído em ambiente isolado.
- [ ] Migrations e grants conferidos.
- [ ] RLS validado com dois usuários de teste.
- [ ] Aplicação conectou ao banco restaurado.
- [ ] RPO/RTO reais registrados.
- [ ] Arquivos temporários eliminados com segurança.
- [ ] Cotas Free e custo zero confirmados.

## 8. Limitações Free
Branching e histórico de restauração do Neon não são backup independente. Confirmar limites de retenção, snapshots, armazenamento e restauração na data do ensaio. Se a necessidade de recuperação não couber no Free, registrar o risco e redesenhar o requisito sem assumir recurso pago.
