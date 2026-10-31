# Estrutura do projeto e responsabilidade dos módulos

| Diretório | Responsabilidade | Execução |
|---|---|---|
| src/app | rotas, layouts, páginas e Route Handlers | servidor por padrão; client explícito |
| src/features | componentes e casos de uso por funcionalidade | conforme módulo |
| src/domain | entidades, regras e interfaces de repositório | ambiente neutro |
| src/server/auth | sessão, identidade e guards | servidor |
| src/server/db | driver, transações e adaptadores | servidor |
| src/server/policies | autorização e contexto tenant | servidor |
| src/server/observability | logging e correlação | servidor |
| src/shared | schemas e tipos sem segredo | compartilhável |

## Regras de importação

1. domain não importa app, features, server, Next.js, React ou SDKs externos.
2. server pode importar domain, mas nunca componentes React.
3. Client Components não importam server, db, auth ou configuração secreta.
4. features não abre conexão direta com Neon.
5. Route Handlers validam payload e delegam a casos de uso.
6. Repositórios traduzem persistência em contratos de domínio.
7. shared não acessa variáveis secretas.

## Convenção de nomes

- Componentes React: PascalCase (ProjectCard.tsx).
- Funções e variáveis: camelCase.
- Tipos e classes: PascalCase.
- Configuração: UPPER_SNAKE_CASE quando necessário.
- Schemas: nome da entidade + Schema.
- Casos de uso: verbo + objeto (createProject, listProjects).
- Repositórios: nome da entidade + Repository.

## Escala

Não criar camadas vazias por antecipação. Projeto pequeno pode manter menos diretórios, preservando segredo/banco server-only, domínio independente, validação de entrada e autorização por operação. Extrair módulos quando houver regras reutilizadas ou necessidade clara de teste isolado.

## Checklist de PR

- [ ] Mudança está na camada correta.
- [ ] Domínio permanece independente de framework e fornecedor.
- [ ] Client bundle não importa código server-only.
- [ ] UI não consulta banco diretamente.
- [ ] Rota valida input e delega a caso de uso.
- [ ] Caso de uso verifica identidade e autorização.
- [ ] Erros são convertidos na borda sem vazamento.
- [ ] Testes cobrem regra e cenário de negação.
