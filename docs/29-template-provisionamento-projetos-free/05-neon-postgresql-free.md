# Neon PostgreSQL — configuração de referência no plano Free

## Elegibilidade e capacidade

Antes de desenhar ambientes, verificar no Neon Console quotas atuais de armazenamento, compute, branches, egress e retenção. Documentação pública pode divergir entre FAQ e página de planos; registrar divergência e adotar limite conservador até confirmação. Não tratar PITR como backup externo independente.

## Configuração padrão inicial

1. Criar/selecionar projeto Neon autorizado.
2. Registrar região e requisitos de latência/residência.
3. Criar database com nome padronizado.
4. Criar roles separadas para migration e runtime.
5. Obter connection string com SSL e pooling para workload serverless.
6. Guardar connection string como secret de servidor.
7. Versionar migrations SQL no repositório.
8. Aplicar migrations em ambiente de desenvolvimento antes de promoção.
9. Confirmar permissões e conectividade com dados sintéticos.

## Topologia de ambientes

Preferir branches isoladas para Development e Preview quando quota permitir. Production deve ter credenciais e permissões próprias. Não assumir que uma branch é backup nem que branches eliminam risco de consumo compartilhado. Se quota não suportar ambientes isolados, documentar risco e não misturar dados reais em testes.

## Roles de referência

Separar ownership, migration e runtime. Exemplo conceitual, a adaptar e revisar para a versão PostgreSQL e o modelo de privilégios do projeto:

~~~sql
CREATE ROLE app_owner NOLOGIN;
CREATE ROLE app_migrator LOGIN NOINHERIT;
GRANT app_owner TO app_migrator;

CREATE ROLE app_runtime LOGIN NOINHERIT NOBYPASSRLS;
~~~

O processo de migration autentica como app_migrator e assume o papel de owner somente durante operação controlada. O runtime não deve ser owner, não deve possuir privilégios de DDL/TRUNCATE e não deve ter BYPASSRLS. Conceder somente privilégios DML necessários em schemas, tabelas e sequences. Não usar superuser na aplicação.

Este SQL é ponto de partida, não script pronto para execução: ajustar LOGIN, memberships, schema, grants, default privileges e credenciais conforme ambiente. Não inserir passwords no arquivo SQL versionado.

## RLS e isolamento multi-tenant

- Habilitar e, quando aplicável, forçar RLS nas tabelas de domínio.
- Criar policies com USING e WITH CHECK.
- Definir contexto de usuário/tenant dentro da mesma transação da consulta.
- Usar valores obtidos da sessão autenticada e validados no servidor.
- Não aceitar tenant ID diretamente do browser sem autorização.
- Testar isolamento com identidades e tenants sintéticos.
- GUCs definidos pelo backend são defesa em profundidade, não fronteira independente contra backend comprometido.

## Driver Neon

~~~typescript
// src/lib/db.ts
import { neon } from "@neondatabase/serverless";

const databaseUrl = process.env.DATABASE_URL;

if (!databaseUrl) {
  throw new Error("DATABASE_URL não configurada");
}

export const sql = neon(databaseUrl);
~~~

Manter o módulo exclusivamente server-side. Para transações interativas, migrations ou operações que dependam de sessão persistente, avaliar driver/API apropriado; não presumir que consulta HTTP stateless suporta todos os padrões transacionais.

## Migrations

- SQL versionado e revisado em Pull Request.
- Um arquivo por alteração incremental, com numeração ordenada.
- Registrar versão aplicada em tabela de controle.
- Testar primeiro em branch/database não produtivo.
- Planejar lock, concorrência, recuperação e rollback.
- Evitar migrations destrutivas sem plano de compatibilidade.
- Nunca executar migration automaticamente a partir de request web.
- Credencial de migration separada da runtime.

## Checklist

- [ ] Quotas Free e região verificadas.
- [ ] Branches e ambientes mapeados.
- [ ] Roles separadas e privilégio mínimo.
- [ ] RLS avaliada para dados sensíveis/multi-tenant.
- [ ] Connection string privada, TLS e pooling avaliados.
- [ ] Migrations versionadas e revisadas.
- [ ] Dados de teste sintéticos.
- [ ] Recuperação e limitações PITR/backup documentadas.

Referências: [Neon Docs](https://neon.com/docs), [Neon Plans](https://neon.com/pricing).
