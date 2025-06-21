<p align="center">
	<img src="./img/aws-icons/aws-SQS.png" alt="aws-sqs-icon" style="height:150px; width:150px;" /> 
  <br />
	<h1 align="center">
    SQS - Simple Queue Service
  </h1>
  <h3 align="center">
    Guia Completo para AWS Developer Associate
  </h3>
</p>	

<br />

## :pushpin: Índice

- [Introdução](#introdução)
- [O que é uma fila?](#o-que-é-uma-fila)
  - [Produtores (Producers)](#produtores-producers)
  - [Consumidores (Consumers)](#consumidores-consumers)
- [Tipos de filas](#tipos-de-filas)
  - [Filas Padrão (Standard Queue)](#filas-padrão-standard-queue)
  - [Filas FIFO](#filas-fifo-first-in-first-out-queue)
- [Conceitos Avançados](#conceitos-avançados)
  - [Fila de Mensagem Morta (Dead Letter Queue)](#fila-de-mensagem-morta-dead-letter-queue)
  - [Fila de Atraso (Delay Queue)](#fila-de-atraso-delay-queue)
  - [Sondagem Curta e Longa (Short and Long Polling)](#sondagem-curta-e-longa-short-and-long-polling)
  - [Message Visibility Timeout](#message-visibility-timeout)
  - [Message Deduplication](#message-deduplication)
  - [Message Group ID](#message-group-id)
- [Integração com Outros Serviços](#integração-com-outros-serviços)
- [Monitoramento e Métricas](#monitoramento-e-métricas)
- [Segurança](#segurança)
- [Perguntas e Respostas para Certificação](#perguntas-e-respostas-para-certificação)
- [Cenários de Uso Comuns](#cenários-de-uso-comuns)
- [Boas Práticas](#boas-práticas)
- [Referências](#books-referências)

<br />

## Introdução

**SQS** é uma sigla para **Simple Queue Service**.

É um sistema de armazenamento de fila/processos que devem ser executados, **ele funciona 100% em pull**, isso significa que todos os serviços puxam informação do SQS.

As **mensagens dos SQS tem um tamanho máximo de 256KB** e podem ser armazenadas entre 1 minuto para até 14 dias (por padrão ela fica 4 dias). Mas é possível armazenar o conteúdo de mensagens maiores que 256 KB usando S3 através do *SQS Extended Client* que é uma biblioteca em Java que pode ser implementada em outras linguagens ou DynamoDB, o SQS criará um ponteiro no objeto do Amazon S3.

**Ponto-chave para a prova:** Se você observar sobre **desacoplamento de aplicativos** pense em SQS.

## O que é uma fila?

Existe uma fila SQS e ela conterá mensagens. Para conter mensagens, alguma coisa precisa enviar essas mensagens para a fila do SQS e tudo que envia uma mensagem para o SQS é chamado de **Produtor (Producer)**. É possível ter um único produtor ou múltiplos produtores enviando mensagens para uma fila SQS. E alguma coisa precisa receber as mensagens para processá-las e estes são chamados de **Consumidores (consumers)**.

### Produtores (Producers)

Os produtores enviarão as mensagens usando a SDK. A mensagem será persistida no SQS enquanto o consumidor não a deletar. As mensagens estarão armazenadas no SQS entre 1 minuto para até 14 dias (por padrão ela fica 4 dias até no máximo 14 dias).

**Atributos importantes da mensagem:**
- Message Body (até 256KB)
- Message Attributes (metadados opcionais)
- Delay Seconds (0-900 segundos)
- Message System Attributes

### Consumidores (Consumers)

Os consumidores são aplicações (software) e eles podem estar sendo executados em instâncias EC2, servidores, Lambda, ECS, etc. É ele que possui a responsabilidade de processar as mensagens.

- Recebem até 10 mensagens por vez da fila no SQS
- Apagam a mensagem na fila do SQS após finalizar o processamento dela (`DeleteMessage` API)
- Quando uma mensagem é pega pelo consumidor ela se torna invisível para outros consumidores (*Message Visibility Timeout*), por padrão o tempo de expiração de invisibilidade da mensagem é de 30 segundos

<br />

## Tipos de filas

### Filas Padrão (Standard Queue)

**Filas padrão (Standard Queue):** Aqui é onde tem o *high throughput* (**TPS**: *transactions per second*) você pode enviar quantas mensagens quiser por segundo e a fila pode armazenar um número ilimitado de transações (*nearly-unlimited*). 

Garante que a mensagem ou processo vai ser entregue no mínimo uma vez. É a oferta mais antiga da AWS, foi um dos primeiros serviços na AWS.

**Características:**
- **Taxa de transferência ilimitada**, **número ilimitado de mensagens na fila**
- Retenção padrão de mensagens: 4 dias, máximo 14 dias
- Baixa latência (< 10ms na publicação e recebimento)
- Limite de envio de 256KB por mensagem
- **Entrega pelo menos uma vez (At-Least-Once Delivery):** Uma mensagem é entregue pelo menos uma vez, mas às vezes, mais de uma cópia da mensagem é entregue. Por isso é possível ter mensagens duplicadas
- **Melhor Ordenação Possível (Best-Effort Ordering):** As mensagens podem ser entregues em uma ordem diferente do que elas foram enviadas

### Filas FIFO (First in, First Out Queue)

As filas FIFO (Primeiro a entrar, primeiro a sair) garante que o primeiro que chegar vai ser o primeiro que vai sair, isso significa que as mensagens são ordenadas, **a primeira mensagem que chegar na fila será a primeira a sair da fila** e também o **one time processing**, ou seja, uma mensagem será processada somente uma vez diferente do tipo de fila *Standard Queue*. Com isso existe a limitação de 300 TPS (ou 3000 com batching).

**Características:**
- **Taxa de transferência LIMITADA** de 300 TPS (ou 3000 com batching)
- **Exactly-once:** A mensagem será processada somente uma vez garantindo assim que não haverá duplicidades
- As mensagens são processadas pelo consumidor na mesma ordem em que foram enviadas pelo produtor
- Requer **Message Deduplication ID** e **Message Group ID**
- Nome da fila deve terminar com `.fifo`

<br />

## Conceitos Avançados

### Fila de Mensagem Morta (Dead Letter Queue)

Se o consumidor processar uma mensagem e falhar, a mensagem voltará para a fila do SQS. O mesmo consumidor ou outro pode tentar processá-la e falhar novamente. Para impedir loops de falha, existe o conceito de **Fila de Mensagem Morta (Dead Letter Queue)** onde é definido um limite (maxReceiveCount) para a quantidade de vezes em que o processamento de uma mensagem pode falhar. Depois desse limite ser excedido, a mensagem vai para o DLQ, útil para debugging.

### Fila de Atraso (Delay Queue)

Essa fila deve atrasar as mensagens para que os consumidores não as vejam imediatamente. O atraso pode ser de até 15 minutos (900 segundos). Por padrão o parâmetro de atraso é de zero segundos e esse período pode ser sobrescrito por uma mensagem específica usando o parâmetro `DelaySeconds`.

### Sondagem Curta e Longa (Short and Long Polling)

- **Sondagem longa:** Quando um consumidor solicita mensagens da fila no SQS, ele pode opcionalmente aguardar a chegada das mensagens se não houver nenhuma na fila. Isso traz o benefício de diminuir chamadas de API na fila do SQS e portanto há uma **diminuição de custo**. A sondagem longa pode ser definida em 1 até 20 segundos usando o parâmetro `ReceiveMessageWaitTimeSeconds`
- **Sondagem curta:** Quando um consumidor solicita mensagens da fila, o SQS responde imediatamente mesmo que na consulta não exista qualquer mensagem. Por padrão, as filas usam a sondagem curta

### Message Visibility Timeout

Quando uma mensagem é recebida por um consumidor, ela fica invisível para outros consumidores durante o período de *Visibility Timeout* (30 segundos por padrão, pode ser de 0 segundos a 12 horas). Se o processamento não for concluído dentro deste tempo, a mensagem voltará a ficar visível na fila.

**APIs importantes:**
- `ChangeMessageVisibility` - altera o timeout de uma mensagem específica
- `ChangeMessageVisibilityBatch` - altera em lote

### Message Deduplication

Para filas FIFO, o SQS usa um ID de desduplicação para evitar mensagens duplicadas:
- **Content-based deduplication:** SQS usa SHA-256 hash do body da mensagem
- **Explicit deduplication:** Você fornece um `MessageDeduplicationId` único

### Message Group ID

Para filas FIFO, mensagens com o mesmo `MessageGroupId` são processadas em ordem, mas diferentes grupos podem ser processados em paralelo.

<br />

## Integração com Outros Serviços

### SNS + SQS (Fan-out Pattern)
- SNS entrega mensagens para múltiplas filas SQS
- Garante processamento independente e paralelo
- Útil para desacoplamento de microsserviços

### Lambda + SQS
- Lambda pode ser triggerizado por mensagens SQS
- Processamento serverless e automático
- Configuração de batch size (1-10 mensagens)

### Auto Scaling com SQS
- Usar métricas do CloudWatch (ApproximateNumberOfMessages)
- Escalar EC2 instances baseado no tamanho da fila

<br />

## Monitoramento e Métricas

**Métricas importantes no CloudWatch:**
- `ApproximateNumberOfMessages` - número aproximado de mensagens na fila
- `ApproximateNumberOfMessagesDelayed` - mensagens com delay
- `ApproximateNumberOfMessagesNotVisible` - mensagens sendo processadas
- `NumberOfMessagesSent` - mensagens enviadas
- `NumberOfMessagesReceived` - mensagens recebidas
- `NumberOfMessagesDeleted` - mensagens deletadas

<br />

## Segurança

### Access Control
- **IAM Policies** - controle granular de acesso
- **SQS Access Policies** - políticas baseadas em recursos
- **VPC Endpoints** - acesso privado sem internet

### Encryption
- **Encryption in transit** - HTTPS
- **Encryption at rest** - SQS KMS
- **Client-side encryption** - SDK encryption client

<br />

## Perguntas e Respostas para Certificação

### 1. Diferenças entre Standard e FIFO

**P: Qual a principal diferença entre filas Standard e FIFO?**

**R:** 
- **Standard:** Alta performance (throughput ilimitado), at-least-once delivery, best-effort ordering, possibilidade de duplicatas
- **FIFO:** Ordenação garantida, exactly-once processing, throughput limitado (300 TPS ou 3000 com batching), nome deve terminar com .fifo

### 2. Message Visibility Timeout

**P: O que acontece se um consumidor SQS não deletar uma mensagem após processá-la?**

**R:** A mensagem volta a ficar visível na fila após o *Visibility Timeout* expirar (30s por padrão). Outros consumidores poderão recebê-la novamente, causando reprocessamento. Por isso é crucial sempre deletar mensagens após processamento bem-sucedido.

### 3. Dead Letter Queue

**P: Quando usar Dead Letter Queue?**

**R:** Use DLQ quando quiser capturar mensagens que falharam no processamento múltiplas vezes. Configure o `maxReceiveCount` (ex: 3 tentativas). Mensagens que excederem esse limite vão para a DLQ, permitindo análise e debugging sem loops infinitos.

### 4. Long Polling vs Short Polling

**P: Qual a vantagem do Long Polling?**

**R:** Long Polling reduz custos e latência ao aguardar até 20 segundos por mensagens, evitando consultas vazias frequentes. Short Polling retorna imediatamente (mesmo sem mensagens), gerando mais chamadas de API e maior custo.

### 5. Batching

**P: Como otimizar performance no SQS?**

**R:** Use batching para enviar/receber até 10 mensagens por vez com `SendMessageBatch` e `ReceiveMessage`. Para FIFO, batching pode aumentar throughput de 300 para 3000 TPS.

### 6. Mensagens Grandes

**P: Como lidar com mensagens maiores que 256KB?**

**R:** Use SQS Extended Client Library que armazena o payload no S3 e mantém apenas uma referência na mensagem SQS. Suporta mensagens até 2GB.

### 7. Delay Queue

**P: Diferença entre Delay Queue e Message Timer?**

**R:** 
- **Delay Queue:** Configuração no nível da fila, afeta todas as mensagens
- **Message Timer:** Parâmetro `DelaySeconds` por mensagem individual, sobrescreve configuração da fila

### 8. FIFO Features

**P: O que são Message Group ID e Deduplication ID?**

**R:**
- **Message Group ID:** Agrupa mensagens para processamento ordenado. Mensagens do mesmo grupo são processadas sequencialmente
- **Deduplication ID:** Evita duplicatas em janela de 5 minutos. Pode ser baseado em conteúdo ou fornecido explicitamente

### 9. Auto Scaling

**P: Como implementar auto scaling baseado em SQS?**

**R:** Use métricas CloudWatch como `ApproximateNumberOfMessages` para criar alarmes que triggeram Auto Scaling Groups. Configure scale-out quando fila cresce e scale-in quando diminui.

### 10. Integração SNS-SQS

**P: Vantagens do padrão Fan-out com SNS+SQS?**

**R:** Permite que uma mensagem SNS seja entregue para múltiplas filas SQS simultaneamente. Cada fila pode ter consumidores independentes, garantindo processamento paralelo e fault-tolerance.

### 11. Lambda Integration

**P: Configurações importantes para Lambda com SQS?**

**R:**
- **Batch Size:** 1-10 mensagens por invocação
- **Maximum Batching Window:** Aguarda até X segundos para formar batch
- **Reserved Concurrency:** Controla paralelismo
- **DLQ:** Para mensagens que falharam no Lambda

### 12. Security

**P: Como implementar encryption no SQS?**

**R:**
- **In-transit:** HTTPS (padrão)
- **At-rest:** SQS KMS usando AWS managed ou customer managed keys
- **Client-side:** SQS Extended Client Library com encryption

### 13. Troubleshooting

**P: Mensagem não aparece na fila após envio?**

**R:** Verifique:
- Delay Queue settings
- Message ainda no Visibility Timeout
- Permissões IAM
- Configuração de região

### 14. Cost Optimization

**P: Como reduzir custos no SQS?**

**R:**
- Use Long Polling para reduzir empty receives
- Implemente batching para reduzir requests
- Configure DLQ para evitar loops infinitos
- Monitore métricas para otimizar polling frequency

### 15. Cross-Region

**P: SQS suporta cross-region replication?**

**R:** Não nativamente. SQS é regional. Para cenários cross-region, use SNS para distribuir para múltiplas regiões ou implemente replicação customizada via Lambda.

<br />

## Cenários de Uso Comuns

### 1. Processamento de Imagens
**Cenário:** Upload de imagem → fila SQS → Lambda processa → salva no S3

### 2. Order Processing
**Cenário:** FIFO queue para garantir ordem de processamento de pedidos

### 3. Email Notifications
**Cenário:** SNS → múltiplas filas SQS → diferentes tipos de notificação

### 4. Batch Processing
**Cenário:** Grandes volumes de dados → SQS → EC2 Auto Scaling

### 5. Microservices Communication
**Cenário:** Desacoplamento entre serviços usando SQS como buffer

<br />

## Boas Práticas

### Performance
- Use batching sempre que possível
- Implemente Long Polling
- Configure Visibility Timeout adequadamente
- Use múltiplos consumidores para paralelismo

### Reliability
- Configure Dead Letter Queues
- Implemente retry logic com exponential backoff
- Monitore métricas do CloudWatch
- Use health checks nos consumidores

### Security
- Aplique princípio do menor privilégio
- Use encryption para dados sensíveis
- Implemente VPC endpoints quando necessário
- Monitore access logs

### Cost Management
- Monitore número de requests
- Otimize polling strategies
- Use reserved capacity quando apropriado
- Implemente lifecycle policies

<br />

## :books: Referências

Para uma compreensão mais profunda sobre SQS recomendo a leitura da documentação oficial:

- [O que é o Amazon SQS?](https://docs.aws.amazon.com/pt_br/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)
- [Filas de atraso do Amazon SQS](https://docs.aws.amazon.com/pt_br/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-delay-queues.html)
- [Sondagem curta e longa do Amazon SQS](https://docs.aws.amazon.com/pt_br/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-short-and-long-polling.html)
- [Amazon SQS FIFO queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html)
- [Amazon SQS dead-letter queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)
- [AWS SQS Best Practices](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-best-practices.html)
