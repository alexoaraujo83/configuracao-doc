# Proteção contra acesso cruzado entre aplicações

## 1. Fronteiras por aplicação
Documentar slug, repositório/owners, projeto Vercel/domínios, Auth0 client/URLs, audience/API/scopes, projeto Neon/schema, roles/credenciais, classificação de dados e responsável.

## 2. Padrão recomendado
- Client Auth0 separado por aplicação; nunca compartilhar client secret.
- Callback/logout/web origins exatos, sem wildcards amplos.
- Audience própria por API distinta; validar audience e scopes no servidor.
- Projeto Neon separado por domínio de dados quando ownership, classificação, ciclo de vida ou blast radius divergirem.
- Schema separado não é fronteira forte se grants/credenciais cruzam esquemas.
- Credenciais runtime distintas e grants mínimos.
- Cookies de sessão não devem permitir leitura por aplicações irmãs.
- Não reutilizar token entre APIs sem audience correta.

## 3. Auth0 tenant único no Free
Tenant único cria fronteira administrativa compartilhada. Clients separados segmentam callback, logout e credenciais, mas não separam tenant, usuários, conexões, políticas globais ou blast radius. Se o requisito exige tenant isolado por aplicação/cliente, marcar BLOQUEADO NO FREE; não simular isolamento com nomes diferentes de clients.

## 4. Quando compartilhar banco
Aceitar somente se domínios e owners forem comuns; grants e policies forem explícitos; credenciais não acessarem tabelas de outras apps; funções/views privilegiadas forem revisadas; testes negativos cross-app passarem; restore e ciclo de vida compartilhados forem aceitáveis.

Separar projetos Neon quando dados/classificação diferirem, times/terceiros tiverem acessos diferentes, restore independente for necessário, compromisso de uma app não puder expor outra ou quotas permitirem a separação.

## 5. Prevenir IDOR e confused deputy
- Não aceitar tenant/app owner do browser como autoridade.
- Resolver tenant permitido por membership validado no servidor.
- Consultar recurso por ID + tenant_id, nunca por ID isolado.
- Validar que recursos relacionados pertencem ao mesmo tenant.
- Revalidar autorização na operação final.
- Ações privilegiadas exigem controles de step-up se disponíveis/elegíveis; não declarar implementado se não disponível no Free.
- Evitar endpoint genérico com credencial privilegiada sem validar o principal.

## 6. Matriz

| Dimensão | Controle mínimo | Evidência |
|---|---|---|
| Identidade | client por app e sessão server-side | callback e login testados |
| Token | audience/scope corretos | token inválido rejeitado |
| API | RBAC e membership | testes 401/403 |
| Banco | roles, GRANT e RLS | testes como runtime |
| Deploy | projeto/env vars por app | painel Vercel |
| Código | repo/owners/permissões | GitHub |
| Logs | sem secrets/PII desnecessária | teste de redaction |

## 7. Bloqueios
NO-GO se uma app lê dados de outra por credencial compartilhada, audience não é validada, callback é amplo, runtime bypassa RLS, não há testes negativos ou requisito de isolamento físico não é atendido.
