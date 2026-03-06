Summary

Documento de acompanhamento das features implementadas e planejadas.

-----

## Implementação

### 1. Nível dos Técnicos (N1 / N2 / N3)

- [X] Enum `NivelTecnico`: `N1`, `N2`, `N3` — padrão `N1` ao criar técnico
- [X] Somente **ADMIN** pode alterar o nível de um técnico
- [X] Regras de atribuição por prioridade:

| Nível | Prioridades permitidas |
|-------|----------------------|
| N1 | P4 e P5 |
| N2 | P2 e P3 |
| N3 | P1, P2, P3, P4 e P5 (qualquer) |

---

### 2. Prioridade dos Chamados (P1 – P5)

- [X] Enum `PrioridadeChamado`: `P1` a `P5` — padrão `P4`

| Prioridade | Descrição |
|------------|-----------|
| P1 | Alta Prioridade |
| P2 | Urgente |
| P3 | Urgente |
| P4 | Baixa Prioridade |
| P5 | Baixa Prioridade |

- [X] Reclassificação permitida apenas para **ADMIN** e **TECNICO N3**

---

### 3. Filas de Atendimento

Dois endpoints de fila separados por prioridade:

- [X] **Fila Baixa Prioridade** — chamados P4 e P5
- [X] **Fila Alta Prioridade** — chamados P1, P2 e P3

Qualquer técnico pode transferir um chamado para qualquer outro, sem restrição de nível.

---

### 4. Comentários nos Chamados

- [X] Sem restrição de perfil — `ADMIN`, `TECNICO` e `USUARIO` podem comentar
- [X] Suporte a comentários internos (visíveis apenas para ADMIN e TECNICO)
- [X] CRUD completo: criar, listar, editar e remover (soft delete)

---

### 5. Anexos nos Chamados

- [X] Armazenamento no **MinIO**
- [X] Limite de **5 MB** por arquivo
- [X] Tipos permitidos: `jpg`, `png`, `gif`, `webp`, `pdf`, `docx`, `xlsx`, `txt`, `csv`
- [X] Sem restrição de perfil para envio e download
- [X] Download via URL assinada (válida por 10 minutos)

---

### 6. Listagem de Chamados

Endpoint `GET /api/chamados` com:

- [X] Paginação (`pagina`, `limite`)
- [X] Busca textual por OS, descrição, nome e email do usuário
- [X] Filtros: `status`, `prioridade`, `tecnicoId`, `usuarioId`, `setor`, `servico`, `semTecnico`, `dataInicio`, `dataFim`
- [X] Ordenação por: `geradoEm`, `atualizadoEm`, `prioridade`, `status`, `OS`
- [X] Escopo automático por perfil: USUARIO vê apenas seus chamados, TECNICO vê apenas os atribuídos a ele

---

### 7. Notificações em Tempo Real

Arquitetura Kafka producer/consumer com Socket.IO:

```
Evento                     Producer                       Consumer
──────────────────────     ─────────────────────────      ──────────────────────────
POST /abertura         →   publicarChamadoAberto       →  salvarNotificacoes (MongoDB)
PATCH /:id/status      →   publicarChamadoAtribuido    →  enviarEmails (Nodemailer)
PATCH /:id/transferir  →   publicarChamadoTransferido  →  emitirSocket (Socket.IO)
PATCH /:id/reabrir     →   publicarChamadoReaberto
PATCH /:id/prioridade  →   publicarPrioridadeAlterada
SLA Job (30min)        →   publicarSLAVencendo
```

Endpoints da API de notificações:

| Método | Rota | Descrição |
|--------|------|-----------|
| `GET` | `/api/notificacoes` | Lista com paginação + contador de não lidas |
| `PATCH` | `/api/notificacoes/:id/lida` | Marca como lida |
| `PATCH` | `/api/notificacoes/marcar-todas-lidas` | Marca todas como lidas |
| `DELETE` | `/api/notificacoes/:id` | Remove notificação |

---

## Planejadas

### Dashboard / Relatórios

Métricas de chamados por status, prioridade e técnico, persistidas no **InfluxDB** e visualizadas no **Grafana**.

**Seção: Chamados por Prioridade**
- [X] Gauges P1 a P5 com thresholds de alerta (P1 fica vermelho a partir de 3 chamados abertos)
- [X] Gráfico de barras empilhadas com evolução por prioridade ao longo do tempo

**Seção: Desempenho por Técnico**
- [X] Tabela com: Técnico, Nível (N1/N2/N3 coloridos), Total, Resolvidos, Em Atendimento, Aguardando, Tempo Médio (h), SLA (%) com gradiente de cor
- [X] Gráfico de barras comparativo entre técnicos
- [X]  Donut de distribuição por nível (N1 / N2 / N3)

**Seção: Tabela de Detalhamento**
- [X] Coluna Prioridade com cor (P1 vermelho → P5 azul)
- [X] Coluna Nível do técnico responsável
- [X] Ordenação por prioridade + tempo de abertura

**Filtros globais do dashboard**
- [X] Filtro por Prioridade (P1–P5)
- [X] Fila Alta vs Fila Baixa — resumo visual comparativo
- [X] SLA por prioridade (P1 com SLA menor que P4)
- [X] Nível do técnico (N1/N2/N3) nos filtros de desempenho
