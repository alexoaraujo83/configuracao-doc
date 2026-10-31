# 03 — Preview e branches

Vercel cria Preview deployments para branches e PRs conforme integração Git. Variáveis Preview podem ser globais ou específicas por branch, conforme disponibilidade do plano.

Neon: usar branch não produtiva e role runtime; criar branches efêmeras somente se quota Free permitir; preferir dados sintéticos/schema-only; documentar expiração e limpeza.

Auth0: registrar callback de Preview específica ou adotar inferência dinâmica suportada pelo SDK com allowlist segura. Não usar wildcard amplo. Se host efêmero não puder ser allowlisted, desabilitar login em previews ou usar preview estável aprovado.

Preview não deve usar DATABASE_URL, AUTH0_SECRET ou client secret de produção.