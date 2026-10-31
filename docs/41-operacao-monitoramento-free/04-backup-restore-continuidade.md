# 04 — Backup, restore e continuidade

## Neon Free
Instant restore/PITR tem janela limitada (6 horas conforme documentação Free) e limite de histórico; não é backup externo independente. Verificar a janela e o estado da branch antes de depender de restore.

## Backup lógico
Planejar pg_dump periódico em ambiente seguro, criptografar o arquivo, limitar acesso, definir retenção e verificar checksum. Não armazenar dump em repositório Git ou artifact público.

Exemplo documental:

~~~bash
pg_dump "$DATABASE_URL" --no-owner --no-privileges -Fc -f backup.dump
~~~

O comando requer cliente PostgreSQL e conexão autorizada. DATABASE_URL não deve aparecer em logs ou histórico de shell compartilhado.

## Restore
1. Selecionar backup validado e confirmar origem/data/checksum.
2. Restaurar em database/branch isolada, nunca diretamente na produção durante teste.
3. Validar schema, constraints, contagem/consistência e aplicação.
4. Documentar RPO (perda máxima aceitável) e RTO (tempo de recuperação) realistas.
5. Aprovar janela e plano antes de substituir ambiente ativo.

## Continuidade
- Repositório Git é fonte do código e migrations, não dos dados.
- Guardar documentação de configuração sem secrets.
- Manter runbook de recuperação de conta Auth0 e acesso aos painéis.
- Documentar dependências e owners/responsáveis.
- Exercitar restore periodicamente em ambiente isolado, se quota permitir.

Nenhum backup ou restore foi executado nesta etapa.