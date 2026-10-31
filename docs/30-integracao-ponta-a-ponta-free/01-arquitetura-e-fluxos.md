# 1. Arquitetura e fluxos

## Fronteiras de confiança

| Componente | Responsabilidade | Regra |
|---|---|---|
| Navegador | Interface | Não é fonte confiável de identidade ou autorização |
| Next.js | BFF, páginas, Route Handlers e Server Actions | Validar sessão e autorizar cada operação |
| Auth0 | Autenticação e sessão | Identidade não equivale a permissão de negócio |
| Neon | PostgreSQL | Acesso server-side com papel runtime |
| GitHub | Código, PR e CI | Segredos mínimos e revisão |
| Vercel | Build e hospedagem | Variáveis segregadas; Hobby não comercial |

~~~mermaid
flowchart LR
 B[Navegador] -->|HTTPS| N[Next.js]
 N <-->|Login e sessão| A[Auth0]
 N -->|SQL parametrizado| D[(Neon)]
 G[GitHub] -->|PR e Actions| V[Vercel]
 V --> N
~~~

## Login
1. Visitante acessa rota protegida.
2. Next.js verifica a sessão pelo SDK Auth0 no servidor.
3. Sem sessão válida, inicia o login pelo mecanismo oficial do SDK.
4. Auth0 autentica e redireciona à callback cadastrada.
5. SDK processa callback e estabelece sessão protegida.
6. Aplicação lê a identidade no servidor e usa o claim sub como identificador externo.
7. Banco resolve usuário interno, membership, tenant e permissões.
8. Operação só é executada após autorização.

## Operação autenticada

~~~mermaid
sequenceDiagram
 participant U as Usuário
 participant N as Next.js
 participant A as Auth0
 participant D as Neon
 U->>N: Requisição
 N->>A: Ler sessão pelo SDK
 A-->>N: Identidade validada
 N->>D: Consultar membership
 D-->>N: Contexto de domínio
 N->>N: Autorizar ação e tenant
 N->>D: SQL parametrizado
 D-->>N: Resultado
 N-->>U: Resposta mínima
~~~

## Identidade e domínio
Persistir issuer + sub com unicidade composta quando houver múltiplos issuers. Não usar e-mail como chave primária. Primeiro login pode criar perfil sem privilégios; membership e papéis são concedidos por fluxo administrativo separado e auditável.

RBAC do Auth0 não substitui membership e autorização do domínio. Toda operação deve verificar usuário ativo, membership ativa, tenant do recurso e permissão específica.

## Ambientes
| Ambiente | Auth0 | Neon | Aplicação |
|---|---|---|---|
| Local | URLs de desenvolvimento | banco/branch dev | localhost |
| Preview | callback explicitamente autorizada | banco isolado e dados sintéticos | preview |
| Produção pessoal | URLs definitivas | banco de produção | Hobby, não comercial |

Não compartilhar banco de produção com preview por padrão. Não expor tokens ao JavaScript do navegador. Não usar ID token como credencial genérica de API.
