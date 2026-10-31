# 04 — Integração Git e Vercel Hobby

## Fluxo preferencial
Usar integração Git nativa da Vercel para criar Preview em branches/PRs e Production na branch configurada, se conta, repositório e uso forem elegíveis ao Hobby.

## Condições e restrições
- Hobby é destinado a uso pessoal e não comercial.
- A documentação da Vercel informa que deploy em Hobby pode ser bloqueado para repositório privado pertencente a organização GitHub; verificar regra atual e proprietário do time.
- Autor do commit pode precisar corresponder ao owner da conta Hobby.
- Pull requests de forks podem exigir autorização antes de deploy para proteger variáveis e OIDC.
- Custom environments não devem ser pressupostos no Hobby.
- Staging em Hobby pode ser modelado como branch Preview com overrides quando disponível; não presumir isolamento completo.

## Variáveis
- Development: valores locais e database branch dev.
- Preview: secrets específicos e database branch não produtiva.
- Production: secrets próprios e database principal.
- APP_BASE_URL: considerar URLs dinâmicas de Preview conforme SDK Auth0 e allowlist de callbacks.
- Alterar env vars não atualiza deployments já criados; novo deployment é necessário.

## Deploy
- PR/branch: Preview.
- Merge em main: Production após gates.
- Não duplicar deploy automático da Vercel e deploy CLI na Actions sem razão explícita.
- Não enviar secrets ao workflow de PR.
- Não expor Vercel OIDC token a código de fork não confiável.

## Quando não for elegível
Não sugerir upgrade pago. Registrar bloqueio e interromper caminho de deploy Vercel Hobby para aquele cenário. Se o uso for comercial, Hobby não é opção permitida; documentar incompatibilidade.

## Referências
- https://vercel.com/docs/git
- https://vercel.com/docs/git/vercel-for-github
- https://vercel.com/docs/plans/hobby
- https://vercel.com/docs/environment-variables
