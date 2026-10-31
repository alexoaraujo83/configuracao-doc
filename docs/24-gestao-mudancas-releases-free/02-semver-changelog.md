# Versionamento Semântico e changelog

## SemVer 2.0.0

Formato MAJOR.MINOR.PATCH:
- MAJOR: mudança incompatível no contrato público.
- MINOR: funcionalidade nova compatível com versões anteriores.
- PATCH: correção compatível.
- Pre-release: versão de validação, por exemplo 2.1.0-rc.1.
- Build metadata: metadados após +, sem alterar precedência.

A API pública precisa estar definida. Para aplicação web, documentar endpoints, payloads, eventos, schemas consumidos por integrações e comportamento de autenticação/autorização.

## Critério de incremento

| Mudança | Versão |
|---|---|
| Corrige bug sem quebrar contrato | PATCH |
| Adiciona endpoint/campo opcional compatível | MINOR |
| Remove/renomeia endpoint ou campo obrigatório | MAJOR |
| Altera autorização de forma incompatível | MAJOR ou migração comunicada |
| Atualiza dependência sem alteração contratual | PATCH/MINOR conforme impacto |
| Apenas documentação | Sem versão de produto obrigatória |

Não usar SemVer para prometer estabilidade que o projeto ainda não oferece.

## Changelog

Manter CHANGELOG.md na raiz com seções:
- Unreleased
- Added
- Changed
- Deprecated
- Removed
- Fixed
- Security

Cada entrada descreve impacto observável e, quando possível, Issue/PR. Não incluir secrets, dados de clientes ou detalhes exploráveis de vulnerabilidade antes da correção.

Exemplo:

~~~markdown
# Changelog

## [Unreleased]

### Added
- Adiciona validação de membership antes de acessar recursos.

### Fixed
- Corrige encerramento de transação em falha de consulta.

## [1.2.0] - 2026-10-31

### Changed
- Atualiza fluxo de sessão sem alterar contratos públicos.
~~~

## Tags Git

- Criar tag anotada após aprovação e validação.
- Formato recomendado: vMAJOR.MINOR.PATCH.
- Não mover tags já publicadas; corrigir por nova versão.
- Associar tag ao commit exato de release.
- Registrar commit SHA, PRs, deployment ID e migration IDs.

Comandos de referência, não executados:

~~~bash
git tag -a v1.2.0 -m "Release v1.2.0"
git push origin v1.2.0
~~~

## Processo

1. Atualizar Unreleased.
2. Revisar mudanças e classificar impacto.
3. Escolher versão conforme SemVer.
4. Substituir Unreleased por versão/data.
5. Criar tag anotada no commit aprovado.
6. Publicar GitHub Release, se apropriado.
7. Registrar deployment e migrations associadas.

## Referências

- https://semver.org/lang/pt-BR/spec/v2.0.0.html
- https://keepachangelog.com/pt-BR/1.1.0/
- https://www.conventionalcommits.org/en/v1.0.0/
