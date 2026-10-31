# 3. Auditoria npm e segurança de GitHub Actions

## 3.1 Inventário reproduzível

Versionar:
- package.json e package-lock.json;
- versão Node.js e npm suportada;
- lista de scripts e dependências de produção/desenvolvimento;
- workflows GitHub Actions e actions externas usadas;
- arquivo dependabot.yml, se adotado.

Em CI, usar npm ci para instalação reproduzível a partir do lockfile. Não atualizar dependências implicitamente durante o build.

## 3.2 Comandos npm

~~~bash
node --version
npm --version
npm ci
npm ls --all
npm audit --json
npm audit --audit-level=high
npm audit signatures
~~~

Interpretação:
- npm audit consulta avisos conhecidos nas dependências descritas no projeto; depende de package-lock/shrinkwrap consistente.
- npm audit --audit-level define o limiar de falha do comando; não remove achados de severidade menor do relatório.
- npm audit signatures verifica assinaturas do registry e atestações de provenance quando suportadas; ausência de attestation não deve ser confundida automaticamente com malware.
- npm ls ajuda a inspecionar árvore e dependências inválidas/extraneous; não é scanner de vulnerabilidade.
- Rodar com versão npm compatível com lockfile e registrar versão usada.

Não executar npm audit fix --force de forma automática. O comando pode alterar árvore de dependências e incluir breaking changes; correção deve ocorrer em branch, com diff revisado e testes.

## 3.3 Dependabot

- Habilitar Dependabot alerts e manter manifests/lockfiles atualizados.
- Configurar version updates para npm e github-actions com frequência controlada (ex.: semanal), revisando cada PR.
- Revisar security updates com prioridade baseada em severidade, reachability, exposição, versão corrigida e impacto de atualização.
- Dependabot verifica a branch padrão; não considerar isso substituto de auditoria no PR atual.
- Para GitHub Actions, alertas Dependabot podem não cobrir actions pinadas por SHA; manter inventário e atualização manual/automatizada do SHA com revisão.
- Dependabot alerts estão disponíveis em repositórios pessoais/organizacionais; confirmar a disponibilidade efetiva do recurso no repositório e a visibilidade de segurança.

## 3.4 Exemplo dependabot.yml

~~~yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5
~~~

É um exemplo de configuração documental; não foi adicionado nem habilitado no repositório.

## 3.5 Segurança de GitHub Actions

- Definir permissions no nível do workflow/job; começar com contents: read e conceder apenas permissões adicionais justificadas.
- Preferir actions confiáveis e pinadas ao SHA completo de 40 caracteres; registrar versão legível em comentário.
- Revisar action source, release, mantenedor, permissões, scripts executados e alterações de SHA.
- Evitar pull_request_target para executar código do PR; nunca fazer checkout/executar código não confiável com token privilegiado ou secrets.
- Não imprimir env, contextos completos, tokens, cookies, DATABASE_URL ou secrets.
- Usar concorrência/cancelamento para evitar jobs redundantes e consumo de quota.
- Não usar secrets de produção em pull_request, forks ou workflows que executem código externo.
- Tratar artifacts, caches e logs como dados potencialmente sensíveis; limitar conteúdo e retenção.
- Não depender de GitHub Environments, environment secrets ou required reviewers em repositórios privados Free quando não elegíveis.

## 3.6 Workflow trust boundaries

| Evento | Confiança | Segredos permitidos |
|---|---|---|
| pull_request de branch do mesmo repo | código ainda requer revisão | nenhum secret de produção |
| pull_request de fork | não confiável | nenhum secret |
| push em branch de feature | código não aprovado | nenhum secret de produção |
| push em main | revisado conforme processo | somente secrets estritamente necessários; preferir nenhum para CI |
| workflow_dispatch | depende de quem pode disparar e do código selecionado | não usar como bypass de aprovação |

## 3.7 Dependências e lockfile

- Lockfile é obrigatório e deve ser revisado junto com package.json.
- Atualizar dependências em pequenos grupos; evitar atualização em massa sem diagnóstico.
- Preferir versões corrigidas compatíveis e releases estáveis.
- Conferir changelog, breaking changes, peer dependencies, Node engine e suporte da versão Next.js.
- Remover dependências sem uso; não instalar pacote apenas para silenciar alerta.
- Conferir licenças e provenance conforme política do projeto.
- Não assumir que pacote popular, assinatura válida ou publicação recente garante segurança.

## 3.8 Critério de aprovação

- npm ci conclui a partir do lockfile.
- Auditoria sem vulnerabilidades bloqueadoras; exceções documentadas e aprovadas.
- Actions revisadas, permissions mínimas, SHA e origem registrados.
- Nenhum secret de produção exposto a PR/fork.
- Evidências sem tokens, credenciais ou conteúdo privado.
