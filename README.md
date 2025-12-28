# Help-Me API

<div align="center">

![Node.js](https://img.shields.io/badge/Node.js-18+-339933?style=for-the-badge&logo=node.js&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![Express](https://img.shields.io/badge/Express-4.18+-000000?style=for-the-badge&logo=express&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-5.0+-2D3748?style=for-the-badge&logo=prisma&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16+-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-7.0+-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-7.0+-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![Kafka](https://img.shields.io/badge/Apache%20Kafka-3.6+-231F20?style=for-the-badge&logo=apache-kafka&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-24.0+-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28+-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)

**Sistema completo de Helpdesk com arquitetura escalável e monitoramento em tempo real**

[Instalação](#-instalação) •
[Recursos](#-recursos) •
[Tecnologias](#-tecnologias) •
[API](#-api) •
[Monitoramento](#-monitoramento)

</div>

---

## 📋 Sobre o Projeto

**Help-Me API** é um sistema robusto de gerenciamento de helpdesk desenvolvido com Node.js e TypeScript. Permite que usuários abram chamados para problemas relacionados a tecnologia, técnicos recebam e gerenciem atendimentos, e administradores tenham visão completa através de dashboards no Grafana sobre todo o ambiente do sistema.

### 🎯 Principais Diferenciais

- **Arquitetura DDD** (Domain-Driven Design) para código escalável e manutenível
- **Banco de Dados Híbrido** com PostgreSQL e MongoDB
- **Mensageria Assíncrona** via Apache Kafka
- **Monitoramento Completo** com Grafana, Prometheus e InfluxDB
- **Cache Inteligente** com Redis
- **DevOps Ready** com Docker, Kubernetes e CI/CD automatizado

---

## ✨ Recursos

### 1. Autenticação e Autorização
- ✅ Sistema completo de autenticação JWT
- ✅ Controle de acesso baseado em roles (ADMIN, TECNICO, USUARIO)
- ✅ Refresh tokens para sessões prolongadas
- ✅ Proteção de rotas por perfil de usuário

### 2. Gestão de Usuários
- ✅ CRUD completo de usuários
- ✅ Gerenciamento de perfis e permissões
- ✅ Hash de senhas com bcrypt

### 3. Gestão de Chamados (Tickets)
- ✅ Abertura, atualização e fechamento de chamados
- ✅ Sistema de prioridades e status
- ✅ Atribuição automática de técnicos
- ✅ Histórico completo de alterações
- ✅ Anexo de arquivos e comentários

### 4. Gestão de Técnicos
- ✅ Cadastro e gerenciamento de técnicos
- ✅ Especialidades e áreas de atuação
- ✅ Disponibilidade e carga de trabalho
- ✅ Métricas de performance

### 5. Gestão de Serviços
- ✅ Catálogo de serviços disponíveis
- ✅ Categorização e classificação
- ✅ SLA por tipo de serviço
- ✅ Templates de resolução

---

## 🛠️ Tecnologias

### Backend & Runtime
- **Node.js 18+** - Runtime JavaScript de alta performance
- **TypeScript 5.0+** - Superset JavaScript com tipagem estática
- **Express.js 4.18+** - Framework web minimalista e flexível

### Banco de Dados & ORM
- **PostgreSQL 16+** - Banco relacional para dados transacionais
- **MongoDB 7.0+** - Banco NoSQL para auditoria e logs
- **Prisma 5.0+** - ORM moderno com type-safety completo

### Cache & Mensageria
- **Redis 7.0+** - Cache in-memory para otimização de performance
- **Apache Kafka 3.6+** - Streaming de eventos e mensageria assíncrona

### Monitoramento & Observabilidade
- **Grafana** - Visualização de métricas e dashboards
- **Prometheus** - Coleta e armazenamento de métricas
- **InfluxDB** - Banco de dados de séries temporais

### DevOps & Infraestrutura
- **Docker & Docker Compose** - Containerização de aplicações
- **Kubernetes** - Orquestração de containers
- **GitHub Actions** - CI/CD automatizado

### Por que essas tecnologias?

**Node.js + TypeScript**: Performance, ecossistema robusto e type-safety para reduzir bugs em produção.

**PostgreSQL + MongoDB**: Solução híbrida aproveitando o melhor dos dois mundos - ACID para transações críticas e flexibilidade para logs de auditoria.

**Redis + Kafka**: Redis para cache de queries frequentes (reduz latência) e Kafka para processamento assíncrono escalável (desacopla serviços).

**Prisma**: ORM type-safe que elimina SQL injection, gera migrations automáticas e oferece excelente DX (Developer Experience).

**Stack de Monitoramento**: Grafana + Prometheus + InfluxDB é o padrão da indústria para observabilidade, permitindo identificar problemas antes que afetem usuários.

**Docker + Kubernetes**: Garante consistência entre ambientes (dev/staging/prod) e permite escalabilidade horizontal automática.

---

## 🚀 Instalação

### Pré-requisitos

- Node.js 18+
- Docker & Docker Compose

### Instalação Rápida
```bash
# Clone o repositório
git clone https://github.com/seu-usuario/help-me-api.git
cd help-me-api

# Configure as variáveis de ambiente
cp .env.example .env
# Edite o arquivo .env com suas configurações

# Inicie todos os serviços com Docker
docker-compose up -d

# Execute as migrações do banco
docker-compose exec api npm run prisma:migrate

# (Opcional) Popule o banco com dados iniciais
docker-compose exec api npm run seed
```

A API estará disponível em `http://localhost:3000`

### Instalação Manual (sem Docker)
```bash
# Instale as dependências
npm install

# Configure o banco de dados
npm run prisma:generate
npm run prisma:migrate

# Inicie em desenvolvimento
npm run dev
```

---

## 📚 Documentação da API

### Swagger UI

Acesse a documentação interativa completa em:
```
http://localhost:3000/api-docs
```

### Principais Endpoints

#### Autenticação
```http
POST   /api/v1/auth/login
POST   /api/v1/auth/refresh
POST   /api/v1/auth/logout
```

#### Usuários
```http
GET    /api/v1/users
POST   /api/v1/users
PUT    /api/v1/users/:id
DELETE /api/v1/users/:id
```

#### Chamados
```http
GET    /api/v1/tickets
POST   /api/v1/tickets
PUT    /api/v1/tickets/:id
PATCH  /api/v1/tickets/:id/status
POST   /api/v1/tickets/:id/comments
```

#### Técnicos
```http
GET    /api/v1/technicians
POST   /api/v1/technicians
GET    /api/v1/technicians/:id/metrics
```

#### Serviços
```http
GET    /api/v1/services
POST   /api/v1/services
PUT    /api/v1/services/:id
```

---

## 📊 Monitoramento

### Dashboards Grafana

O sistema inclui dashboards pré-configurados:

**Dashboard de Chamados**
- Total de chamados abertos/fechados
- Tempo médio de resolução
- Taxa de cumprimento de SLA
- Distribuição por prioridade

**Dashboard de Técnicos**
- Performance individual
- Carga de trabalho
- Taxa de resolução
- Tempo médio de atendimento

**Dashboard de Sistema**
- Métricas de API (latência, throughput)
- Saúde dos serviços
- Uso de recursos (CPU, memória)

### Acesso
```
Grafana:    http://localhost:3001
Prometheus: http://localhost:9090
InfluxDB:   http://localhost:8086
```

Credenciais padrão: `admin / admin`

---

## 🧪 Testes
```bash
# Testes unitários
npm run test

# Testes de integração
npm run test:integration

# Testes E2E
npm run test:e2e

# Cobertura de testes
npm run test:coverage
```

---

## 🚢 Deploy

### Docker Compose
```bash
docker-compose -f docker-compose.yml up -d
```

### Kubernetes
```bash
kubectl apply -f k8s/
kubectl get pods -n helpme-api
```

---

## 📝 Licença

Este projeto está sob a licença MIT.

---

## 👥 Autor

**Diego Ferreira** - Desenvolvimento e Arquitetura

---

<div align="center">

**[⬆ Voltar ao topo](#help-me-api)**

Feito com ❤️ e ☕

</div>
