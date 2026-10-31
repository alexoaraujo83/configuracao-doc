# Segurança, testes e critérios de validação da arquitetura

## Fronteiras

| Fronteira | Controle requerido |
|---|---|
| Browser → Next.js | HTTPS, cookies seguros e validação de entrada |
| Next.js → Auth0 | SDK v4, callback/origins mínimos e secrets server-side |
| Next.js → Neon | driver server-only, TLS e pooled URL quando aplicável |
| Caso de uso → recurso | autorização server-side por operação e tenant |
| PostgreSQL → dados | role runtime restrita e RLS |
| CI → repositório | permissions mínimas e sem secrets em PR não confiável |
| Logs → observabilidade | redaction de tokens, cookies e PII |

## Testes unitários

Cobrir usuário sem sessão, usuário sem associação, tenant incorreto, recurso de outro usuário, papel sem permissão, entrada inválida, recurso inexistente e conflito de estado. Usar doubles/fakes de repositório para testar regras sem depender de serviços externos.

## Testes de integração

Em ambiente não produtivo e com dados sintéticos:
- adaptador Neon e consultas parametrizadas;
- transação e rollback;
- contexto tenant/user de RLS na mesma transação;
- policies com role runtime e tentativas fora do tenant;
- mapeamento de erros PostgreSQL;
- serialização de resposta sem campos internos.

Nunca usar credenciais de produção em testes ou Preview. Não executar migrations de produção no pipeline de PR.

## Testes de autenticação

Cobrir sessão ausente/expirada, callback válido, logout, identidade sem perfil interno, usuário desativado e autorização negada para recurso/tenant. Não registrar cookies, tokens, authorization headers ou conteúdo privado.

## Pipeline de qualidade

Usar apenas recursos disponíveis no GitHub Free. Não depender de secrets de Environment ou proteções indisponíveis na modalidade/repositório. Checks recomendados:
1. instalação reproduzível pelo lockfile;
2. lint;
3. TypeScript typecheck;
4. testes unitários;
5. build;
6. testes de integração opcionais contra banco isolado, sem secrets de produção.

## Gate de aceite

- [ ] Camadas e dependências respeitam o desenho.
- [ ] Segredos permanecem server-only.
- [ ] Auth0 SDK v4 e API compatíveis com Next.js selecionado.
- [ ] Autorização em cada caso de uso protegido.
- [ ] SQL parametrizado.
- [ ] RLS validada com runtime role.
- [ ] Contexto e consultas na mesma transação.
- [ ] Erros não expõem stack, SQL, tokens ou connection strings.
- [ ] Logs possuem redaction.
- [ ] Testes cobrem autorização positiva e negativa.
- [ ] CI não usa secrets em PR não confiável.
- [ ] Plano Free e elegibilidade contratual confirmados.
- [ ] Rollback e recuperação documentados.

**Estado atual:** checklist de referência. Marcar itens como verificados somente após evidência em ambiente autorizado.
