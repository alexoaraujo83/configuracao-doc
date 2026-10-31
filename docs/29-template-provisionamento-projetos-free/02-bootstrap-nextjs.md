# Bootstrap Next.js + TypeScript

## Configuração padrão inicial

### Pré-requisitos

- Node.js LTS compatível com a versão de Next.js escolhida.
- npm e Git.
- Editor e navegador atualizados.
- Repositório GitHub criado pelo responsável.
- Nenhuma credencial real necessária para iniciar o scaffold.

Registrar versões de Node.js, npm, Next.js, React e TypeScript. Não atualizar versões principais automaticamente sem avaliar compatibilidade com Auth0 SDK, Neon driver e Vercel.

### Criar a aplicação

~~~bash
npx create-next-app@latest nome-do-projeto
cd nome-do-projeto
npm run dev
~~~

Escolhas recomendadas no assistente:
- TypeScript: Yes.
- ESLint: Yes.
- App Router: Yes.
- src/ directory: conforme convenção.
- Import alias: @/*.
- Tailwind: decisão do projeto; não é requisito da integração.

Após gerar o scaffold, conferir package.json, scripts dev/build/start/lint, versionar lockfile npm, fixar Node por .nvmrc ou engines e criar .env.example com placeholders. Confirmar que .gitignore exclui arquivos .env reais.

### Estrutura recomendada

~~~text
nome-do-projeto/
├── .github/workflows/ci.yml
├── database/migrations/
├── public/
├── src/
│   ├── app/
│   │   ├── api/
│   │   ├── (public)/
│   │   └── (private)/
│   ├── lib/
│   │   ├── auth0.ts
│   │   ├── db.ts
│   │   ├── env.ts
│   │   └── authorization.ts
│   └── types/
├── .env.example
├── .gitignore
├── .nvmrc
├── package.json
├── package-lock.json
└── tsconfig.json
~~~

Ajustar a estrutura ao projeto. Route Groups não criam fronteiras de segurança; proteção deve ocorrer em cada operação server-side.

## Dependências

Instalar somente quando o projeto for efetivamente implementado e após confirmar versões compatíveis:

~~~bash
npm install @auth0/nextjs-auth0 @neondatabase/serverless
~~~

Não declarar que dependências estão instaladas apenas porque aparecem neste guia. Fixar versões por lockfile e revisar changelogs, Node compatibility e breaking changes.

## Configuração completa

- Server Components por padrão; Client Components apenas quando interação de browser exigir.
- Route Handlers para endpoints HTTP.
- Validação de entrada no servidor.
- Consultas parametrizadas.
- Separar módulos server-only e client-side.
- Não importar módulos de banco ou secrets em componentes cliente.
- Tratar erros sem expor stack trace, SQL, tokens ou connection strings.
- Definir timeouts, limites de payload e rate limiting de acordo com a capacidade disponível.
- Implementar autorização em cada recurso protegido.

## Auth0 SDK v4 e Next.js

Usar API atual do SDK v4, sem copiar exemplos legados do SDK v3. A integração de roteamento deve seguir a versão principal do Next.js instalada:
- Next.js 15: convenção middleware.ts.
- Next.js 16: convenção proxy.ts.

Não criar ambos por tentativa. Confirmar o guia de migração do Next.js e a versão exata do SDK antes de adotar o arquivo de entrada. O módulo Auth0 deve ser server-side e não expor secrets.

## Validação local

~~~bash
npm ci
npx tsc --noEmit
npm run lint
npm run build
~~~

Executar no ambiente de desenvolvimento do projeto; documentar versão, commit, data e resultado. Este documento não executa esses comandos.
