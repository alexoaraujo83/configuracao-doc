# Next.js: logs estruturados e correlação

## Objetivo

Permitir que uma falha seja rastreada por request, deployment e operação sem gravar conteúdo sensível. O exemplo é um padrão de implementação; não foi instalado nem testado neste repositório.

## Convenção de evento

Emitir um objeto JSON por evento com campos estáveis:

| Campo | Finalidade |
|---|---|
| timestamp | instante UTC em ISO 8601 |
| level | debug, info, warn ou error |
| service | nome lógico da aplicação |
| environment | development, preview ou production |
| release | commit SHA curto ou identificador de deployment |
| requestId | identificador único por requisição |
| traceId | identificador de trace quando instrumentação estiver habilitada |
| method | método HTTP |
| route | rota normalizada, sem query string |
| statusCode | status HTTP |
| durationMs | duração em milissegundos |
| event | nome estável, por exemplo http.request.completed |
| errorCode | código interno não sensível |

Não incluir URL completa com query string, Authorization, Cookie, Set-Cookie, tokens, e-mail, telefone, corpo de requisição, SQL com valores, nem stack trace que possa conter secrets.

## Exemplo TypeScript

Criar uma função de logging server-only, sem dependência externa:

    type LogLevel = "info" | "warn" | "error";

    type LogEvent = {
      level: LogLevel;
      event: string;
      requestId?: string;
      traceId?: string;
      route?: string;
      method?: string;
      statusCode?: number;
      durationMs?: number;
      errorCode?: string;
      release?: string;
    };

    export function logEvent(entry: LogEvent) {
      const record = {
        timestamp: new Date().toISOString(),
        service: process.env.SERVICE_NAME ?? "web",
        environment: process.env.VERCEL_ENV ?? process.env.NODE_ENV ?? "development",
        release: process.env.VERCEL_GIT_COMMIT_SHA?.slice(0, 12) ?? "local",
        ...entry,
      };

      const line = JSON.stringify(record);
      if (entry.level === "error") console.error(line);
      else if (entry.level === "warn") console.warn(line);
      else console.info(line);
    }

Usar somente em código de servidor. Não importar em Client Components.

## Request ID

- Preferir request ID gerado pelo runtime/plataforma quando disponível e confiável.
- Se aceitar um identificador vindo do cliente, validar tamanho e charset e nunca confiar nele como identidade/autorização.
- Gerar UUID interno quando não houver ID confiável.
- Propagar o mesmo ID para logs da aplicação, chamadas internas e resposta HTTP por header documentado, como X-Request-Id.
- Não reutilizar requestId como userId, tenantId ou token de sessão.
- Para tarefas assíncronas, registrar correlationId de origem e um novo executionId.

## Instrumentation e OpenTelemetry

Next.js App Router suporta o arquivo instrumentation.ts na raiz ou em src, exportando register(), chamado na inicialização de uma instância do servidor. A instrumentação nativa do framework e OpenTelemetry ajudam a correlacionar spans, mas exportar traces para um backend externo pode exigir serviço adicional e não deve ser presumido gratuito.

Exemplo mínimo:

    export async function register() {
      if (process.env.NEXT_RUNTIME === "nodejs") {
        // Registrar instrumentação server-side compatível com o runtime.
      }
    }

Não inicializar SDKs específicos de Node em Edge Runtime. Seguir a documentação da versão efetiva do Next.js e do runtime Vercel.

## Métricas mínimas

- total de requests por rota normalizada e status;
- taxa de 5xx e 4xx;
- duração p50/p95 calculada a partir de amostra ou logs disponíveis;
- falhas de autenticação/autorização por código genérico;
- falhas de dependências externas;
- duração e resultado de operações críticas.

Sem backend de métricas pago, usar logs nativos da Vercel e amostras agregadas em relatórios manuais. Não transformar logs de produção em analytics de usuário.

## Segurança e privacidade

- Redigir segredos antes de serializar erros.
- Limitar tamanho de mensagens e stack traces.
- Evitar logging de objetos arbitrários recebidos de SDKs.
- Aplicar retenção mínima e acesso restrito aos painéis.
- Em incidente, preservar apenas evidência necessária e seguir política de privacidade.
- Não registrar identificadores pessoais se um identificador técnico pseudônimo for suficiente.

## Validação documental

Antes de liberar: revisar tipos de evento, garantir ausência de secrets em testes sintéticos, confirmar requestId ponta a ponta e verificar que logs não expõem dados em Preview/Production.

## Referências

- https://nextjs.org/docs/app/guides/instrumentation
- https://nextjs.org/docs/15/pages/guides/open-telemetry
- https://vercel.com/docs/logs/runtime
