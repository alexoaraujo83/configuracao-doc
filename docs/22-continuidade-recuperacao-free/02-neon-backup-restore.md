# Neon Free — backup e restore PostgreSQL

## Capacidade Free
A documentação Neon consultada informa history window de 6 horas no Free para Instant Restore/Time Travel/branching de estado anterior. Confirmar em Console → Settings → Instant restore se está habilitado e o valor efetivo. Reduzir a zero desabilita restore histórico. Não confundir essa janela com retenção de dump lógico externo nem com backups internos do provedor.

## Estratégia em camadas
1. PITR/branch histórico Neon dentro da janela efetiva.
2. Dump lógico independente, criptografado e armazenado fora do Neon e GitHub.
3. Schema/migrations versionados em Git, sem dados reais.
4. Restore de teste isolado, validando schema, constraints, RLS e amostras.
5. Cópia offline cifrada em meio já disponível; não pressupor armazenamento externo gratuito.

## Exemplo de dump lógico
Em máquina controlada; DATABASE_URL obtida por canal seguro e nunca impressa:

    umask 077
    export PGCONNECT_TIMEOUT=10
    pg_dump "$DATABASE_URL" --format=custom --no-owner --no-acl --file="backup.dump"
    sha256sum backup.dump

Validar compatibilidade do endpoint com pg_dump. Não colocar connection string no histórico do shell, logs ou argumentos de CI. Cifrar arquivo antes de transferir e custodiar chave separadamente. Registrar projeto/branch, UTC, versão, tamanho, SHA-256 e resultado. Frequência derivada do RPO; exemplo não agenda backup.

## Restore isolado
    pg_restore --dbname="$RESTORE_DATABASE_URL" --no-owner --no-acl --exit-on-error backup.dump

Criar database/branch pelo Console/CLI conforme procedimento Neon vigente. Comando é modelo e não deve apontar para produção sem aprovação.

## PITR e recuperação seletiva
1. Determinar timestamp UTC anterior ao incidente.
2. Criar branch restaurada usando Console/CLI.
3. Conectar read-only e comparar schema, contagens e registros.
4. Exportar somente linhas necessárias e reconciliar gravações posteriores.
5. Validar roles, grants, ownership, RLS ENABLE/FORCE, policies, extensões, sequences e constraints.
6. Para cutover total, congelar gravações, planejar nova DATABASE_URL e obter aprovação.
7. Preservar branch original e evidências até encerrar.

## Segurança
Backups contêm dados sensíveis: cifrar, limitar acesso e retenção. Nunca armazenar dumps em Git/Git LFS, Actions artifacts ou Vercel deployments. Não restaurar com role runtime. Hash comprova integridade desde a coleta, não completude nem restaurabilidade.

## Registro
Projeto/branch; tipo PITR ou dump; ponto UTC; RPO calculado; tamanho/hash; criptografia/custodiante; localização; último restore testado; duração; resultado; owner.

## Referências
- [Neon history window](https://neon.com/docs/introduction/branching#history-window)
- [Neon restore a branch](https://neon.com/docs/guides/branching-neon#restore-a-branch-to-a-previous-point-in-time)
- [Neon migrate from PostgreSQL](https://neon.com/docs/import/migrate-from-postgres)
- [PostgreSQL pg_dump](https://www.postgresql.org/docs/current/app-pgdump.html)
- [PostgreSQL pg_restore](https://www.postgresql.org/docs/current/app-pgrestore.html)
