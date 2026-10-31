# Governança de configuração e dependências

## Papéis mínimos

| Papel | Responsabilidade |
|---|---|
| Proprietário do projeto | finalidade, classificação de dados, aprovação e quotas Free |
| Mantenedor técnico | runtime, dependências, CI e documentação |
| Revisor de segurança | Auth0, autorização, roles, RLS, secrets e exceções |
| Responsável por release | critérios, janela, validação e rollback |

Uma pessoa pode acumular papéis em projeto pequeno, mas mudanças sensíveis devem receber revisão independente sempre que possível.

## Ciclo de revisão

- A cada PR: revisar impacto, lockfile, variáveis, permissões do workflow e documentação.
- Mensalmente: verificar dependências críticas, avisos de segurança e quotas consumidas.
- Trimestralmente: revisar elegibilidade Free, callbacks, roles, RLS, membros e acessos.
- Antes de release: verificar variáveis, migrations pendentes, dados sintéticos, critérios de aceite e rollback.
- Após incidente: revogar credenciais afetadas, preservar evidências não sensíveis, registrar causa e atualizar runbook.

## Política de dependências

1. Manter lockfile versionado.
2. Usar instalação reprodutível em CI.
3. Atualizar dependências em PRs pequenos e rastreáveis.
4. Conferir breaking changes de Next.js, Auth0 SDK, Neon driver e actions.
5. Não aceitar atualização major sem plano de migração e rollback.
6. Não declarar que dependências estão atualizadas sem executar verificação autorizada.
7. Fixar versões revisadas de GitHub Actions; avaliar pin por SHA completo conforme a política do projeto.

## Política de GitHub Actions Free

- Usar permissões mínimas no bloco permissions.
- Não imprimir secrets nem habilitar debug que possa expô-los.
- Não executar código não confiável de Pull Request com credenciais privilegiadas.
- Não depender de Environment protection/secrets como se estivessem disponíveis em todo repositório Free privado.
- Usar banco de teste e dados sintéticos.
- Controlar concorrência para evitar execuções redundantes.
- Verificar quotas e limites atuais no painel/documentação do GitHub.

## Revisão de quotas

Manter um registro por fornecedor: recurso, quota exibida, unidade, período de reset, consumo, data/hora e URL da documentação. Nunca extrapolar a partir de quotas antigas ou de outro tipo de conta. Ao atingir limite, reduzir escopo ou suspender o recurso; não sugerir upgrade ou trial dentro deste padrão.

## Registro de exceções

Cada exceção deve conter requisito não atendido; limitação Free observada e evidência; risco residual; alternativa sem custo considerada; responsável e aprovador; prazo de revisão; critério de encerramento.

## Gate de conformidade

O projeto não deve ser considerado pronto se houver segredo versionado, Preview conectado à base Production, privilégio de runtime excessivo, callback Auth0 amplo sem justificativa, dependência de recurso pago ou uso comercial do Vercel Hobby.
