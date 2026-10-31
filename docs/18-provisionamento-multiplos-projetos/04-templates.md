# 04 — Templates reutilizáveis

## 1. Objetivo

Templates aceleram novos projetos sem carregar credenciais ou configurações específicas de outro projeto. São contratos documentais, não scripts de provisionamento automático.

## 2. Arquivos

- `templates/provisionamento/project.yaml`: manifesto de identidade e recursos.
- `templates/provisionamento/.env.example`: nomes e placeholders.
- `templates/provisionamento/checklist.md`: passos e aceite.
- `templates/provisionamento/README.md`: regras de cópia e manutenção.

## 3. Regras de reutilização

1. Copiar os templates para o repositório do novo projeto.
2. Substituir apenas identificadores não secretos e placeholders.
3. Revisar os campos que dependem de plano/quotas.
4. Manter o manifesto sem valores secretos.
5. Não copiar `.env.local`, connection strings ou exports de tenants.
6. Versionar alterações nos templates por Pull Request.
7. Registrar a versão do template usada pelo projeto.

## 4. Manifesto versus configuração real

O manifesto descreve o que se pretende provisionar. Não comprova que o recurso existe. Para cada recurso, registrar:
- `status: planned | existing | verified | blocked`;
- URL/ID não secreto, quando conhecido;
- ambiente;
- responsável;
- data de verificação;
- referência da evidência sem credenciais.

Nunca armazenar tokens de Management API, Auth0 Client Secret, Vercel token ou DATABASE_URL.

## 5. Templates e configuração as code

- Auth0 Deploy CLI pode exportar/importar configurações do tenant; excluir recursos não disponíveis no Free, por exemplo log streams e custom domains conforme orientação da CLI.
- A exportação de tenant não inclui usuários; planejar export/import de usuários separadamente se aplicável.
- Vercel environment variables devem ser geridas no painel/CLI e escopadas ao ambiente/branch. O arquivo `.env.example` não deve conter valores reais.
- Neon schema e roles devem ser versionados por migrations SQL; connection strings são fornecidas fora do Git.
- GitHub workflow deve usar permissões mínimas e não executar workflows privilegiados sobre código não confiável.

## 6. Validação de template

Antes de declarar template aprovado:
- [ ] YAML/Markdown válido.
- [ ] Nenhum segredo ou dado pessoal.
- [ ] Slugs e nomes dentro do padrão.
- [ ] Ambientes coerentes com limitações Free.
- [ ] Variáveis documentadas.
- [ ] URLs de callback não usam wildcard amplo.
- [ ] Checklist tem passos de validação e rollback.
- [ ] Links oficiais atualizados.
