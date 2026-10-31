# Auth0: monitoramento de autenticação, erros e sessões

## Fontes de sinal

No Auth0 Dashboard, consultar Monitoring > Logs. Eventos de tenant podem incluir autenticações bem-sucedidas e malsucedidas, alterações de senha, ações administrativas e eventos de segurança. A disponibilidade e a nomenclatura dos eventos variam conforme o fluxo e o recurso usado.

## Indicadores operacionais

- logins bem-sucedidos e falhos por intervalo;
- aumento de falhas por client/application;
- eventos de bloqueio, rate limit, password leak e credential stuffing quando disponíveis;
- falhas de callback, redirect URI e troca de authorization code;
- falhas de refresh token e erros de sessão;
- mudanças administrativas, alterações de connection e configuração de application/API;
- aumento inesperado de eventos de MFA ou consentimento, quando aplicável.

Não interpretar todo erro de login como ataque: separar erro de credencial, usuário inexistente, callback inválido, indisponibilidade de IdP externo e bloqueios de proteção.

## Consultas

Usar os filtros do Dashboard e a sintaxe de busca de logs do Auth0. Para análise de incidente, restringir a janela temporal e filtrar pelo tipo de evento, client ID e identificador de request disponível. Não copiar logs integrais para tickets públicos.

A Management API Search log events pode ser usada em exportações moderadas, respeitando paginação, rate limits e retenção. A documentação de suporte alerta que consultas grandes podem sofrer limitações de paginação e recomenda dividir por faixas de data. Não criar polling frequente desnecessário.

## Retenção e streaming no Free

A página oficial de preços Auth0 consultada em 2026-10-31 informa:
- Free: retenção de logs por 1 dia;
- Essentials: 5 dias;
- Professional: 10 dias;
- Enterprise: 30 dias;
- Log Streaming não aparece como recurso incluído no Free na matriz consultada.

Portanto, o procedimento Free-only é consultar e registrar evidências dentro da janela disponível, exportar pontualmente via Management API quando permitido e armazenar somente dados minimizados sob controle do projeto. Não depender de Log Streams, extensões pagas ou serviço externo.

A retenção e o entitlement podem mudar; confirmar no tenant e na página de preços na data de operação. Não assumir que o Dashboard mostra todo tipo de evento para qualquer role.

## Sessões e tokens

Monitorar por configuração e eventos, não pelo conteúdo dos tokens:
- duração de sessão e inatividade conforme política configurada;
- falhas de refresh e revogação;
- rotação e reutilização de refresh token quando o recurso estiver habilitado;
- expiração de access token e falhas de audience/scope;
- alteração inesperada de callback URLs, logout URLs e allowed web origins.

Nunca registrar access token, ID token, refresh token, cookie de sessão, senha ou Authorization header. Em logs da aplicação, registrar apenas categoria do erro e requestId. Claims devem ser minimizados.

## Runbook: pico de falhas de login

1. Delimitar horário UTC e comparar com baseline.
2. Agrupar por application/client, connection e tipo de evento.
3. Verificar se falhas são credenciais inválidas, callback mismatch, rate limiting ou IdP externo.
4. Consultar mudanças recentes no tenant e deployments correspondentes.
5. Se houver suspeita de credencial comprometida, seguir runbook de incidente; não desativar proteções em massa.
6. Registrar evidência minimizada, decisão, responsável e resultado.

## Limitações

A ausência de evento não prova ausência de falha: alguns erros em provedores de identidade externos podem não completar o fluxo e não retornar ao Auth0. Dashboard/Management API também têm limites de consulta e acesso por role.

## Referências

- https://auth0.com/pt/pricing
- https://auth0.com/docs/deploy-monitor/logs
- https://auth0.com/docs/api/management/v2/logs/get-logs
- https://support.auth0.com/center/s/article/Viewing-Large-Numbers-of-Logs-in-the-Auth0-Dashboard
- https://support.auth0.com/center/s/article/Unexpected-Usage-of-the-Auth0-Log-Endpoint-api-v2-logs
