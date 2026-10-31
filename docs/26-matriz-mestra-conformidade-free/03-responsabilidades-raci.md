# Responsabilidades e governança

## Papéis

Atribuir pessoas reais a cada papel no projeto consumidor. Uma pessoa pode acumular papéis em projeto pequeno, mas a aprovação de risco crítico deve ter revisão independente sempre que possível.

| Papel | Responsabilidade |
|---|---|
| Product/Service Owner | Define finalidade, usuários, dados, criticidade e elegibilidade de uso |
| Tech Lead / Architect | Aprova arquitetura, padrões técnicos e critérios de release |
| Engineering Owner | Implementa aplicação, testes e correções |
| IAM Owner | Administra Auth0, fluxos OIDC, sessão e políticas de identidade |
| DBA / Data Owner | Administra schema, roles, RLS, migrations, backup e integridade |
| Platform/DevOps Owner | GitHub, Actions, Vercel, ambientes, deploy e observabilidade |
| Security Reviewer | Revisa riscos, dependências, secrets, autorização e exceções |
| Release Owner | Consolida evidências, executa gate e registra GO/NO-GO |
| Evidence Custodian | Mantém manifesto, links, validade e sanitização das evidências |

## Matriz RACI

R = executa; A = responde pela decisão; C = consultado; I = informado.

| Atividade | Owner | Tech Lead | Engineering | IAM | DBA | Platform | Security | Release |
|---|---|---|---|---|---|---|---|---|
| Classificar uso comercial/pessoal | A/R | C | I | I | I | C | C | I |
| Definir arquitetura | C | A | R | C | C | C | C | I |
| Configurar Auth0 | I | C | C | A/R | I | C | C | I |
| Implementar RBAC de domínio | C | A | R | C | C | I | C | I |
| Configurar PostgreSQL/RLS | I | C | R | I | A | C | C | I |
| Auditar dependências | I | A | R | I | I | C | C | I |
| Proteger GitHub Actions | I | C | C | I | I | A/R | C | I |
| Validar Vercel e env | I | C | C | C | I | A/R | C | I |
| Executar restore de teste | I | C | C | I | A/R | C | C | I |
| Consolidar evidências | I | C | R | R | R | R | C | A |
| Avaliar risco residual | A | R | C | C | C | C | R | C |
| Decidir release | A | C | I | I | C | C | C | R |

## Segregação de funções

- Quem implementa uma mudança crítica não deve ser o único aprovador dessa mudança.
- Mudanças de IAM, RLS, grants, migrations destrutivas e secrets exigem revisão técnica específica.
- A pessoa responsável pelo release confirma evidências, mas não pode dispensar sozinha um gate crítico.
- Em equipes de uma pessoa, registrar a limitação e instituir revisão posterior por outra pessoa antes de ampliar exposição ou criticidade.

## Registro de aprovação

Aprovação deve registrar:
- identificador do release e commit SHA;
- escopo e ambiente;
- requisitos críticos avaliados;
- evidências verificadas;
- exceções e risco residual;
- decisão GO, GO condicionado ou NO-GO;
- nome/função do aprovador e data;
- condições, prazo e gatilho de reavaliação.

Uma aprovação não transfere responsabilidades legais ou operacionais do proprietário do sistema.
