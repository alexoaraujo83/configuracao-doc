# 01 — Matriz de cobertura e critérios de aceite

| Área | Conteúdo documental | Evidência de conclusão | Estado |
|---|---|---|---|
| Arquitetura Next.js/TypeScript | App Router, camadas, server-only | PR #27 e #28 | Documentada |
| Contratos HTTP | DTO, endpoints, erros, paginação | PR #29 | Documentada |
| Persistência | ER, dicionário, migrations, índices, backup | PR #30 | Documentada |
| Segurança | Auth0, cookies, CSRF, autorização, RLS | PR #31 | Documentada |
| CI/CD | GitHub Actions, Neon migration gates, Vercel Git | PR #32 | Documentada |
| Ambientes | Development, Preview, Staging lógico, Production | PR #33 | Documentada |
| Testes | Vitest, RTL, integração, contratos, RLS, Playwright | PR #34 | Documentada |
| Operação | Logs, quotas, incidentes, backup/restore | PR #35 | Documentada |
| Consolidação | Matriz, lacunas, ordem de revisão | PR #37 | Integrada à main |

## Critérios para aceite documental
- Cada área possui guia, pré-requisitos, passos, parâmetros, riscos, validação e fontes.
- Cada recurso é classificado como Free disponível, limitado, não disponível ou pendente de confirmação.
- Exemplos são claramente ilustrativos e não se passam por código executado.
- Variáveis e secrets são descritos sem valores reais.
- Referências internas só são consideradas publicadas após incorporação dos respectivos PRs.
- Limites voláteis têm fonte e data de consulta.

## Critérios para aceite operacional (fora do escopo atual)
- Revisão e merge dos PRs documentais autorizados (PR #37 incorporado à `main` em 31/10/2026).
- Instalação reproduzível e typecheck/build.
- Migrations aplicadas em banco de teste.
- Testes unitários, integração, RLS e E2E executados.
- Configuração Auth0/Vercel/Neon validada em cada ambiente.
- Backup e restore testados em ambiente isolado.
- CI executada e quotas verificadas.

Nenhum desses critérios operacionais é marcado como concluído por esta entrega documental.
