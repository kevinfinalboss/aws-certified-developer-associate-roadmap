# AWS Lambda

<p align="center">
	<img src="./img/aws-icons/aws-Lambda.png" alt="aws-lambda-icon" style="height:150px; width:150px;" /> 
  <br />
	<h1 align="center">
    Lambda
  </h1>
</p>	

## 📋 Índice

- [Conceitos Fundamentais](#conceitos-fundamentais)
- [Arquitetura Serverless](#arquitetura-serverless)
- [Linguagens e Runtime](#linguagens-e-runtime)
- [Modelos de Invocação](#modelos-de-invocação)
- [Integrações Principais](#integrações-principais)
- [Lambda com ALB](#lambda-com-alb)
- [Event Source Mapping](#event-source-mapping)
- [Segurança e IAM](#segurança-e-iam)
- [Lambda e VPC](#lambda-e-vpc)
- [Concorrência e Performance](#concorrência-e-performance)
- [Lambda Layers](#lambda-layers)
- [Versões e Aliases](#versões-e-aliases)
- [Container Images](#container-images)
- [Monitoramento e Debugging](#monitoramento-e-debugging)
- [Limites e Quotas](#limites-e-quotas)
- [Pricing](#pricing)
- [Perguntas e Respostas](#perguntas-e-respostas)
- [Referências](#referências)

---

## Conceitos Fundamentais

### O que é AWS Lambda?
AWS Lambda é um serviço de computação **serverless** que executa código em resposta a eventos sem necessidade de provisionar ou gerenciar servidores. É um dos serviços mais importantes para o exame AWS Developer.

### Características Essenciais:
- **Event-driven**: Executa em resposta a eventos
- **Stateless**: Cada execução é independente
- **Automatically Scaling**: Escala automaticamente conforme demanda
- **Pay-per-use**: Paga apenas pelo tempo de execução
- **Managed Service**: AWS gerencia toda infraestrutura

### Casos de Uso Principais:
- APIs backend (com API Gateway)
- Processamento de dados em tempo real
- Processamento de arquivos (S3 triggers)
- Automação e orquestração
- Microserviços
- ETL (Extract, Transform, Load)

---

## Arquitetura Serverless

### O que é Serverless?
Paradigma onde você não gerencia servidores, focando apenas no código. A AWS cuida da infraestrutura, scaling, patching e disponibilidade.

### Serviços Serverless na AWS:
- **Compute**: Lambda, Fargate
- **Storage**: S3, EFS
- **Database**: DynamoDB, Aurora Serverless
- **Integration**: API Gateway, EventBridge, SNS, SQS
- **Analytics**: Kinesis, Athena
- **Monitoring**: CloudWatch, X-Ray

### Benefícios Serverless:
- **Sem gerenciamento de servidor**
- **Scaling automático**
- **Billing por uso**
- **Foco no código de negócio**
- **Alta disponibilidade nativa**

---

## Linguagens e Runtime

### Runtimes Oficiais:
- **Python**: 3.8, 3.9, 3.10, 3.11
- **Node.js**: 16.x, 18.x, 20.x
- **Java**: 8, 11, 17, 21
- **C#**: .NET 6, .NET 8
- **Go**: 1.x
- **Ruby**: 3.2
- **PowerShell**: 7.x

### Custom Runtime:
- **Provided.al2**: Para qualquer linguagem
- **Container Images**: Suporte completo via Docker
- **Runtime API**: Interface para criar runtimes customizados

### Estrutura da Função:
```python
def lambda_handler(event, context):
    # event: dados do trigger
    # context: informações do runtime
    return {
        'statusCode': 200,
        'body': 'Hello World'
    }
```

---

## Modelos de Invocação

### 1. Invocação Síncrona
- **Aguarda resposta**: Cliente espera resultado
- **Timeout**: Máximo 15 minutos
- **Error Handling**: Erros retornados diretamente
- **Exemplos**: API Gateway, ALB, CLI/SDK

**Fluxo:**
```
Client → Lambda → Response
```

### 2. Invocação Assíncrona
- **Fire and forget**: Cliente não espera resposta
- **Retry**: 2 tentativas automáticas em caso de erro
- **DLQ**: Dead Letter Queue para falhas
- **Exemplos**: S3, SNS, EventBridge, CloudWatch Events

**Fluxo:**
```
Event Source → Event Queue → Lambda
```

### 3. Event Source Mapping
- **Poll-based**: Lambda consulta a fonte
- **Batch Processing**: Processa lotes de registros
- **Parallelization**: Múltiplos workers simultâneos
- **Exemplos**: Kinesis, DynamoDB Streams, SQS

**Fluxo:**
```
Event Source ← Lambda (polling)
```

---

## Integrações Principais

### API Gateway + Lambda
- **RESTful APIs**: Criação de APIs completas
- **Authentication**: Cognito, IAM, Custom Authorizers
- **Rate Limiting**: Throttling e quotas
- **Caching**: Cache de respostas
- **CORS**: Cross-Origin Resource Sharing

### S3 + Lambda
- **Event Notifications**: Object created, deleted, modified
- **Event Filtering**: Por prefixo, sufixo, tamanho
- **Permissions**: Resource-based policy necessária
- **Considerations**: Evitar loops infinitos

### DynamoDB + Lambda
- **DynamoDB Streams**: Captura mudanças em tempo real
- **Event Types**: INSERT, MODIFY, REMOVE
- **Batch Size**: 1-10.000 registros
- **Parallelization**: Por shard

### SQS + Lambda
- **Standard Queue**: Até 10 mensagens por invocação
- **FIFO Queue**: Processa em ordem
- **Batch Size**: Configurável (1-10.000)
- **Visibility Timeout**: Importante para reprocessamento

---

## Lambda com ALB

### Configuração:
1. **Target Group**: Criar com tipo "Lambda function"
2. **Health Checks**: Não aplicável para Lambda
3. **Multi-Value Headers**: Suporte a arrays em headers/query params

### Request Format:
```json
{
    "requestContext": {
        "elb": {
            "targetGroupArn": "arn:aws:elasticloadbalancing:..."
        }
    },
    "httpMethod": "GET",
    "path": "/path",
    "queryStringParameters": {"param": "value"},
    "headers": {"Host": "example.com"},
    "body": "",
    "isBase64Encoded": false
}
```

### Response Format:
```json
{
    "statusCode": 200,
    "statusDescription": "200 OK",
    "headers": {"Content-Type": "application/json"},
    "body": "Hello World",
    "isBase64Encoded": false
}
```

### Multi-Header Values:
```bash
# URL: http://example.com/path?name=João&name=Maria
# Lambda Event:
"multiValueQueryStringParameters": {
    "name": ["João", "Maria"]
}
```

---

## Event Source Mapping

### Conceito:
Configuração que instrui Lambda a ler de event sources que não fazem push de eventos (Kinesis, DynamoDB Streams, SQS).

### Características:
- **Lambda faz polling**: Consulta a fonte regularmente
- **Batch Processing**: Processa múltiplos registros
- **Error Handling**: Retry logic configurável
- **Scaling**: Aumenta workers conforme necessário

### Configurações Importantes:
- **Batch Size**: Quantos registros por invocação
- **Maximum Batching Window**: Tempo para formar batch
- **Starting Position**: TRIM_HORIZON, LATEST, AT_TIMESTAMP
- **Parallelization Factor**: Número de workers por shard

### Error Handling:
- **Retry**: Configurável por source
- **Dead Letter Queue**: Para mensagens que falharam
- **Bisect on Error**: Divide batch em caso de erro
- **Maximum Record Age**: Descarta registros antigos

---

## Segurança e IAM

### Execution Role (Obrigatória):
Role que Lambda assume para executar a função.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        }
    ]
}
```

### Resource-Based Policy:
Controla quem pode invocar a função.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {"Service": "s3.amazonaws.com"},
            "Action": "lambda:InvokeFunction",
            "Resource": "arn:aws:lambda:us-east-1:123456789012:function:MyFunction"
        }
    ]
}
```

### Managed Policies Importantes:
- **AWSLambdaBasicExecutionRole**: Logs básicos
- **AWSLambdaVPCAccessExecutionRole**: Acesso VPC
- **AWSLambdaKinesisExecutionRole**: Kinesis streams
- **AWSLambdaDynamoDBExecutionRole**: DynamoDB streams

### Environment Variables:
- **Encryption**: KMS encryption disponível
- **Size Limit**: 4KB total
- **Runtime Access**: Disponível via `os.environ`

---

## Lambda e VPC

### Por Padrão:
Lambda executa em VPC gerenciada pela AWS com acesso à internet e serviços AWS.

### VPC Configuration:
- **Subnet IDs**: Pelo menos 2 subnets
- **Security Groups**: Controla tráfego de rede
- **ENI**: Elastic Network Interface criada

### Limitações em VPC:
- **Sem Internet**: Por padrão não tem acesso à internet
- **Cold Start**: Mais lento devido setup de rede
- **ENI Limits**: Limites de interfaces de rede

### Acesso à Internet em VPC:
- **NAT Gateway**: Em subnet pública
- **Route Table**: Direcionamento correto
- **Internet Gateway**: Na VPC

### Casos de Uso:
- **RDS Access**: Banco em VPC privada
- **ElastiCache**: Cache em VPC
- **Internal APIs**: Serviços internos da empresa

---

## Concorrência e Performance

### Concorrência:
- **Default Limit**: 1000 execuções simultâneas
- **Reserved Concurrency**: Garantia de capacidade
- **Provisioned Concurrency**: Pré-aquece funções

### Reserved Concurrency:
- **Allocation**: Reserva parte do limite total
- **Isolation**: Garante que outras funções não consumam
- **Cost**: Sem custo adicional (apenas reserva)

### Provisioned Concurrency:
- **Pre-warmed**: Instâncias sempre prontas
- **Zero Cold Start**: Eliminação do cold start
- **Cost**: Cobrança adicional por instância provisionada
- **Auto Scaling**: Pode escalar automaticamente

### Cold Start:
- **Definição**: Tempo para iniciar nova instância
- **Fatores**: Runtime, package size, VPC config
- **Otimização**: Reduzir dependencies, connection pooling
- **Typical Times**: 100ms-10s dependendo do runtime

### Performance Optimization:
- **Memory Allocation**: Mais CPU com mais memória
- **Connection Reuse**: Pool de conexões DB
- **Lazy Loading**: Carregar apenas quando necessário
- **Reduce Package Size**: Apenas dependencies necessárias

---

## Lambda Layers

### Conceito:
Mecanismo para compartilhar código, libraries, custom runtimes entre funções.

### Características:
- **Size Limit**: 250MB (unzipped)
- **Max Layers**: 5 layers por função
- **Versioning**: Layers são versionados
- **Sharing**: Pode ser compartilhado entre contas

### Estrutura:
```
layer.zip
└── python/
    └── lib/
        └── python3.8/
            └── site-packages/
                └── requests/
```

### Casos de Uso:
- **Common Libraries**: pandas, requests, boto3
- **Shared Business Logic**: Código comum entre funções
- **Custom Runtimes**: Linguagens não suportadas
- **Configuration**: Arquivos de configuração

### Benefits:
- **Code Reuse**: Evita duplicação
- **Smaller Packages**: Função menor, deploy mais rápido
- **Centralized Updates**: Atualiza library em um lugar
- **Version Management**: Controle de versões

---

## Versões e Aliases

### Versões ($LATEST):
- **Mutable**: $LATEST é sempre editável
- **Immutable**: Versões numeradas são imutáveis
- **ARN**: Cada versão tem ARN único
- **Snapshots**: Versão é snapshot do código + config

### Exemplo ARNs:
```
# $LATEST (mutable)
arn:aws:lambda:us-east-1:123456789012:function:MyFunction:$LATEST

# Version 1 (immutable)
arn:aws:lambda:us-east-1:123456789012:function:MyFunction:1
```

### Aliases:
- **Pointer**: Aponta para versão específica
- **Mutable**: Pode ser redirecionado
- **Blue/Green**: Deployment com weighted traffic
- **Environment**: DEV, STAGING, PROD

### Traffic Splitting:
```json
{
    "Name": "PROD",
    "FunctionVersion": "1",
    "AdditionalVersionWeights": {
        "2": 0.1
    }
}
```

### Use Cases:
- **A/B Testing**: Testa duas versões simultaneamente
- **Canary Deployments**: Gradual rollout
- **Environment Management**: Separação clara de ambientes
- **Rollback**: Rápido retorno à versão anterior

---

## Container Images

### Conceito:
Empacote função Lambda como container Docker.

### Características:
- **Image Size**: Até 10GB
- **Base Images**: AWS fornece base images
- **ECR**: Deploy via Elastic Container Registry
- **Local Testing**: Teste local com Docker

### Base Images Oficiais:
```dockerfile
FROM public.ecr.aws/lambda/python:3.9

COPY app.py ${LAMBDA_TASK_ROOT}
COPY requirements.txt ${LAMBDA_TASK_ROOT}

RUN pip install -r requirements.txt

CMD ["app.lambda_handler"]
```

### Custom Images:
- **Runtime Interface Client**: Biblioteca necessária
- **Runtime Interface Emulator**: Para teste local
- **Multi-stage builds**: Otimização de tamanho

### Vantagens:
- **Larger Dependencies**: Até 10GB vs 250MB zip
- **Familiar Tooling**: Docker ecosystem
- **Local Testing**: Ambiente idêntico
- **CI/CD Integration**: Pipelines existentes

---

## Monitoramento e Debugging

### CloudWatch Metrics:
- **Invocations**: Número de execuções
- **Duration**: Tempo de execução
- **Errors**: Número de erros
- **Throttles**: Invocações limitadas
- **DeadLetterErrors**: Falhas na DLQ

### CloudWatch Logs:
- **Log Groups**: Criados automaticamente
- **Log Streams**: Por execução
- **Structured Logging**: JSON para melhor análise
- **Log Retention**: Configurável

### X-Ray Tracing:
- **Distributed Tracing**: Rastreamento end-to-end
- **Service Map**: Visualização de dependências
- **Performance Analysis**: Identificação de bottlenecks
- **Error Analysis**: Root cause analysis

### Environment Variables para X-Ray:
- `_X_AMZN_TRACE_ID`: Trace ID atual
- `AWS_XRAY_CONTEXT_MISSING`: Comportamento quando sem context
- `AWS_XRAY_DAEMON_ADDRESS`: Endereço do daemon

### Lambda Insights:
- **System Metrics**: CPU, Memory, Network
- **Application Metrics**: Custom metrics
- **Enhanced Monitoring**: Visão detalhada performance

---

## Limites e Quotas

### Runtime Limits:
- **Memory**: 128MB - 10.008MB (incrementos de 1MB)
- **Timeout**: 15 minutos máximo
- **Temp Storage (/tmp)**: 512MB - 10.240MB
- **Environment Variables**: 4KB total
- **Request/Response**: 6MB payload

### Deployment Limits:
- **Package Size (zip)**: 50MB compressed, 250MB uncompressed
- **Container Image**: 10GB
- **Layers**: 5 layers max, 250MB uncompressed total
- **Code + Layers**: 250MB uncompressed total

### Concurrency Limits:
- **Account Limit**: 1000 concurrent executions (default)
- **Function Limit**: 1000 concurrent executions (max)
- **Burst Limit**: 3000 initial burst, depois 500/minute increase

### Service Limits:
- **Function Name**: 64 characters
- **Description**: 256 characters
- **Functions per Region**: 1000 (soft limit)

### Network (VPC):
- **ENIs**: 250 per security group
- **Hyperplane ENIs**: Newer, more efficient networking

---

## Pricing

### Componentes de Cobrança:
1. **Requests**: Número de invocações
2. **Duration**: Tempo de execução (GB-seconds)
3. **Provisioned Concurrency**: Instâncias pré-aquecidas

### Free Tier:
- **Requests**: 1 milhão por mês
- **Duration**: 400.000 GB-seconds por mês

### Pricing Details:
- **Requests**: $0.20 por 1M requests (após free tier)
- **Duration**: $0.0000166667 por GB-second
- **Provisioned Concurrency**: $0.0000041667 por GB-second

### Cálculo Example:
```
Function: 512MB, 100ms execution, 1M invocations/month
Duration: (512MB/1024) * 0.1s * 1M = 50,000 GB-seconds
Cost: $0.83 per month (dentro free tier)
```

### Cost Optimization:
- **Right-size Memory**: Encontrar sweet spot price/performance
- **Reduce Execution Time**: Código otimizado
- **Use Layers**: Reduzir size, faster deploys
- **Reserved Concurrency**: Apenas quando necessário

---

## Perguntas e Respostas

### Q1: Qual a diferença entre invocação síncrona e assíncrona?
**R:** **Síncrona**: Cliente aguarda resposta, usado por API Gateway/ALB. **Assíncrona**: Fire-and-forget, usado por S3/SNS, inclui retry automático e DLQ. **Event Source Mapping**: Lambda faz polling da fonte (Kinesis/SQS).

### Q2: Como configurar Lambda para acessar RDS em VPC privada?
**R:** Configure Lambda com Subnet IDs (privadas), Security Group (permitir acesso ao RDS), e IAM role com `AWSLambdaVPCAccessExecutionRole`. Lambda criará ENI para comunicação com VPC.

### Q3: Qual a diferença entre Reserved e Provisioned Concurrency?
**R:** **Reserved**: Reserva parte do limite de concorrência, sem custo adicional, para garantir capacidade. **Provisioned**: Pré-aquece instâncias, elimina cold start, mas tem custo adicional por instância provisionada.

### Q4: Como funciona o Event Source Mapping?
**R:** Lambda faz polling da fonte de eventos (Kinesis, DynamoDB Streams, SQS), processa em batches, e escala automaticamente baseado no volume. Usado para fontes que não fazem push de eventos.

### Q5: Quais são os tipos de ARN para versões Lambda?
**R:** **Qualified ARN**: Inclui versão específica (`:1`, `:$LATEST`). **Unqualified ARN**: Sem versão, sempre aponta para $LATEST. Aliases têm ARNs próprios e podem fazer traffic splitting.

### Q6: Como implementar Blue/Green deployment com Lambda?
**R:** Use Aliases com weighted traffic routing. Exemplo: Alias PROD aponta 90% para versão 1 e 10% para versão 2, permitindo canary deployment e rollback rápido.

### Q7: Quais são os limites principais do Lambda?
**R:** **Runtime**: 15min timeout, 10GB memory, 10GB temp storage. **Package**: 50MB zip, 250MB uncompressed, 10GB container. **Concurrency**: 1000 simultâneas default. **Payload**: 6MB max.

### Q8: Como otimizar cold start no Lambda?
**R:** Use Provisioned Concurrency, reduza package size, evite VPC se possível, use connection pooling, lazy loading, e considere runtime mais rápido (Go, Java compiled).

### Q9: Qual a diferença entre Execution Role e Resource-based Policy?
**R:** **Execution Role**: O que Lambda pode acessar (outbound). **Resource-based Policy**: Quem pode invocar Lambda (inbound). Ambas são necessárias para integração completa.

### Q10: Como configurar DLQ (Dead Letter Queue) no Lambda?
**R:** Configure SQS ou SNS como DLQ na configuração da função (apenas para invocações assíncronas). Para Event Source Mapping, configure DLQ no próprio mapping.

### Q11: O que são Lambda Layers e quando usar?
**R:** Compartilham código/libraries entre funções. Use para: common libraries, shared business logic, custom runtimes. Limite: 5 layers, 250MB total, versionadas.

### Q12: Como funciona o pricing do Lambda?
**R:** Cobrança por requests ($0.20/1M após free tier) + duration (GB-seconds). Free tier: 1M requests + 400K GB-seconds/mês. Provisioned Concurrency tem custo adicional.

### Q13: Diferença entre Container Images e ZIP deployment?
**R:** **ZIP**: 250MB limit, mais rápido cold start, tradicional. **Container**: 10GB limit, teste local idêntico, familiar para Docker users, suporte a dependencies maiores.

### Q14: Como configurar environment variables e encryption?
**R:** Configure via console/CLI, limite 4KB total. Para dados sensíveis use KMS encryption. Variables ficam disponíveis via `os.environ` no runtime.

### Q15: Quais integrações requerem Resource-based Policy?
**R:** S3, API Gateway, ALB, SNS, EventBridge. Services como Kinesis, DynamoDB Streams, SQS usam Execution Role pois Lambda faz polling.

### Q16: Como monitorar e debuggar Lambda functions?
**R:** **CloudWatch**: Metrics automáticos + logs. **X-Ray**: Distributed tracing. **Lambda Insights**: System metrics detalhados. Use structured logging (JSON) para melhor análise.

### Q17: Qual configuração de memory escolher?
**R:** Memory aloca proporcionalmente CPU. Teste diferentes valores para encontrar sweet spot entre performance e custo. Mais memory = mais CPU = execução mais rápida (pode compensar custo).

### Q18: Como implementar error handling robusto?
**R:** **Síncronas**: Try/catch + status codes apropriados. **Assíncronas**: Configure retry + DLQ. **Event Source Mapping**: Configure bisect on error + DLQ + maximum record age.

### Q19: Diferença entre S3 Event Notifications e EventBridge?
**R:** **S3 Events**: Direto S3→Lambda, mais rápido, formato específico. **EventBridge**: S3→EventBridge→Lambda, permite filtering/routing complexo, formato padrão, mais flexível.

### Q20: Como configurar Lambda com ALB?
**R:** Crie Target Group tipo "Lambda function", adicione função, configure health checks (opcional para Lambda). Lambda recebe formato específico ALB e deve retornar statusCode + body.

---

## Referências

- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/)
- [AWS Lambda Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [Lambda Event Source Mappings](https://docs.aws.amazon.com/lambda/latest/dg/invocation-eventsourcemapping.html)
- [Lambda Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html)
- [Lambda Container Images](https://docs.aws.amazon.com/lambda/latest/dg/lambda-images.html)
- [Lambda Security](https://docs.aws.amazon.com/lambda/latest/dg/lambda-security.html)
- [Lambda Monitoring](https://docs.aws.amazon.com/lambda/latest/dg/lambda-monitoring.html)
- [Serverless Application Lens](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/)

---

## 💡 Dicas para o Exame

1. **Memorize os limites**: 15min timeout, 10GB memory, 1000 concurrency default
2. **Entenda modelos de invocação**: Síncrona vs Assíncrona vs Event Source Mapping  
3. **Saiba quando usar cada integração**: API Gateway, ALB, S3, SQS, Kinesis
4. **Compreenda IAM**: Execution Role vs Resource-based Policy
5. **Entenda Versioning**: $LATEST, numbered versions, aliases, traffic splitting
6. **Saiba sobre VPC**: ENI creation, internet access, performance impact
7. **Compreenda concurrency**: Reserved vs Provisioned, cold start mitigation
8. **Conheça Lambda Layers**: Sharing code, size limits, versioning
9. **Entenda error handling**: DLQ, retry logic, diferentes por invocation type
10. **Pratique troubleshooting**: CloudWatch logs, X-Ray tracing, common errors
11. **Saiba pricing model**: Requests + GB-seconds, free tier limits
12. **Entenda container support**: 10GB limit, ECR integration, local testing
13. **Conheça event formats**: Diferentes para cada source (S3, API Gateway, ALB)
14. **Pratique deployment**: ZIP vs Container, CI/CD patterns
15. **Entenda performance**: Memory sizing, connection pooling, optimization techniques
