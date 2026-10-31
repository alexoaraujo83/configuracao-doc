# Perfil mestre de projeto — preencher uma cópia por projeto

> Não inserir valores de secrets, tokens, senhas, connection strings ou dados pessoais.

## 1. Identificação
- ID do projeto:
- Nome funcional:
- Finalidade:
- Proprietário técnico:
- Aprovador:
- Data de criação/revisão:
- Classificação dos dados:

## 2. Elegibilidade e plano
- GitHub: Free (confirmar tipo de repositório e recursos disponíveis):
- Auth0: Free (confirmar limites e recursos):
- Neon: Free (registrar limites observados):
- Vercel: Hobby somente se uso pessoal e não comercial:
- Restrições ou exceções:

## 3. Código e runtime
- Repositório GitHub:
- Branch padrão:
- Node.js:
- Next.js:
- TypeScript:
- Package manager:
- Lockfile:
- Estratégia de atualização:

## 4. Ambientes
| Ambiente | Branch Git | Vercel target | Auth0 Application | Neon branch | URL |
|---|---|---|---|---|---|
| Development | | | | | |
| Preview | | | | | |
| Staging | | | | | |
| Production | | | | | |

## 5. Auth0
- Tenant (identificador não secreto):
- Application por ambiente:
- Allowed Callback URLs:
- Allowed Logout URLs:
- Allowed Web Origins:
- SDK e versão:
- Estratégia de sessão:
- Estratégia de autorização:
- Limitações Free confirmadas:

## 6. Neon PostgreSQL
- Projeto:
- Database:
- Branch de produção:
- Branches não produtivas:
- Role de migration:
- Role de runtime:
- RLS/policies:
- Pooling:
- Retenção/recuperação observada:
- Limites Free confirmados:

## 7. Variáveis (nomes somente)
| Nome | Ambiente | Consumidor | Pública/segredo | Finalidade | Responsável | Rotação/revogação |
|---|---|---|---|---|---|---|
| APP_BASE_URL | | | | | | |
| AUTH0_DOMAIN | | | | | | |
| AUTH0_CLIENT_ID | | | | | | |
| AUTH0_CLIENT_SECRET | | | segredo | | | |
| AUTH0_SECRET | | | segredo | | | |
| DATABASE_URL | | | segredo | | | |

## 8. CI, qualidade e release
- Workflow:
- Checks obrigatórios:
- Estratégia de testes:
- Dados sintéticos:
- Critérios de aprovação:
- Responsável pelo release:
- Procedimento de rollback:

## 9. Riscos e evidências
| Item | Estado (verificado/recomendado/pendente) | Evidência/data | Ação |
|---|---|---|---|
| Elegibilidade Hobby | | | |
| Quotas Auth0 | | | |
| Quotas Neon | | | |
| GitHub Actions | | | |
| Isolamento entre ambientes | | | |
| Secrets fora do Git | | | |
| RLS e roles | | | |

## 10. Aprovação
- Revisão técnica:
- Revisão de segurança:
- Exceções aprovadas:
- Data da próxima revisão:
