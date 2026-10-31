# 02 — Testes unitários e de componentes

## Vitest
Adequado para funções puras, schemas Zod, conversões, regras de autorização, validação de parâmetros e helpers de paginação.

Casos prioritários:
- Payload válido/inválido.
- Campos desconhecidos e limites de tamanho.
- Normalização de filtros e allowlist de sort.
- Mapeamento de erros para status HTTP.
- Matriz role × action × resource.
- Cursor inválido, adulterado ou expirado.

## React Testing Library
Testar componentes pelo que o usuário vê: roles, labels, nomes acessíveis e eventos. Evitar acoplamento a classes CSS ou estado interno.

## Cobertura
Cobertura é indicador, não prova de segurança. Priorizar branches de autorização, tratamento de erros e validação. Não buscar 100% mecânico.

O guia Next.js observa limitações de suporte para async Server Components no Vitest; cobrir fluxos assíncronos com E2E quando necessário.

Fontes: https://nextjs.org/docs/app/guides/testing/vitest ; https://vitest.dev/guide/ ; https://testing-library.com/docs/react-testing-library/intro/