# Ambientes, CI, segurança, testes e operação — baseline Free

> Revisão documental: 31/10/2026. Esta página descreve o processo de referência. Não configura serviços nem executa deploy.

## 1. Regra de elegibilidade do ambiente

O ambiente lógico (Development, Preview, Staging, Production) não garante que o plano do fornecedor permita o uso. Validar sempre três dimensões: disponibilidade técnica, quota Free e permissão contratual de uso.

**Vercel Hobby é restrito a uso pessoal ou não comercial.** Se o produto for SaaS comercial, aplicação empresarial comercial ou serviço oferecido a clientes, Vercel Hobby está **NÃO DISPONÍVEL PARA ESTE USO**. A documentação pode explicar o fluxo Vercel para projetos pessoais/não comerciais; para produto comercial, avaliar self-host ou provedor cujo plano gratuito autorize expressamente esse uso. Não pressupor que preço zero signifique uso comercial permitido.

Os Termos da Vercel atualizados em 19/10/2026 também autorizam uso do conteúdo enviado por contas Hobby para treinamento de modelos de IA/ML e compartilhamento com terceiros para esse fim, com possibilidade de opt-out nas configurações da Team. Antes de enviar código-fonte, dados de usuários ou conteúdo confidencial, verificar e registrar o estado efetivo do opt-out. Se não for possível comprovar a desativação, não enviar conteúdo sensível.

Fonte: [Vercel Terms of Service](https://vercel.com/legal/terms), seção Hobby e conteúdo enviado.

## 2. Matriz de ambientes

| Dimensão | Development | Preview | Staging | Production |
|---|---|---|---|---|
| Next.js | local | deployment temporário se permitido | host Free elegível ou local controlado | host elegível ao uso |
| Auth0 | aplicação/tenant de desenvolvimento | somente host aprovado | configuração de homologação | configuração de produção |
| Neon | branch dev | branch efêmera dentro da quota | branch/projeto isolado | branch principal |
| Secrets | locais | próprios | próprios | próprios |
| Dados | sintéticos | sintéticos/mascarados | mascarados | reais |
| Deploy | `npm run dev` | PR workflow | promoção aprovada | release aprovada |
| Uso Vercel Hobby | pessoal/não comercial | somente dentro do uso permitido | não presume uso comercial | não usar para SaaS comercial |

Se Auth0 ou Neon não puderem fornecer isolamento suficiente dentro do Free, reduzir ambientes persistentes e documentar risco. Não compartilhar secrets entre ambientes. Não usar Preview como ambiente de produção.

## 3. Nomenclatura

Padrão sugerido:

- Aplicação: `<produto>-<dominio>`.
- Auth0 Application: `<produto>-<ambiente>-web`.
- Auth0 API: `<produto>-api`.
- Neon Project: `<produto>-<ambiente>` somente se projeto separado for necessário.
- Neon Branch: `main`, `development`, `preview-<identificador>`.
- Git branches: `feature/<assunto>`, `fix/<assunto>`, `docs/<assunto>`.

Não incluir dados pessoais, nomes de clientes ou secrets em nomes de branches.

## 4. GitHub Free: fluxo de revisão e CI

Para repositório privado no GitHub Free, rulesets/branch protection server-side não estão disponíveis. Não apresentar CI como bloqueio obrigatório de merge quando não existe enforcement no plano.

Controles processuais:

1. Restringir escrita na branch padrão ao mínimo possível.
2. Trabalhar em branches curtas.
3. Abrir Pull Request para cada mudança.
4. Executar CI em `pull_request`.
5. Revisar diff e logs antes de incorporar.
6. Registrar aprovação no PR; isso é processo, não enforcement técnico.
7. Não tornar repositório privado público para obter rulesets gratuitos.

Workflow ilustrativo `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 24
          cache: npm
      - run: npm ci
      - run: npm run typecheck
      - run: npm run lint
      - run: npm run build
```

Antes de uso em produção, revisar versões das Actions e fixar tags/SHAs confiáveis. Não passar secrets na linha de comando. Em repositório privado, monitorar minutos e storage de Actions na área Billing. O uso self-hosted evita minutos de runner hospedado, mas transfere custos e responsabilidades de isolamento, atualização e disponibilidade para o operador.

## 5. Vercel Hobby: uso permitido e configuração

Somente para projeto pessoal/não comercial:

1. Importar repositório Git.
2. Confirmar framework Next.js e Root Directory.
3. Revisar build command e output.
4. Configurar variáveis por ambiente.
5. Fazer deployment de preview.
6. Validar login, callback e banco.
7. Promover para Production somente após checklist.
8. Conferir limites e termos vigentes.

Não assumir Custom Environments (ex.: Staging) no Hobby. Usar apenas Development, Preview e Production conforme funcionalidades realmente incluídas no plano. Projeto separado não elimina a restrição contratual de uso comercial.

### Variáveis

| Nome | Development | Preview | Production |
|---|---|---|---|
| `APP_BASE_URL` | localhost | host exato de preview ou inferência controlada | origem canônica HTTPS |
| `AUTH0_DOMAIN` | tenant dev | tenant autorizado | tenant prod |
| `AUTH0_CLIENT_ID` | app dev | app/host autorizado | app prod |
| `AUTH0_CLIENT_SECRET` | secret dev | secret próprio | secret prod |
| `AUTH0_SECRET` | segredo local | segredo próprio | segredo próprio |
| `DATABASE_URL` | branch dev pooled | branch preview pooled | branch prod pooled |

Revalidar callback URLs Auth0 após criação/alteração de domínio. Não cadastrar wildcard irrestrito. Não colocar secrets em build args, logs, código ou screenshots.

## 6. Auth0: ambientes e isolamento

- Preferir aplicações separadas por ambiente quando isso reduzir risco e for compatível com a quota Free.
- Não compartilhar Client Secret entre ambientes.
- Usar callback/logout URL específica de cada origem.
- Preview com URL dinâmica requer estratégia explícita; se o host não estiver aprovado no Auth0, desabilitar login no Preview em vez de abrir wildcard.
- Manter requisito de Access Token lifetime máximo de 900 segundos e consent skipping desabilitado como itens a verificar.
- A quota M2M é 1.000 autenticações/tokens por mês; monitorar e reutilizar tokens.
- MFA OTP/Duo e RBAC por Organization não estão incluídos no Free.

## 7. Neon: branches e dados

- Runtime usa pooled connection string.
- Migrations usam direct connection e role separada.
- Limite de 10 branches por projeto; definir expiração para branches Preview.
- Não clonar dados pessoais reais para ambiente não produtivo sem mascaramento.
- RLS e grants devem ser testados com role de runtime.
- Branch protection não está incluída no Free; controles SQL e revisão de migrations são compensatórios.
- Backup externo é obrigatório para retenção maior que o histórico Free.

## 8. Testes funcionais e de segurança

### Autenticação

- login e callback;
- logout;
- sessão expirada;
- callback não autorizado;
- token com issuer/audience incorretos;
- API sem sessão;
- API com usuário sem membership.

### Autorização e isolamento

- usuário A não lê registros de B;
- tenant A não lê registros de tenant B;
- alterar ID na URL não contorna ownership;
- role runtime não tem BYPASSRLS;
- RLS funciona com contexto transaction-local;
- tentativa de inserir tenant_id arbitrário é rejeitada;
- operações administrativas não estão expostas a usuário comum.

### Banco e migrations

- connection pooled funciona no runtime;
- direct connection funciona para migration;
- migration em branch de teste;
- migration repetida não corrompe schema;
- backup e restore validados;
- erro de quota tratado sem expor connection string.

### Aplicação

- `npm ci`;
- `npm run typecheck`;
- `npm run lint`;
- `npm run build`;
- nenhuma variável secreta no bundle cliente;
- respostas de erro sem stack trace/segredos.

## 9. Operação, limites e resposta a quota

Definir política fail-closed:

| Limite | Resposta operacional |
|---|---|
| Auth0 M2M quota aproximando-se de 1.000 | reutilizar tokens, suspender workloads não críticos e monitorar |
| Neon compute 100 CU-hours/projeto/mês | reduzir consultas/jobs; preparar suspensão controlada |
| Neon storage 1 GB/projeto | limitar escrita, arquivar/exportar dados e revisar crescimento |
| Neon transfer 5 GB/projeto/mês | reduzir transferência e consultas pesadas; preparar indisponibilidade |
| Neon 10 branches | remover somente branches expiradas após validar dependências |
| GitHub Actions minutes/storage | reduzir frequência/artifacts e usar execução local/self-hosted com segurança |
| Vercel Hobby quotas | limitar workloads pessoais; não habilitar recurso pago |

Não ativar cobrança automática, upgrade ou trial para contornar quota. Se o requisito de negócio não cabe no Free, registrar **BLOCKED — plano Free insuficiente** e não implantar sob pressupostos incorretos.

## 10. Backup, recuperação e rollback

- SQL migrations são versionadas; rollback destrutivo deve ser migration compensatória revisada.
- Export lógico do Neon com `pg_dump` em periodicidade definida pelo RPO/RTO.
- Restaurar periodicamente em branch/projeto não produtivo.
- Guardar backup criptografado e independente da conta Neon.
- Manter cópia do código no Git e releases identificadas.
- Rollback de aplicação não reverte automaticamente schema; adotar estratégia expand/contract para mudanças incompatíveis.
- Documentar owner, horário, duração, resultado e evidência de cada exercício de recuperação.

## 11. Checklist de implantação (documental)

- [ ] finalidade pessoal/não comercial confirmada para Vercel Hobby;
- [ ] estado de opt-out de Model Training verificado antes de enviar conteúdo;
- [ ] planos Free e quotas verificados;
- [ ] callback/logout URLs exatas;
- [ ] secrets isolados por ambiente;
- [ ] roles SQL de privilégio mínimo;
- [ ] RLS testada com role real;
- [ ] migrations revisadas e testadas em branch não produtiva;
- [ ] CI com typecheck/lint/build;
- [ ] testes de autorização e isolamento aprovados;
- [ ] backup independente e restore testado;
- [ ] política de quotas e fail-closed aprovada;
- [ ] rollback de aplicação e schema documentados.

## 12. Fontes oficiais

- [GitHub Rulesets availability](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)
- [GitHub Actions billing](https://docs.github.com/en/billing/managing-billing-for-your-products/managing-billing-for-github-actions/about-billing-for-github-actions)
- [Vercel Terms of Service](https://vercel.com/legal/terms)
- [Vercel Hobby Plan](https://vercel.com/docs/plans/hobby)
- [Vercel Environment Variables](https://vercel.com/docs/environment-variables)
- [Auth0 Pricing](https://auth0.com/pt/pricing)
- [Auth0 Next.js SDK](https://auth0.github.io/nextjs-auth0/)
- [Neon Plans](https://neon.com/docs/introduction/plans)
- [Neon RLS](https://neon.com/docs/guides/row-level-security)
