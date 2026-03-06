# Summary

Documento de acompanhamento das features implementadas e planejadas do sistema **help-me**.

---

## Nível dos Técnicos (N1 / N2 / N3)

Técnicos possuem níveis hierárquicos que determinam quais chamados podem ser atribuídos a eles. O nível padrão ao criar um técnico é **N1**, e somente o **ADMIN** pode alterar o nível.

**Regras de atribuição por prioridade:**

| Nível | Prioridades permitidas |
|-------|------------------------|
| N1    | P4 e P5                |
| N2    | P2 e P3                |
| N3    | P1, P2, P3, P4 e P5    |

- [x] Enum `NivelTecnico`: `N1`, `N2`, `N3` — padrão `N1` ao criar técnico
- [x] Somente **ADMIN** pode alterar o nível de um técnico
- [x] Regras de atribuição por prioridade implementadas

---

## Prioridade dos Chamados (P1 – P5)

Chamados possuem prioridade definida no momento da abertura, com padrão **P4**. A reclassificação é permitida apenas para **ADMIN** e **TECNICO N3**.

| Prioridade | Descrição         |
|------------|-------------------|
| P1         | Alta Prioridade   |
| P2         | Urgente           |
| P3         | Urgente           |
| P4         | Baixa Prioridade  |
| P5         | Baixa Prioridade  |

- [x] Enum `PrioridadeChamado`: `P1` a `P5` — padrão `P4`
- [x] Reclassificação restrita a **ADMIN** e **TECNICO N3**

---

## Filas de Atendimento

Dois endpoints de fila separados por prioridade. Qualquer técnico pode transferir um chamado para qualquer outro, sem restrição de nível.

- [x] **Fila Baixa Prioridade** — chamados P4 e P5
- [x] **Fila Alta Prioridade** — chamados P1, P2 e P3

---

## Comentários nos Chamados

- [x] Sem restrição de perfil — `ADMIN`, `TECNICO` e `USUARIO` podem comentar
- [x] Suporte a comentários internos (visíveis apenas para ADMIN e TECNICO)
- [x] CRUD completo: criar, listar, editar e remover (soft delete)

---

## Anexos nos Chamados

Arquivos enviados são armazenados no **MinIO** com controle de tipo e tamanho. Downloads são feitos via URL assinada com validade de 10 minutos.

- [x] Armazenamento no **MinIO**
- [x] Limite de **5 MB** por arquivo
- [x] Tipos permitidos: `jpg`, `png`, `gif`, `webp`, `pdf`, `docx`, `xlsx`, `txt`, `csv`
- [x] Sem restrição de perfil para envio e download
- [x] Download via URL assinada (válida por 10 minutos)

---

## Listagem de Chamados

Endpoint `GET /api/chamados` com suporte completo a filtros, paginação e ordenação. O escopo é aplicado automaticamente por perfil do usuário autenticado.

- [x] Paginação via `pagina` e `limite`
- [x] Busca textual por OS, descrição, nome e e-mail do usuário
- [x] Filtros: `status`, `prioridade`, `tecnicoId`, `usuarioId`, `setor`, `servico`, `semTecnico`, `dataInicio`, `dataFim`
- [x] Ordenação por: `geradoEm`, `atualizadoEm`, `prioridade`, `status`, `OS`
- [x] Escopo automático: USUARIO vê apenas seus chamados, TECNICO vê apenas os atribuídos a ele

**Filtros disponíveis:**

| Parâmetro                | Exemplo                 | Quem pode usar  |
|--------------------------|-------------------------|-----------------|
| `status`                 | `ABERTO,EM_ATENDIMENTO` | todos           |
| `prioridade`             | `P1,P2`                 | todos           |
| `busca`                  | `INC0001` ou `maria`    | todos           |
| `servico`                | `Suporte`               | todos           |
| `dataInicio` / `dataFim` | `2026-01-01`            | todos           |
| `tecnicoId`              | `cuid...`               | ADMIN           |
| `usuarioId`              | `cuid...`               | ADMIN           |
| `setor`                  | `FINANCEIRO`            | ADMIN, TECNICO  |
| `semTecnico`             | `true`                  | ADMIN, TECNICO  |
| `ordenarPor`             | `prioridade`            | todos           |
| `ordem`                  | `asc` / `desc`          | todos           |
| `pagina`                 | `1`                     | todos           |
| `limite`                 | `20` (máx `100`)        | todos           |

---

## Notificações em Tempo Real

Arquitetura baseada em Kafka producer/consumer integrada ao Socket.IO para entrega em tempo real.

```
Evento                     Producer                        Consumer
─────────────────────────  ──────────────────────────────  ──────────────────────────
POST /abertura             publicarChamadoAberto           salvarNotificacoes (MongoDB)
PATCH /:id/status          publicarChamadoAtribuido        enviarEmails (Nodemailer)
PATCH /:id/transferir      publicarChamadoTransferido      emitirSocket (Socket.IO)
PATCH /:id/reabrir         publicarChamadoReaberto
PATCH /:id/prioridade      publicarPrioridadeAlterada
SLA Job (30min)            publicarSLAVencendo
```

**Endpoints da API de notificações:**

| Método   | Rota                                   | Descrição                                   |
|----------|----------------------------------------|---------------------------------------------|
| `GET`    | `/api/notificacoes`                    | Lista com paginação + contador de não lidas |
| `PATCH`  | `/api/notificacoes/:id/lida`           | Marca como lida                             |
| `PATCH`  | `/api/notificacoes/marcar-todas-lidas` | Marca todas como lidas                      |
| `DELETE` | `/api/notificacoes/:id`                | Remove notificação                          |

---

## Exemplos de Uso

### LISTAGEM

```
GET /api/chamados?pagina=1&limite=10&status=ABERTO&prioridade=P1,P2&ordenarPor=prioridade&ordem=asc
GET /api/chamados?busca=INC0001
GET /api/chamados?semTecnico=true&setor=FINANCEIRO
GET /api/chamados?dataInicio=2026-01-01&dataFim=2026-01-31
```

---

### ABERTURA

```
POST /api/chamados/abertura-chamado
Content-Type: multipart/form-data

descricao: "Computador não está ligando após queda de energia"
servico: "Suporte de Hardware"
arquivos: [foto.jpg]  (opcional)
```

---

### EDITAR CHAMADO

```
PATCH /api/chamados/:id
Content-Type: multipart/form-data

descricao: "Computador não liga, já verifiquei a tomada e o cabo"
```

---

### ATUALIZAR STATUS

```
PATCH /api/chamados/:id/status
Content-Type: application/json

{ "status": "EM_ATENDIMENTO" }

{ "status": "ENCERRADO", "descricaoEncerramento": "Problema resolvido, cabo estava danificado" }

{ "status": "CANCELADO" }
```

---

### ALTERAR PRIORIDADE

```
PATCH /api/chamados/:id/prioridade
Content-Type: application/json

{ "prioridade": "P1", "motivo": "Servidor de produção fora do ar" }
```

---

### TRANSFERIR

```
PATCH /api/chamados/:id/transferir
Content-Type: application/json

{
  "tecnicoNovoId": "cuid...",
  "motivo": "Chamado requer nível N3, fora da minha alçada"
}
```

---

### REABRIR

```
PATCH /api/chamados/:id/reabrir-chamado
Content-Type: application/json

{ "atualizacaoDescricao": "Problema voltou a ocorrer no mesmo equipamento" }
```

---

### CANCELAR

```
PATCH /api/chamados/:id/cancelar-chamado
Content-Type: application/json

{ "descricaoEncerramento": "Usuário resolveu o problema por conta própria" }
```

---

### HISTÓRICO

```
GET /api/chamados/:id/historico
```

---

### TRANSFERÊNCIAS

```
GET /api/chamados/:id/transferencias
```

---

### DELETE (soft)

```
DELETE /api/chamados/:id
```

### DELETE permanente

```
DELETE /api/chamados/:id?permanente=true
```

---

### ANEXOS — upload

```
POST /api/chamados/:id/anexos
Content-Type: multipart/form-data

arquivos: [print_erro.png, log.txt]
```

### ANEXOS — listar

```
GET /api/chamados/:id/anexos
```

### ANEXOS — download

```
GET /api/chamados/:id/anexos/:anexoId/download
```

### ANEXOS — remover

```
DELETE /api/chamados/:id/anexos/:anexoId
```

---

### COMENTÁRIOS — criar

```
POST /api/chamados/:id/comentarios
Content-Type: application/json

{ "comentario": "Já verificamos o equipamento, aguardando peça de reposição" }

{ "comentario": "Nota interna: cliente é reincidente", "visibilidadeInterna": true }
```

### COMENTÁRIOS — listar

```
GET /api/chamados/:id/comentarios
```

### COMENTÁRIOS — editar

```
PUT /api/chamados/:id/comentarios/:comentarioId
Content-Type: application/json

{ "comentario": "Peça chegou, iniciando reparo agora" }
```

### COMENTÁRIOS — remover

```
DELETE /api/chamados/:id/comentarios/:comentarioId
```

---

## Dashboard / Relatórios

Métricas persistidas no **InfluxDB** e visualizadas no **Grafana**.

**Chamados por Prioridade**
- [x] Gauges P1–P5 com thresholds de alerta (P1 fica vermelho a partir de 3 chamados abertos)
- [x] Gráfico de barras empilhadas com evolução por prioridade ao longo do tempo

**Desempenho por Técnico**
- [x] Tabela com: Técnico, Nível (N1/N2/N3 coloridos), Total, Resolvidos, Em Atendimento, Aguardando, Tempo Médio (h), SLA (%) com gradiente de cor
- [x] Gráfico de barras comparativo entre técnicos
- [x] Donut de distribuição por nível (N1 / N2 / N3)

**Tabela de Detalhamento**
- [x] Coluna Prioridade com cor (P1 vermelho → P5 azul)
- [x] Coluna Nível do técnico responsável
- [x] Ordenação por prioridade + tempo de abertura

**Filtros globais**
- [x] Filtro por Prioridade (P1–P5)
- [x] Fila Alta vs Fila Baixa — resumo visual comparativo
- [x] SLA por prioridade (P1 com SLA menor que P4)
- [x] Nível do técnico (N1/N2/N3) nos filtros de desempenho
