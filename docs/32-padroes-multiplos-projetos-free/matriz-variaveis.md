# Matriz de variáveis por ambiente

Esta matriz define nomes e regras, não valores. Cada projeto mantém seu inventário próprio. Configurações reais devem ser registradas nos painéis do fornecedor, nunca no Git.

| Variável | Local | Preview | Staging | Production | Exposição |
|---|---|---|---|---|---|
| APP_BASE_URL | URL local | URL preview | URL staging | URL produção | configuração |
| AUTH0_DOMAIN | domínio tenant | domínio tenant | domínio tenant | domínio tenant | identificador |
| AUTH0_CLIENT_ID | client dev | client preview | client staging | client prod | identificador |
| AUTH0_CLIENT_SECRET | segredo dev | segredo preview | segredo staging | segredo prod | segredo |
| AUTH0_SECRET | segredo local | segredo preview | segredo staging | segredo prod | segredo |
| DATABASE_URL | banco dev | banco preview | banco staging | banco prod | segredo |

## Regras

1. Não reutilizar AUTH0_CLIENT_SECRET, AUTH0_SECRET ou DATABASE_URL entre ambientes.
2. Se tenant Free único obrigar compartilhamento do tenant Auth0, criar Applications separadas por projeto/ambiente quando os limites permitirem; não reutilizar a mesma credencial como atalho.
3. Cada callback e logout URL deve corresponder exatamente ao host autorizado daquele ambiente.
4. DATABASE_URL só pode ser consumida no servidor. Não prefixar com NEXT_PUBLIC_.
5. Variáveis adicionadas/alteradas na Vercel aplicam-se a novos deployments; planejar novo deployment para incorporar a configuração.
6. Preview não deve conectar ao banco Production.
7. Não colocar valores reais em .env.example; usar placeholders não funcionais.
8. Registrar responsável e procedimento de revogação, sem registrar o segredo.

## Checklist de revisão

- [ ] Existe inventário separado por projeto?
- [ ] Existe inventário separado por ambiente?
- [ ] Os valores reais estão somente no mecanismo de secrets/configuração do fornecedor ou ambiente local protegido?
- [ ] Nenhuma variável secreta possui prefixo NEXT_PUBLIC_?
- [ ] Preview e Development não apontam para banco Production?
- [ ] Callbacks/origins Auth0 estão restritos aos hosts necessários?
- [ ] Novo deployment foi planejado após alteração de variáveis Vercel?
