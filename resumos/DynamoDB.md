<p align="center">
	<img src="./img/aws-icons/aws-DynamoDB.png" alt="aws-dynamodb-icon" style="height:150px; width:150px;" /> 
  <br />
	<h1 align="center">
    DynamoDB - Guia Completo AWS Developer Associate
  </h1>
</p>	

<br />

## :pushpin: Índice

- [Introdução](#introdução)
- [Conceitos Fundamentais](#conceitos-fundamentais)
- [Tabelas e Estrutura de Dados](#tabelas-e-estrutura-de-dados)
- [Keys (Chaves)](#keys-chaves)
- [Capacidade de Leitura/Gravação](#capacidade-de-leituragravação)
  - [Modo Provisionado](#modo-provisionado)
  - [Modo Sob Demanda](#modo-sob-demanda)
  - [Auto Scaling](#auto-scaling)
- [Consistência de Dados](#consistência-de-dados)
- [API Operations](#api-operations)
  - [Operações de Escrita](#operações-de-escrita)
  - [Operações de Leitura](#operações-de-leitura)
  - [Operações de Exclusão](#operações-de-exclusão)
  - [Operações em Lote](#operações-em-lote)
- [Índices Secundários](#índices-secundários)
- [Streams](#streams)
- [TTL (Time To Live)](#ttl-time-to-live)
- [Transações](#transações)
- [PartiQL](#partiql)
- [Backup e Restore](#backup-e-restore)
- [Global Tables](#global-tables)
- [DAX (DynamoDB Accelerator)](#dax-dynamodb-accelerator)
- [Segurança](#segurança)
- [Monitoramento](#monitoramento)
- [Best Practices](#best-practices)
- [Pricing](#pricing)
- [Questões de Prova](#questões-de-prova)
- [Referências](#books-referências)

<br />

## Introdução

Amazon **DynamoDB** é um banco de dados NoSQL totalmente gerenciado, altamente disponível e de baixa latência. É um dos serviços mais importantes para a certificação AWS Developer Associate.

**Características principais:**
- NoSQL totalmente gerenciado
- Latência de single-digit milliseconds
- Escalabilidade automática
- Integração nativa com outros serviços AWS
- Disponibilidade de 99.99%
- Suporte a ACID transactions

**Casos de uso comuns:**
- Aplicações mobile e web
- Gaming
- IoT
- Real-time analytics
- Shopping carts
- Session stores

<br />

## Conceitos Fundamentais

**DynamoDB vs Bancos Relacionais:**
- Não possui schemas fixos
- Não suporta JOINs complexos
- Escalabilidade horizontal automática
- Performance consistente independente do tamanho

**Modelos de dados suportados:**
- **Document:** JSON, HTML, XML
- **Key-Value:** Pares chave-valor simples

<br />

## Tabelas e Estrutura de Dados

### Componentes da Tabela

**Table (Tabela):**
- Coleção de dados
- Nome único por região e conta AWS
- Sem limite de armazenamento

**Item:**
- Registro individual (equivale a uma linha)
- Tamanho máximo: **400 KB**
- Identificado unicamente pela Primary Key

**Attribute:**
- Campo de dados (equivale a uma coluna)
- Pode ser obrigatório ou opcional
- Tipos de dados variados

### Tipos de Dados Suportados

**Scalar Types:**
- **String (S):** Texto
- **Number (N):** Inteiros e decimais
- **Binary (B):** Dados binários
- **Boolean (BOOL):** True/False
- **Null (NULL):** Valor nulo

**Document Types:**
- **List (L):** Array de valores
- **Map (M):** Objeto JSON

**Set Types:**
- **String Set (SS):** Conjunto de strings
- **Number Set (NS):** Conjunto de números
- **Binary Set (BS):** Conjunto de dados binários

<br />

## Keys (Chaves)

### Primary Key

**Obrigatória para toda tabela. Dois tipos:**

#### 1. Partition Key (Simple Primary Key)
- **HASH** - Valor único na tabela
- Determina a partição física onde o item será armazenado
- Deve ter boa distribuição para evitar hot partitions

#### 2. Composite Primary Key
- **Partition Key + Sort Key (HASH + RANGE)**
- Partition Key pode ser duplicada
- Combinação Partition Key + Sort Key deve ser única
- Permite consultas range na Sort Key

**Exemplo de estrutura:**
```
Partition Key: user_id
Sort Key: timestamp
Item: {user_id: "123", timestamp: "2025-01-01", data: "..."}
```

<br />

## Capacidade de Leitura/Gravação

### Modo Provisionado

**Características:**
- Você especifica RCU/WCU antecipadamente
- Preço previsível
- Suporte a Auto Scaling
- Pode configurar minimum/maximum capacity

#### Write Capacity Units (WCU)

**1 WCU = 1 item de até 1KB por segundo**

**Cálculos:**
- Item > 1KB: arredondar para cima
- Exemplo 1: 10 items/sec de 2KB = 10 × 2 = 20 WCUs
- Exemplo 2: 6 items/sec de 4.5KB = 6 × 5 = 30 WCUs

#### Read Capacity Units (RCU)

**1 RCU = 1 strongly consistent read ou 2 eventually consistent reads de até 4KB por segundo**

**Cálculos:**
- Eventually consistent: divide por 2
- Item > 4KB: arredondar para cima
- Exemplo 1: 10 strongly consistent reads/sec de 4KB = 10 × 1 = 10 RCUs
- Exemplo 2: 16 eventually consistent reads/sec de 12KB = (16/2) × (12/4) = 24 RCUs

#### Burst Capacity
- DynamoDB reserva capacidade extra para picos temporários
- Não deve ser considerado no planejamento

#### Adaptive Capacity
- Redistribui automaticamente capacidade não utilizada
- Ajuda com hot partitions temporárias

### Modo Sob Demanda

**Características:**
- Escalabilidade automática instantânea
- Paga apenas pelo que usar
- Até 2.5x mais caro que provisionado
- Ideal para workloads imprevisíveis

**Request Units:**
- 1 WRU = 1 WCU
- 1 RRU = 1 RCU

### Auto Scaling

**Funcionalidades:**
- Monitora consumo de capacidade
- Ajusta automaticamente RCU/WCU
- Define limites mínimo/máximo
- Baseado em CloudWatch metrics

<br />

## Consistência de Dados

### Eventually Consistent Reads (Padrão)
- Retorna dados que podem estar desatualizados
- Máximo 1 segundo de atraso
- Consome menos RCU
- Usar quando dados recentes não são críticos

### Strongly Consistent Reads
- Retorna sempre os dados mais recentes
- Consome 2x mais RCU
- Parâmetro: `ConsistentRead=true`
- Usar quando dados atuais são críticos

**Importante:** Strongly consistent reads não estão disponíveis em:
- Global Secondary Indexes
- Global Tables cross-region reads

<br />

## API Operations

### Operações de Escrita

#### PutItem
- Cria novo item ou substitui completamente
- Consome WCUs baseado no tamanho do item
- Suporte a Conditional Writes

```python
response = dynamodb.put_item(
    TableName='Users',
    Item={'user_id': '123', 'name': 'João'},
    ConditionExpression='attribute_not_exists(user_id)'
)
```

#### UpdateItem
- Modifica atributos existentes ou adiciona novos
- Consome WCUs apenas pelos atributos modificados
- Suporte a Atomic Counters
- Operações: SET, ADD, REMOVE, DELETE

```python
response = dynamodb.update_item(
    TableName='Users',
    Key={'user_id': '123'},
    UpdateExpression='SET #name = :name, #count = #count + :inc',
    ExpressionAttributeNames={'#name': 'name', '#count': 'login_count'},
    ExpressionAttributeValues={':name': 'João Silva', ':inc': 1}
)
```

#### Conditional Writes
- Executam apenas se condição for verdadeira
- Previnem overwrites acidentais
- Useful para otimistic locking

### Operações de Leitura

#### GetItem
- Busca item específico pela Primary Key
- Altamente eficiente (O(1))
- Suporte a ProjectionExpression

```python
response = dynamodb.get_item(
    TableName='Users',
    Key={'user_id': '123'},
    ConsistentRead=True,
    ProjectionExpression='#name, email',
    ExpressionAttributeNames={'#name': 'name'}
)
```

#### Query
- Busca baseada na Partition Key
- Opcionalmente filtra pela Sort Key
- Suporte a filtros adicionais
- Retorna até 1MB de dados
- Suporte a paginação

```python
response = dynamodb.query(
    TableName='Orders',
    KeyConditionExpression='user_id = :uid AND order_date BETWEEN :start AND :end',
    ExpressionAttributeValues={
        ':uid': '123',
        ':start': '2025-01-01',
        ':end': '2025-01-31'
    }
)
```

#### Scan
- Examina toda a tabela
- Operação custosa e lenta
- Suporte a filtros
- Suporte a Parallel Scans
- Usar apenas quando necessário

```python
response = dynamodb.scan(
    TableName='Users',
    FilterExpression='age > :age',
    ExpressionAttributeValues={':age': 25}
)
```

**Query vs Scan:**
- Query: Eficiente, usa índices
- Scan: Ineficiente, examina todos os items

### Operações de Exclusão

#### DeleteItem
- Remove item específico
- Suporte a Conditional Deletes
- Retorna item deletado se solicitado

```python
response = dynamodb.delete_item(
    TableName='Users',
    Key={'user_id': '123'},
    ConditionExpression='attribute_exists(user_id)',
    ReturnValues='ALL_OLD'
)
```

### Operações em Lote

#### BatchGetItem
- Busca até 100 items de uma ou múltiplas tabelas
- Máximo 16MB de dados
- Unprocessed keys retornadas se houver throttling

#### BatchWriteItem
- Escreve/deleta até 25 items
- Não suporta UpdateItem
- Máximo 16MB de dados
- Unprocessed items retornados se houver throttling

**Importante:** Operações batch não são transacionais.

<br />

## Índices Secundários

### Global Secondary Index (GSI)

**Características:**
- Partition Key e Sort Key diferentes da tabela base
- Capacidade RCU/WCU independente
- Eventual consistency apenas
- Até 20 GSIs por tabela
- Pode ser criado/deletado após criação da tabela

**Projeções:**
- **KEYS_ONLY:** Apenas chaves
- **INCLUDE:** Chaves + atributos específicos
- **ALL:** Todos os atributos (mais custoso)

### Local Secondary Index (LSI)

**Características:**
- Mesma Partition Key, Sort Key diferente
- Compartilha capacidade com tabela base
- Suporte a strongly consistent reads
- Até 5 LSIs por tabela
- Deve ser criado na criação da tabela
- Limite de 10GB por partition key

<br />

## Streams

**DynamoDB Streams** capturam mudanças na tabela em tempo real.

**Características:**
- Ordenação por shard
- Retenção de 24 horas
- Processamento via Lambda ou Kinesis

**Tipos de informação capturada:**
- **KEYS_ONLY:** Apenas chaves
- **NEW_IMAGE:** Item após modificação
- **OLD_IMAGE:** Item antes da modificação  
- **NEW_AND_OLD_IMAGES:** Ambos

**Casos de uso:**
- Analytics em tempo real
- Replicação de dados
- Notificações
- Auditoria

<br />

## TTL (Time To Live)

**Funcionalidades:**
- Remove automaticamente items expirados
- Baseado em timestamp Unix
- Processo em background (até 48h de atraso)
- Não consome WCUs
- Items deletados aparecem no Stream

```python
response = dynamodb.put_item(
    TableName='Sessions',
    Item={
        'session_id': '123',
        'data': 'session_data',
        'ttl': int(time.time()) + 3600  # expira em 1 hora
    }
)
```

<br />

## Transações

**ACID Transactions** para múltiplos items/tabelas.

### TransactWriteItems
- Até 25 operações
- All-or-nothing
- Consome 2x WCUs

### TransactGetItems  
- Até 25 operações
- Snapshot isolation
- Consome 2x RCUs

**Casos de uso:**
- Transferências bancárias
- Inventory management
- Operações que requerem consistência

<br />

## PartiQL

**SQL-compatible query language** para DynamoDB.

**Suporte a:**
- SELECT, INSERT, UPDATE, DELETE
- Joins limitados
- Functions e expressions

```sql
SELECT * FROM Users WHERE age > 25 AND city = 'São Paulo'
```

<br />

## Backup e Restore

### On-Demand Backup
- Backup completo manual
- Sem impacto na performance
- Retenção até você deletar

### Point-in-Time Recovery (PITR)
- Backup contínuo automático
- Restore para qualquer momento nos últimos 35 dias
- Granularidade de segundos

### Cross-Region Backup
- Backup em região diferente
- Disaster recovery

<br />

## Global Tables

**Multi-region, multi-master replication.**

**Características:**
- Eventual consistency entre regiões
- Automatic conflict resolution (last writer wins)
- Requer Streams habilitado
- RCU/WCU por região

**Casos de uso:**
- Aplicações globais
- Disaster recovery
- Baixa latência global

<br />

## DAX (DynamoDB Accelerator)

**In-memory cache** para DynamoDB.

**Características:**
- Latência microseconds
- Throughput 10x maior
- Cache write-through
- Multi-AZ
- Compatible com DynamoDB APIs

**Casos de uso:**
- Aplicações que requerem latência extremamente baixa
- Read-heavy workloads
- Gaming leaderboards

<br />

## Segurança

### IAM
- Políticas baseadas em recursos
- Fine-grained access control
- Condition keys específicas

### Encryption
- **At rest:** Encryption automática (AWS owned, managed, customer managed keys)
- **In transit:** TLS 1.2

### VPC Endpoints
- Acesso privado via VPC
- Sem tráfego internet

<br />

## Monitoramento

### CloudWatch Metrics
- **ConsumedReadCapacityUnits/ConsumedWriteCapacityUnits**
- **ProvisionedReadCapacityUnits/ProvisionedWriteCapacityUnits**
- **ReadThrottleEvents/WriteThrottleEvents**
- **SystemErrors/UserErrors**

### CloudTrail
- API calls logging
- Compliance e auditoria

<br />

## Best Practices

### Design da Tabela
- Use partition keys com boa distribuição
- Evite hot partitions
- Prefira composite keys quando apropriado

### Performance
- Use ProjectionExpression para reduzir dados transferidos
- Prefira Query sobre Scan
- Use GSI para access patterns diferentes

### Cost Optimization
- Use TTL para limpeza automática
- Monitore capacidade não utilizada
- Considere Reserved Capacity para workloads previsíveis

<br />

## Pricing

### Modo Provisionado
- **RCU:** $0.00013 por hora
- **WCU:** $0.00065 por hora
- **Storage:** $0.25 por GB/mês

### Modo Sob Demanda
- **RRU:** $0.25 por milhão
- **WRU:** $1.25 por milhão

### Custos Adicionais
- **GSI:** RCU/WCU separadas
- **Backup:** $0.10 per GB/mês
- **Streams:** $0.02 per 100K records

<br />

## Questões de Prova

### Questão 1
**Cenário:** Uma aplicação precisa armazenar dados de sessão de usuário com expiração automática após 30 minutos de inatividade.

**Qual é a melhor abordagem?**

A) Usar Lambda function para deletar sessões expiradas  
B) Configurar TTL no DynamoDB  
C) Usar CloudWatch Events para limpeza  
D) Implementar lógica na aplicação  

**Resposta:** B) Configurar TTL no DynamoDB
**Explicação:** TTL é a funcionalidade nativa do DynamoDB para expiração automática de items, não consome WCUs e é a solução mais eficiente.

### Questão 2
**Cenário:** Uma tabela DynamoDB está experienciando throttling frequente em uma partition específica durante picos de tráfego.

**Qual das seguintes opções resolve o problema?**

A) Aumentar RCU/WCU globalmente  
B) Usar Adaptive Capacity  
C) Redesenhar a partition key para melhor distribuição  
D) Todas as anteriores  

**Resposta:** D) Todas as anteriores
**Explicação:** Todas são válidas, mas C é a solução definitiva para hot partitions.

### Questão 3
**Cenário:** Você precisa implementar um contador que incrementa a cada visualização de página, garantindo precisão mesmo com alta concorrência.

**Qual operação usar?**

A) GetItem seguido de PutItem  
B) UpdateItem com ADD operation  
C) TransactWriteItems  
D) BatchWriteItem  

**Resposta:** B) UpdateItem com ADD operation
**Explicação:** ADD operation é atomic e thread-safe para contadores.

### Questão 4
**Cenário:** Uma aplicação precisa buscar todos os pedidos de um usuário específico ordenados por data.

**Qual é a melhor estrutura de chave?**

A) Partition Key: order_id, Sort Key: user_id  
B) Partition Key: user_id, Sort Key: order_date  
C) Partition Key: order_date, Sort Key: user_id  
D) Usar apenas Partition Key: user_id  

**Resposta:** B) Partition Key: user_id, Sort Key: order_date
**Explicação:** Permite buscar todos os pedidos de um usuário (Query na partition key) ordenados por data (sort key).

### Questão 5
**Cenário:** Uma aplicação global precisa de baixa latência de leitura em múltiplas regiões com dados eventualmente consistentes.

**Qual solução usar?**

A) DynamoDB com DAX em cada região  
B) Global Tables  
C) Cross-region replication via Lambda  
D) ElastiCache em cada região  

**Resposta:** B) Global Tables
**Explicação:** Global Tables fornece replicação multi-region nativa com eventual consistency.

### Questão 6
**Cenário:** Você precisa processar mudanças em uma tabela DynamoDB em tempo real para atualizar um índice de busca.

**Qual abordagem usar?**

A) Polling regular da tabela  
B) DynamoDB Streams com Lambda  
C) CloudWatch Events  
D) SQS polling  

**Resposta:** B) DynamoDB Streams com Lambda
**Explicação:** Streams capturam mudanças em tempo real e Lambda pode processar automaticamente.

### Questão 7
**Cenário:** Uma consulta DynamoDB retorna "ProvisionedThroughputExceededException".

**Quais são as possíveis causas?**

A) Hot partition  
B) Capacidade insuficiente  
C) Burst capacity esgotada  
D) Todas as anteriores  

**Resposta:** D) Todas as anteriores
**Explicação:** Throttling pode ocorrer por qualquer uma dessas razões.

### Questão 8
**Cenário:** Você precisa implementar uma operação que atualiza inventory e cria um pedido atomicamente.

**Qual abordagem usar?**

A) UpdateItem seguido de PutItem  
B) TransactWriteItems  
C) BatchWriteItem  
D) Duas operações separadas  

**Resposta:** B) TransactWriteItems
**Explicação:** Transações garantem atomicidade (all-or-nothing) para múltiplas operações.

### Questão 9
**Cenário:** Uma aplicação faz muitas consultas por um atributo que não é parte da primary key.

**Qual é a melhor solução?**

A) Usar Scan com FilterExpression  
B) Criar Global Secondary Index  
C) Reestruturar a primary key  
D) Usar multiple GetItem calls  

**Resposta:** B) Criar Global Secondary Index
**Explicação:** GSI permite queries eficientes em atributos não-chave.

### Questão 10
**Cenário:** Uma tabela DynamoDB precisa suportar queries range em timestamps para relatórios, mas a primary key é user_id.

**Qual solução usar?**

A) Usar Scan com FilterExpression  
B) Criar GSI com timestamp como sort key  
C) Criar LSI com timestamp como sort key  
D) Reestruturar a tabela completamente  

**Resposta:** B) Criar GSI com timestamp como sort key
**Explicação:** GSI permite different access patterns. LSI requer mesma partition key.

<br />

## :books: Referências

- [Amazon DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/)
- [DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
- [DynamoDB API Reference](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/)
- [AWS Well-Architected Framework - DynamoDB](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/dynamodb.html)
