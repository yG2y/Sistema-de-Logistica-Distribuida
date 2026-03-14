# 🎬 Demonstrações em Vídeo — Sistema de Logística Distribuída

Esta pasta contém os vídeos de demonstração do sistema, cobrindo as três frentes do trabalho: Microsserviços, Cloud/Notificações e Mobile.

---

## 📹 Vídeos

### 1. 🔧 Microsserviços — Fluxo End-to-End
**[▶️ Assistir no YouTube](https://youtu.be/iIB_3QDwI-4)**

Demonstra a orquestração completa do ecossistema de logística distribuída, focando na resiliência da comunicação entre microsserviços e na validação de regras de negócio baseadas em geolocalização.

#### Funcionalidades Demonstradas
- **Gerenciamento de Ciclo de Vida de Pedidos** — Fluxo completo desde a criação (`Order Creation`) até a finalização (`Delivery Confirmation`).
- **Geofencing e Proximidade** — Validação em tempo real da posição do motorista em relação aos pontos de coleta e entrega.
- **Match de Disponibilidade** — Algoritmo de cancelamento automático para pedidos sem prestadores em raio de alcance definido.
- **Mensageria para Incidentes** — Registro e processamento assíncrono de eventos críticos via RabbitMQ.

#### Telas e Serviços em Exibição
| Componente | Papel no Vídeo |
|---|---|
| **Postman** | Interage com o API Gateway (`:8000`) simulando requisições com JWT Bearer Token |
| **Flutter Mobile App** | Interface do motorista: aceite de corridas, atualização de status, reporte de incidentes |
| **RabbitMQ Console** | Monitoramento das filas, demonstrando baixo acoplamento no reporte de incidentes |
| **Docker Desktop** | Orquestração dos containers (DBs + broker) |
| **Logs Spring Boot** | Visualização da comunicação inter-service e processamento de regras de negócio |

#### Fluxo End-to-End
1. **Bootstrapping & Auth** — Containers iniciados via Docker. Requisições passam pelo API Gateway com validação de tokens.
2. **Sincronização Geográfica** — Motorista (Flutter) envia coordenadas; serviço de logística atualiza disponibilidade.
3. **Processamento de Pedido (Sucesso)** — Pedido criado → motorista mais próximo identificado → aceite → status `AGUARDANDO_COLETA`.
4. **Geofencing** — Sistema bloqueia confirmação de coleta se coordenadas GPS não coincidirem com a origem do pedido (`400 Bad Request`).
5. **Incidente Assíncrono** — Motorista reporta incidente → microsserviço publica no RabbitMQ → Lambda consome e persiste logs sem bloquear a thread principal.
6. **Encerramento** — Motorista confirma entrega → estatísticas atualizadas → motorista liberado para novas demandas.

> 💡 **Estratégia de Teste:** Coordenadas internacionais (Berlim/Lisboa) foram utilizadas para contornar limitações de precisão de mapas em ambiente de desenvolvimento.

---

### 2. ☁️ Cloud — Notificações e Integração AWS
**[▶️ Assistir no YouTube](https://youtu.be/Z_I-dodPACo)**

Demonstra o ecossistema de notificações e campanhas integrado com serviços de nuvem (AWS SQS + Gmail), focando na arquitetura event-driven e observabilidade.

#### Funcionalidades Demonstradas
- **Orquestração Logística End-to-End** — Autenticação, criação de pedidos, aceite e finalização completos.
- **Sistema de Notificações e Campanhas** — Disparo de cupons e atualizações de status via Push (Mobile) e E-mail.
- **Observabilidade Cloud** — Monitoramento em tempo real das filas AWS SQS.
- **Geofencing** — Validação GPS para confirmação de coleta/entrega.

#### Telas e Serviços em Exibição
| Componente | Papel no Vídeo |
|---|---|
| **Postman** | Disparos ao API Gateway: autenticação JWT e gatilhos de campanhas de marketing |
| **Flutter Mobile App** | Notificações push em tempo real, lista de entregas atribuídas e atualizações de estado |
| **AWS Console (SQS)** | Monitoramento da fila `email-notifications` (taxa de recebimento e exclusão) |
| **VS Code / Terminal** | Infraestrutura Docker Compose e logs dos microsserviços Spring Boot |
| **Gmail** | Validação do recebimento de e-mails transacionais disparados pelo sistema |

#### Fluxo End-to-End
1. **Bootstrap & Auth** — Containers iniciados. Login via Postman retorna Bearer Token validado pelo Gateway.
2. **Preparação** — Cadastro de clientes e motoristas. Motorista loga no Flutter e atualiza disponibilidade geográfica.
3. **Ciclo do Pedido** — Pedido criado → microsserviço identifica motorista → notificação enviada → motorista aceita no mobile (`AGUARDANDO_COLETA`).
4. **Integração Cloud** — Gatilho de "Campanha de Cupons" publicado via Postman → mensagem vai para AWS SQS → serviço consome e dispara simultaneamente **Push** (Flutter) + **E-mail** (Gmail).
5. **Geofencing & Finalização** — Confirmação de entrega bloqueada longe do destino; ao chegar no local correto, status atualizado para `ENTREGUE`.
6. **Auditoria** — Verificação nos painéis AWS confirmando que todas as mensagens da fila foram processadas.

> 💡 **Destaques:** Arquitetura Event-Driven desacoplando e-mails da lógica principal; Internal Tokens para segurança inter-service.

---

### 3. 📱 Mobile — Interface Flutter do Motorista
**[▶️ Assistir no YouTube](https://youtu.be/YwyG8zP1-TU)**

Demonstra a integração do cliente mobile com o ecossistema de microsserviços, focando na jornada do motorista e no rastreamento de cargas em tempo real.

#### Funcionalidades Demonstradas
- **Gestão de Sessão e Autenticação** — Login seguro integrado ao microsserviço de usuários via API Gateway (JWT).
- **Controle de Disponibilidade** — Alteração do status do motorista para recebimento de novas demandas.
- **Gerenciamento de Entregas** — Visualização de ordens de serviço atribuídas e histórico.
- **Rastreamento Geográfico (Last-Mile)** — Mapa com marcadores dinâmicos de Origem (O), Veículo (V) e Destino (D).

#### Telas e Componentes Visíveis
| Tela | Descrição |
|---|---|
| **Login** | Campos e-mail/senha com integração backend para emissão de JWT |
| **Dashboard do Motorista** | Status atual, cards de "Entregas Atribuídas" com tipo de carga, distância e tempo estimado |
| **Rastreamento (Mapa)** | Marcadores dinâmicos O / V / D com integração à API de mapas |
| **Configurações** | Dark Mode, toggle de Notificações Push e E-mail (acionam filas RabbitMQ/SQS) |

#### Fluxo End-to-End no Mobile
1. **Autenticação** — Login consome `/auth` do API Gateway, que valida credenciais no microsserviço de usuários.
2. **Sincronização de Perfil** — App carrega estado do motorista ("Pedro Motorista") e busca entregas pendentes por ID.
3. **Configurações** — Alternância de notificações dispara requisições ao microsserviço de notificações para atualizar preferências de subscrição.
4. **Monitoramento de Entrega** — Pedido selecionado (ex: `Pedido #23 - Eletrônicos`) abre mapa consumindo coordenadas em tempo real do serviço de rastreamento.
5. **Atualização de Status** — Interações do motorista enviam eventos ao backend, transitando o estado do pedido no PostgreSQL e refletindo instantaneamente na UI.

> 💡 **Destaques:** Consumo de API reativo com atualizações fluidas; validação de HTTP 200 visível no console Flutter; Design System Material com estados coloridos por status.

---

## 🗺️ Mapa de Casos de Teste Demonstrados

Os vídeos cobrem os 4 casos de teste definidos no [`TESTES.md`](../TESTES.md):

| Caso de Teste | Vídeo |
|---|---|
| **CT-1** Fluxo Feliz (pedido criado → aceito → entregue) | Microsserviços + Cloud |
| **CT-2** Cancelamento por falta de motoristas (Lisboa) | Microsserviços |
| **CT-3** Reporte e resolução de incidentes (RabbitMQ) | Microsserviços + Cloud |
| **CT-4** Validações geográficas na coleta/entrega (Geofencing) | Microsserviços + Mobile |

---

## 🛠️ Stack Demonstrada

- **Backend:** Java Spring Boot · API Gateway (Spring Cloud) · PostgreSQL
- **Mensageria:** RabbitMQ · AWS SQS · Lambda Simulator (Python)
- **Mobile:** Flutter (Dart) · Geolocator · Google Maps
- **Infra:** Docker · Docker Compose
- **Testes:** Postman · Coordenadas Berlim (52.51°N) / Lisboa (38.73°N)
