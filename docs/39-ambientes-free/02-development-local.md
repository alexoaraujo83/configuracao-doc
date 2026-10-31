# 02 — Development local

Pré-requisitos: Node.js LTS compatível, npm, Auth0 Regular Web Application de desenvolvimento, Neon branch dev e role app_runtime.

1. Configurar APP_BASE_URL=http://localhost:3000.
2. Registrar callback http://localhost:3000/auth/callback no Auth0.
3. Registrar logout URL http://localhost:3000.
4. Gerar AUTH0_SECRET exclusivo para Development.
5. Obter DATABASE_URL da branch dev e role runtime.
6. Manter .env.local no gitignore.
7. Usar dados sintéticos.

Validar login/logout, cookie HttpOnly, ausência de secrets no bundle/logs e conexão exclusivamente à branch dev. Procedimento não executado.