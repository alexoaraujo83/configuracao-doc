# 1. Preparar estação e criar projeto Next.js

## Pré-requisitos
- Node.js 20 LTS ou versão LTS mais recente compatível com Next.js e Auth0 SDK.
- npm e Git instalados.
- Editor e navegador atualizados.
- Acesso aos painéis, caso a etapa de configuração seja posteriormente autorizada.

A documentação do Auth0 SDK v4 exige Node.js 20 LTS ou superior. Respeitar o requisito mais restritivo entre framework e dependências.

## Verificar ferramentas
~~~bash
node --version
npm --version
git --version
~~~

Registrar versões no inventário do projeto. Fixar versão Node suportada por .nvmrc ou mecanismo equivalente.

## Criar aplicação
~~~bash
npx create-next-app@latest sistema-web
cd sistema-web
~~~

No assistente, selecionar TypeScript, ESLint, App Router, src directory e alias @/*. Tailwind é opcional conforme o padrão visual. Opções podem mudar por versão; revisar o resultado gerado.

## Dependências
~~~bash
npm install @auth0/nextjs-auth0 @neondatabase/serverless zod
~~~

- Auth0 SDK: autenticação e sessão.
- Neon driver: PostgreSQL serverless.
- Zod: validação de ambiente e entrada.

## Estrutura de referência
~~~text
sistema-web/
├── src/
│   ├── app/
│   ├── components/
│   └── lib/
│       ├── auth0.ts
│       ├── db.ts
│       ├── env.ts
│       └── security/
├── database/migrations/
├── docs/
├── .env.example
├── .nvmrc
├── package.json
└── README.md
~~~

## Ambiente e Git
Garantir que .env*, exceto .env.example, .next/, node_modules/ e .vercel/ estejam no .gitignore. O .env.example deve conter apenas nomes e placeholders.

## Validação local
Confirmar scripts no package.json antes de executar:
~~~bash
npm ci
npm run lint
npx tsc --noEmit
npm run build
~~~

Não mascarar falhas. Se lint não estiver configurado, adicionar configuração compatível com a versão do Next.js.

## Critério de conclusão
Versões registradas, projeto App Router/TypeScript gerado, dependências no lockfile, arquivos locais ignorados e nenhum segredo versionado.
