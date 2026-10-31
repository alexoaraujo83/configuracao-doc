# Padrões de isolamento e reutilização

## Conceitos
- **Separação lógica:** clients, schemas, roles, tenant_id e RLS no mesmo recurso.
- **Separação por recurso:** projetos, aplicações, repositórios ou deployments independentes.
- **Separação por tenant/conta:** fronteira administrativa e de identidade independente.
- **Isolamento operacional:** implantar, reverter, auditar e recuperar um componente sem afetar outro.
- **Isolamento de dados:** impedir acesso, inferência ou restauração cruzada.

Um client Auth0 não é tenant separado. Uma branch Neon não é projeto independente. Um ambiente lógico não é automaticamente uma fronteira de segurança.

## GitHub Free
Reutilizar templates e workflows. Em repo privado Free, não depender de GitHub Environments, environment secrets ou deployment branch rules. Não passar secrets de produção a workflows de PR/fork. A quota publicada de Actions em privados é 2.000 minutos/mês; revalidar billing. Ações em repositórios públicos têm execução padrão gratuita.

## Auth0 Free
Um tenant Free por conta. Clients distintos por aplicação e callbacks/logout/web origins com allowlists exatas. Scopes mínimos e autorização no backend. Se exigir tenants separados por ambiente/cliente, o desenho Free não atende.

## Vercel Hobby
Preferir projeto por aplicação para separar variáveis, domínios e release. Preview branch-scoped pode representar staging, mas não é Custom Environment. Uso pessoal/não comercial apenas; não contornar por separar projetos, domínios ou contas.

## Neon Free
Projeto Neon é a fronteira recomendada por aplicação/domínio de dados. Quotas atuais consultadas: 100 projetos, 10 branches/projeto, 100 CU-hours/projeto/mês, 1 GB por projeto, 20 GB agregados e 5 GB de transferência pública/projeto/mês. O agregado de storage impede assumir 100 projetos com 1 GB cada sem limite global. Branches não são backup independente nem fronteira física forte. Separar credenciais e roles; runtime nunca usa owner/migrator.

## Multi-tenancy no PostgreSQL
1. tenant_id NOT NULL em tabelas multi-tenant.
2. FKs compostas quando necessário para impedir referências cruzadas.
3. Índices com tenant_id para consultas seletivas.
4. RLS habilitada com USING e WITH CHECK.
5. Backend valida membership e autorização por ação.
6. Contexto de tenant/user aplicado no servidor, na mesma transação/conexão da query.
7. Não reutilizar contexto de sessão entre requisições sem reset comprovado.
8. Testes negativos automatizados de leitura/escrita entre tenants.
9. Runtime sem BYPASSRLS, ownership ou superuser.
10. Avaliar SQL injection, SECURITY DEFINER, jobs, migrations, caches, arquivos e storage externo.

RLS isolada não é segurança completa: todas as vias de acesso precisam aplicar a mesma fronteira de autorização.

## Gatilhos para separar recursos
- Exigência legal/contratual de isolamento físico.
- Requisitos incompatíveis de retenção, residência, criptografia ou recuperação.
- Restore de um cliente sem afetar outros.
- Equipes/terceiros com acessos distintos.
- Credencial única com blast radius excessivo.
- Quotas/concorrência ameaçam disponibilidade de outros apps.
- Isolamento não comprovável por testes/revisão.

Separar o recurso na fronteira necessária; se não couber no Free, marcar BLOQUEADO NO FREE.

## Dados e ambientes
Preferir dados sintéticos. Não copiar dados pessoais/produção para branches de desenvolvimento sem base legal, minimização e mascaramento. Preview nunca usa connection string de produção. Secrets não entram em manifestos, exemplos, issues, PRs ou logs.

## Fontes
[Auth0](https://support.auth0.com/center/s/article/Second-Tenant-on-a-Free-Plan) · [Neon](https://neon.com/docs/introduction/plans) · [GitHub environments](https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments) · [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions) · [Vercel](https://vercel.com/docs/limits/fair-use-guidelines)
