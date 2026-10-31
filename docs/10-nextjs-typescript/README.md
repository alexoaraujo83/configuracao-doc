# Next.js + TypeScript — padrão de implementação

> Revisão documental: 31/10/2026. Stack de referência: Next.js App Router, TypeScript, Node.js e npm. Este guia descreve procedimentos; não instala nem altera uma aplicação real.

## 1. Objetivo e limites

Definir uma base reutilizável para aplicações web com apenas recursos dos planos Free. Nenhum passo depende de upgrade, trial, crédito promocional ou add-on pago. Onde o plano não atender a um requisito, registrar o cenário como BLOCKED.

A base tecnológica usa:

- Next.js App Router.
- TypeScript em modo strict.
- React Server Components por padrão.
- Client Components somente para interação que exige estado/eventos do navegador.
- Route Handlers para endpoints HTTP.
- Auth0 para autenticação.
- Neon PostgreSQL para persistência.
- npm e lockfile versionado.

**Não faz parte deste escopo:** executar `create-next-app`, instalar dependências, criar tenant/projeto, configurar DNS, conectar Vercel ou publicar deployment. Esses comandos são documentados para execução posterior por um operador autorizado.

## 2. Pré-requisitos

| Ferramenta | Padrão | Observação |
|---|---|---|
| Node.js | 24 LTS, se compatível com a versão escolhida de Next.js | Next.js exige no mínimo Node.js 20.9 |
| npm | versão fornecida com Node LTS | versionar `package-lock.json` |
| Git | versão estável suportada | usar branch e PR |
| Editor | VS Code ou equivalente | TypeScript language service |
| PostgreSQL client | `psql` | somente para migrations/administração |
| Browser | navegador moderno | validar cookies e redirects |

A documentação oficial do Next.js informa Node.js mínimo 20.9 e TypeScript mínimo 5.1. O SDK Auth0 para Next.js v4 exige Node.js 20 LTS ou superior. Fixar uma versão LTS compatível e usar a mesma major version em desenvolvimento, CI e hospedagem. Não usar `latest` como versão permanente de produção sem revisão. Fontes: [Next.js Installation](https://nextjs.org/docs/app/getting-started/installation) e [Auth0 Next.js SDK](https://auth0.github.io/nextjs-auth0/). 

## 3. Criar uma aplicação de referência

Comandos documentados para execução posterior:

```bash
npx create-next-app@latest sistema-web
cd sistema-web
```

No assistente, selecionar:

- TypeScript: Yes.
- Linter: ESLint.
- React Compiler: decisão explícita do projeto; baseline pode manter No.
- Tailwind CSS: opcional; não é requisito de arquitetura.
- `src/` directory: Yes.
- App Router: Yes.
- Import alias: `@/*`.
- Package manager: npm.

Executar localmente:

```bash
npm run dev
```

A URL local padrão é `http://localhost:3000`. A configuração recomendada do create-next-app habilita TypeScript, ESLint, App Router e alias `@/*`. Não copiar a lista de prompts como se fosse uma versão fixa: o assistente pode mudar entre versões.

## 4. Dependências da aplicação

Pacotes de runtime propostos:

```bash
npm install @auth0/nextjs-auth0 @neondatabase/serverless zod
```

Responsabilidades:

| Pacote | Responsabilidade |
|---|---|
| `next`, `react`, `react-dom` | framework e interface |
| `typescript` | verificação estática |
| `@auth0/nextjs-auth0` | sessão e fluxos OIDC |
| `@neondatabase/serverless` | SQL/PostgreSQL no ambiente serverless |
| `zod` | validação de entrada e configuração |

Usar versões resolvidas e compatíveis, salvar o lockfile e executar `npm ci` em CI. Não adicionar ORM até existir uma decisão documentada que justifique sua abstração e custo operacional.

## 5. Estrutura de diretórios de referência

```text
sistema-web/
├── .github/
│   └── workflows/
│       └── ci.yml
├── database/
│   └── migrations/
│       ├── 001_roles_and_schema.sql
│       ├── 002_users_and_rls.sql
│       └── README.md
├── public/
├── src/
│   ├── app/
│   │   ├── api/
│   │   │   ├── health/route.ts
│   │   │   └── me/route.ts
│   │   ├── dashboard/page.tsx
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── components/
│   ├── lib/
│   │   ├── auth0.ts
│   │   ├── db.ts
│   │   ├── env.ts
│   │   ├── repositories/
│   │   └── security/
│   └── types/
├── .env.example
├── .gitignore
├── eslint.config.mjs
├── next.config.ts
├── package.json
├── package-lock.json
├── tsconfig.json
└── README.md
```

Regras:

1. `src/app` contém rotas, layouts e Route Handlers.
2. `src/lib` centraliza integração com Auth0, Neon, validação e segurança.
3. `src/components` contém interface; não importa drivers, secrets ou repositórios de banco.
4. `database/migrations` contém SQL versionado, imutável depois de aplicado.
5. `.env.example` contém apenas nomes e exemplos não secretos.
6. `.env.local` nunca é versionado.
7. A aplicação não acessa Neon diretamente do browser.

## 6. TypeScript strict

Manter `strict: true` no `tsconfig.json`. Configuração ilustrativa compatível com o baseline:

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": false,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "react-jsx",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": { "@/*": ["./src/*"] }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

Antes de adotar, comparar com o `tsconfig.json` gerado pela versão instalada de Next.js. Não substituir automaticamente opções específicas do framework.

Scripts recomendados no `package.json`:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "typecheck": "tsc --noEmit",
    "lint": "eslint ."
  }
}
```

A partir do Next.js 16, `next build` não executa lint automaticamente. O CI deve chamar lint e typecheck explicitamente.

## 7. Server Components, Client Components e Route Handlers

- Server Components são o padrão. Utilizá-los para carregar dados privados no servidor e compor páginas.
- Client Components recebem apenas dados necessários à interface; nunca recebem secrets, connection strings ou tokens de servidor.
- Route Handlers ficam em `src/app/api/**/route.ts`; validar método, autenticação, autorização, entrada e formato da resposta.
- Server Actions devem ser tratadas como endpoints públicos: autenticar, autorizar, validar input e impedir mutações cross-tenant.
- Não considerar o fato de um componente ser Server Component como autorização automática.

## 8. Variáveis de ambiente

Modelo de nomes:

```dotenv
# URL pública da aplicação; valor local ilustrativo
APP_BASE_URL=http://localhost:3000

# Auth0 — valores fornecidos pelo tenant
AUTH0_DOMAIN=
AUTH0_CLIENT_ID=
AUTH0_CLIENT_SECRET=
AUTH0_SECRET=

# Neon — conexão pooled para runtime
DATABASE_URL=

# Neon — conexão direta para migrations administrativas
DATABASE_URL_MIGRATION=
```

Regras:

- Não prefixar secrets com `NEXT_PUBLIC_`.
- Não importar `DATABASE_URL` em módulos Client Components.
- `DATABASE_URL` usa conexão pooled do Neon para requisições de runtime.
- `DATABASE_URL_MIGRATION` usa conexão direta, privilégio de migration e fica disponível somente ao operador/CI autorizado quando necessário.
- Separar valores por ambiente. Nunca compartilhar senha ou segredo entre development, preview e production.
- Em Preview dinâmico, não definir um `APP_BASE_URL` estático apontando para produção; seguir o guia Auth0 e registrar somente hosts de callback aprovados.

## 9. Validação de ambiente no servidor

Exemplo `src/lib/env.ts`:

```typescript
import "server-only";
import { z } from "zod";

const schema = z.object({
  APP_BASE_URL: z.string().url().optional(),
  AUTH0_DOMAIN: z.string().min(1),
  AUTH0_CLIENT_ID: z.string().min(1),
  AUTH0_CLIENT_SECRET: z.string().min(1),
  AUTH0_SECRET: z.string().regex(/^[a-fA-F0-9]{64}$/),
  DATABASE_URL: z.string().min(1),
});

export const env = schema.parse({
  APP_BASE_URL: process.env.APP_BASE_URL || undefined,
  AUTH0_DOMAIN: process.env.AUTH0_DOMAIN,
  AUTH0_CLIENT_ID: process.env.AUTH0_CLIENT_ID,
  AUTH0_CLIENT_SECRET: process.env.AUTH0_CLIENT_SECRET,
  AUTH0_SECRET: process.env.AUTH0_SECRET,
  DATABASE_URL: process.env.DATABASE_URL,
});
```

`openssl rand -hex 32` gera 32 bytes codificados em hexadecimal para `AUTH0_SECRET`. Gerar um valor independente para cada ambiente. Nunca imprimir valores de ambiente em logs ou respostas HTTP.

## 10. Testes e critérios de aceite

| Teste | Critério |
|---|---|
| `npm ci` | instala a partir do lockfile sem resolver versões novas |
| `npm run typecheck` | zero erros TypeScript |
| `npm run lint` | zero erros de lint |
| `npm run build` | build concluído |
| Inspeção do bundle | nenhum secret ou driver DB no bundle cliente |
| Busca por env real | nenhum `.env.local` ou secret no Git |
| Navegação | rotas públicas e privadas comportam-se conforme política |
| Erro de configuração | falha controlada sem expor valor sensível |

## 11. Checklist de configuração completa

- [ ] Node LTS compatível fixado e documentado.
- [ ] Next.js App Router e TypeScript strict.
- [ ] Lockfile versionado e npm ci no CI.
- [ ] ESLint e typecheck separados do build.
- [ ] Variáveis validadas no servidor.
- [ ] Secrets isolados por ambiente.
- [ ] Server/Client Components com fronteiras explícitas.
- [ ] Route Handlers e Server Actions com autorização.
- [ ] Banco acessado exclusivamente no servidor.
- [ ] Dependências revisadas e sem versões principais atualizadas automaticamente.

## 12. Fontes oficiais

- [Next.js Installation](https://nextjs.org/docs/app/getting-started/installation)
- [Next.js App Router](https://nextjs.org/docs/app)
- [Next.js Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components)
- [Next.js Route Handlers](https://nextjs.org/docs/app/getting-started/route-handlers)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Node.js Releases](https://nodejs.org/en/about/previous-releases)
- [Auth0 Next.js SDK](https://auth0.github.io/nextjs-auth0/)
