# Não conformidades, riscos e exceções

## Não conformidade (NC)

Abrir uma NC sempre que requisito aplicável não for atendido, evidência estiver ausente/inválida, controle falhar ou plano Free não suportar requisito obrigatório.

Campos:
- ID, título, data e projeto/ciclo.
- Requisito(s) e ambiente afetado.
- Descrição factual e evidência.
- Impacto e escopo conhecido.
- Severidade definida pela política do projeto.
- Causa conhecida ou “não determinada”.
- Ação corretiva, responsável e prazo.
- Critério verificável de encerramento.
- Revisor, data de validação e evidência de fechamento.
- Estado: aberta, em tratamento, aguardando validação, aceita como risco ou encerrada.

Não registrar hipótese como causa confirmada. Não realizar investigação intrusiva; este modelo é registro documental.

## Exceção

Exceção não apaga a NC nem altera o estado técnico do requisito. É uma aceitação temporária e explícita de risco residual por autoridade competente.

Campos obrigatórios:
- ID e NC relacionada.
- Requisito e escopo exato.
- Justificativa e necessidade de negócio.
- Risco residual e impacto potencial.
- Controles compensatórios realmente existentes e evidenciados.
- Responsável que solicita.
- Autoridade que aprova.
- Data de início e vencimento.
- Plano de encerramento e condição de revogação.
- Revisão periódica.

Não conceder exceção para violar termos do fornecedor, limites legais, exposição de credenciais, bypass de controles essenciais ou uso comercial incompatível com Vercel Hobby. Nesses casos, manter NO-GO.

## Escalonamento

- Crítico: bloquear release e escalar imediatamente ao proprietário.
- Alto: bloquear release até correção ou decisão formal de NO-GO; exceção somente se política e obrigações permitirem.
- Médio/baixo: registrar prazo e risco; release condicionada somente com aprovação e monitoramento documentados.

A severidade deve seguir a política interna do projeto; não inferir classificação sem contexto.

## Encerramento

Uma NC só é encerrada após evidência nova, válida, vinculada à versão/ambiente e revisada. Fechar a tarefa sem evidência não encerra a NC.

## Regra de expiração

Exceção vencida volta a bloquear o requisito e deve impedir decisão GO até renovação formal ou correção. Renovação exige nova análise, não extensão automática.
