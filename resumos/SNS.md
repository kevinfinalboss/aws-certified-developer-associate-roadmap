<p align="center">
	<img src="./img/aws-icons/aws-SNS.png" alt="aws-sns-icon" style="height:150px; width:150px;" /> 
  <br />
	<h1 align="center">
    SNS - Simple Notification Service
  </h1>
  <h3 align="center">
    Guia Completo para AWS Developer Associate
  </h3>
</p>	

<br />

## :pushpin: Índice

- [Introdução](#introdução)
- [Como funciona?](#como-funciona)
  - [Publicação direta](#publicação-direta)
  - [Publicação através de serviços AWS](#publicação-através-de-serviços-aws)
- [Tipos de Tópicos](#tipos-de-tópicos)
  - [Standard Topics](#standard-topics)
  - [FIFO Topics](#fifo-topics)
- [Assinantes (Subscribers)](#assinantes-subscribers)
- [Conceitos Avançados](#conceitos-avançados)
  - [Filtragem de mensagens](#filtragem-de-mensagens)
  - [Message Attributes](#message-attributes)
  - [Delivery Retry Policy](#delivery-retry-policy)
  - [Dead Letter Queue](#dead-letter-queue)
- [Padrões de Integração](#padrões-de-integração)
  - [Fan-out Pattern](#fan-out-pattern)
  - [Application Integration](#application-integration)
- [Segurança](#segurança)
- [Monitoramento e Métricas](#monitoramento-e-métricas)
- [Mobile Push Notifications](#mobile-push-notifications)
- [Perguntas e Respostas para Certificação](#perguntas-e-respostas-para-certificação)
- [Cenários de Uso Comuns](#cenários-de-uso-comuns)
- [Boas Práticas](#boas-práticas)
- [Referências](#books-referências)

<br />

## Introdução

SNS é uma sigla para **Simple Notification Service**.

O produtor de eventos (*event producer*) enviará mensagens somente para um tópico do SNS, e vários recebedores de eventos (*subscriptions*) irão ouvir as notificações do tópico SNS.

Cada assinante (*subscriber*) irá receber todas as mensagens (a menos que use filtros), existindo um recurso para filtrar as mensagens baseado em atributos.

**Limites importantes:**
- 12.500.000 de assinantes por tópico
- Limite de 100.000 tópicos por região
- Tamanho máximo da mensagem: 256 KB
- Retenção de mensagem: Não há retenção (entrega imediata)

**Assinantes podem ser:**
- Email/Email-JSON
- HTTP/HTTPS endpoints
- AWS Lambda
- Amazon SQS
- Amazon Kinesis Data Firehose
- SMS
- Push notifications móveis (Apple, Google, Amazon, Microsoft)

O SNS possui integração com muitos serviços da AWS, permitindo que diversos serviços enviem dados ou notificações diretamente para o SNS.

<br />

## Como funciona?

### Publicação direta

Para publicar em um tópico use a SDK através da API `Publish`. A partir disso pode-se criar uma ou várias assinaturas.

**Fluxo básico:**
1. Criar tópico SNS
2. Criar assinaturas (subscriptions)
3. Publicar mensagens no tópico
4. SNS entrega mensagens para todos os assinantes

### Publicação através de serviços AWS

Muitos serviços AWS podem publicar automaticamente no SNS:
- CloudWatch Alarms
- AWS Config
- CloudFormation
- Auto Scaling Groups
- S3 Events
- Lambda (error notifications)
- RDS Events
- ElastiCache Events

<br />

## Tipos de Tópicos

### Standard Topics

- **High throughput:** Até 300.000 mensagens por segundo
- **At-least-once delivery:** Mensagens podem ser entregues mais de uma vez
- **Best-effort ordering:** Não garante ordem das mensagens
- **Suporte a todos os tipos de assinantes**

### FIFO Topics

- **Exactly-once delivery:** Cada mensagem é entregue exatamente uma vez
- **Message ordering:** Mantém ordem das mensagens
- **Throughput limitado:** Até 300 mensagens por segundo (3.000 com batching)
- **Suporte limitado:** Apenas SQS FIFO como assinante
- **Requer Message Group ID**
- **Nome deve terminar com .fifo**

<br />

## Assinantes (Subscribers)

### Email/Email-JSON
- **Email:** Formato texto simples
- **Email-JSON:** Formato JSON com metadados
- Confirmação manual necessária

### HTTP/HTTPS
- Endpoints web que recebem POST requests
- Confirmação automática ou manual
- Retry policy configurável
- Suporte a confirmação de assinatura

### AWS Lambda
- Invocação assíncrona
- Processamento serverless
- Retry automático
- DLQ support

### Amazon SQS
- Padrão Fan-out
- Desacoplamento e durabilidade
- Processamento assíncrono
- Buffer para picos de carga

### SMS
- Envio de mensagens de texto
- Opt-out automático
- Suporte internacional
- Cobrança por mensagem

### Mobile Push
- iOS, Android, Windows, macOS
- Device tokens necessários
- Payload customizável
- Suporte a sandboxes

<br />

## Conceitos Avançados

### Filtragem de mensagens

Por padrão os assinantes recebem todas as mensagens publicadas no tópico. Para receber um subconjunto de mensagens, um assinante deve ter uma **política de filtro**.

A política de filtro é um arquivo JSON usado para filtrar as mensagens que são enviadas para os assinantes do tópico SNS baseado em **message attributes**.

**Exemplo de filtro:**
```json
{
  "store": ["example_corp"],
  "price": [{"numeric": [">=", 100]}],
  "event": ["order-placed"]
}
```

**Operadores suportados:**
- String: exact match, anything-but, prefix, suffix
- Numeric: =, !=, <, <=, >, >=, between
- Null: is null, is not null
- Array: contains

### Message Attributes

Metadados da mensagem usados para filtragem:
- **Nome:** Até 256 caracteres
- **Tipo:** String, Number, Binary, String.Array, Number.Array
- **Valor:** Até 256 caracteres
- **Máximo:** 10 atributos por mensagem

### Delivery Retry Policy

Configuração de tentativas de entrega:
- **Immediate retry:** 3 tentativas sem delay
- **Pre-backoff phase:** 2 tentativas com delay mínimo
- **Backoff phase:** Delay exponencial até máximo
- **Post-backoff phase:** Delay máximo até expirar

### Dead Letter Queue

Para mensagens que falharam na entrega:
- **SQS DLQ:** Para análise de falhas
- **Configuração por assinante**
- **Separada da DLQ do SQS**

<br />

## Padrões de Integração

### Fan-out Pattern

**SNS + SQS:** Distribui uma mensagem para múltiplas filas
- Processamento independente
- Durabilidade das mensagens
- Escalabilidade individual
- Fault tolerance

**Vantagens:**
- Desacoplamento total
- Processamento paralelo
- Diferentes velocidades de processamento
- Reliability individual

### Application Integration

**Event-driven architecture:**
- Microservices communication
- Loose coupling
- Scalability
- Resilience

<br />

## Segurança

### Access Control
- **IAM Policies:** Controle granular
- **Topic Policies:** Resource-based policies
- **Subscription Policies:** Controle por assinante

### Encryption
- **In-transit:** HTTPS/TLS
- **At-rest:** KMS encryption
- **Message-level:** Client-side encryption

### VPC Endpoints
- Acesso privado sem internet
- Controle de rede
- Compliance requirements

<br />

## Monitoramento e Métricas

**CloudWatch Metrics:**
- `NumberOfMessagesPublished`
- `NumberOfNotificationsDelivered`
- `NumberOfNotificationsFailed`
- `NumberOfNotificationsFilteredOut`
- `PublishSize`

**CloudWatch Logs:**
- Delivery status logging
- Failed delivery analysis
- Success rate monitoring

**X-Ray:**
- Distributed tracing
- Performance analysis
- Error tracking

<br />

## Mobile Push Notifications

### Plataformas Suportadas
- **Apple Push Notification Service (APNS)**
- **Google Firebase Cloud Messaging (FCM)**
- **Amazon Device Messaging (ADM)**
- **Microsoft Push Notification Service (MPNS)**
- **Baidu Cloud Push**

### Configuração
1. Criar platform application
2. Registrar device tokens
3. Criar endpoints
4. Publicar mensagens

### Payload Customization
- Diferentes payloads por plataforma
- Message attributes para customização
- TTL (Time To Live) support

<br />

## Perguntas e Respostas para Certificação

### 1. SNS vs SQS

**P: Qual a principal diferença entre SNS e SQS?**

**R:** SNS é um serviço de **notificação push** (pub/sub) onde mensagens são enviadas imediatamente para todos os assinantes. SQS é um serviço de **fila** (queue) onde mensagens são armazenadas até serem puxadas pelos consumidores. SNS é 1:N (um para muitos), SQS é 1:1 (um para um).

### 2. Fan-out Pattern

**P: Como implementar o padrão Fan-out?**

**R:** Use SNS como publisher e múltiplas filas SQS como subscribers. Uma mensagem publicada no SNS é automaticamente enviada para todas as filas SQS subscritas, permitindo processamento independente e paralelo por diferentes aplicações.

### 3. Message Filtering

**P: Como funciona a filtragem de mensagens no SNS?**

**R:** Use **message attributes** na publicação e **filter policies** na assinatura. O filtro é um JSON que especifica quais valores dos atributos devem ser aceitos. Apenas mensagens que atendem aos critérios são entregues ao assinante, reduzindo custos e processamento desnecessário.

### 4. FIFO vs Standard

**P: Quando usar SNS FIFO?**

**R:** Use SNS FIFO quando precisar de:
- Ordem garantida das mensagens
- Exactly-once delivery
- Integração com SQS FIFO
- Processamento sequencial crítico
Limitação: Apenas SQS FIFO como assinante e menor throughput (300 TPS).

### 5. Delivery Retry

**P: Como funciona a política de retry no SNS?**

**R:** SNS tem retry automático em 4 fases:
1. **Immediate retry:** 3 tentativas sem delay
2. **Pre-backoff:** 2 tentativas com delay mínimo
3. **Backoff:** Delay exponencial
4. **Post-backoff:** Delay máximo até TTL expirar
Configure Dead Letter Queue para capturar falhas permanentes.

### 6. Mobile Push

**P: Como implementar push notifications multiplataforma?**

**R:** 
1. Crie platform applications para cada plataforma (APNS, FCM, etc.)
2. Registre device tokens
3. Crie endpoints SNS
4. Use message attributes para customizar payload por plataforma
5. Publique uma vez, SNS entrega para todas as plataformas

### 7. Security

**P: Como controlar acesso ao SNS?**

**R:** Use três níveis:
- **IAM Policies:** Controle de usuários/roles
- **Topic Policies:** Controle resource-based
- **Subscription Policies:** Controle por assinante
Combine com VPC Endpoints para acesso privado e KMS para encryption.

### 8. Integration Patterns

**P: Como integrar SNS com Lambda?**

**R:** SNS pode invocar Lambda assincronamente. Configure:
- Lambda como subscriber do tópico
- Permissions adequadas (SNS precisa de lambda:InvokeFunction)
- Error handling com DLQ
- Filtering para processar apenas mensagens relevantes

### 9. Cost Optimization

**P: Como reduzir custos no SNS?**

**R:**
- Use message filtering para reduzir entregas desnecessárias
- Implemente batching quando possível
- Configure TTL adequados
- Monitor failed deliveries
- Use SQS como buffer para reduzir re-deliveries

### 10. HTTP/HTTPS Endpoints

**P: Como configurar endpoints HTTP no SNS?**

**R:**
1. Endpoint deve estar acessível publicamente
2. Confirmar subscription (automático ou manual)
3. Processar mensagens POST do SNS
4. Validar signatures para segurança
5. Retornar HTTP 200 para confirmação

### 11. Error Handling

**P: Como tratar erros de entrega no SNS?**

**R:**
- Configure **Dead Letter Queue** para falhas permanentes
- Monitor CloudWatch metrics para failed deliveries
- Use retry policies apropriadas
- Implement circuit breaker patterns nos endpoints
- Log delivery status para análise

### 12. Cross-Region

**P: SNS funciona cross-region?**

**R:** SNS é regional, mas pode:
- Ter subscribers HTTP/HTTPS em outras regiões
- Integrar com serviços cross-region via endpoints
- Usar replication customizada com Lambda
- Não suporta cross-region replication nativa

### 13. Message Size

**P: Como lidar com mensagens grandes no SNS?**

**R:** Limite é 256KB. Para mensagens maiores:
- Armazene payload no S3
- Envie apenas referência/URL via SNS
- Use Extended Client Library (similar ao SQS)
- Implemente compression quando possível

### 14. Throughput Limits

**P: Quais os limites de throughput do SNS?**

**R:**
- **Standard:** 300.000 mensagens/segundo
- **FIFO:** 300 mensagens/segundo (3.000 com batching)
- **SMS:** Varia por região
- **Email:** 14 mensagens/segundo
- Solicite aumento de limits via AWS Support

### 15. Event-Driven Architecture

**P: Como usar SNS em arquiteturas event-driven?**

**R:** SNS é ideal como **event bus**:
- Services publicam events em tópicos
- Multiple subscribers processam events
- Use message attributes para event metadata
- Implement event sourcing patterns
- Combine com EventBridge para routing complexo

<br />

## Cenários de Uso Comuns

### 1. Notificações de Sistema
**Cenário:** CloudWatch Alarm → SNS → Email/SMS/Slack
**Uso:** Alertas de infraestrutura, monitoramento de aplicações

### 2. Processamento de Pedidos
**Cenário:** Order Service → SNS → [Inventory SQS, Payment SQS, Shipping SQS]
**Uso:** Desacoplamento de microservices, processamento paralelo

### 3. Upload de Arquivos
**Cenário:** S3 Upload → SNS → [Thumbnail Lambda, Virus Scan SQS, Analytics Firehose]
**Uso:** Processamento multimedia, análise de dados

### 4. Mobile Notifications
**Cenário:** Backend → SNS → [iOS APNS, Android FCM, Web Push]
**Uso:** Engajamento de usuários, notificações push

### 5. Compliance e Auditoria
**Cenário:** Config Changes → SNS → [Compliance Lambda, Audit SQS, Notification Email]
**Uso:** Governança, compliance automático

<br />

## Boas Práticas

### Design Patterns
- Use Fan-out para desacoplamento
- Implemente idempotência nos subscribers
- Use message attributes para metadata
- Mantenha payloads pequenos e estruturados

### Performance
- Configure retry policies adequadas
- Use batching quando possível
- Monitor métricas de delivery
- Implemente circuit breakers

### Security
- Aplique princípio do menor privilégio
- Use VPC endpoints para comunicação interna
- Valide signatures em endpoints HTTP
- Encrypt dados sensíveis

### Cost Management
- Use message filtering para reduzir custos
- Monitor failed deliveries
- Configure TTL apropriados
- Analise padrões de uso regularmente

### Monitoring
- Configure CloudWatch alarms
- Use X-Ray para tracing
- Log delivery status
- Monitore subscriber health

<br />

## :books: Referências

Para uma compreensão mais profunda sobre SNS recomendo a leitura da documentação oficial:

- [O que é o Amazon SNS?](https://docs.aws.amazon.com/pt_br/sns/latest/dg/welcome.html)
- [Fanout para filas do Amazon SQS](https://docs.aws.amazon.com/pt_br/sns/latest/dg/sns-sqs-as-subscriber.html)    
- [Filtragem de mensagens do Amazon SNS](https://docs.aws.amazon.com/pt_br/sns/latest/dg/sns-message-filtering.html)
- [Amazon SNS FIFO topics](https://docs.aws.amazon.com/sns/latest/dg/sns-fifo-topics.html)
- [Amazon SNS mobile push notifications](https://docs.aws.amazon.com/sns/latest/dg/sns-mobile-push-notifications.html)
- [Amazon SNS message delivery retry](https://docs.aws.amazon.com/sns/latest/dg/sns-message-delivery-retries.html)
- [AWS SNS Best Practices](https://docs.aws.amazon.com/sns/latest/dg/sns-best-practices.html)
