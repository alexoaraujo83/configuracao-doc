# Catálogo de controles e evidências

## Padrão mínimo de evidência

Cada evidência deve possuir:
- ID único e requisito relacionado;
- nome do projeto e ambiente lógico;
- data/hora UTC e fuso original quando relevante;
- commit SHA, versão da aplicação ou migration;
- fonte (GitHub, CI, Auth0, Neon, Vercel ou teste local);
- responsável pela coleta e revisor;
- resultado esperado e observado;
- artefato sanitizado e localização;
- validade/revisão prevista.

Nunca anexar: secrets, access/refresh/ID tokens, cookies, connection strings, dados pessoais reais, dumps de produção ou logs contendo payloads sensíveis.

## Catálogo de artefatos aceitos

| Código | Evidência | Conteúdo aceitável | Não é evidência suficiente |
|---|---|---|---|
| EV-CODE | Revisão de código | SHA, arquivos, aprovação e comentários resolvidos | Link genérico ao repositório |
| EV-CI | Pipeline | Execução, workflow SHA, jobs e logs sanitizados | YAML que nunca foi executado |
| EV-TEST | Relatório de testes | Framework/versão, ambiente, casos, resultado e data | Testes descritos em documentação |
| EV-IAM | Auth0 | Configuração exportada redigida, callback allowlist, teste browser | Screenshot sem data ou tenant |
| EV-DB | PostgreSQL | DDL, grants, policies, catálogo e testes sintéticos | Diagrama sem banco/DDL |
| EV-MIG | Migration | Hash, ordem, resultado e validação pós-migration | Arquivo SQL isolado |
| EV-DEP | Dependências | package-lock, npm audit JSON redigido, triagem e decisão | Apenas npm outdated |
| EV-ACT | GitHub Actions | Workflow, SHA fixo das Actions, permissões e run | Nome da action sem versão |
| EV-VER | Vercel | Deployment ID, commit SHA, ambiente, build status e smoke test | URL sem deployment/commit |
| EV-BKP | Backup/restore | Dump sintético ou protegido, checksum, restore log e validação | Backup criado sem restore |
| EV-OPS | Operação | Runbook, exercício tabletop e decisão | Runbook sem owner |
| EV-FREE | Plano Free | Página oficial consultada, data, painel e cota observada | Captura antiga ou estimativa |
| EV-APR | Aprovação | decisão, escopo, aprovador, data e condições | Emoji ou comentário informal |

## Procedimento de coleta

1. Abrir o requisito na matriz mestra.
2. Confirmar o ambiente e a versão/commit sob avaliação.
3. Executar apenas o teste autorizado no ambiente apropriado.
4. Sanitizar o resultado antes de armazenar.
5. Vincular artefato ao ID do requisito.
6. Registrar responsável, revisor e data.
7. Definir validade de acordo com volatilidade do controle.
8. Recoletar após mudança material, incidente ou expiração.

## Validade sugerida

| Evidência | Validade interna máxima sugerida |
|---|---|
| Plano, preço e quotas | 30 dias ou antes de qualquer mudança do fornecedor |
| Configuração Auth0 | 90 dias e após qualquer mudança |
| Variáveis/ambientes Vercel | 30 dias e após redeploy/config change |
| Roles, grants e RLS | 90 dias e após migration |
| Dependências/CVEs | 7 dias para release; revalidar no merge |
| CI e testes | Por commit candidato |
| Restore | 90 dias ou após mudança do processo |
| Aprovação release | Apenas para o artefato/commit aprovado |

Os prazos são políticas internas propostas, não garantias nem exigências dos fornecedores. A organização pode torná-los mais restritivos.

## Evidência documental versus evidência operacional

- **Documental:** especifica o que fazer; estado Planejada.
- **Configuração:** mostra como o serviço está configurado; estado Coletada até revisão.
- **Execução:** mostra o teste efetivamente executado; estado Coletada.
- **Verificada:** revisão confirmou origem, escopo, data, integridade e resultado.
- **Expirada:** evidência antiga, inválida para o release atual ou superada por mudança.

A existência deste repositório e de PRs de documentação não certifica serviços, ambientes, pipelines ou aplicação.
