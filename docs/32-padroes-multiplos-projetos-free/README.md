# Padrões para múltiplos projetos e ambientes — somente Free

**Status:** Recomendada; validação por projeto pendente.  
**Escopo:** Next.js + TypeScript, Auth0, Neon PostgreSQL, GitHub e Vercel.  
**Restrição:** não pressupõe recursos pagos, trials, créditos promocionais ou add-ons.

## 1. Objetivos e princípios

1. Repetibilidade: cada projeto parte do mesmo contrato documental.
2. Isolamento: separar identidades, dados, credenciais e deployments por projeto e ambiente sempre que os limites Free permitirem.
3. Menor privilégio: cada credencial tem finalidade, escopo e responsável definidos.
4. Evidência: distinguir recomendação de configuração observada.
5. Portabilidade: manter configurações não sensíveis versionadas e secrets fora do Git.
6. Limites explícitos: quotas, retenção, recursos e termos são revalidados antes de cada adoção.
7. Sem falsa segregação: nomes de ambientes, branches ou variáveis não constituem isolamento de segurança por si só.

## 2. Modelo de organização

~~~text
organização
├── projeto-a
│   ├── development
│   ├── preview
│   ├── staging (opcional)
│   └── production
├── projeto-b
│   ├── development
│   ├── preview
│   └── production
└── documentação-compartilhada
~~~

Cada projeto deve ter repositório GitHub próprio quando possível, projeto Vercel próprio e projeto Neon próprio. O Auth0 Free deve ser planejado para um tenant por conta e até 10 Applications por tenant conforme a matriz Free deste repositório; essa quota é compartilhada entre todos os projetos. Aplicações e callbacks precisam ser segregados por projeto/ambiente dentro do tenant compartilhado, sem presumir isolamento de tenant independente. Revalidar o limite no pricing/painel antes de cada adoção.

Não compartilhar banco de produção entre aplicações distintas por conveniência. Quando a quota impedir projetos Neon separados, registrar formalmente a exceção, avaliar o risco e não declarar isolamento equivalente.

## 3. Convenção de identificadores

Use identificadores minúsculos, ASCII, sem espaços, sem dados pessoais e sem secrets.

| Elemento | Convenção | Exemplo |
|---|---|---|
| Projeto | `<produto>-<dominio>` | portal-clientes |
| Repositório | `<projeto>` | portal-clientes |
| Vercel project | `<projeto>-web` | portal-clientes-web |
| Neon project | `<projeto>-db` | portal-clientes-db |
| Auth0 Application | `<projeto>-<ambiente>-web` | portal-clientes-preview-web |
| Branch Git | `feat/<assunto>`, `fix/<assunto>`, `docs/<assunto>` | feat/perfil |
| Neon branch | `<ambiente>-<projeto>` | preview-portal-clientes |
| Variável | UPPER_SNAKE_CASE | DATABASE_URL |
| Schema PostgreSQL | nome funcional | billing |

Não colocar nome de cliente, e-mail, identificador de usuário ou informação confidencial em nomes públicos de branch, URL ou projeto.

## 4. Matriz de ambientes

| Ambiente lógico | GitHub | Vercel | Auth0 | Neon |
|---|---|---|---|---|
| Development | branch de trabalho | execução local | aplicação de desenvolvimento | branch/database não produtivo |
| Preview | Pull Request | Preview Deployment | aplicação/callback de Preview | branch isolada se quota permitir |
| Staging | branch de release | Preview com configuração específica, se suportado | aplicação/callback de Staging | branch de Staging, se disponível |
| Production | release aprovada | Production apenas quando elegível nos termos do plano | aplicação/callback de Production | branch de produção |

**Importante:** Vercel Hobby é restrito a uso pessoal e não comercial. Se o projeto tiver uso comercial, a arquitetura Hobby não é elegível; interromper a adoção dessa hospedagem no escopo Free, sem recomendar upgrade. Preview e Production são ambientes lógicos, não garantias de isolamento físico.

O GitHub Free não deve ser presumido como oferecendo controles de proteção ou secrets de Environment equivalentes aos planos pagos em repositórios privados. Projetar CI sem depender desses recursos; validar disponibilidade efetiva na conta e visibilidade do repositório.

## 5. Modelo mestre por projeto

Copiar e preencher templates/projeto/PROJECT_PROFILE.md para cada novo projeto. Cada campo deve ser preenchido sem incluir valores secretos.

Campos mínimos: identificador e finalidade; proprietário técnico e aprovadores; classificação dos dados; repositório, branch padrão e política de revisão; versões de Node.js, Next.js, TypeScript e package manager; projeto Vercel e elegibilidade Hobby; tenant e Application Auth0, callback/logout/origins; projeto Neon, branches, database e roles; inventário de variáveis por ambiente (somente nomes e finalidade); fluxos de CI, validações e critérios de release; quotas observadas, data de verificação e links oficiais; riscos, exceções, plano de retorno e estado de conformidade.

## 6. Configuração compartilhada versus específica

Compartilhar somente padrões não sensíveis: convenções, configurações de lint, TypeScript, workflow-base, templates e documentação. Fixar versões e revisar atualizações em PR.

Manter específico por projeto: Auth0 client ID, client secret, callback e logout URLs; DATABASE_URL e roles; Vercel project e variáveis de ambiente; domínios, origens permitidas e identificadores de integração; políticas de dados, tenant, permissões e retenção.

Segredos nunca devem ser colocados em repositório compartilhado, arquivo de exemplo, issue, log, comentário de PR ou documentação. Um valor fictício em .env.example deve ser claramente não funcional.

## 7. Inventário de variáveis

O inventário deve conter: nome, ambiente, serviço consumidor, classificação (pública/segredo), finalidade, origem, responsável, data de rotação e procedimento de revogação. Não registrar valor, hash ou trecho de segredo.

Variáveis típicas:
- Next/Auth0: APP_BASE_URL, AUTH0_DOMAIN, AUTH0_CLIENT_ID, AUTH0_CLIENT_SECRET, AUTH0_SECRET.
- Neon: DATABASE_URL (server-only).
- Aplicação: APP_ENV, LOG_LEVEL (sem dados sensíveis).

Variáveis NEXT_PUBLIC_* são expostas ao navegador no build; nunca usar esse prefixo para segredos ou connection strings. AUTH0_CLIENT_ID pode ser identificador público conforme configuração SDK, mas seguir a documentação atual do SDK.

## 8. Isolamento de credenciais e dados

- Preferir uma credencial por aplicação e ambiente.
- Não reutilizar credenciais de produção em Preview ou Development.
- Usar role de runtime sem privilégios de DDL, ownership ou BYPASSRLS.
- Aplicar RLS e validar tenant/user no servidor; não confiar em identificador enviado pelo cliente.
- Guardar connection strings apenas em configuração server-side.
- Restringir callbacks, logout URLs e web origins do Auth0 aos hosts necessários.
- Revogar credencial exposta e registrar incidente; não apenas editar o arquivo.
- Quando o plano Free impedir segregação adequada, documentar a limitação e não classificar a solução como isolamento forte.

## 9. Versionamento e atualização

- Fixar Node e package manager por projeto (por exemplo, .nvmrc e lockfile).
- Usar npm ci em CI quando o projeto usa npm e mantém lockfile.
- Fixar actions por versão revisada; para ambientes de maior garantia, avaliar SHA completo após revisão.
- Atualizar dependências por PR pequeno, com changelog, impacto e rollback.
- Não automatizar atualização direta para Production.
- Revisar mensalmente dependências críticas e, no mínimo, trimestralmente inventário de limites e compatibilidade Free.
- Revalidar SDK Auth0, APIs Next.js e runtime Vercel antes de atualizar versões principais.

## 10. Governança de mudanças

Toda mudança deve informar: projeto, ambiente, motivo, arquivos/serviços afetados, impacto em dados, secrets envolvidos (somente nomes), validação, janela, aprovador e rollback. Mudanças documentais não autorizam execução operacional.

Mudanças em autenticação, autorização, RLS, roles, schema ou variáveis de Production exigem revisão técnica específica. Nunca copiar secrets de um ambiente para outro durante troubleshooting.

## 11. Critérios de aceitação de novo projeto

- [ ] Perfil do projeto preenchido e proprietário definido.
- [ ] Uso do Vercel Hobby confirmado como pessoal e não comercial, se aplicável.
- [ ] Limites Free de Auth0, Neon, GitHub e Vercel conferidos na data.
- [ ] Tenant/Auth0 Application e URLs documentados sem expor secrets.
- [ ] Banco, roles e estratégia RLS documentados.
- [ ] Variáveis inventariadas por ambiente e nenhum segredo versionado.
- [ ] CI sem dependência de recursos pagos.
- [ ] Política de branches, PR e release documentada.
- [ ] Dados sintéticos usados em testes.
- [ ] Plano de rollback e recuperação compatível com o Free.
- [ ] Exceções e riscos aceitos formalmente.
- [ ] Documentação revisada por outra pessoa.

## 12. Limitações conhecidas

Este padrão não garante quotas, alta disponibilidade, backups externos, retenção prolongada, ambientes isolados ou recursos empresariais. Limites Free mudam e devem ser consultados nos painéis e documentos oficiais. Não assumir que um recurso existe porque aparece em documentação geral do fornecedor: confirmar disponibilidade no plano Free efetivamente utilizado.

## 13. Referências oficiais

- GitHub Docs: https://docs.github.com/pt
- GitHub Actions billing: https://docs.github.com/en/billing/managing-billing-for-your-products/managing-billing-for-github-actions/about-billing-for-github-actions
- Auth0 Docs: https://auth0.com/docs
- Auth0 Pricing: https://auth0.com/pricing
- Vercel Hobby: https://vercel.com/docs/plans/hobby
- Vercel Environment Variables: https://vercel.com/docs/environment-variables
- Neon plans: https://neon.com/docs/introduction/plans
- Neon branching: https://neon.com/docs/introduction/branching

**Aviso:** este guia é documental. Não cria projetos, tenants, aplicações, branches, secrets, bancos ou deployments.
