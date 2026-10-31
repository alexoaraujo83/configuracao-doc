# 06 — Segredos, headers e logging

## Variáveis de ambiente

Segredos server-only:
- AUTH0_CLIENT_SECRET
- AUTH0_SECRET
- DATABASE_URL
- chaves de assinatura, webhook e integração, quando existirem.

Regras:
- Nunca prefixar secrets com NEXT_PUBLIC_.
- Nunca incluir valores reais em .env.example, README, workflow ou issue.
- .env.local fica fora do Git.
- Separar valores por Development, Preview e Production.
- Preview nunca utiliza role/database de produção.
- Não compartilhar AUTH0_SECRET entre ambientes.
- Limitar acesso às variáveis no painel Vercel.
- Em caso de exposição, revogar/rotacionar no provedor e avaliar sessões, conexões e logs afetados.

## Headers

Configurar e testar conforme a aplicação:
- Strict-Transport-Security em produção HTTPS, após confirmar domínio e subdomínios.
- Content-Security-Policy compatível com Next.js e Auth0; evitar unsafe-eval/unsafe-inline sem análise.
- X-Content-Type-Options: nosniff.
- Referrer-Policy restritiva.
- frame-ancestors via CSP para reduzir clickjacking.
- Permissions-Policy mínima.

Não copiar uma CSP genérica sem testar scripts, nonce, callback Auth0 e assets. Headers não substituem escaping e prevenção de XSS.

## Logs

Registrar:
- request_id/correlation_id.
- resultado de autenticação/autorização (sem tokens).
- ação, tipo de recurso e tenant interno quando permitido.
- códigos de erro e latência.
- eventos administrativos e mudanças de membership.

Não registrar:
- cookies, Authorization header, access/refresh/ID tokens.
- client secret, AUTH0_SECRET, DATABASE_URL.
- payloads completos com dados pessoais.
- SQL contendo parâmetros sensíveis.
- códigos OAuth ou query strings sensíveis.

Definir retenção mínima necessária e acesso restrito aos logs. Free tier pode ter retenção e capacidade limitadas; não assumir exportação, alertas avançados ou retenção longa sem confirmar disponibilidade Free.

## Rate limiting

Aplicar limites por usuário autenticado e por IP para endpoints sensíveis, com limites diferenciados por operação. Não depender de memória local de função serverless como contador global confiável. Se não houver armazenamento compartilhado incluído no Free, documentar limitação e adotar proteção no nível de negócio sem inventar serviço pago.

## Cache

Respostas autenticadas ou personalizadas devem usar private, no-store. Não compartilhar resposta que depende de sessão, usuário ou tenant via CDN. Rever também cache de dados, fetch e headers de deployment.

## Referências

- https://nextjs.org/docs/app/guides/data-security
- https://vercel.com/docs/headers
- https://github.com/auth0/nextjs-auth0/blob/main/SECURITY.md
- https://owasp.org/www-project-secure-headers/
