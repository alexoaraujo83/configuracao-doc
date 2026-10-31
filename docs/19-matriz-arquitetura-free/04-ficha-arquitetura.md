# Ficha de arquitetura por projeto

Copiar por aplicação. Não inserir valores secretos.

## Identificação
- Projeto / slug:
- Responsável técnico:
- Data e revisão:
- Arquétipo: aplicação simples | SaaS multiusuário | API/backend | múltiplas aplicações
- Finalidade: pessoal/não comercial | comercial | desconhecida
- Estado: recomendado | condicional | bloqueado | pendente

## Classificação e requisitos
- Tipos/classificação dos dados:
- Dados pessoais/sensíveis:
- Retenção:
- Fronteiras de confiança:
- Isolamento físico/lógico requerido:
- RPO/RTO:

## Topologia

| Serviço | Conta/tenant/projeto | Ambiente | Identificador não secreto | Plano | Evidência |
|---|---|---|---|---|---|
| GitHub | | | | Free | |
| Vercel | | | | Hobby | |
| Auth0 | | | | Free | |
| Neon | | | | Free | |

## Isolamento
- Tenant Auth0 compartilhado?:
- Clients separados por app/ambiente?:
- Callbacks/logout/web origins revisados?:
- Projeto Vercel por app?:
- Projeto Neon por domínio de dados?:
- RLS habilitada e testada?:
- Roles owner/migrator/runtime separadas?:
- Secrets separados por ambiente?:
- Blast radius e riscos residuais:
- Responsável pela aceitação:

## Quotas verificadas
- Neon projetos/branches/CU/storage/transferência:
- GitHub Actions minutos:
- Vercel Hobby limites:
- Auth0 quotas:
- Data e URLs oficiais:

## Testes
- [ ] Login/logout e callback allowlists.
- [ ] JWT issuer/audience/expiry/signature.
- [ ] Autorização server-side por recurso/ação.
- [ ] Testes negativos cross-tenant.
- [ ] RLS USING e WITH CHECK.
- [ ] Preview não acessa Production.
- [ ] Runtime DB role sem owner/migrator.
- [ ] Secrets ausentes de Git/browser/logs/artifacts.
- [ ] Migrations revisadas e rollback documentado.
- [ ] Backup/restore ensaiado.
- [ ] Quotas e comportamento no limite verificados.
- [ ] Elegibilidade pessoal/não comercial do Vercel Hobby confirmada.

## Decisão
- Resultado: GO | GO CONDICIONAL | PENDENTE | NO-GO
- Bloqueios:
- Mitigações:
- Riscos aceitos:
- Aprovador e data:
- Próxima revisão:
