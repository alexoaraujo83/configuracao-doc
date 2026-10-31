# 05 — Procedimento de provisionamento repetível

**Importante:** este documento descreve como provisionar após autorização. Nenhum passo abaixo foi executado nesta etapa.

## Fase 0 — Registro e decisão

1. Criar cópia do [manifesto](../../templates/provisionamento/project.yaml).
2. Reservar project slug e nomes.
3. Definir proprietário, contatos operacionais, criticidade e classificação dos dados.
4. Confirmar se o projeto é pessoal/não comercial para elegibilidade Vercel Hobby.
5. Confirmar limites e uso atual dos planos Free.
6. Definir Development, Preview, Staging e Production que serão realmente utilizados.
7. Aprovar arquitetura e risco residual do Auth0 tenant único.

**Saída:** manifesto revisado e status Planned.

## Fase 1 — GitHub

1. Criar ou selecionar o repositório.
2. Definir visibilidade e branch padrão.
3. Adicionar README, .gitignore, LICENSE quando aplicável e templates não secretos.
4. Definir convenção de branches e revisão de PR.
5. Configurar workflow CI sem secrets de produção.
6. Revisar permissions e Actions de terceiros.
7. Confirmar que nenhum .env real, token ou connection string está versionado.

**Validação:** clone limpo, CI executável sem credenciais e revisão de histórico.

## Fase 2 — Auth0 Free

1. Confirmar o tenant Free existente; não criar segundo tenant.
2. Registrar domínio do tenant no manifesto.
3. Definir se haverá clients separados por ambiente e projeto, conforme quotas.
4. Criar/selecionar Regular Web Application conforme a arquitetura Next.js.
5. Cadastrar callback, logout e web origins exatos de cada URL.
6. Confirmar Database Connection e políticas de login aprovadas.
7. Registrar scopes/permissions mínimos.
8. Guardar client secret e segredo de sessão no destino correto, nunca no Git.
9. Testar login, callback, logout e sessão em cada URL autorizada.

**Validação:** fluxo OIDC funciona e callbacks não autorizados são rejeitados. Documentar que o tenant continua compartilhado.

## Fase 3 — Neon Free

1. Conferir limite atual de projetos, branches, compute, storage e transfer.
2. Criar/selecionar o projeto do app.
3. Criar branches persistentes somente para ambientes necessários.
4. Criar roles owner/migrator/runtime conforme manual de banco.
5. Obter connection string da role runtime para a aplicação e conexão de migration separada.
6. Aplicar migrations somente após aprovação e em ambiente isolado.
7. Validar TLS, permissões, RLS e isolamento entre usuários.
8. Registrar nomes e IDs não secretos no manifesto.

**Validação:** conexão runtime mínima, migrations auditadas e testes negativos de RLS aprovados.

## Fase 4 — Vercel Hobby

1. Confirmar elegibilidade de uso pessoal/não comercial.
2. Criar/selecionar o projeto e importar o repositório.
3. Definir Root Directory, framework, build e Production Branch.
4. Cadastrar variáveis de Development, Preview e Production separadamente.
5. Para Staging, usar branch específica e overrides Preview se disponíveis.
6. Configurar domínio e URLs callback no Auth0.
7. Executar deployment Preview e validar.
8. Promover Production somente após aprovação e revisão do impacto.
9. Alterações de environment variables exigem novo deployment para surtir efeito.

**Validação:** build, callbacks, endpoint de saúde e banco testados sem expor segredos.

## Fase 5 — Integração cruzada

- [ ] GitHub PR dispara CI sem credenciais de produção.
- [ ] Vercel Preview recebe variáveis de Preview.
- [ ] Auth0 aceita apenas callback/logout/origin esperados.
- [ ] Neon Preview conecta à branch não produtiva.
- [ ] Production usa credenciais e banco próprios.
- [ ] Logs não contêm tokens, cookies ou connection strings.
- [ ] A aplicação verifica autorização no servidor.
- [ ] Vercel Hobby não está sendo usado para workload comercial.

## Fase 6 — Registro de evidências

Para cada recurso registrar:
- estado (planned/existing/verified/blocked);
- data e responsável;
- ambiente;
- recurso/ID não secreto;
- procedimento realizado;
- teste e resultado;
- quota consumida;
- limitações e pendências.

Não incluir prints que revelem tokens, segredos, dados pessoais ou connection strings.

## Fase 7 — Mudança e rollback

Toda mudança em produção exige:
1. PR revisado.
2. Backup/estratégia de recuperação conforme risco.
3. Plano de rollback de aplicação e banco.
4. Janela e responsável.
5. Validação pós-deploy.
6. Registro do resultado.

Documentar não autoriza executar; a aprovação para provisionamento ou produção é independente.
