# 01 — Next.js App Router e TypeScript

**Estado:** Documentado; não executado.  
**Plano:** ferramentas locais gratuitas; deploy tratado no manual Vercel Hobby.

## 1. Pré-requisitos

- Node.js compatível com a versão escolhida do Next.js (a documentação atual de instalação indica Node.js 20.9 ou superior; usar versão LTS suportada).
- npm, Git e editor.
- Terminal e navegador atualizados.
- Acesso ao repositório GitHub.
- Não é necessário cartão, trial ou serviço pago para criar e executar localmente a aplicação.

Verificar versões:

```bash
node --version
npm --version
git --version
```

Registrar as versões no README do projeto. Não usar `latest` como garantia de reprodutibilidade: após criar a aplicação, manter `package-lock.json` versionado e usar `npm ci` em CI.

## 2. Criar a aplicação

```bash
npx create-next-app@latest sistema-web
cd sistema-web
```

Selecionar App Router, TypeScript, ESLint e diretório `src/`. Tailwind é opcional e não é requisito de arquitetura. Evitar ativar recursos experimentais sem decisão registrada.

Instalar dependências da aplicação:

```bash
npm install @auth0/nextjs-auth0 @neondatabase/serverless zod
```

Executar:

```bash
npm run dev
```

Abrir `http://localhost:3000`.

## 3. Estrutura de referência

```text
sistema-web/
├── src/
│   ├── app/
│   │   ├── api/
│   │   │   ├── health/route.ts
│   │   │   └── me/route.ts
│   │   ├── dashboard/page.tsx
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── components/
│   └── lib/
│       ├── auth0.ts
│       ├── db.ts
│       ├── env.ts
│       └── users.ts
├── database/migrations/
├── docs/
├── .env.example
├── .gitignore
├── package.json
├── package-lock.json
└── tsconfig.json
```

Regra: integração com Auth0 e Neon exclusivamente no servidor, em módulos de `src/lib`. Não importar esses módulos em Client Components.

## 4. Scripts e validação estática

Confirmar os scripts gerados em `package.json`. O conjunto mínimo esperado:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint .",
    "typecheck": "tsc --noEmit"
  }
}
```

Se o scaffold gerar comandos ESLint diferentes, manter os comandos reais e documentá-los. Não sobrescrever o `package.json` sem comparar com o scaffold instalado.

Executar:

```bash
npm ci
npm run typecheck
npm run lint
npm run build
```

O build deve terminar com código zero antes de abrir PR ou fazer deploy.

## 5. Variáveis de ambiente

Criar `.env.example` sem valores reais:

```dotenv
APP_BASE_URL=http://localhost:3000
AUTH0_DOMAIN=tenant.example.auth0.com
AUTH0_CLIENT_ID=replace-me
AUTH0_CLIENT_SECRET=replace-me
AUTH0_SECRET=replace-with-random-secret
DATABASE_URL=postgresql://user:password@host/db?sslmode=require
```

Criar `.env.local` com valores reais e mantê-lo fora do Git. Gerar segredo de sessão localmente:

```bash
node -e "console.log(require('node:crypto').randomBytes(32).toString('hex'))"
```

Regras:
- Nunca adicionar valores reais ao `.env.example`.
- Nunca usar `NEXT_PUBLIC_` para secrets, tokens ou connection strings.
- Usar segredos diferentes em Development, Preview e Production.
- Validar variáveis no servidor; não registrar seus valores nos logs.
- Reiniciar o servidor local após alterar variáveis.

## 6. Validação de ambiente

Exemplo de validação com Zod em `src/lib/env.ts`:

```typescript
import "server-only";
import { z } from "zod";

const schema = z.object({
  APP_BASE_URL: z.string().url(),
  AUTH0_DOMAIN: z.string().min(1),
  AUTH0_CLIENT_ID: z.string().min(1),
  AUTH0_CLIENT_SECRET: z.string().min(1),
  AUTH0_SECRET: z.string().min(32),
  DATABASE_URL: z.string().min(1),
});

export const env = schema.parse({
  APP_BASE_URL: process.env.APP_BASE_URL,
  AUTH0_DOMAIN: process.env.AUTH0_DOMAIN,
  AUTH0_CLIENT_ID: process.env.AUTH0_CLIENT_ID,
  AUTH0_CLIENT_SECRET: process.env.AUTH0_CLIENT_SECRET,
  AUTH0_SECRET: process.env.AUTH0_SECRET,
  DATABASE_URL: process.env.DATABASE_URL,
});
```

O pacote `server-only` é disponibilizado pelo ecossistema Next.js. A importação explícita evita que o módulo seja acidentalmente utilizado em código cliente. A validação deve ocorrer no servidor.

## 7. Padrões de implementação

- Server Components por padrão.
- Client Components apenas quando houver interação de navegador.
- Route Handlers para endpoints HTTP.
- No Next.js 16, usar `proxy.ts` com export `proxy`; `middleware.ts` é a convenção legada/depreciada. Proxy executa no runtime Node.js e é uma barreira de rede/otimização, não substitui autorização em DAL, Server Actions ou Route Handlers.
- Cada Server Action deve ser tratada como endpoint público: validar sessão, entrada, autorização e escopo do recurso dentro da própria action; não confiar em layout, UI ou Proxy como controle suficiente.
- Validar entrada no limite da aplicação (por exemplo, Zod).
- Não confiar em parâmetros enviados pelo cliente para identificar o usuário.
- Consultas SQL parametrizadas; não concatenar entrada em SQL.
- Retornar erros públicos genéricos e manter detalhes técnicos apenas em logs protegidos.
- Aplicar autorização junto ao acesso ao recurso, não somente em redirecionamentos.

## 8. Testes de aceite

| Verificação | Resultado esperado |
|---|---|
| `npm ci` | Dependências instaladas pelo lockfile |
| `npm run typecheck` | Sem erros TypeScript |
| `npm run lint` | Sem violações bloqueantes |
| `npm run build` | Build concluído |
| `/ ` | Página inicial responde |
| Variável obrigatória ausente | Falha controlada no servidor |
| Busca no bundle cliente | Nenhum segredo ou URL de banco |

## 9. Segurança

- Atualizar dependências com revisão; não aplicar upgrades principais automaticamente.
- Executar `npm audit` e avaliar cada alerta por impacto e correção.
- Manter lockfile e revisar diffs de dependências.
- Não incluir dumps, logs com PII, arquivos .env ou tokens no Git.
- Definir política de atualização e revisão periódica.

## 10. Referências

- [Instalação Next.js](https://nextjs.org/docs/app/getting-started/installation)
- [App Router](https://nextjs.org/docs/app)
- [Environment Variables](https://nextjs.org/docs/app/guides/environment-variables)
- [Server-only modules](https://nextjs.org/docs/app/getting-started/server-and-client-components)

