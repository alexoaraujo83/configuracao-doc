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