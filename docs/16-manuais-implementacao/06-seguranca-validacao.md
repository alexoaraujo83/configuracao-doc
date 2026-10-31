# 06 — Segurança, validação e operação

**Estado:** critérios documentados; testes de aplicação pendentes.  
**Escopo:** controles técnicos que podem ser adotados sem depender de planos pagos.

## 1. Modelo de ameaça mínimo

Proteger contra:
- Roubo ou exposição de Client Secret, AUTH0_SECRET e DATABASE_URL.
- Acesso horizontal entre usuários (IDOR).
- SQL injection.
- Falsificação de identidade por parâmetros enviados pelo navegador.
- Sessões inválidas, callback aberto e configuração incorreta de cookies.
- Exposição de PII e tokens em logs.
- Dependências vulneráveis e workflows de CI com permissões excessivas.
- Exaustão de cotas Free e indisponibilidade por suspensão de compute.

## 2. Regras obrigatórias

1. Autenticar no Auth0; não armazenar senhas de usuários na aplicação.
2. Validar sessão em cada operação privada no servidor.
3. Autorizar por recurso, ação e usuário; não confiar apenas no Proxy.
4. Obter `sub` da sessão validada, nunca do body/query como identidade.
5. Usar SQL parametrizado e allowlists para identificadores dinâmicos.
6. Aplicar RLS nas tabelas multiusuário e testar isolamento entre identidades.
7. Separar owner, migrator e runtime no PostgreSQL.
8. Não usar role administrativa como credencial de runtime.
9. Guardar secrets somente nos mecanismos de ambiente apropriados.
10. Não prefixar segredos com `NEXT_PUBLIC_`.
11. Cookies de sessão devem manter defaults seguros do SDK; produção somente HTTPS.
12. Respostas e logs não devem conter tokens, connection strings ou dados pessoais desnecessários.
13. Dependências e GitHub Actions devem ser revisadas.
14. Migrations devem ser versionadas e revisadas antes de produção.
15. Não depender de recursos pagos ou de trial.

## 3. Validação local e CI

Executar antes de cada PR:

```bash
npm ci
npm run typecheck
npm run lint
npm audit
npm run build
```

Investigar alertas do `npm audit`; não executar `npm audit fix --force` sem revisão de breaking changes.

## 4. Testes funcionais e de autorização

| ID | Cenário | Esperado |
|---|---|---|
| SEC-01 | Usuário anônimo acessa dashboard | Redirecionamento/login |
| SEC-02 | Usuário anônimo chama API privada | 401 |
| SEC-03 | Usuário A consulta recurso de B | 403 ou 404 conforme política |
| SEC-04 | Cliente envia `auth0_sub` de B | Ignorado; identidade vem da sessão |
| SEC-05 | SQL injection em campo de busca | Tratado como dado, sem alterar SQL |
| SEC-06 | RLS sem contexto | SELECT/INSERT/UPDATE negados |
| SEC-07 | RLS com contexto A consulta B | Nenhuma linha de B |
| SEC-08 | RLS INSERT com subject A | Permitido somente para subject A |
| SEC-09 | RLS INSERT forjado com subject B | Negado |
| SEC-10 | RLS UPDATE cruzado ou troca de subject | Negado |
| SEC-08 | Sessão ausente ou expirada | Acesso negado |
| SEC-09 | Secret ausente | Inicialização falha sem revelar valor |
| SEC-10 | Banco indisponível | Erro controlado, sem detalhes internos |
| SEC-11 | Inspecionar bundle cliente | Nenhum segredo |
| SEC-12 | Logout | Sessão encerrada |
| SEC-13 | Callback não permitido | Provedor rejeita redirect |
| SEC-14 | CI de PR | Build/typecheck/lint executados |

## 5. Teste de isolamento multiusuário

Criar dois usuários de teste com subjects diferentes. Inserir dados vinculados a cada identidade. Com a sessão A:
- Consultar registro de A: permitido.
- Consultar registro de B: negado/zero linhas.
- Inserir registro com subject A: permitido.
- Inserir registro com subject B: negado pela policy INSERT.
- Atualizar registro de B ou alterar o subject de A para B: negado.
- Tentar forjar subject B no request: não altera identidade usada pelo servidor.

Repetir com B. Executar testes com a role `app_runtime`, não com owner ou role administrativa. Guardar somente resultado, identificadores fictícios e data, nunca tokens ou dados reais.

## 6. Gestão de segredos e rotação

- Inventariar nomes de secrets e sistemas que os consomem, sem copiar valores.
- Rotacionar imediatamente qualquer segredo exposto.
- Atualizar o segredo no provedor e no ambiente de execução.
- Reimplantar e validar login/conexão.
- Revogar credencial anterior após confirmar a nova.
- Registrar data, responsável e resultado sem registrar o segredo.

## 7. Incidentes

**Suspeita de credencial comprometida**
1. Revogar/rotacionar credencial no provedor.
2. Atualizar variáveis nos ambientes.
3. Invalidar sessões/tokens conforme capacidade do Auth0.
4. Revisar logs e histórico GitHub.
5. Reimplantar e validar.
6. Documentar escopo e ações sem incluir o segredo.

**Vazamento entre usuários**
1. Desabilitar endpoint/operação afetada.
2. Preservar logs necessários com controle de acesso.
3. Corrigir autorização e/ou policy RLS.
4. Testar A/B e casos negativos.
5. Reabrir somente após revisão.
6. Avaliar obrigações legais e notificações aplicáveis.

## 8. Recuperação de banco

- Manter migrations versionadas.
- Executar backup lógico periódico.
- Testar restore em ambiente isolado.
- Documentar dependências, extensão e versão PostgreSQL.
- Definir RPO/RTO realistas considerando cotas Free.
- Não prometer recuperação instantânea ou PITR além do que o plano efetivamente oferece.

## 9. Evidências de conclusão

Para cada etapa, registrar:
- Data e ambiente.
- Versões (Node, Next.js, SDK Auth0, driver Neon).
- Comando ou procedimento executado.
- Resultado (passou/falhou/bloqueado).
- Link para PR/commit ou evidência não sensível.
- Limitação Free observada.
- Pendências e responsável.

Não marcar como “validado” com base apenas em leitura de documentação ou revisão estática.

