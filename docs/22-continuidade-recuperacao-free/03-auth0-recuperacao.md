# Auth0 — continuidade, exportação e recuperação

## Capacidades e limites
Auth0 Public Cloud não fornece backup/restore gerenciado do tenant ao cliente. O owner/admin mantém export próprio. Auth0 Deploy CLI exporta configuração, não usuários. Exportação/importação de usuários via Management API tem limitações; password hashes de usuários Auth0 Database não são exportados normalmente pela API/extension. Não prometer recuperação transparente de senha.

## Export de configuração
- Usar Auth0 Deploy CLI em ambiente controlado.
- Excluir recursos não elegíveis ao Free; a documentação CLI cita logStreams e customDomains em AUTH0_EXCLUDED para Free.
- AUTH0_EXPORT_SECRETS=false; não exportar segredos reais para Git.
- AUTH0_ALLOW_DELETE=false em operações normais; revisar dry-run/diff.
- Marketplace Actions, billing, custom domains e algumas configurações podem exigir tratamento manual.
- Export completo não significa clone funcional do tenant.

Exemplo:

    a0deploy export --config_file config.json --format yaml --output_folder ./auth0-export

Credenciais devem ser variáveis temporárias seguras; nunca commitar config com credenciais.

## Inventário não secreto
Tenant domain/region/owner; Applications e callback/logout/origins; APIs/audience/scopes; Connections; Universal Login; Actions/triggers; nomes de secrets e custodiante; RBAC/permissions/claims; políticas de sessão/refresh/MFA elegíveis; email provider e integrações não exportadas.

## Recuperação
1. Recuperar email e identidade do administrador primeiro.
2. Confirmar tenant e owner; não criar substituto automaticamente.
3. Se tenant existe, aplicar configuração revisada sem exclusões.
4. Se tenant perdido, seguir processo oficial; não assumir usuários/password hashes recuperáveis.
5. Recriar clients/APIs somente com aprovação e inventário; gerar novos secrets.
6. Atualizar Vercel env, issuer, audience e callbacks após confirmar domínio.
7. Testar login/logout/refresh, audience, scopes, RBAC e membership.
8. Avaliar tokens/sessões antigos; JWT access token emitido pode permanecer válido até expirar.

Se o projeto depende de password hashes não exportáveis, documentar reset/re-enrollment. Considerar tenant único e limites de Applications Free conforme matriz vigente.

## Referências
- [Auth0 backup/export tenant and users](https://support.auth0.com/center/s/article/Backup-Auth0-Data)
- [Auth0 Deploy CLI configuration](https://github.com/auth0/auth0-deploy-cli/blob/master/docs/configuring-the-deploy-cli.md)
- [Auth0 Deploy CLI export/import](https://github.com/auth0/auth0-deploy-cli/blob/master/docs/using-as-cli.md)
