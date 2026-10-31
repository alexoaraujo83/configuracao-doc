# 26 — Matriz mestra de conformidade Free

> Estado: padrão documental recomendado. Data-base de consulta dos limites: 2026-10-31. Nenhum controle de serviço foi aplicado ou certificado por esta documentação.

## Objetivo

Consolidar requisitos, controles, evidências, responsáveis, limitações dos fornecedores e critérios de aprovação para a stack Next.js + TypeScript + Auth0 + Neon PostgreSQL + GitHub + Vercel, sob a restrição exclusiva de planos Free.

A matriz serve como instrumento de governança e decisão. Não é certificação, auditoria independente, parecer jurídico, atestado de segurança ou prova de execução.

## Regras de leitura

Cada controle recebe dois estados separados:

- **Estado do requisito:** Não avaliado, Conforme, Não conforme, Parcial ou Não aplicável.
- **Estado da evidência:** Ausente, Planejada, Coletada, Verificada ou Expirada.

A documentação de um procedimento não prova que o controle foi implementado. Uma evidência só será **Verificada** quando houver artefato observável, escopo, data, responsável e resultado reproduzível.

## Navegação

1. [Matriz mestra de requisitos](01-matriz-requisitos.md)
2. [Catálogo de controles e evidências](02-controles-evidencias.md)
3. [Responsabilidades e governança](03-responsabilidades-raci.md)
4. [Limitações dos planos Free](04-limitacoes-free.md)
5. [Quality gates e aprovação](05-gates-aprovacao.md)
6. [Registro de evidências e exceções](06-registro-evidencias.md)
7. [Checklist de aprovação](07-checklist-aprovacao.md)

## Escopo técnico

- Aplicação: Next.js App Router + TypeScript + Node.js.
- Identidade: Auth0 Free, sem pressupor recursos pagos.
- Persistência: Neon PostgreSQL Free.
- Código e automação: GitHub Free.
- Hospedagem: Vercel Hobby, apenas quando o uso for pessoal e não comercial.
- Dependências: npm, lockfile versionado e ferramentas de análise disponíveis sem contratação.
- Ambientes lógicos: Development, Preview, Staging e Production; não presumir isolamento contratual ou técnico equivalente a planos pagos.

## Princípios obrigatórios

1. Free significa preço recorrente de US$ 0 e ausência de trial, crédito promocional, add-on ou recurso pago.
2. Não cadastrar cartão para desbloquear funcionalidade opcional.
3. Não presumir que limites publicados hoje permanecerão iguais.
4. Não usar Vercel Hobby para produto, serviço ou atividade comercial.
5. Não depender de Auth0 Organizations/RBAC gerenciado, tenants adicionais ou recursos indisponíveis no Free.
6. Não presumir GitHub Environments ou environment secrets em repositório privado Free.
7. Não tratar Neon PITR como backup externo nem como garantia de RPO.
8. Não armazenar credenciais, tokens, dados pessoais ou dumps reais nos artefatos de evidência.
9. Bloquear release se um controle crítico estiver não conforme, sem evidência ou com evidência expirada.
10. Registrar limitações e risco residual; não converter limitação do fornecedor em aprovação tácita.

## Relação com as etapas anteriores

Esta seção consolida padrões das documentações de arquitetura, RBAC/RLS, ameaças e incidentes, continuidade, observabilidade, releases e testes. Os PRs dessas etapas podem permanecer em revisão independente; a matriz não presume que estejam mesclados na branch principal. Antes da aprovação definitiva, reconciliar os links e versões efetivamente incorporados em main.

## Uso em novos projetos

Copiar a matriz por projeto, atribuir responsáveis nominais no ambiente de governança do projeto e substituir os campos de evidência por referências a artefatos reais. Não copiar segredos nem evidências contendo dados pessoais.

## Critério global

Um projeto só recebe resultado **APROVADO PARA O ESCOPO DECLARADO** quando todos os controles críticos aplicáveis estão Conforme, todas as evidências críticas estão Verificadas e vigentes, não existem exceções críticas abertas, o uso é compatível com os termos Free e os aprovadores designados registraram decisão e data. Isso não representa garantia de segurança absoluta ou SLA dos fornecedores.
