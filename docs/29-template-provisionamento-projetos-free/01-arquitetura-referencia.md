# Arquitetura de referência

## Visão lógica

~~~mermaid
flowchart TD
    U[Browser] -->|HTTPS| V[Vercel Hobby - pessoal e nao comercial]
    V --> N[Next.js App Router / Node.js]
    N -->|Authorization Code + OIDC| A[Auth0 Free tenant]
    A -->|Callback / sessao| N
    N -->|SQL parametrizado TLS| D[Neon PostgreSQL]
    G[GitHub Free repository] -->|PR / CI| V
~~~

O navegador não acessa diretamente Neon nem recebe client secret, connection string, tokens internos ou credenciais administrativas. O servidor Next.js valida a sessão e executa autorização antes de acessar dados.

## Responsabilidades por camada

| Camada | Responsabilidade | Não deve fazer |
|---|---|---|
| Browser | Interface e interação | Guardar segredo de servidor ou decidir autorização final |
| Next.js Server Components | Renderização e leitura autenticada | Expor variáveis privadas |
| Route Handlers / Server Actions | Validar sessão, autorização e entrada | Confiar em IDs/roles enviados pelo cliente |
| Auth0 | Autenticar identidade e emitir tokens | Substituir autorização de domínio |
| Neon | Persistir dados e aplicar privilégios/RLS | Ser acessado diretamente pelo browser |
| GitHub | Versionar, revisar e validar código | Armazenar secrets em arquivos versionados |
| Vercel | Executar build e hospedar aplicação | Ser usado comercialmente no Hobby |

## Fluxo de autenticação

1. Usuário inicia login no Next.js.
2. SDK Auth0 inicia Authorization Code Flow e redireciona ao tenant.
3. Auth0 autentica o usuário e retorna ao callback registrado.
4. SDK valida a resposta e gerencia sessão segura.
5. Camada server-side obtém a identidade autenticada.
6. Aplicação verifica permissões e escopo do recurso.
7. Somente então consulta Neon com credenciais privadas.

O identificador estável de identidade é o claim sub; email não deve ser chave primária de identidade.

## Fluxo de dados

- Consultas SQL parametrizadas.
- Validar autorização no servidor em cada operação protegida.
- Aplicar menor privilégio nas roles PostgreSQL.
- Em aplicações multi-tenant, aplicar filtros de domínio e RLS conforme arquitetura aprovada.
- Contexto de tenant/user definido pelo backend dentro da mesma transação não substitui proteção contra comprometimento do backend.
- Dados de CI e Preview devem ser sintéticos ou anonimizados.

## Estratégia de ambientes

| Ambiente | Código | Auth0 | Neon | Vercel |
|---|---|---|---|---|
| Development | branch local | aplicação/callback de desenvolvimento no tenant Free | branch/database de desenvolvimento, se quota permitir | local |
| Preview | PR branch | callback Preview registrado e validado | branch de teste isolada, se quota permitir | Preview |
| Staging | branch/release candidata | configuração dedicada ou callback aprovado | branch isolada, se quota permitir | Preview com variáveis escopadas, se suportado |
| Production | commit aprovado | configuração de produção | database/branch de produção | Production, somente se elegível ao Hobby |

Ambientes lógicos não garantem isolamento. Se não houver capacidade Free para bancos ou aplicações separados, documentar o risco e não simular isolamento apenas com nomes diferentes.

## Arquitetura multi-projeto

Auth0 Free pode impor tenant único; separar aplicações dentro do tenant não significa isolamento administrativo completo. Neon pode hospedar mais de uma branch, mas quotas e blast radius devem ser considerados. Não compartilhar automaticamente banco, roles, segredos ou dados entre projetos. Cada projeto deve ter registro, schema, permissões, variáveis e ciclo de release próprios.

## Decisões arquiteturais obrigatórias

Registrar finalidade e classificação dos dados; uso pessoal/não comercial para Vercel Hobby; estratégia de tenant e applications Auth0; topologia Neon, branches, roles e RLS; política de sessão e autorização; ambientes e isolamento; região e requisitos de latência/residência; limites Free observados e margem de capacidade; retenção, recuperação e risco residual.
