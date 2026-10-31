# 02 — GitHub Actions seguro

## Permissões

~~~yaml
permissions:
  contents: read
~~~

Conceder permissão adicional somente ao job que precisa. Não usar contents: write, pull-requests: write ou id-token: write por conveniência.

## Eventos
- pull_request: recomendado para validar contribuições sem expor secrets a forks.
- push em main: validações em código confiável.
- workflow_dispatch: somente operações controladas; não equivale a aprovação de deployment.
- Evitar pull_request_target para compilar/executar código de PR externo.
- Não executar artifacts de workflow não confiável em workflow privilegiado sem validar origem e conteúdo.

## Actions de terceiros
- Fixar actions por SHA completo revisado, não apenas tag mutável.
- Revisar origem, permissões, releases e dependências transitivas.
- Atualizar SHAs deliberadamente e documentar a revisão.
- Definir timeout-minutes e concurrency.
- Usar persist-credentials: false quando checkout não precisa fazer push.

## Secrets
- Não colocar secrets em YAML, outputs, logs, artifacts ou argumentos CLI.
- Não imprimir env nem usar set -x em etapa que manipule segredo.
- Restringir secret ao workflow/job/step que realmente necessita.
- Em PR de fork, assumir que secrets não estão disponíveis; nunca contornar essa proteção.
- GitHub Free em repositórios privados não deve ser presumido como oferecendo Environment secrets e required reviewers. Validar disponibilidade por visibilidade/plano.
- Se proteção de ambiente não estiver disponível, manter deploy/migration como procedimento manual fora do workflow automático e usar secrets apenas em contexto confiável.

## OIDC
GitHub OIDC exige id-token: write no workflow/job e o provedor de destino deve validar issuer, audience e subject. Só usar se o serviço de destino suportar confiança federada e a configuração estiver disponível no plano Free. Não inventar suporte OIDC para Auth0 ou Neon sem documentação específica.

Para Vercel CLI, documentar token de escopo mínimo apenas se deploy pela Actions for realmente necessário. Preferir integração Git nativa para Hobby quando elegível.

## Matriz de risco
| Padrão | Avaliação |
|---|---|
| PR workflow read-only sem secrets | Recomendado |
| Secret de produção em PR workflow | Proibido |
| pull_request_target + checkout do fork + npm test | Proibido |
| Actions fixadas por SHA completo | Recomendado |
| Permissões write globais | Evitar |
| Vercel token em argumento CLI | Evitar |
| Build/test de código de fork com token | Proibido |

## Referências
- https://docs.github.com/en/actions/reference/security/secure-use
- https://docs.github.com/en/actions/reference/security/securely-using-pull_request_target
- https://docs.github.com/en/actions/reference/security/oidc
- https://github.com/pricing
