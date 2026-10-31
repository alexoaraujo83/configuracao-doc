# Vercel Hobby — hospedagem e Preview

## Restrição determinante

Vercel Hobby é destinado a uso pessoal e não comercial. Aplicação comercial, SaaS, prestação de serviços ou uso empresarial não pode ser aprovado sob Hobby. Não recomendar trial, créditos ou upgrade como solução dentro deste escopo; registrar NO-GO e reavaliar arquitetura/serviço elegível em processo separado.

## Configuração padrão inicial

1. Confirmar e documentar finalidade pessoal e não comercial.
2. Criar projeto Vercel a partir do repositório GitHub autorizado.
3. Confirmar framework Next.js, Root Directory e comandos de build.
4. Configurar variáveis por ambiente sem expor valores.
5. Configurar domínios e HTTPS quando elegíveis.
6. Registrar callback e logout URLs correspondentes no Auth0.
7. Configurar connection string Neon correta para ambiente.
8. Aguardar build e verificar logs sem revelar secrets.
9. Validar autenticação, autorização e banco com dados sintéticos.

## Ambientes

Vercel disponibiliza Development, Preview e Production conforme plano e configuração. No Hobby, Staging pode ser modelado como Preview com branch/configuração específica apenas se recurso estiver disponível; isso não cria automaticamente ambiente independente.

Não confundir variável configurada no projeto, variável incluída em novo deployment e deployment já publicado. Uma mudança de variável exige novo deployment para que runtime receba o novo valor.

## GitHub integration e releases

- Vincular repositório autorizado.
- Revisar quais branches geram Preview.
- Definir branch de produção.
- Validar variáveis antes do deployment.
- Registrar commit, URL, data e resultado.
- Revisar limites de build, functions, bandwidth e logs no painel.
- No Hobby, rollback pode ser limitado ao deployment de produção imediatamente anterior; verificar comportamento atual antes de depender dele.

## Checklist

- [ ] Uso pessoal/não comercial confirmado.
- [ ] Plano Hobby e quotas verificados.
- [ ] Repositório e branch corretos.
- [ ] Root Directory e build configurados.
- [ ] Variáveis por ambiente corretas.
- [ ] Auth0 callback/logout URLs correspondem aos domínios.
- [ ] Neon connection string é privada e do ambiente correto.
- [ ] Logs não exibem dados sensíveis.
- [ ] Estratégia de rollback documentada e compatível com Hobby.

Referências: [Vercel Docs](https://vercel.com/docs), [Hobby Plan](https://vercel.com/docs/plans/hobby), [Limits](https://vercel.com/docs/limits).
