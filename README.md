## v1.2.5

---

## O que mudou nesta versão

### Novo: Distributed Tracing com Grafana Tempo

Foi implementado o rastreamento distribuído completo utilizando **Grafana Tempo** integrado ao stack de observabilidade existente (Loki + Prometheus + Grafana).

**O que foi adicionado:**

- Coleta de traces via **OpenTelemetry** (OTLP gRPC na porta `4317` e HTTP na porta `4318`)
- Integração do Tempo como datasource no Grafana, permitindo correlacionar logs (Loki) com traces (Tempo) diretamente nos dashboards
- `tracingMiddleware` no Express para propagar `traceId` e `spanId` em todas as requisições HTTP
- Suporte a trace context propagation entre serviços (W3C TraceContext)
- `traceId` incluído nas respostas de erro para facilitar debugging em produção

**Stack de observabilidade atual:**

| Ferramenta | Função | Porta |
|------------|--------|-------|
| Grafana | Visualização | 3001 |
| Loki | Logs | 3100 |
| Tempo | Traces | 3200 |
| Prometheus | Métricas | 9090 |
| InfluxDB | Métricas de negócio | 8086 |

**Como usar:**

Acesse o Grafana em `http://localhost:3001`, navegue até **Explore** e selecione o datasource **Tempo**. Para correlacionar um trace com seus logs, clique no ícone de logs ao lado de qualquer span — o Grafana abre automaticamente o Loki filtrado pelo `traceId`.

```bash
# Verificar se o Tempo está recebendo traces
curl http://localhost:3200/ready

# Buscar um trace específico via API
curl "http://localhost:3200/api/traces/{traceId}"
```

---

### Correção: Falha no ingresso de tópico no Kafka

Foi corrigida uma falha que impedia o consumer Kafka de se inscrever corretamente em tópicos após reconexão ou restart do broker.

**Causa raiz:**

O consumer era instanciado uma única vez no módulo e reutilizado entre reconexões. Após uma desconexão, o estado interno do `kafkajs` marcava o consumer como encerrado, e a chamada de `subscribe` subsequente lançava um erro silencioso — o consumer ficava conectado mas não recebia mensagens.

**O que foi corrigido:**

- A instância do consumer agora é **recriada a cada `startConsumer`** em vez de reutilizada
- Adicionado reset de estado (`consumerInstance = null`) no bloco `catch` do `startConsumer`, garantindo que uma nova instância seja criada na próxima tentativa
- Adicionado `resetConsumerState()` para uso em testes e em cenários de restart controlado
- Opções de tuning adicionadas à criação do consumer para reduzir timeouts em ambientes instáveis:

```typescript
kafka.consumer({
  groupId: 'chamado-group',
  sessionTimeout: 60000,    // dobrado do padrão de 30s
  heartbeatInterval: 5000,
  maxWaitTimeInMs: 5000,
})
```

**Comportamento anterior vs corrigido:**

| Cenário | Antes | Depois |
|---------|-------|--------|
| Restart do broker | Consumer conecta mas não recebe mensagens | Consumer recria instância e resubscreve corretamente |
| Falha no `connect()` | Estado `isRunning` podia ficar `true` | Estado sempre resetado no `catch` |
| Re-start após erro | `subscribe` lançava erro silencioso | Nova instância criada, subscribe funciona |

---

## Como atualizar

```bash
# Parar os serviços
docker compose down

# Atualizar a imagem
docker pull seu-usuario/helpme-api:1.2.5

# Subir com Tempo habilitado
docker compose up -d
```

Certifique-se de que as portas `4317` e `4318` estão disponíveis no host — são usadas pelo Tempo para receber traces via OTLP.

---

## Variáveis de ambiente adicionadas

| Variável | Descrição | Padrão |
|----------|-----------|--------|
| `TEMPO_PORT` | Porta HTTP do Tempo | `3200` |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | Endpoint OTLP para envio de traces | `http://tempo_helpme:4317` |
| `OTEL_SERVICE_NAME` | Nome do serviço nos traces | `helpme-api` |

---

## Dependências atualizadas

| Pacote | Versão anterior | Versão atual |
|--------|----------------|--------------|
| `@opentelemetry/sdk-node` | — | `0.x` |
| `@opentelemetry/auto-instrumentations-node` | — | `0.x` |
| `@opentelemetry/exporter-trace-otlp-grpc` | — | `0.x` |
| `grafana/tempo` (Docker) | — | `2.4.0` |

---

## Problemas conhecidos

- Em ambientes com ESM/CommonJS mistos usando `tsx`, as auto-instrumentações do OpenTelemetry podem não capturar todos os spans automaticamente. O `tracingMiddleware` manual garante cobertura mínima das requisições HTTP enquanto esse cenário é investigado.

---

## Links

- [Documentação da API](http://localhost:3000/api-docs)
- [Grafana](http://localhost:3001)
- [Docker Hub](https://hub.docker.com/r/seu-usuario/helpme-api)
