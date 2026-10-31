# Inventário dos painéis — GitHub, Auth0, Vercel e Neon

> Revisão documental: 31/10/2026. Escopo: navegação técnica dos painéis e recursos compatíveis com planos Free. Referência documental; não representa captura nem auditoria visual das contas conectadas.

## Objetivo

Mapear menus e submenus necessários para configurar, proteger e operar:
- GitHub Free — repositórios, colaboração, Pull Requests, Actions e segurança.
- Auth0 Free — tenant, aplicações, APIs, conexões, usuários, sessões, tokens e segurança.
- Vercel Hobby — projeto, Git, deployments, variáveis, domínios e observabilidade, respeitando uso pessoal/não comercial.
- Neon Free — projeto PostgreSQL, branches, databases, roles, SQL Editor, conexão, métricas e recuperação.

## Como interpretar

Cada item registra caminho, finalidade, configuração relevante, elegibilidade Free e evidência. Os painéis mudam de navegação, agrupamento e nomenclatura. Quando a interface divergir, registrar o texto efetivamente exibido, data, plano e contexto, sem registrar secrets.

## Estado de evidência

| Estado | Significado |
|---|---|
| DOCUMENTADO | Caminho ou comportamento descrito em documentação oficial |
| OBSERVADO NA CONTA | Captura/leitura direta com data e contexto |
| RECOMENDADO | Padrão proposto para projetos novos |
| PENDENTE | Requer confirmação no painel, plano ou teste |

Os caminhos deste conjunto são DOCUMENTADOS ou RECOMENDADOS. Nenhum submenu é declarado como observado nesta revisão.

## Índice

- [GitHub Free](github.md)
- [Auth0 Free](auth0.md)
- [Vercel Hobby](vercel.md)
- [Neon Free](neon.md)

## Matriz resumida

| Plataforma | Área | Limitação condicionante |
|---|---|---|
| GitHub Free | Repositórios públicos e privados, PRs, Actions | Recursos avançados de proteção em repositórios privados dependem de plano superior; rulesets Free aplicam-se a repositórios públicos |
| Auth0 Free | Identidade e autenticação | Quotas e recursos variam; não presumir MFA OTP/Duo, M2M Add-on ou RBAC por Organization |
| Vercel Hobby | Projetos pessoais | Uso pessoal/não comercial; BLOCKED para SaaS ou aplicação comercial |
| Neon Free | PostgreSQL e branching | Quotas de compute, armazenamento, transferência, branches e recuperação; sem protected branches nativas |

Consulte a [Matriz Free](../09-planos-free/README.md), os guias [Next.js](../10-nextjs-typescript/README.md), [Auth0 + Next.js](../11-auth0-nextjs/README.md), [Neon](../12-neon-postgresql/README.md) e [ambientes/operação](../13-ambientes-testes-operacao/README.md).

## Procedimento de auditoria visual futura

1. Acessar o painel com identidade autorizada e MFA habilitado.
2. Registrar fornecedor, URL, conta/organização/tenant/team/projeto/branch, plano e data/hora UTC.
3. Capturar somente navegação e estado não secreto.
4. Não abrir, copiar ou registrar senhas, tokens, client secrets, API keys ou connection strings.
5. Marcar o item como visível, oculto por permissão, indisponível no plano ou ausente.
6. Comparar rótulos reais com este inventário.
7. Atualizar a documentação somente após confirmar diferenças em fonte oficial.

## Critérios de aceite

- [ ] Cada plataforma possui inventário de menus e submenus.
- [ ] Cada item tem finalidade e uso técnico.
- [ ] Limites e indisponibilidades Free estão explícitos.
- [ ] Nenhum recurso pago é apresentado como disponível.
- [ ] Caminhos sujeitos a variação estão marcados.
- [ ] Configuração real não é confundida com recomendação.
- [ ] Fontes oficiais constam em cada página.
