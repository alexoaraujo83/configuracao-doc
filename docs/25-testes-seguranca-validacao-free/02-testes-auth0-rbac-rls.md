# 2. Testes Auth0, RBAC e RLS

## 2.1 Separar autenticação de autorização

- **Autenticação:** Auth0 confirma a identidade e Next.js SDK v4 estabelece a sessão.
- **Autorização:** aplicação decide se o principal pode executar a ação sobre o recurso e tenant.
- **Persistência:** PostgreSQL aplica grants e políticas RLS como defesa em profundidade.
- Claims do token não substituem a consulta de membership/estado no banco quando a autorização depende do domínio.

A chave interna de identidade deve considerar issuer + subject (iss + sub). Não usar e-mail como identificador imutável.

## 2.2 Ambiente Auth0 de teste

1. Inventariar quota do tenant Free e aplicações existentes antes de criar client.
2. Usar aplicação de teste separada quando houver quota disponível; callbacks/logout URLs devem apontar somente para ambiente de teste.
3. Habilitar uma Database Connection de teste com usuário sintético, se compatível com o fluxo adotado.
4. Nunca usar conta real de cliente, credenciais de produção ou conexão social de produção em automação.
5. Guardar client secret somente em ambiente local/CI confiável; nunca em PR não confiável, fixture, artifact ou log.
6. Respeitar o domínio padrão do tenant quando custom domain não estiver elegível no Free.

Não criar aplicações adicionais nem alterar tenant nesta etapa documental.

## 2.3 Testar o login real e o comportamento pós-login

- Para validar a interface real do Universal Login, usar automação de navegador (Playwright) e percorrer o fluxo de identificação como usuário.
- Não chamar endpoints internos de login do Universal Login nem pular etapas do fluxo Identifier First.
- Um cookie de sessão simulado pode isolar testes da aplicação após autenticação, mas **não prova** que o login, callback, emissão de token ou Universal Login funcionam.
- Se usar helper oficial para gerar cookie de sessão, manter segredo e cookie exclusivamente no ambiente de teste, com expiração curta, e nunca habilitar bypass em produção.
- Não habilitar Resource Owner Password Grant apenas para facilitar testes; a orientação Auth0 ressalta limitações, inclusive incompatibilidade com MFA e provedores sociais. Preferir navegador para testar o Universal Login real.

## 2.4 Matriz de autenticação

| Cenário | Resultado esperado |
|---|---|
| Sem cookie/sessão | redirect para login ou 401 conforme endpoint |
| Cookie inválido/adulterado | sessão rejeitada |
| Sessão expirada | reautenticação |
| Callback com state/nonce inválido | fluxo recusado |
| Issuer/audience incorretos | token rejeitado |
| Logout | sessão local encerrada; retorno permitido |
| Usuário autenticado sem membership | acesso negado |
| Usuário desativado/revogado | acesso negado após regra de atualização definida |
| Erro Auth0/rede | falha segura, sem revelar segredo |

## 2.5 Matriz RBAC

Construir tabela de ações por papel, alinhada ao domínio real. Exemplo ilustrativo:

| Ação | anonymous | member | manager | admin |
|---|---:|---:|---:|---:|
| Ler recurso próprio | negar | permitir | permitir | permitir |
| Criar recurso no tenant | negar | permitir | permitir | permitir |
| Ler recursos de outro tenant | negar | negar | negar | conforme escopo explícito |
| Gerenciar membership | negar | negar | conforme regra | permitir |
| Alterar configuração global | negar | negar | negar | permitir |

Para cada ação, testar: permitido, negado, recurso inexistente, tenant divergente, membership revogado e papel desconhecido. Verificar também que UI oculta ações não autorizadas, mas a proteção efetiva ocorre no servidor em cada DAL, Server Action e Route Handler.

## 2.6 RLS: pré-condições

- Runtime usa role dedicada, sem ownership, superuser ou BYPASSRLS.
- Habilitar RLS e FORCE ROW LEVEL SECURITY nas tabelas que exigem isolamento, conforme modelo.
- Definir políticas separadas para SELECT, INSERT, UPDATE e DELETE, quando necessário.
- Conferir USING e WITH CHECK; UPDATE deve validar a linha existente e o novo valor.
- Definir default deny quando não houver contexto/policy aplicável.
- Contexto tenant/user é definido no servidor dentro da mesma transação da query, com parâmetro e set_config(..., true) / SET LOCAL; não persistir contexto entre transações.
- A aplicação nunca aceita tenantId/userId de input do browser como fonte de autoridade.

## 2.7 Matriz RLS obrigatória

Usar pelo menos dois tenants sintéticos (A e B) e dois usuários distintos:

| Teste com app_runtime | Resultado esperado |
|---|---|
| SELECT com contexto A | somente linhas A |
| SELECT com contexto B | somente linhas B |
| Sem contexto tenant | zero linhas ou erro seguro |
| INSERT com tenant A | permite somente tenant A |
| INSERT declarando tenant B sob contexto A | rejeita |
| UPDATE de linha A para tenant B | rejeita via WITH CHECK |
| DELETE de linha B sob contexto A | nenhuma linha afetada |
| Contexto trocado após commit | não herda tenant anterior |
| Role owner/migrator | usada somente em teste administrativo isolado, nunca para provar RLS de runtime |
| Role runtime | não consegue DDL, TRUNCATE, alterar policy ou assumir role privilegiada |

Não confiar em testes executados como owner: owners normalmente podem ignorar RLS, a menos que FORCE ROW LEVEL SECURITY esteja aplicado; superusers e BYPASSRLS sempre ignoram.

## 2.8 Exemplo de teste SQL documental

Adaptar nomes e políticas reais; executar somente em banco de teste:

~~~sql
BEGIN;
SET LOCAL ROLE app_runtime;
SELECT set_config('app.tenant_id', 'tenant-a-test', true);
SELECT set_config('app.user_sub', 'auth0|user-a-test', true);

-- A consulta deve retornar exclusivamente registros do tenant A.
SELECT id, tenant_id
FROM app.documents
ORDER BY id;

ROLLBACK;
~~~

O contexto GUC configurado pela aplicação é uma defesa em profundidade, não uma fronteira independente contra backend comprometido ou SQL arbitrário executado com a mesma role. A validação RBAC/membership deve ocorrer no servidor.

## 2.9 Critérios de aprovação

- Nenhuma rota, action ou DAL sensível sem autorização server-side.
- Nenhum teste cruzado A/B revela, modifica ou remove dados do outro tenant.
- Runtime não possui ownership, BYPASSRLS ou DDL.
- Sem contexto ou identidade ambígua, acesso é negado.
- Evidência registra role SQL efetiva, tenant sintético, migration/schema revision e resultado.
