# Índice da documentação

Esta documentação é organizada por domínio técnico e ciclo operacional. Cada página deve apresentar primeiro a **configuração padrão inicial** (mínimo seguro para começar) e depois a **configuração completa** (opções, decisões, dependências, validação e operação).

## Seções

1. [Padrões gerais](01-padroes-gerais/README.md)
2. [Auth0](02-auth0/README.md)
3. [Vercel — guia principal](03-vercel/README.md)
4. [Vercel — guia detalhado](03-vercel/guia-detalhado.md)
5. [Neon](04-neon/README.md)
6. [Integrações](05-integracoes/README.md)
7. [Templates por projeto](06-templates-projeto/README.md)
8. [Operação e governança](07-operacao-governanca/README.md)
9. [Referências e histórico](08-referencias-historico/README.md)
10. [GitHub — governança e Actions](09-github/README.md)

## Convenção de evidências

- **Verificada:** observada diretamente no painel, API ou CLI, com data e escopo registrados.
- **Recomendada:** padrão técnico proposto, ainda não necessariamente aplicado.
- **Pendente de validação:** depende de acesso, decisão arquitetural ou confirmação do responsável.

Não registrar valores secretos. Exemplos de nomes de variáveis são permitidos; valores devem ser fictícios ou omitidos.

## Stack Next.js + TypeScript + Neon

11. [Next.js App Router + TypeScript](10-nextjs-typescript/README.md)
12. [Auth0 + Next.js SDK v4](11-auth0-nextjs/README.md)
13. [Neon PostgreSQL — driver, roles, RLS e migrations](12-neon-postgresql/README.md)
14. [Ambientes, CI, segurança, testes e operação](13-ambientes-testes-operacao/README.md)
15. [Inventário dos painéis GitHub, Auth0, Vercel e Neon](14-inventario-paineis/README.md)

A stack é somente documental nesta etapa. Os exemplos não significam que pacotes foram instalados, migrations executadas ou deployments realizados. Consulte a matriz Free antes de aplicar qualquer procedimento.

## Arquitetura e segurança

16. [Matriz de arquitetura por tipo de projeto — somente Free](19-matriz-arquitetura-free/README.md)
17. [Segurança, RBAC, RLS e isolamento multi-tenant — somente Free](20-seguranca-rbac-rls-free/README.md)
18. [Modelagem de ameaças e resposta a incidentes — somente Free](21-modelagem-ameacas-incidentes-free/README.md)

A modelagem de ameaças e os runbooks são padrões documentais. Não significam que controles foram aplicados, incidentes ocorreram ou exercícios foram executados.


## Continuidade, qualidade e governança

As seções 19–23 documentam recuperação, observabilidade, releases, testes e conformidade. Não representam execução de exercícios, validação de produção ou certificação.

## Revisões técnicas

19. [Continuidade de negócio e recuperação de desastres — Free](22-continuidade-recuperacao-free/README.md)
20. [Observabilidade, monitoramento e capacidade — Free](23-observabilidade-monitoramento-capacidade-free/README.md)
21. [Gestão de mudanças, releases e versionamento — Free](24-gestao-mudancas-releases-free/README.md)
22. [Testes de qualidade, segurança e validação — Free](25-testes-seguranca-validacao-free/README.md)
23. [Matriz mestra de conformidade Free](26-matriz-mestra-conformidade-free/README.md)
24. [Revisão técnica aprofundada — Next.js 16 + Auth0 SDK v4 + Neon PostgreSQL](27-revisao-tecnica-next16-auth0-neon-free/README.md)

A revisão registra correções documentais e limites de compatibilidade; não substitui validação operacional autorizada.


## Manual operacional de implantação Free

26. [Procedimentos operacionais para Next.js + Auth0 + Neon + GitHub + Vercel](31-manual-operacional-implantacao-free/README.md)

Inclui preparação de estação, configuração por serviço, CI, validação, troubleshooting, rollback e checklist. Conteúdo de referência não equivale à configuração real verificada.
