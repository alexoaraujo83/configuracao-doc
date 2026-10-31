# 6. Testes e critérios de aceitação

Testes são recomendações documentais. Executar somente em ambiente isolado, com dados sintéticos e autorização explícita.

## Camadas
| Camada | Verificações |
|---|---|
| Unitário | validação, autorização, mapeamento de erros |
| Integração | migrations, constraints, repositórios em banco de teste |
| Segurança | sem sessão, tenant incorreto, membership inativa |
| E2E | login, callback, sessão, logout, rotas protegidas |
| CI | lint, TypeScript e build |
| Deploy | variáveis, callbacks e smoke tests não destrutivos |

## Matriz de casos
| Caso | Resultado esperado |
|---|---|
| Visitante em rota protegida | login ou 401 |
| Sessão expirada | não autenticar; solicitar login |
| Sessão válida | identidade obtida no servidor |
| Subject adulterado pelo cliente | ignorar; usar sessão |
| Sem membership | negar acesso |
| Membership inativa | negar acesso |
| Tenant não autorizado | negar leitura e escrita |
| Payload de SQL injection | tratar como dado parametrizado |
| RLS sem contexto | nenhuma linha protegida acessível ou erro controlado |
| Runtime tenta DDL | operação negada |
| Neon indisponível | erro genérico, sem bypass |
| Auth0 indisponível | não conceder sessão |
| Variável ausente | falha explícita sem imprimir segredo |
| Preview | não acessar dados/segredos de produção |

## Critérios de aceitação
- [ ] Identidade vem de sessão validada no servidor.
- [ ] Nenhum endpoint mutável confia em tenant, subject ou role do navegador.
- [ ] Membership, estado ativo e permissão são verificados por operação.
- [ ] RLS protege tabelas multi-tenant.
- [ ] Contexto tenant é transacional e na mesma conexão das consultas.
- [ ] Runtime não é owner e não possui DDL ou BYPASSRLS.
- [ ] SQL parametrizado e validação de entrada.
- [ ] Nenhum segredo versionado ou impresso em logs.
- [ ] CI executa lint, tipos e build sem secrets de produção.
- [ ] Preview usa configuração e dados isolados.
- [ ] Callback/logout URLs correspondem ao ambiente.
- [ ] Hobby usado somente para fins pessoais e não comerciais.
- [ ] Migrations revisadas e com estratégia de compatibilidade/rollback.
- [ ] Limites Free conferidos na data da implantação.

## Revisão de PR
1. Conferir versão real do Next.js e SDK Auth0.
2. Validar APIs contra documentação da versão.
3. Revisar políticas RLS e grants dos papéis.
4. Procurar secrets e dados pessoais no diff.
5. Conferir segregação de ambientes.
6. Confirmar que nenhuma migration ou deploy foi disparado inadvertidamente.
7. Registrar limitações e decisões pendentes em ADR.
