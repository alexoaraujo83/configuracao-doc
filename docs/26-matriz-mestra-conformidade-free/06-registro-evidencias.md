# Registro de evidências, exceções e não conformidades

## Modelo de registro de evidência

Copiar uma linha por artefato:

| Campo | Valor |
|---|---|
| Evidence ID | EV-AAAA-NNN |
| Requirement ID | GOV-01 / APP-01 / etc. |
| Projeto | nome lógico |
| Ambiente | Development / Preview / Staging / Production |
| Commit SHA / versão | hash ou versão exata |
| Fonte | GitHub / Auth0 / Neon / Vercel / CI / teste |
| Artefato | URL interna ou caminho protegido |
| Data de coleta | ISO 8601 |
| Coletado por | papel/responsável |
| Revisado por | papel/revisor |
| Resultado | aprovado / reprovado / inconclusivo |
| Validade até | data |
| Sanitização | confirmada / pendente |
| Observação | sem segredos ou PII |

## Modelo de não conformidade

| Campo | Descrição |
|---|---|
| NC ID | identificador único |
| Requirement ID | requisito afetado |
| Descrição | condição observada, sem especulação |
| Criticidade | Crítica / Alta / Média / Baixa |
| Impacto | confidencialidade, integridade, disponibilidade, conformidade |
| Evidência | artefato que demonstra a condição |
| Causa | confirmada ou ainda desconhecida |
| Contenção | medida aprovada |
| Correção | ação e responsável |
| Prazo | data acordada |
| Reteste | resultado e evidência |
| Encerramento | revisor e data |

## Modelo de exceção

Uma exceção deve conter:
- requisito e motivo;
- justificativa de negócio e técnica;
- impacto e risco residual;
- controle compensatório;
- responsável pela execução;
- aprovador independente;
- início e expiração;
- gatilho de revogação;
- plano para eliminar a exceção.

Não permitir exceção para: violação dos termos de uso, uso comercial de Hobby, uso de recurso pago sob política Free-only, exposição intencional de segredo, bypass de autorização ou aceitação de acesso cruzado indevido.

## Manifesto de release

Exemplo de estrutura sem valores secretos:

~~~yaml
release:
  project: "nome-do-projeto"
  version: "1.2.3"
  commit_sha: "SHA_COMPLETO"
  deployment_id: "ID_DA_VERCEL"
  environment: "production"
  reviewed_at: "2026-10-31T00:00:00Z"
  plan_eligibility:
    github: "free-confirmed"
    auth0: "free-confirmed"
    neon: "free-confirmed"
    vercel: "hobby-personal-non-commercial-confirmed"
  gates:
    typecheck: "pass"
    lint: "pass"
    unit: "pass"
    integration: "pass"
    e2e: "pass"
    dependency_audit: "pass"
    auth0_rbac: "pass"
    postgres_rls: "pass"
    migrations: "pass"
    restore: "pass"
    production_smoke: "pass"
  evidence_index: "URI_INTERNA_DO_MANIFESTO"
  decision: "GO"
  approved_by:
    - "ROLE_OWNER"
    - "ROLE_SECURITY_REVIEWER"
~~~

O exemplo é um modelo, não uma evidência nem um resultado de teste real.

## Armazenamento e retenção

- Guardar evidências em local com acesso mínimo necessário.
- Não versionar logs completos, dados pessoais ou artefatos de produção no repositório público.
- Para cada artefato, registrar checksum quando aplicável.
- Definir retenção conforme obrigações do projeto e capacidade do plano Free.
- Se o serviço gratuito não oferecer retenção adequada, manter cópia externa segura sem incorrer em serviço pago.
- Remover ou redigir informações sensíveis antes de compartilhar evidência com revisores.
