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
## Stack de implementação Next.js + TypeScript + Neon

11. [Next.js App Router + TypeScript](10-nextjs-typescript/README.md)
12. [Auth0 + Next.js SDK v4](11-auth0-nextjs/README.md)
13. [Neon PostgreSQL — driver, roles, RLS e migrations](12-neon-postgresql/README.md)
14. [Ambientes, CI, segurança, testes e operação](13-ambientes-testes-operacao/README.md)

A stack é somente documental nesta etapa. Os exemplos não significam que pacotes foram instalados, migrations executadas ou deployments realizados. Consulte a matriz Free antes de aplicar qualquer procedimento.
