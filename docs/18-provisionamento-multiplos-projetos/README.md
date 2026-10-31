# Provisionamento reproduzível para múltiplos projetos — Free

**Estado:** padrão de referência documentado. Nenhum tenant, projeto, banco, secret ou deployment foi criado ou alterado.
**Stack:** Next.js App Router + TypeScript + Auth0 + Neon PostgreSQL + GitHub Free + Vercel Hobby.
**Política:** exclusivamente Free; sem trials, créditos promocionais, upgrades ou add-ons.

## Objetivo

Estabelecer um padrão replicável para iniciar e manter vários projetos sem compartilhar credenciais, dados ou identidades de execução. Cada projeto recebe um identificador estável, manifesto não secreto, matriz de ambientes, inventário de variáveis e checklist de auditoria.

A especificação de origem exige distinguir padrão recomendado, configuração existente e procedimento de implantação. Este conjunto mantém essas três categorias separadas; nenhuma configuração real de fornecedor é declarada como verificada sem evidência de painel/API/CLI.

## Documentos

1. [Convenções de nomenclatura](./01-nomenclatura.md)
2. [Ambientes e isolamento Free](./02-ambientes-free.md)
3. [Matriz de variáveis de ambiente](./03-matriz-variaveis.md)
4. [Templates reutilizáveis](./04-templates.md)
5. [Provisionamento passo a passo](./05-provisionamento-repetivel.md)
6. [Auditoria de permissões, secrets e quotas](./06-auditoria-free.md)

## Templates do repositório

- [Manifesto não secreto](../../templates/provisionamento/project.yaml)
- [Modelo de variáveis](../../templates/provisionamento/.env.example)
- [Checklist por projeto](../../templates/provisionamento/checklist.md)
- [Instruções dos templates](../../templates/provisionamento/README.md)

## Arquitetura Free: restrições que mudam o desenho

| Serviço | Restrição relevante | Decisão documental |
|---|---|---|
| Auth0 Free | Um tenant Free por conta conforme suporte Auth0 | Um tenant compartilhado; separar aplicações/clients por projeto/ambiente quando quotas e configuração permitirem; isolar callbacks e nunca presumir tenants dev/stage/prod independentes |
| Vercel Hobby | Ambientes padrão Local/Development, Preview e Production; uso pessoal e não comercial | Staging é branch específica dentro de Preview; não documentar Custom Environment como disponível no Hobby |
| GitHub Free | Environments e environment secrets não estão disponíveis em repositórios privados no Free | CI sem secrets de produção; preferir variáveis no painel Vercel; evitar depender de deployment protection privada |
| Neon Free | Limites de projetos, branches, compute, storage e transferência | Projeto por aplicação ou domínio de isolamento; branches limitadas para dev/staging/teste; auditar quotas antes de criar branch por PR |

## Princípios obrigatórios

- Um projeto não herda automaticamente os secrets de outro.
- Compartilhar templates e padrões; nunca compartilhar connection strings, Auth0 client secrets, tokens, cookies ou dados.
- Usar nomes e tags não secretas para identificar ownership, finalidade e ambiente.
- Cada ambiente tem URL base, callback URLs, client credentials e database connection próprios sempre que possível.
- No Auth0 Free, a separação de ambientes é lógica dentro de um tenant único; isso não equivale a isolamento entre tenants.
- Na Vercel Hobby, Staging é Preview branch-scoped; revisar a elegibilidade pessoal/não comercial.
- Não criar branch Neon automaticamente para cada Pull Request sem verificar o limite de branches.
- Provisionamento é manual/documentado nesta fase; nenhum comando de criação deve ser executado sem autorização separada.
- Toda evidência deve ser classificada: Documentado, Executado, Validado ou Bloqueado.

## Fluxo recomendado

1. Registrar o projeto no manifesto.
2. Reservar o slug e aplicar nomenclatura.
3. Verificar a elegibilidade Free e quotas.
4. Definir ambientes que realmente serão usados.
5. Mapear Auth0, GitHub, Vercel e Neon sem criar recursos ainda.
6. Criar/ajustar recursos manualmente somente após aprovação do responsável.
7. Cadastrar variáveis por ambiente em canais apropriados.
8. Validar callbacks, banco, permissões e deploy.
9. Registrar evidências sem valores secretos.
10. Repetir auditoria em cada release e revisão periódica.

## Fontes oficiais consultadas

- [Auth0 Free — segundo tenant](https://support.auth0.com/center/s/article/Second-Tenant-on-a-Free-Plan)
- [Auth0 Deploy CLI — multi-environment workflow](https://github.com/auth0/auth0-deploy-cli/blob/master/docs/multi-environment-workflow.md)
- [Auth0 Deploy CLI — configuração e limitações Free](https://github.com/auth0/auth0-deploy-cli/blob/master/docs/configuring-the-deploy-cli.md)
- [Vercel — Environments](https://vercel.com/docs/deployments/environments)
- [Vercel — Environment Variables](https://vercel.com/docs/environment-variables)
- [GitHub — Managing environments](https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments)
- [Neon — Free plan limits](https://neon.com/docs/introduction/pro-plan)
- [Neon — Free plan quotas](https://neon.com/docs/introduction/plans)

> Conferir os limites e termos diretamente na data de provisionamento. A presença de uma funcionalidade no painel não comprova elegibilidade no plano Free.
