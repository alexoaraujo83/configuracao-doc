# 5. Configurar Vercel Hobby

## Elegibilidade
Hobby é gratuito, porém os termos Vercel restringem seu uso a fins pessoais ou não comerciais. Não utilizar para SaaS, clientes, empresa ou atividade comercial. Recursos, limites e condições podem mudar.

## Importar projeto
1. Vercel Dashboard → Add New → Project.
2. Selecionar repositório GitHub autorizado.
3. Confirmar preset Next.js e Root Directory.
4. Revisar Install Command, Build Command e versão Node.
5. Não iniciar publicação de produção sem revisão e autorização.

## Variáveis por ambiente
Configurar Development, Preview e Production separadamente:

| Variável | Tipo | Uso |
|---|---|---|
| AUTH0_DOMAIN | Config | domínio do tenant |
| AUTH0_CLIENT_ID | Config | client correto por ambiente |
| AUTH0_CLIENT_SECRET | Secret | segredo Auth0 |
| AUTH0_SECRET | Secret | segredo de sessão exclusivo |
| APP_BASE_URL | Config | host absoluto quando fixo |
| DATABASE_URL | Secret | Neon branch/role apropriada |

A interface atual usa Config e Secret. Secrets são write-only após salvar. Não reproduzir valores em documentação ou logs. Após alterar variáveis, criar novo deployment para aplicar as mudanças; deployments existentes preservam configuração anterior.

## Callback Auth0
Cadastrar apenas URLs efetivamente usadas:
- Local: http://localhost:3000/auth/callback
- Produção: https://dominio-controlado.example/auth/callback

Cadastrar também logout URLs exatas. Preview dinâmico exige compatibilidade entre host inferido pelo SDK e allowlist Auth0. Não usar wildcard amplo.

## Neon por ambiente
Preview não deve usar DATABASE_URL de produção. Preferir branch Neon isolada e role runtime limitada. Branch automática por preview depende da integração e das quotas; não presumir disponibilidade ilimitada. Se não houver isolamento Free adequado, desabilitar operações persistentes no preview ou usar dados sintéticos em ambiente de teste controlado.

## Fluxo de publicação
1. Abrir PR.
2. Aguardar CI.
3. Revisar preview e logs.
4. Confirmar variáveis e isolamento.
5. Aprovar e mesclar conforme processo humano autorizado.
6. Confirmar deployment e smoke test não destrutivo.

Não configurar publicação automática de produção sem aprovação expressa.

## Domínio
Adicionar somente domínio sob controle do responsável. Seguir registros DNS exibidos no painel; não inventar valores. Após configurar, validar DNS/HTTPS e atualizar URLs Auth0. Não comprar domínio nesta etapa.

## Rollback e retenção
Rollback de aplicação não reverte schema. Planejar migrations compatíveis (expand/contract) e recuperação do banco separadamente. Conferir limites atuais de retenção e armazenamento Hobby. Não depender de recursos exclusivos de planos pagos.

## Troubleshooting
- Build falha: logs, Node, lockfile e variáveis.
- Runtime sem variável: escopo e novo deployment.
- Callback falha: host e URL cadastrada.
- Banco não conecta: DATABASE_URL, branch, role e pooling.
- Preview acessa dados reais: corrigir isolamento antes de continuar.
- Deploy não aparece no PR: integração GitHub e permissões.

Referências: https://vercel.com/legal/terms, https://vercel.com/docs/environment-variables, https://vercel.com/docs/plans/hobby, https://vercel.com/docs/deployments
