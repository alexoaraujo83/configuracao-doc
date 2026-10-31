# Cadastro e escopo do projeto

## Finalidade

Criar identidade estável para cada avaliação de conformidade, evitando misturar recursos, evidências e decisões de projetos diferentes.

## Identificadores

Adotar um slug imutável e não sensível, por exemplo `portal-interno`. Não incluir nomes de clientes, e-mails, IDs de conta, domínio privado ou informação confidencial no slug.

Padrão de identificação:
- Projeto: `PRJ-<slug>`
- Ciclo: `CYC-AAAA-MM-NN`
- Evidência: `EV-<domínio>-NNNN`
- Não conformidade: `NC-NNNN`
- Exceção: `EX-NNNN`
- Decisão: `REL-AAAA-MM-DD-NN`

## Dados obrigatórios

- Nome e slug do projeto.
- Objetivo, proprietário do produto e responsável técnico.
- Repositório e branch de referência.
- Stack e versões efetivamente identificadas.
- Classificação de dados e finalidade do tratamento.
- Ambientes adotados e respectivos identificadores não secretos.
- Auth0: tenant lógico e aplicação por ambiente; nunca registrar client secret.
- Vercel: projeto e escopo Hobby; confirmar uso pessoal/não comercial.
- Neon: projeto, database, branches e roles por ambiente; nunca registrar connection string.
- Dependências externas, integrações e responsáveis.
- Requisitos aplicáveis, exclusões justificadas e riscos conhecidos.
- Data de início, ciclo, próxima revisão e status.

## Matriz de ambientes

Para cada ambiente, registrar apenas nomes e identificadores não sensíveis:

| Campo | Development | Preview | Staging (se adotado) | Production |
|---|---|---|---|---|
| Git ref | | | | |
| Vercel project/scope | | | | |
| Auth0 application alias | | | | |
| Auth0 callback origin | | | | |
| Neon project/database alias | | | | |
| Neon branch alias | | | | |
| Data class | | | | |
| Owner | | | | |

Ambientes lógicos não garantem isolamento. Validar separação real de credenciais, banco, callbacks e permissões. Não copiar credenciais entre ambientes.

## Gate de elegibilidade Free

Antes de avaliar os demais controles, registrar:
- Vercel Hobby: uso pessoal e não comercial? Se não, NO-GO para este plano.
- Auth0: número de tenants, aplicações e recursos necessários dentro da oferta Free vigente?
- Neon: armazenamento, compute, branches, egress e retenção dentro das quotas atuais?
- GitHub Free: recursos de proteção, Actions e secrets disponíveis para visibilidade e tipo de repositório?
- Ausência de dependência em trials, créditos, cartão ou add-ons pagos.

Guardar URL oficial, data de consulta e captura sanitizada ou exportação pública pertinente. Se a oferta divergir entre páginas, registrar a divergência e aplicar limite conservador até confirmação.

## Critério de aceite do cadastro

O cadastro está completo quando todos os campos obrigatórios têm valor ou justificativa explícita “não aplicável/pendente”, há responsáveis nomeados por função e a elegibilidade Free foi analisada. Cadastro completo não significa aprovação técnica.
