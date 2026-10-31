# 01 — Mapa de endpoints e contratos

Todos os caminhos abaixo são propostas de contrato, não rotas já criadas.

| Método | Caminho | Finalidade | Permissão |
|---|---|---|---|
| GET | /api/v1/me | identidade e perfil próprio | autenticado |
| PATCH | /api/v1/me | atualizar perfil editável | próprio perfil |
| GET | /api/v1/tenants | listar tenants do usuário | autenticado |
| GET | /api/v1/tenants/{tenantId}/members | listar membros | members:read |
| POST | /api/v1/tenants/{tenantId}/members/invitations | convidar membro | members:manage |
| PATCH | /api/v1/tenants/{tenantId}/members/{userId} | alterar papel/status | members:manage |
| GET | /api/v1/tenants/{tenantId}/projects | listar projetos | projects:read |
| POST | /api/v1/tenants/{tenantId}/projects | criar projeto | projects:create |
| GET | /api/v1/tenants/{tenantId}/projects/{projectId} | obter projeto | projects:read |
| PATCH | /api/v1/tenants/{tenantId}/projects/{projectId} | atualizar projeto | projects:update |
| DELETE | /api/v1/tenants/{tenantId}/projects/{projectId} | excluir projeto | projects:delete |

## Semântica

- GET não produz efeitos colaterais.
- POST cria recurso e responde 201 + Location.
- PATCH altera somente campos permitidos e responde 200 com DTO atualizado.
- DELETE pode responder 204 sem corpo; escolher exclusão lógica/física na política do domínio.
- PUT não é usado enquanto não houver contrato de substituição integral.
- /me representa usuário da sessão, nunca recebe userId no path.
- Rotas de convite dependem de política de convite definida. Não implementar envio de e-mail até documentar provedor e limites Free.

## Query parameters

Listagens aceitam limit, cursor e filtros explicitamente documentados. Parâmetros desconhecidos devem ser rejeitados ou ignorados de modo consistente; a recomendação é rejeitar para detectar erro de cliente. Ordenação só por allowlist.

## Headers

Request: Content-Type: application/json para mutações; Accept: application/json. Resposta: Content-Type, Cache-Control: private, no-store para dados pessoais/tenant-scoped, X-Request-Id para correlação. Não aceitar tenant context em header customizado como fonte de autorização.

## Auth

Sessão web Auth0 via cookie HttpOnly. Requisições sem sessão válida recebem 401. A sessão válida não concede acesso automático a tenant ou recurso.
