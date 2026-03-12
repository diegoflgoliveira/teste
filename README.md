# v1.3.0

> **Branch:** `feature/sla-vinculos`  
> **Reviewers:** @diego-ferreira  
> **Labels:** `feature` `breaking-change` `migration`

---

## Contexto

Esta PR implementa dois sistemas independentes — **SLA** e **vinculação de chamados** — além de atualizar os seeds medium e big para os novos formatos do schema.

Os quatro itens são entregues juntos porque compartilham a migration que adiciona os campos `slaDeadline`, `slaViolado`, `slaVioladoEm`, `chamadoPaiId` e `chamadoFilhosIds` ao model `Chamado`.

---

## O que foi feito

### 1 — Calculadora de SLA para chamados críticos (`P1` / `P2`)

Arquivo: `src/domain/sla/sla.calculator.ts`

Calcula o `slaDeadline` considerando **janela de expediente** e **feriados**. Chamados críticos não pausam o relógio fora do expediente — o prazo corre 24/7.

| Prioridade | Prazo | Modo |
|---|---|---|
| P1 | 1 hora | Contínuo (24/7) |
| P2 | 4 horas | Contínuo (24/7) |

```ts
// Exemplo de uso
const deadline = slaCalculator.calcularDeadline(new Date(), 'P1');
// → agora + 1h, sem pausas
```

---

### 2 — Calculadora de SLA para chamados comuns (`P3`, `P4`, `P5`)

Mesmo arquivo: `src/domain/sla/sla.calculator.ts`

Chamados comuns **pausam** fora do expediente do técnico atribuído. O prazo desconta o tempo em que nenhum técnico está disponível.

| Prioridade | Prazo útil | Modo |
|---|---|---|
| P3 | 8 horas úteis | Expediente |
| P4 | 24 horas úteis | Expediente |
| P5 | 72 horas úteis | Expediente |

A lógica usa a tabela `expedientes` para determinar o horário de cada técnico. Se nenhum técnico está atribuído, usa expediente padrão `08:00–18:00`.

---

### 3 — Vinculação de chamados

Arquivos: `chamado.routes.ts`, `chamado.service.ts`

#### Endpoints adicionados

| Método | Rota | Descrição |
|---|---|---|
| `POST` | `/chamados/:id/vincular` | Vincula um chamado filho ao pai |
| `DELETE` | `/chamados/:id/vincular/:filhoId` | Remove vínculo |
| `GET` | `/chamados/:id/hierarquia` | Retorna a hierarquia completa |

#### Regras de negócio

- Hierarquia **ilimitada** — um filho pode ter filhos próprios
- Ao vincular, o filho é **encerrado automaticamente** com `descricaoEncerramento = "Chamado vinculado ao chamado {OS_PAI}"`
- Quando o pai é **encerrado ou cancelado**, todos os filhos diretos são encerrados em cascata
- Chamados com qualquer status podem ser vinculados como filho (incluindo já encerrados/cancelados)
- Um chamado não pode ser vinculado a si mesmo
- Um chamado não pode ser vinculado a um descendente seu (evita ciclo)

#### Campos adicionados ao model `Chamado` (migration)

```prisma
model Chamado {
  // ... campos existentes ...

  chamadoPaiId  String?   @map("chamado_pai_id")
  chamadoPai    Chamado?  @relation("hierarquia", fields: [chamadoPaiId], references: [id])
  chamadoFilhos Chamado[] @relation("hierarquia")

  vinculadoEm  DateTime? @map("vinculado_em")  @db.Timestamptz(3)
  vinculadoPor String?   @map("vinculado_por")

  slaDeadline   DateTime? @map("sla_deadline")   @db.Timestamptz(3)
  slaViolado    Boolean   @default(false)         @map("sla_violado")
  slaVioladoEm  DateTime? @map("sla_violado_em")  @db.Timestamptz(3)

  @@index([chamadoPaiId])
  @@index([slaDeadline])
  @@index([slaViolado])
  @@index([slaViolado, status])
}
```

---

### 4 — Seeds medium e big atualizados

Os seeds `seed-medium.ts` e `seed-big.ts` (500 chamados) foram atualizados para popular os novos campos:

- `slaDeadline` calculado com base em `geradoEm + SLA_HORAS[prioridade]`
- `slaViolado = true` nos chamados do grupo **VENCIDO**
- `slaVioladoEm` definido nos vencidos
- `chamadoPaiId` definido nas âncoras de hierarquia (PAI/FILHO)
- `vinculadoEm` e `vinculadoPor` definidos nos filhos

---

## Fluxo de ponta a ponta

```
POST /chamados
  └→ criarChamado()
       └→ sla.service.calcularEPersistir(chamadoId, prioridade, tecnicoId?)
            ├→ sla.calculator.calcularDeadline(now(), prioridade, expediente?)
            ├→ prisma.chamado.update({ slaDeadline })
            └→ kafka.publish("sla.calculado")

CRON */5 min
  └→ sla-checker.job
       └→ SELECT WHERE slaDeadline < now() AND slaViolado = false
            └→ UPDATE slaViolado=true, slaVioladoEm=now()
                 └→ kafka.publish("sla.violado")

POST /chamados/:id/vincular
  └→ vincularChamado(paiId, filhoId)
       ├→ validar: não cria ciclo, pai existe
       ├→ prisma.chamado.update filho: { chamadoPaiId, vinculadoEm, status: ENCERRADO, descricaoEncerramento }
       └→ kafka.publish("chamado.vinculado")

GET /chamados/:id
  └→ response inclui { slaDeadline, slaViolado, statusSLA }
       └→ statusSLA calculado em tempo real via sla.validator:
            "NO_PRAZO" | "VENCENDO" | "VENCIDO"
```

---

## Checklist de review

- [X] Migration reversível (`prisma migrate dev` + `prisma migrate reset` funcionando)
- [X] `slaDeadline` nunca nulo após criação de chamado
- [X] Chamados P1/P2 **não** pausam fora do expediente
- [X] Chamados P3/P4/P5 **pausam** corretamente fora do expediente
- [X] Vínculo cria histórico no MongoDB (`tipo: 'VINCULO'`)
- [X] Encerramento em cascata ao encerrar/cancelar pai
- [X] Ciclo de hierarquia bloqueado (A → B → A)
- [X] Seeds retornam `slaViolado: true` nos 112 chamados VENCIDO do seed big
- [X] Testes unitários: `sla.calculator.test.ts`, `sla.validator.test.ts`
- [X] Testes e2e: `chamado.e2e.test.ts` — blocos `vincular` e `hierarquia`

---

## Migration

```bash
pnpm prisma migrate dev --name sla_e_hierarquia_chamados
```

**Campos adicionados:**

| Campo | Tipo | Default | Nullable |
|---|---|---|---|
| `chamado_pai_id` | `uuid` | — | sim |
| `vinculado_em` | `timestamptz(3)` | — | sim |
| `vinculado_por` | `uuid` | — | sim |
| `sla_deadline` | `timestamptz(3)` | — | sim |
| `sla_violado` | `boolean` | `false` | não |
| `sla_violado_em` | `timestamptz(3)` | — | sim |

**Indexes adicionados:** `chamado_pai_id`, `sla_deadline`, `sla_violado`, `(sla_violado, status)`

> ⚠️ Migration **não destrutiva** — todos os novos campos são nullable ou têm default. Não há remoção de colunas.

---

## Arquivos modificados

```
prisma/
  migrations/
    XXXXXX_sla_e_hierarquia_chamados/
      migration.sql                      ← nova migration
  schema.prisma                          ← model Chamado atualizado

src/
  domain/
    sla/
      sla.calculator.ts                  ← novo
      sla.validator.ts                   ← novo
      sla.service.ts                     ← novo
      sla.config.ts                      ← novo
    chamado/
      chamado.service.ts                 ← vincularChamado(), encerrarCascata()
      chamado.routes.ts                  ← POST /:id/vincular, DELETE, GET hierarquia
  infrastructure/
    jobs/
      sla-checker.job.ts                 ← novo (cron */5min)
      sla.job.ts                         ← novo (cron diário/resumo)

prisma/
  seeds/
    seed-medium.ts                       ← atualizado: slaDeadline, vínculos
    seed-big.ts                          ← atualizado: 500 chamados com SLA
```

---

## Como testar localmente

```bash
# 1. Rodar a migration
pnpm prisma migrate dev

# 2. Popular com seed big
pnpm seed-big

# 3. Verificar chamados vencidos (deve retornar 112)
psql $DATABASE_URL -c "SELECT COUNT(*) FROM chamados WHERE sla_violado = true;"

# 4. Verificar hierarquia
psql $DATABASE_URL -c "SELECT os, chamado_pai_id FROM chamados WHERE chamado_pai_id IS NOT NULL LIMIT 10;"

# 5. Rodar testes
pnpm test src/__tests__/unit/domain/sla/
pnpm test src/__tests__/e2e/chamado.e2e.test.ts
```
