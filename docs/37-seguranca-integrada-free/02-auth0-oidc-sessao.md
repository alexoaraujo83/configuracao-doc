# 02 — Auth0, OIDC e sessão

## Aplicação Auth0

Para Next.js SSR, documentar uma **Regular Web Application** e usar o SDK oficial v4. Registrar callback, logout e web origins por ambiente, sem curingas amplos.

- Desenvolvimento: callback local /auth/callback e logout localhost.
- Preview: registrar URLs efetivamente utilizadas ou adotar estratégia dinâmica documentada pelo SDK; nunca confiar apenas no Host recebido.
- Produção: callback e logout no domínio canônico HTTPS.
- Não aceitar returnTo externo arbitrário. Preferir caminhos relativos e validar destinos.

## Fluxo

1. O usuário inicia login no Next.js.
2. SDK redireciona para Auth0 usando Authorization Code Flow.
3. Auth0 autentica e retorna ao callback allowlisted.
4. SDK valida state/nonce e troca o code conforme o protocolo.
5. SDK cria cookie de sessão protegido.
6. Requisições seguintes leem a sessão no servidor.
7. O backend deriva user sub da sessão e executa autorização do domínio.

Não implementar manualmente troca de tokens quando o SDK já gerencia o fluxo.

## Sessão e cookies

O SDK v4 fornece defaults de cookie seguros. Não substituir atributos sem requisito documentado. A configuração deve preservar:
- HttpOnly (o SDK mantém HttpOnly).
- Secure em HTTPS/produção.
- SameSite adequado ao fluxo OIDC; não definir Strict indiscriminadamente.
- Path restrito ao necessário.
- Chave AUTH0_SECRET forte, distinta entre ambientes.
- Expiração e renovação conforme política documentada e risco.

O cookie de sessão é uma credencial bearer: não o registrar, copiar para logs, disponibilizar ao JavaScript ou enviar a terceiros.

Rolling sessions do SDK podem adicionar Set-Cookie em respostas que leem sessão. Portanto, respostas que consultam sessão não podem ser cacheadas publicamente.

## Tokens

- ID Token: informação de autenticação do usuário para o cliente confiável.
- Access Token: autorização para acessar API/resource server; validar audience, issuer, assinatura, expiração e scopes no resource server.
- Refresh Token: só habilitar quando necessário e com política de rotação/revogação compatível; não armazenar no browser.
- Não usar ID Token como access token para APIs.
- Não aceitar token apenas decodificado; validar criptograficamente.

## Claims e identidade

- Identificador estável de usuário: claim sub.
- Email não é chave primária nem prova de identidade estável.
- Verificar email_verified quando a regra de negócio exigir email confirmado.
- Não confiar em role, tenant ou permission enviada pelo browser.
- Claims customizadas devem usar namespace próprio e ser minimizadas.

## Logout e revogação

Documentar logout local e do provedor conforme comportamento do SDK. Para incidente, definir procedimento de invalidação de sessões e credenciais; não presumir que rotacionar AUTH0_SECRET ou client secret seja operação sem impacto.

## Free-only

Não assumir Organizations, RBAC avançado, custom domains, MFA avançado ou recursos add-on como disponíveis. O domínio padrão Auth0 e autorização própria no banco são a base documentada. Verificar a matriz de preços e entitlement na data de uso.

## Referências

- https://auth0.github.io/nextjs-auth0/
- https://github.com/auth0/nextjs-auth0/blob/main/V4_MIGRATION_GUIDE.md
- https://github.com/auth0/nextjs-auth0/blob/main/SECURITY.md
- https://auth0.com/docs/secure/tokens
