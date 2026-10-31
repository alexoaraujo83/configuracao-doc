# 06 — Templates por projeto

Use estes modelos para iniciar um novo projeto sem copiar segredos nem assumir configurações específicas de outro produto.

## Ficha de projeto

- Nome e repositório:
- Responsável técnico:
- Framework/runtime:
- Região e requisitos de residência:
- Ambientes necessários:
- Auth0 tenant/application/API por ambiente:
- Vercel team/project por ambiente:
- Neon project/branch/database/role por ambiente:
- Domínios e callbacks:
- Variáveis requeridas (nomes, nunca valores):
- Dados tratados e classificação:
- Requisitos de disponibilidade, RPO e RTO:
- Plano e limites de cada fornecedor:

## Checklist de onboarding

- [ ] Definir arquitetura e classificação de dados.
- [ ] Provisionar recursos não produtivos.
- [ ] Configurar Auth0 e URLs allowlisted.
- [ ] Conectar GitHub e Vercel.
- [ ] Criar branches/roles Neon.
- [ ] Cadastrar variáveis por ambiente.
- [ ] Executar testes de autenticação, autorização e banco.
- [ ] Revisar segurança e observabilidade.
- [ ] Aprovar plano de produção e rollback.

## Template de inventário

Copiar a ficha para cada aplicação e manter IDs/nomes de recursos, owners, região, plano, data da última verificação e links administrativos. Não incluir valores secretos.

## Verificação obrigatória de planos Free

Adicionar à ficha de cada projeto e preencher antes de provisionar:

- Plano Free e link da página oficial:
- Data da verificação e responsável:
- Limite mensal de compute/minutos/invocações:
- Limite de storage/transfer/branches:
- Retenção de logs e restore:
- Uso comercial permitido pelos termos? (sim/não/pendente):
- Recursos desejados não incluídos no Free:
- Alternativa open source/self-hosted:
- Requisitos que ficam BLOCKED sem recurso pago:

Não iniciar um provisionamento se o serviço exigir trial, cartão, add-on pago ou upgrade. Não assumir que a disponibilidade de uma integração equivale a inclusão no Free.
