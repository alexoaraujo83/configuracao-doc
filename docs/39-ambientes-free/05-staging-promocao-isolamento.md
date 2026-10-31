# 05 — Staging, promoção e isolamento

Vercel documenta Local, Preview e Production como ambientes padrão; Custom Environments não devem ser pressupostos no Hobby. Modelar Staging como branch Preview dedicada e overrides por branch quando disponíveis, registrando que não é ambiente customizado isolado.

Promoção: validar Preview, revisar migration, confirmar variáveis Production, aprovar merge, confirmar deployment, executar smoke tests e registrar commit/deployment.

Branch separada não equivale a conta separada: tenant Auth0, projeto Vercel e projeto Neon compartilham quotas e governança. Não usar Preview com dados reais. Vercel Hobby é pessoal/não comercial.