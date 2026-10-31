# 5. Migrations e integridade PostgreSQL/Neon

## 5.1 Objetivo

Demonstrar que o histórico de migrations produz o schema esperado, preserva invariantes e aplica privilégios/policies corretos. Testar somente em banco isolado e com dados sintéticos.

## 5.2 Estratégia de validação

Para cada release de schema:
1. Criar banco/branch descartável com versão PostgreSQL compatível.
2. Aplicar todas as migrations desde zero.
3. Comparar schema resultante com o schema esperado/versionado.
4. Aplicar a migration sobre snapshot/branch representativa da versão anterior.
5. Executar testes de constraints, índices, triggers, funções, grants e RLS.
6. Inserir fixtures sintéticas e testar integridade referencial.
7. Simular falha em transação e verificar rollback/atomicidade.
8. Medir duração e locks de migrations que alterem tabelas grandes.
9. Repetir a partir de banco limpo para provar determinismo.
10. Registrar checksum, ordem, resultado e versão do PostgreSQL/driver.

## 5.3 Regras de migration

- Migration aplicada é imutável; correção deve ser nova migration.
- Um único mecanismo de migration deve ser fonte de verdade; não misturar ferramentas sem governança.
- Aplicar migration por role migrator dedicada; runtime não executa DDL.
- Usar transação quando a operação suportar; documentar operações PostgreSQL que não podem ocorrer em transação.
- Usar estratégia expand-contract para mudanças incompatíveis: adicionar, compatibilizar código, migrar dados, validar, remover em release posterior.
- Evitar DROP/rename destrutivo no mesmo release que altera código consumidor.
- Não executar migration de produção automaticamente em Preview/PR.
- Não rodar migration contra produção para “ver se passa”.
- Registrar lock/checksum para impedir reordenação e edição silenciosa do histórico.

## 5.4 Matriz de integridade

| Área | Teste |
|---|---|
| PK/FK | duplicidade rejeitada; referência órfã rejeitada |
| UNIQUE | valores duplicados rejeitados, incluindo escopo composto |
| NOT NULL/CHECK | dados inválidos rejeitados |
| Defaults | valores gerados conforme contrato |
| Índices | presença e definição esperadas; plano de consulta quando relevante |
| Enums/types | valores válidos e compatibilidade de aplicação |
| Triggers/functions | efeitos esperados, segurança e permissões |
| Grants/roles | runtime só executa operações necessárias |
| RLS | isolamento por tenant para SELECT/INSERT/UPDATE/DELETE |
| Migration ledger | sequência e checksum consistentes |
| Transações | rollback não deixa estado parcial |
| Concorrência | constraints e idempotência sob requisições simultâneas |

## 5.5 Roles e RLS

Usar a separação definida no projeto:
- app_owner: NOLOGIN, proprietário do schema/objetos;
- app_migrator: login dedicado, assume app_owner apenas durante migration controlada;
- app_runtime: LOGIN, NOINHERIT, NOBYPASSRLS, sem ownership e sem DDL/TRUNCATE.

Validar grants efetivos e memberships; não basta inspecionar o SQL declarativo. Executar testes de RLS como app_runtime. Testar FORCE ROW LEVEL SECURITY e confirmar que a aplicação não abre conexão com role migrator.

## 5.6 Branches Neon e quota Free

- Usar branch de teste apenas quando quota de branches permitir; contabilizar branches persistentes e efêmeras.
- Definir nome, origem, finalidade, owner e política de exclusão.
- Preferir dados sintéticos ou anonimizados; não copiar produção com dados pessoais para branch de CI.
- Excluir branch descartável somente após confirmar que não é necessária como evidência.
- Se não houver quota, usar PostgreSQL local isolado em CI; documentar que isso não valida recursos específicos da Neon.
- Confirmar quotas Free atuais antes de desenhar branch-per-PR; não presumir automação ou retenção ilimitada.

## 5.7 Verificação de backup e restore

Para evidência de integridade, manter procedimento separado de backup/restore:
- pg_dump gera export consistente de uma base; não substitui estratégia completa de backup de produção.
- verificar exit code, tamanho, checksum e legibilidade do artefato;
- restaurar em base isolada, nunca sobre produção durante teste;
- comparar contagens/constraints e executar smoke tests;
- garantir que role de backup consiga ler todos os dados necessários sem depender inadvertidamente de RLS filtrando linhas;
- proteger e restringir acesso aos dumps; não guardar dados pessoais em artifacts comuns de CI.

PITR/instant restore do plano Free tem janela e quota limitadas; não tratá-lo como único backup ou garantia de RPO.

## 5.8 Gate de migration

Bloquear se: migration não reproduzível, checksum divergente, perda de dados não aprovada, role runtime com DDL/BYPASSRLS, policy sem teste, rollback/forward recovery indefinido, ou execução de produção embutida no CI.
