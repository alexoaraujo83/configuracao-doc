# 02 — Ambientes Development, Preview, Staging e Production no Free

## 1. Objetivo

Definir isolamento prático sem presumir recursos pagos. Os quatro ambientes são categorias lógicas do ciclo de vida; não significam quatro ambientes nativos em cada fornecedor.

## 2. Matriz por fornecedor

| Ambiente lógico | GitHub | Vercel Hobby | Auth0 Free | Neon Free |
|---|---|---|---|---|
| Development | branch de trabalho; CI sem secrets | Local/Development | aplicação/credencial de dev no tenant único, se quota permitir | branch `development` |
| Preview | Pull Request e branch | Preview automático | callback de Preview registrado de forma restrita; client dedicado quando viável | branch de teste compartilhada ou efêmera, sujeita à quota |
| Staging | branch persistente `staging` | Preview com override de variáveis para branch `staging` | configuração lógica no tenant único; client separado se possível | branch `staging` |
| Production | branch protegida por revisão/fluxo compatível com Free | Production | aplicação/client de produção no mesmo tenant | branch root de produção |

## 3. Auth0: limite estrutural do tenant Free

O Free permite um tenant por conta conforme a referência de suporte. Consequências:
- Dev, Preview, Staging e Production não terão tenants independentes dentro da mesma conta Free.
- Preferir clients separados por ambiente quando a quota do tenant permitir.
- Configurar callback, logout e origins exatos para cada URL.
- Considerar que configurações de tenant, Connections, branding e políticas são compartilhadas.
- Testes e usuários de desenvolvimento devem ser identificáveis e isolados por convenção.
- Não usar a mesma credencial de client em todos os ambientes.
- Se não for possível separar clients, documentar o risco residual e reduzir callbacks/origins autorizados.

A CLI de Deploy pode exportar/importar configurações, mas determinados recursos pagos devem ser excluídos no Free (por exemplo, log streams e custom domains conforme documentação do projeto da CLI). Exportar tenant config não exporta usuários; backup de usuários é um processo separado. Não assumir que a exportação representa backup integral.

## 4. Vercel Hobby: Staging como Preview branch-scoped

Vercel documenta três ambientes padrão: Development, Preview e Production. Custom Environments são direcionados a planos superiores. No Hobby:
1. Definir `main` como Production Branch.
2. Criar branch Git `staging`.
3. Configurar variáveis Preview com escopo para a branch `staging` quando a interface/CLI disponibilizar branch-specific variables.
4. Usar o URL Preview para homologação.
5. Não chamar isso de Custom Environment ou Production-like isolation nativa.
6. Proteger acesso e dados de staging na própria aplicação; Preview URL não é controle de autorização.

Variáveis alteradas só afetam novos deployments; redeploy é necessário para aplicar mudanças. Não usar Vercel Hobby para serviço comercial, SaaS comercial ou workload comercial: elegibilidade pessoal/não comercial deve ser confirmada.

## 5. GitHub Free: branch e secrets

- Manter CI de Pull Request sem credenciais de produção.
- Não depender de GitHub Environments/secrets para repositórios privados no Free.
- O repositório público torna o código e os metadados visíveis; nunca colocar secrets em arquivos, Actions logs ou commits.
- Preferir que a integração GitHub→Vercel faça o deploy usando as variáveis mantidas no projeto Vercel.
- Não executar workflow privilegiado com código de PR não confiável.

## 6. Neon: branch por ambiente

Estratégia padrão para um projeto com quota pequena:
- Root branch: Production.
- Branch persistente: Development.
- Branch Staging: somente se necessária.
- Preview: compartilhar branch de integração não produtiva ou criar branch efêmera sob demanda e removê-la após validação.

Branches não substituem backup independente. Não usar dados pessoais/produção em branches de teste sem avaliação de privacidade e mascaramento. Respeitar quotas por projeto e conta.

## 7. Fluxo de promoção

```text
feature branch
   |
   v
Pull Request -> CI sem secrets
   |
   v
Vercel Preview + Neon test branch
   |
   v
merge/revisão para staging
   |
   v
Vercel Preview (branch staging) + Neon staging
   |
   v
aprovação explícita de release
   |
   v
main -> Vercel Production + Neon root
```

O diagrama descreve um processo documental. Não cria ambientes nem promove código automaticamente.

## 8. Decisões por projeto

Antes de adotar os quatro ambientes, responder no manifesto:
- O projeto é pessoal/não comercial? Se não, Vercel Hobby não é elegível.
- Precisa de isolamento de tenant Auth0? O Free de tenant único não atende isolamento físico entre ambientes.
- Precisa de database isolada por ambiente? Verificar se branches Neon atendem ao risco e à quota.
- O repositório é privado? Não assumir GitHub Environment secrets no Free.
- Staging é realmente necessário? Não criar ambiente redundante que consuma quota.

## Referências
- [Auth0 — Free second tenant](https://support.auth0.com/center/s/article/Second-Tenant-on-a-Free-Plan)
- [Auth0 Deploy CLI — multi-environment](https://github.com/auth0/auth0-deploy-cli/blob/master/docs/multi-environment-workflow.md)
- [Vercel — Environments](https://vercel.com/docs/deployments/environments)
- [Vercel — Environment Variables](https://vercel.com/docs/environment-variables)
- [GitHub — Environments availability](https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments)
- [Neon — Plans and quotas](https://neon.com/docs/introduction/plans)
