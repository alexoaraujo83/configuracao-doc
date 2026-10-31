# Gate de continuidade e recuperação

Marcar Verificado, Pendente ou NO-GO com evidência, owner e data. Documento não equivale a teste.

| Controle | Evidência | Estado inicial |
|---|---|---|
| BIA/processos críticos | BIA aprovada | Pendente |
| RTO/RPO | metas aprovadas vs capacidade Free | Pendente |
| Neon PITR | history window conferido; Free até 6h conforme docs consultadas | Pendente |
| Dump lógico | cópia externa cifrada + checksum | Pendente |
| Restore Neon | restore isolado e duração medidos | Pendente |
| Schema/migrations | versionados e validados | Pendente |
| Auth0 export | export sem secrets + diff | Pendente |
| Auth0 usuários | limitação e re-enrollment aceitos | Pendente |
| GitHub account | 2FA/recovery codes + clone verificado | Pendente |
| GitHub pipeline | workflows reconstruíveis | Pendente |
| Vercel | Preview rebuild + smoke test | Pendente |
| Vercel rollback | limite Hobby conhecido | Pendente |
| Env vars | nomes/escopo e custódia documentados | Pendente |
| Domínio/DNS | titular, zona exportada e acesso | Pendente |
| Exercício integrado | tabletop/restore com relatório | Pendente |
| Free-only | sem trial/add-on/recurso pago | Pendente |
| Vercel eligibility | uso pessoal/não comercial | Pendente |

## Bloqueios
RPO menor que capacidade real; PITR Free tratado como retenção maior que 6h ou backup externo; dump sem criptografia/checksum/restore testado; password hashes Auth0 sem alternativa; conta sem recovery/owner; domínio sem titularidade; deployment antigo como único artefato; secrets recuperados de logs/commits/artifacts; uso comercial no Hobby; RTO/RPO apresentados como SLA.

## Aprovação
Projeto/versão; owner; data UTC; RTO/RPO; limitações Free aceitas; testes/evidências; riscos residuais; pendências; GO/NO-GO e aprovadores.

Declaração: esta documentação não comprova backup, restore, recovery codes, export Auth0, domínio/DNS, conta, deployment ou teste realizado. Todos os estados iniciais são Pendente.
