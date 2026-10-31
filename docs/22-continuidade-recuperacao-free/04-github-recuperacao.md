# GitHub Free — recuperação de conta, repositório e pipeline

## Preparação
- Habilitar 2FA e guardar recovery codes offline; manter método alternativo suportado.
- Proteger email de recuperação.
- Manter clone local atualizado e validar git fsck periodicamente.
- Git versiona código, migrations e configuração não secreta; nunca dumps ou .env.
- Inventariar owners, colaboradores, deploy keys, Apps, PATs, webhooks e Actions permissions.
- Não depender de GitHub Environments/secrets por ambiente em repo privado Free sem confirmar elegibilidade.

## Recuperar conta
1. Usar recovery codes, passkey ou método alternativo cadastrado.
2. Se perder todos os métodos 2FA, seguir fluxo oficial; não prometer recuperação pelo suporte.
3. Se email/conta comprometidos, proteger email e revogar tokens/sessões suspeitos após recuperar acesso.
4. Revisar owners, membros, PATs, SSH/deploy keys, Apps, webhooks e workflows.

## Recuperar código
    git status --short
    git fsck --full
    git log --oneline --decorate -n 30
    git remote -v

Validar commit, branches e tags. Criar novo repositório somente após decisão do owner; publicar código revisado sem secrets. Não usar git push --mirror sem inspecionar refs e histórico. Remover segredo do histórico não substitui rotação. Recriar workflows com permissions mínimos, Actions pinadas a SHA revisado e sem secrets para fork não confiável.

## Pipeline
Confirmar branch padrão, permissões, Node/lockfile, Actions, scripts e secrets necessários. Recriar secrets manualmente de fonte autorizada; nunca recuperar de artifacts/logs. Executar CI com dados sintéticos; reconectar Vercel após verificar repo, branch e commit.

GitHub é fonte de código, não backup de banco nem cofre recuperável de segredos.

## Referências
- [GitHub 2FA recovery](https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/recovering-your-account-if-you-lose-your-2fa-credentials)
- [Download source archives](https://docs.github.com/en/repositories/working-with-files/using-files/downloading-source-code-archives)
- [Git fsck](https://git-scm.com/docs/git-fsck)
