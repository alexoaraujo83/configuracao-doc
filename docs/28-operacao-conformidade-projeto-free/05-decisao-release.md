# Gate de conformidade e decisão de release

## Escopo da decisão

Toda decisão é específica para projeto, versão/commit, ambiente e janela temporal. Não transferir aprovação de uma versão para outra sem avaliar mudanças.

## Pré-condições

- Cadastro do projeto atualizado.
- Escopo e finalidade documentados.
- Elegibilidade Free confirmada.
- Requisitos aplicáveis avaliados.
- Evidências críticas válidas e sanitizadas.
- NCs e exceções revisadas.
- Revisão de segurança e privacidade registrada.
- Plano de rollback/recuperação documentado e compatível com o plano Free.
- Responsável pela decisão identificado.

## Resultados possíveis

### GO

Somente quando requisitos obrigatórios e controles críticos estiverem conformes, evidências válidas, sem bloqueio de plano/termos e sem risco residual não aceito.

### GO condicionado

Somente para pendências não bloqueadoras, com justificativa, escopo limitado, responsável, prazo, condição de revogação e aprovação formal. Não usar para contornar requisito legal, exposição de segredo, controle crítico ausente ou incompatibilidade de plano.

### NO-GO

Obrigatório quando houver:
- incompatibilidade com termos ou elegibilidade Free;
- uso comercial em Vercel Hobby;
- requisito obrigatório não atendido;
- evidência crítica ausente, inválida ou expirada;
- NC crítica/alta não resolvida conforme política;
- exceção vencida;
- risco não aceito pelo responsável autorizado;
- impossibilidade de demonstrar isolamento ou autorização exigidos.

## Registro da decisão

Registrar ID, data/hora, projeto, ciclo, commit, ambiente, versão da matriz, resultado, justificativa, requisitos bloqueadores, NCs/exceções, aprovadores, condições e data de expiração. A decisão deve ser reavaliada após mudança relevante.

## Separação de responsabilidades

Quem preparou o release fornece evidências; responsável técnico avalia; proprietário do projeto aceita risco residual; revisor independente participa quando viável. A falta de segregação deve ser declarada.

## Limites do documento

A decisão registrada é governança documental. Não aciona pipeline, deploy, promoção, rollback ou alteração em fornecedor.
