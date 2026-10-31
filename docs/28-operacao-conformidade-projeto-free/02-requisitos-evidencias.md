# Avaliação de requisitos e evidências

## Como instanciar a matriz

1. Obter a versão vigente da matriz mestra aprovada.
2. Copiar IDs, domínio, descrição e critério sem alterar o significado.
3. Adicionar contexto específico do projeto: ambiente, componente, responsável e aplicabilidade.
4. Definir estado, evidência necessária, validade e próxima revisão.
5. Manter referência à versão/commit da matriz de origem.

Não criar IDs duplicados nem substituir o requisito original por uma interpretação local.

## Campos por requisito

| Campo | Regra |
|---|---|
| ID de origem | Identificador preservado da matriz mestra |
| Ambiente/escopo | Componente e ambiente efetivamente avaliados |
| Aplicabilidade | Aplicável ou não aplicável com justificativa |
| Estado | Um dos estados padronizados |
| Responsável | Pessoa ou função responsável pela avaliação |
| Evidência | ID(s) do manifesto |
| Avaliado em | Data ISO 8601 |
| Validade | Data ou evento que invalida a evidência |
| Revisão seguinte | Data prevista |
| Observação | Limitações, dependências e resultado |
| Revisor | Pessoa distinta, quando viável |

## Requisitos mínimos de uma evidência

- Identificador único e título objetivo.
- Requisito(s) relacionado(s).
- Sistema, ambiente e escopo observados.
- Método de coleta e origem.
- Data/hora UTC e responsável.
- Versão/commit quando aplicável.
- Resultado observado, sem extrapolação.
- Integridade: hash opcional do artefato não secreto.
- Sanitização e classificação.
- Validade e gatilho de nova coleta.
- Localização controlada do artefato, sem URL pública para material restrito.

## Evidência aceitável e não aceitável

Aceitável: saída de comando sanitizada, configuração exportada sem segredo, captura de painel sem dados pessoais, resultado de teste com dados sintéticos, revisão de código com commit identificado.

Não aceitável: screenshot sem data/escopo, texto “funciona”, configuração proposta apresentada como real, token ou segredo, log bruto contendo dados pessoais, resultado de teste sem versão ou ambiente.

## Coleta segura

1. Preferir metadados e saídas mínimas.
2. Remover tokens, cookies, headers Authorization, connection strings, e-mails e identificadores pessoais.
3. Não anexar arquivos .env, dumps de produção ou dados reais.
4. Guardar artefatos em local com acesso restrito e retenção definida.
5. No repositório público, versionar somente metadados não sensíveis e exemplos fictícios.
6. Se houver exposição acidental, não reproduzir o segredo no registro; abrir incidente e seguir o procedimento de resposta.

## Validade

A evidência expira quando ocorrer mudança relevante de código, configuração, fornecedor, plano, quota, ambiente, ameaça ou requisito. Evidências de elegibilidade Free devem ser revalidadas antes de cada release. Definir periodicidade por criticidade; não tratar prazo sugerido como garantia.

## Regras de conclusão

- “Conforme” exige evidência válida que satisfaça o critério.
- “Parcial” exige descrever exatamente o que falta.
- “Não conforme” exige NC vinculada.
- “Bloqueado” exige dependência, responsável e data para reavaliação.
- “Não aplicável” exige justificativa e aprovação.
- Evidência expirada invalida o estado até reavaliação.
