# Exemplos de código de referência

Fragmentos didáticos. Conferir as versões instaladas de Next.js, Auth0 SDK e Neon Driver antes de copiar. Não são uma aplicação completa compilável.

## Auth0 server-only — SDK v4

~~~ts
// src/server/auth/auth0.ts
import "server-only";
import { Auth0Client } from "@auth0/nextjs-auth0/server";

export const auth0 = new Auth0Client();
~~~

Não usar o padrão legado v3 handleAuth ou /api/auth/[...auth0]. Next.js 15 usa middleware.ts; Next.js 16 usa proxy.ts, conforme compatibilidade do SDK e versão efetiva.

## Identidade no servidor

~~~ts
// src/server/auth/current-user.ts
import "server-only";
import { auth0 } from "./auth0";

export async function getCurrentIdentity() {
  const session = await auth0.getSession();
  if (!session?.user?.sub) return null;

  return {
    subject: session.user.sub,
    issuer: session.user.iss,
  };
}
~~~

A identidade externa estável é issuer + subject. Confirmar as propriedades do tipo User na versão instalada; se iss não estiver exposto, obter issuer de configuração confiável do tenant, nunca do browser. Não usar e-mail como chave primária.

## Guard de autenticação

~~~ts
// src/server/auth/require-user.ts
import "server-only";
import { redirect } from "next/navigation";
import { getCurrentIdentity } from "./current-user";

export async function requireIdentity() {
  const identity = await getCurrentIdentity();
  if (!identity) redirect("/");
  return identity;
}
~~~

Esse guard resolve autenticação, não autorização de recurso. Cada caso de uso deve verificar permissão e tenant.

## Cliente Neon server-only

~~~ts
// src/server/db/neon.ts
import "server-only";
import { neon } from "@neondatabase/serverless";

const connectionString = process.env.DATABASE_URL;
if (!connectionString) throw new Error("DATABASE_URL não configurada");

export const sql = neon(connectionString);
~~~

## Consulta parametrizada

~~~ts
const rows = await sql(
  "select id, name from projects where owner_sub = $1 order by created_at desc",
  [subject],
);
~~~

Este formato ilustra parâmetros posicionais. Confirmar a assinatura suportada pela versão instalada do driver. Não concatenar entrada externa para montar SQL. Filtrar por owner não substitui autorização nem RLS.

## Route Handler — composição

~~~ts
// src/app/api/projects/route.ts
import { NextResponse } from "next/server";
import { requireIdentity } from "@/server/auth/require-user";
import { listProjects } from "@/features/projects/queries/list-projects";

export async function GET() {
  const identity = await requireIdentity();
  const projects = await listProjects(identity);
  return NextResponse.json({ data: projects });
}
~~~

O exemplo omite tratamento completo. Em produção, mapear erros de validação, autenticação, autorização e falhas internas para status apropriados. Não capturar e converter indiscriminadamente exceções de controle do Next.js (por exemplo, redirect) em 500.

## Contrato de domínio

~~~ts
export type ProjectSummary = {
  id: string;
  name: string;
};

export interface ProjectRepository {
  listForOwner(subject: string): Promise<ProjectSummary[]>;
}
~~~

O domínio declara o contrato; um adaptador em src/server/db/repositories implementa o acesso PostgreSQL. A UI não conhece o driver.

## Caso de uso

~~~ts
import "server-only";
import { projectRepository } from "@/server/db/repositories/project-repository";

type Identity = { issuer?: string; subject: string };

export async function listProjects(identity: Identity) {
  if (!identity.subject) throw new Error("UNAUTHENTICATED");
  return projectRepository.listForOwner(identity.subject);
}
~~~

Em sistema multi-tenant, resolver e validar a associação do usuário ao tenant e a permissão da operação. Não confiar em tenantId enviado pelo cliente.

## Variáveis de ambiente — nomes somente

~~~dotenv
APP_BASE_URL=
AUTH0_DOMAIN=
AUTH0_CLIENT_ID=
AUTH0_CLIENT_SECRET=
AUTH0_SECRET=
DATABASE_URL=
~~~

AUTH0_CLIENT_SECRET, AUTH0_SECRET e DATABASE_URL são segredos. Não podem ser enviados ao browser nem prefixados com NEXT_PUBLIC_. Valores locais ficam em arquivo ignorado pelo Git; valores remotos no mecanismo seguro do fornecedor.
