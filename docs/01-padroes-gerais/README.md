# 01 — Padrões gerais

## Configuração padrão inicial

- Manter Development, Preview, Staging e Production separados logicamente.
- Aplicar menor privilégio em identidades, roles e credenciais.
- Usar secrets por ambiente; nunca commitar `.env` real, tokens ou connection strings.
- Definir nomenclatura consistente para projetos, aplicações, branches, databases e variáveis.
- Registrar proprietário, finalidade, criticidade, ambiente e procedimento de rollback para cada recurso.

## Configuração completa

### Nomenclatura

Formato recomendado: `<produto>-<dominio>-<ambiente>`, respeitando limites e convenções de cada fornecedor. Identificadores de recursos não devem conter dados pessoais ou segredos.

### Matriz de ambientes

| Ambiente | Uso | Dados | Acesso |
|---|---|---|---|
| Development | desenvolvimento local e integração | sintéticos ou anonimizados | equipe técnica |
| Preview | validação por branch/PR | sintéticos ou isolados | equipe e revisores |
| Staging | ensaio pré-produção | mascarados/sintéticos | acesso restrito |
| Production | tráfego real | dados operacionais | menor privilégio, acesso auditado |

### Gestão de configuração

- Definir catálogo de variáveis com nome, finalidade, ambiente, sensibilidade e consumidor.
- Separar configuração pública de segredo. Variáveis expostas ao browser nunca podem conter credenciais.
- Validar variáveis obrigatórias no startup/build sem imprimir seus valores.
- Preferir identidade e credenciais de curta duração quando suportadas.

### Mudanças e auditoria

Toda mudança deve registrar motivação, responsável, ambientes, pré-requisitos, impacto, validação e rollback. Produção exige aprovação explícita e janela/estratégia compatível com criticidade.

## Checklist de revisão

- [ ] Isolamento entre ambientes confirmado.
- [ ] Segredos fora do Git e com escopo mínimo.
- [ ] Logs sem tokens, senhas ou dados pessoais desnecessários.
- [ ] Backup/restore e rollback documentados.
- [ ] Proprietários e responsáveis definidos.

## Política obrigatória de compatibilidade Free

Este repositório utiliza somente planos gratuitos. Nenhuma arquitetura, checklist ou exemplo deve depender de upgrade, trial, crédito promocional, add-on ou recurso pago. Antes de habilitar qualquer item, confirmar preço, termos, quotas, retenção, região e elegibilidade comercial na página oficial vigente.

- Marcar explicitamente como **NÃO DISPONÍVEL NO FREE** qualquer controle pago.
- Oferecer alternativa gratuita, open source ou self-hosted apenas quando tecnicamente viável.
- Não chamar controle compensatório de equivalente a enforcement nativo.
- Se não existir alternativa gratuita que satisfaça um requisito de segurança, classificar como **BLOCKED — requisito incompatível com Free**.
- Considerar custos indiretos de infraestrutura própria, energia, conectividade, backup e manutenção.
- Registrar plano, data, evidência, limites e responsável por cada recurso.

A Vercel Hobby é apenas para projetos pessoais/não comerciais. Não utilizá-la para hospedar SaaS ou aplicações comerciais.
