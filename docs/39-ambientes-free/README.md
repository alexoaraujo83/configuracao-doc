# Ambientes Development, Preview e Production — somente Free

Padrão documental não aplicado. Define matriz de configuração, isolamento de credenciais, URLs Auth0, branches Neon e Vercel.

## Matriz
- Development: local, dados sintéticos, Auth0 dev, Neon branch dev.
- Preview: validação de PR, secrets próprios, Neon branch isolada se quota permitir.
- Staging: branch Preview dedicada; não presumir custom environment no Hobby.
- Production: domínio estável, secrets exclusivos, Neon root e role runtime.

## Documentos
- 01 matriz de variáveis
- 02 Development local
- 03 Preview e branches
- 04 Production e Auth0
- 05 Staging, promoção e isolamento

Nenhuma configuração real foi alterada.

Fontes: https://vercel.com/docs/deployments/environments ; https://vercel.com/docs/environment-variables ; https://auth0.github.io/nextjs-auth0/ ; https://neon.com/docs/introduction/branching
Consulta: 31/10/2026.