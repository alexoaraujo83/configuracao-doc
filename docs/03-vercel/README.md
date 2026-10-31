# 03 — Vercel

## Configuração padrão inicial

1. Conectar o repositório GitHub ao projeto Vercel correto.
2. Confirmar framework preset, root directory, build command, install command e output conforme o repositório.
3. Mapear Development, Preview e Production e documentar a semântica de cada target.
4. Cadastrar variáveis por ambiente e escopo; nunca usar segredo como variável `NEXT_PUBLIC_`.
5. Restringir domains e redirects; confirmar domínio principal e aliases por ambiente.
6. Definir Node.js runtime compatível com o lockfile, framework e dependências.
7. Ativar logs e observabilidade adequados ao plano; documentar limites e retenção.

## Configuração completa — inventário do painel

| Área | Itens a registrar |
|---|---|
| Project | nome, framework, root directory, comandos, região e runtime |
| Git | repositório, production branch, deploy por PR e branch exclusions |
| Environments | Development, Preview, Production e custom environments se disponíveis |
| Environment Variables | nome, target, sensibilidade, escopo e rotação (nunca o valor) |
| Domains | domínio, aliases, DNS, TLS e redirects |
| Deployments | build, logs, status, promoção e rollback |
| Functions | runtime, região, duração, memória e limites do plano |
| Networking | firewall, proteção, trusted IPs e private networking se disponível |
| Storage / Integrations | integrações habilitadas e permissões concedidas |
| Observability | runtime logs, analytics, alerts e retenção |
| Team / Access | membros, roles, SSO e políticas disponíveis no plano |

## Estratégia de ambientes

- Development: desenvolvimento local; variáveis sincronizadas com escopo controlado.
- Preview: deployments por branch/PR; não assumir que todo Preview é seguro para dados reais.
- Staging: usar projeto separado ou custom environment se necessário e disponível; explicitar custos/limites.
- Production: branch protegida, revisão obrigatória e promoção controlada.

A nomenclatura de ambientes da Vercel não corresponde automaticamente a quatro ambientes independentes. Registrar a implementação real escolhida para cada projeto.

## Deploy e rollback

Documentar gatilhos, checks, build output, health checks, promoção, domínio associado e rollback. Antes de rollback, avaliar compatibilidade entre código e schema do banco. Usar migrations compatíveis com versões anterior e nova durante rollout.

## Guia complementar

Consulte o [guia detalhado de configuração e operação](guia-detalhado.md), que aprofunda Settings, Git Integration, ambientes, variáveis, domains, Functions, segurança, observabilidade e CLI.

## Referências

- [Vercel Documentation](https://vercel.com/docs)
- [Vercel CLI](https://vercel.com/docs/cli)
- [Deployment Environments](https://vercel.com/docs/deployments/environments)

## Restrições Vercel Hobby — baseline Free

Vercel Hobby é restrito a uso pessoal ou não comercial pelos termos oficiais. Não hospedar SaaS, produto comercial ou aplicações empresariais oferecidas a clientes nesse plano.

Os Termos de Serviço também dizem que conteúdo enviado por contas Hobby pode ser usado para treinar modelos de IA/ML e compartilhado com terceiros para essa finalidade. Verificar a opção de opt-out nas configurações da Team antes de enviar código-fonte ou dados confidenciais; se a opção não estiver acessível ou não puder ser comprovada, não enviar conteúdo sensível. Custom Environments, Deployment Protection avançada, recursos de equipe e quotas de observabilidade variam conforme plano.

Para projetos pessoais/não comerciais, utilizar somente os ambientes e limites efetivamente incluídos no Hobby. Para aplicações comerciais com orçamento de licença zero, considerar self-host com infraestrutura própria já disponível ou outro host cujo Free autorize explicitamente uso comercial. Não presumir que preço zero permita uso comercial.
