# 04 — Production e Auth0

Production usa domínio canônico HTTPS, variáveis exclusivas, DATABASE_URL da branch principal com app_runtime e AUTH0_SECRET/client secret próprios.

Auth0 Allowed Callback URLs: domínio canônico + /auth/callback. Allowed Logout URLs: domínio canônico. Allowed Web Origins: domínio canônico somente quando necessário. Não cadastrar wildcards amplos.

O tenant Auth0 Free pode impor limite de tenants e recursos. Separar clients por ambiente quando elegível, sem presumir isolamento de tenant completo. Não depender de Organizations ou add-ons.