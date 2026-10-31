# 05 — Vercel Hobby e deploy Next.js

**Estado:** Documentado; projeto Vercel não importado nem publicado nesta etapa.  
**Plano:** Vercel Hobby.

## 1. Elegibilidade obrigatória

Hobby destina-se a uso pessoal e não comercial. **Não usar este plano para SaaS comercial, operação comercial ou serviço prestado comercialmente.** Se o caso de uso for comercial, interromper o deploy neste plano; não contornar a restrição nem iniciar trial/upgrade. O repositório pode continuar documentado e executado localmente.

Antes de publicar, confirmar que o projeto atende às condições atuais do Hobby e aos limites de uso. Cotas e recursos podem mudar.

## 2. Pré-requisitos

- Projeto Next.js com build local aprovado.
- Repositório GitHub.
- Conta Vercel elegível para Hobby.
- Auth0 e Neon configurados separadamente.
- Domínio padrão Vercel é suficiente; não depender de add-ons pagos.

## 3. Importar projeto

No Vercel Dashboard:
1. Importar repositório GitHub.
2. Selecionar framework Next.js.
3. Confirmar Root Directory.
4. Confirmar comando de instalação e build detectados.
5. Não cadastrar secrets até confirmar ambiente e domínio.
6. Criar deployment de Preview somente após configurar variáveis adequadas.
7. Testar antes de considerar qualquer produção.

## 4. Variáveis por ambiente

Configurar em Project Settings → Environment Variables, com valores distintos onde aplicável:

| Variável | Development | Preview | Production |
|---|---|---|---|
| `APP_BASE_URL` | localhost | URL exata de preview | URL de produção |
| `AUTH0_DOMAIN` | tenant | tenant autorizado | tenant |
| `AUTH0_CLIENT_ID` | app dev | app preview ou app compartilhado com callback exato | app production |
| `AUTH0_CLIENT_SECRET` | segredo dev | segredo preview | segredo produção |
| `AUTH0_SECRET` | segredo local | segredo exclusivo preview | segredo exclusivo produção |
| `DATABASE_URL` | role/branch dev | role/branch de preview | role/branch produção |

Não copiar o mesmo `AUTH0_SECRET` entre ambientes. Preferir separar aplicações Auth0 e credenciais Neon por ambiente se as cotas permitirem. Se a separação não for possível, documentar riscos e restringir URLs e permissões.

Após alterar variáveis, realizar novo deployment para que o build/runtime receba a configuração atualizada. Nunca imprimir valores no log.

## 5. Configuração Auth0 do domínio publicado

Depois de obter URL estável, cadastrar no Auth0:
- Allowed Callback URL: `https://<dominio>/auth/callback`
- Allowed Logout URL: `https://<dominio>`
- Allowed Web Origin: `https://<dominio>`

O caminho `/auth/callback` corresponde ao SDK v4. Não usar `/api/auth/callback` de exemplos antigos. Para Preview, cadastrar somente URLs específicas necessárias; não permitir wildcard amplo.

## 6. Deploy e validação

O fluxo Git é:
1. Criar branch.
2. Abrir Pull Request.
3. Executar GitHub CI.
4. Avaliar Preview, quando elegível e configurado.
5. Revisar código, variáveis e callback.
6. Merge autorizado.
7. Verificar deployment de produção, se permitido pelo uso pessoal/não comercial.

CLI opcional:

```bash
npm install --global vercel
vercel login
vercel link
vercel
```

Não executar `vercel --prod` até que a elegibilidade Hobby, o ambiente, os secrets e os testes tenham sido confirmados. Não passar secrets como argumentos CLI.

## 7. Limites e observabilidade

Consultar [Vercel Limits](https://vercel.com/docs/limits) e [Hobby Plan](https://vercel.com/docs/plans/hobby) na data do deploy. Limites de builds, funções, CPU, memória, invocações, logs e transferência têm janelas e condições específicas. Não copiar cotas antigas sem registrar data e fonte.

No Hobby, logs de runtime e recursos de observabilidade são limitados. Não desenhar operação que dependa de log drains, analytics avançado, proteção paga, RBAC avançado ou outros recursos não incluídos no plano.

## 8. Rollback

- Identificar último deployment válido.
- Verificar compatibilidade de schema antes de reverter aplicação.
- Preferir migrations retrocompatíveis (expand/contract).
- Não reverter schema automaticamente se houver escrita de dados incompatível.
- Registrar quem autorizou rollback, motivo, horário e validações.
- Confirmar limites e ações disponíveis no Hobby antes de depender de promoção/rollback específico.

## 9. Checklist de aceite

- [ ] Uso enquadrado como pessoal e não comercial.
- [ ] Build local aprovado.
- [ ] Variáveis configuradas por ambiente.
- [ ] Secrets únicos por ambiente.
- [ ] Callback Auth0 exato.
- [ ] Connection string Neon de menor privilégio.
- [ ] HTTPS e cookies seguros.
- [ ] Login, logout e páginas privadas testados.
- [ ] Endpoint de saúde não expõe detalhes internos.
- [ ] Limites Hobby revisados na data da publicação.

## 10. Referências

- [Vercel Hobby](https://vercel.com/docs/plans/hobby)
- [Vercel Limits](https://vercel.com/docs/limits)
- [Vercel Environment Variables](https://vercel.com/docs/projects/environment-variables)
- [Deploy Next.js](https://vercel.com/docs/frameworks/nextjs)

