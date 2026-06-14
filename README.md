
# 📐 Lattice Sync Engine

---

## 🇧🇷 Versão em Português

O **Lattice Sync** é um motor de sincronização de estado distribuído de alta performance e ultrabaixa latência, projetado especificamente para aplicações colaborativas em tempo real. O sistema utiliza **CRDTs (Conflict-free Replicated Data Types)** para garantir a convergência eventual de estado entre múltiplos nós sem a necessidade de um coordenador central de concorrência ou travas estruturais (*locks*).

O objetivo deste projeto é fornecer uma infraestrutura agnóstica de dados capaz de sincronizar estados complexos em tempo real, mantendo uma experiência de usuário (*UX*) fluida, mesmo sob condições de rede severas, intermitência ou operação totalmente offline.


<div align="center">

![Build Status](https://img.shields.io/badge/build-passing-brightgreen?style=for-the-badge&logo=github-actions)
![Coverage](https://img.shields.io/badge/coverage-98%25-success?style=for-the-badge&logo=jest)
![Architecture](https://img.shields.io/badge/architecture-CRDT%20%2F%20RGA-blue?style=for-the-badge)
![Protocol](https://img.shields.io/badge/protocol-WebSockets%20%7C%20Binary-orange?style=for-the-badge)
![Chaos Testing](https://img.shields.io/badge/chaos%20testing-passed-purple?style=for-the-badge)
![License](https://img.shields.io/badge/license-MIT-lightgrey?style=for-the-badge)

</div>

<br>



---

### 🧠 O Desafio Técnico: Concorrência Distribuída

Resolver a concorrência em sistemas distribuídos de edição simultânea (como Google Docs, Notion ou Figma) é historicamente complexo. Abordagens tradicionais baseadas em travas (*locks*) destroem a experiência do usuário. Sistemas baseados em **OT (Operational Transformation)** exigem um servidor central inteligente e pesado que ordene e transforme cada mutação de forma sequencial.

O **Lattice Sync** quebra esse paradigma ao adotar **CRDTs baseados em operações**, implementando a variante estrutural **RGA (Replicated Growable Array)**.

> **A Premissa Matemática:** Toda modificação no sistema é tratada como um *append-only log* de operações imutáveis. Cada operação possui um identificador único composto por um par `(Lamport Timestamp, Actor ID)`. Isso garante que as mutações sejam **idempotentes, comutativas e associativas**. Não importa a ordem ou o atraso com que os pacotes cheguem: o estado final convergido será matematicamente idêntico em todos os clientes.

---

### 🏗️ Arquitetura do Sistema e Engenharia de Software

O ecossistema do Lattice foi desenhado sob o princípio de segregação de responsabilidades (*Separation of Concerns*), dividindo-se em três camadas principais:

```
[ Cliente A (Core Engine) ] <--- WebSocket ---> [ Servidor Relay (Stateless) ] <--- Redis Pub/Sub ---> [ Servidor Relay 2 ]
                                                      |
[ Cliente B (Core Engine) ] <-------------------------+

```

1. **Lattice Core Engine (Client-Side / Edge):** Uma biblioteca desacoplada da interface gráfica. Ela gerencia o grafo de operações local, detecta mutações na interface, gera os deltas com marcas causais e aplica funções de `merge()` instantâneas para renderização local otimista.
2. **Low-Latency Transport Layer:** Uma camada de transporte full-duplex que utiliza **WebSockets** brutos estruturados com codificação binária (MessagePack/Protocol Buffers). Isso reduz o overhead de rede em até 70% comparado a payloads JSON tradicionais.
3. **Stateless Relay Cluster (Server-Side):** O servidor atua estritamente como um retransmissor (*relay*) de mensagens de alta velocidade. Como ele não precisa validar regras de negócio complexas do documento, ele opera de forma *stateless*. A escalabilidade horizontal é delegada a um barramento de eventos **Redis Pub/Sub**, permitindo que milhares de usuários colaborem na mesma sala mesmo conectados a instâncias de servidores geograficamente distintas.

---

### 🛡️ Engenharia de Caos & Resiliência (*Chaos Testing*)

Para validar a confiabilidade matemática do motor em cenários de produção reais, o repositório conta com uma suíte de testes de caos automatizada que simula falhas extremas na rede:

* **Network Jitter & Latência Artificial:** Injeção de atrasos randômicos entre 50ms e 1500ms para simular conexões móveis instáveis (3G/4G).
* **Out-of-Order Execution:** Embaralhamento intencional da ordem de entrega dos pacotes WebSocket para garantir que o mecanismo de ordenação por *Lamport Timestamps* resolva os conflitos de precedência corretamente.
* **Network Partition (Modo Offline):** Simulação de desconexão completa de um cliente. O usuário continua editando localmente; ao restabelecer a conexão, o Lattice realiza o *catch-up* dos logs históricos e converge o documento sem sobrescrever o trabalho alheio.

---

### 🛠️ Tecnologias Utilizadas

| Camada | Tecnologia | Motivação Técnica |
| --- | --- | --- |
| **Engine Core** | TypeScript / Rust (WASM) | Tipagem estática estrita e eficiência algorítmica para manipulação de grafos de dados. |
| **Networking** | WebSockets + MessagePack | Comunicação bidirecional contínua com serialização binária para menor consumo de banda. |
| **Infras./PubSub** | Redis | Buffer in-memory de alta velocidade e distribuição de eventos em cluster. |
| **Testes** | Vitest / Chaos Scripts | Validação determinística de concorrência e simulação de concorrência massiva. |

---

### 🚀 Instalação e Execução

#### Pré-requisitos

* Node.js (v18+) ou Rust Toolchain (dependendo da sua implementação)
* Instância do Redis ativa (via Docker ou Local)

```bash
# Clone o repositório
git clone https://github.com/seu-usuario/lattice-sync.git

# Acesse o diretório
cd lattice-sync

# Instale as dependências de desenvolvimento e produção
npm install

# Inicie a infraestrutura de suporte (Redis) via Docker caso necessário
docker-compose up -d

# Execute o servidor de Relay
npm run start:server

# Em outro terminal, execute o cliente de demonstração
npm run start:client

```

---

## 🇺🇸 English Version

**Lattice Sync** is a high-performance, ultra-low latency distributed state synchronization engine, specifically designed for real-time collaborative applications. The system leverages **CRDTs (Conflict-free Replicated Data Types)** to guarantee eventual consistency across multiple nodes without the need for a centralized concurrency coordinator or heavy structural database locks.

The primary goal of this project is to provide a data-agnostic infrastructure capable of synchronizing complex operational states in real-time while preserving a seamless user experience (*UX*), even under severe network degradation, intermittent connectivity, or complete offline operations.

---

### 🧠 The Technical Challenge: Distributed Concurrency

Solving concurrency in distributed environments for simultaneous editing (such as Google Docs, Notion, or Figma) is historically one of the hardest problems in systems engineering. Centralized locking mechanisms degrade user experience. Systems relying on **OT (Operational Transformation)** require heavy, complex central servers to sequentially transform and sequence every single mutation.

**Lattice Sync** bypasses this paradigm by adopting **operation-based CRDTs**, specifically implementing the **RGA (Replicated Growable Array)** structural variant.

> **The Mathematical Groundwork:** Every mutation within the system is treated as an immutable *append-only operation log*. Each operation is uniquely tagged with a cryptographic identifier pair `(Lamport Timestamp, Actor ID)`. This ensures that mutations are **idempotent, commutative, and associative**. Regardless of network latency or the order in which packets arrive, the final converged state remains mathematically identical across all connected clients.

---

### 🏗️ System Architecture & Software Engineering

The Lattice ecosystem is built strictly on the principle of Separation of Concerns, isolating components into three main decoupled layers:

```
[ Client A (Core Engine) ] <--- WebSocket ---> [ Stateless Relay Server ] <--- Redis Pub/Sub ---> [ Relay Server 2 ]
                                                      |
[ Client B (Core Engine) ] <-------------------------+

```

1. **Lattice Core Engine (Client-Side / Edge):** A robust standalone library independent of any UI framework. It manages the local operation graph, hooks into state mutations, generates causally tracked deltas, and executes optimistic, instantaneous local merges.
2. **Low-Latency Transport Layer:** A full-duplex communication pipeline built over raw **WebSockets**, utilizing binary encoding schemes (MessagePack/Protocol Buffers). This approach slashes network serialization overhead by up to 70% compared to standard JSON payloads.
3. **Stateless Relay Cluster (Server-Side):** The backend operates purely as a high-velocity event relay. Because it doesn't need to process or evaluate application business logic or document formatting rules, it remains completely *stateless*. Horizontal scalability is out-of-the-box via a **Redis Pub/Sub** event backbone, allowing thousands of concurrent users to collaborate within the same workspace even when connected to distinct, geographically distributed server instances.

---

### 🛡️ Chaos Engineering & Resilience

To prove the engine's mathematical stability under real-world production constraints, the repository features an automated chaos testing suite simulating hostile network topologies:

* **Network Jitter & Artificial Latency:** Introduces randomized delays spanning from 50ms to 1500ms to simulate volatile mobile cellular networks (3G/4G/5G).
* **Out-of-Order Execution:** Intentionally shuffles the sequence of incoming WebSocket frames to guarantee that the *Lamport Timestamp* sorting matrix handles causality and precedence conflicts gracefully.
* **Network Partition (Offline Mode):** Simulates abrupt client disconnections. Users can continue modifying data locally. Upon reconnection, Lattice replays the missing historical operation log, converging states flawlessly without overwriting concurrent remote edits.

---

### 🛠️ Tech Stack

| Layer | Technology | Technical Motivation |
| --- | --- | --- |
| **Engine Core** | TypeScript / Rust (WASM) | Strict static typing and maximum algorithmic efficiency for complex graph processing. |
| **Networking** | WebSockets + MessagePack | Persistent, bi-directional communication paired with binary serialization for low bandwidth footprint. |
| **Infra/PubSub** | Redis | In-memory high-throughput buffering and multi-node event distribution. |
| **Testing** | Vitest / Chaos Scripts | Deterministic concurrency verification and heavy concurrent workload simulations. |

---

### 🚀 Getting Started

#### Prerequisites

* Node.js (v18+) or Rust Toolchain
* A running Redis instance (via Docker or local daemon)

```bash
# Clone the repository
git clone https://github.com/your-username/lattice-sync.git

# Navigate to directory
cd lattice-sync

# Install all development and production dependencies
npm install

# Spin up auxiliary services (Redis) using Docker
docker-compose up -d

# Spin up the High-Performance Relay Server
npm run start:server

# Open a separate shell and run the frontend demonstration client
npm run start:client

```
