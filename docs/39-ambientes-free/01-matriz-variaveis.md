# 01 — Matriz de variáveis

| Variável | Development | Preview | Production |
|---|---|---|---|
| APP_BASE_URL | localhost ou inferência | URL preview dinâmica/allowlisted | domínio HTTPS canônico |
| AUTH0_DOMAIN | tenant Free | tenant Free | tenant Free |
| AUTH0_CLIENT_ID | client dev | client preview | client prod |
| AUTH0_CLIENT_SECRET | segredo dev | segredo preview | segredo prod |
| AUTH0_SECRET | exclusivo dev | exclusivo preview | exclusivo prod |
| DATABASE_URL | Neon branch dev | branch preview | branch principal |

NODE_ENV é controlado pelo framework/runtime. Não compartilhar secrets nem connection strings entre ambientes. Nunca usar DATABASE_URL de produção no Preview. Não colocar secrets em NEXT_PUBLIC_.

Modelo de .env.example sem valores:

~~~env
APP_BASE_URL=
AUTH0_DOMAIN=
AUTH0_CLIENT_ID=
AUTH0_CLIENT_SECRET=
AUTH0_SECRET=
DATABASE_URL=
~~~
