<p align="center">
	<img src="./img/aws-icons/aws-SAM.png" alt="aws-SAM-icon" style="height:150px; width:150px;" /> 
  <br />
	<h1 align="center">
    SAM
  </h1>
</p>	
<br />

## :pushpin: Índice
- [Introdução](#introdução)
- [Transform Declaration](#transform-declaration)
- [Recursos SAM](#recursos-sam)
  - [AWS::Serverless::Function](#awsserverlessfunction)
  - [AWS::Serverless::Api](#awsserverlessapi)
  - [AWS::Serverless::SimpleTable](#awsserverlesssimpletable)
- [Comandos SAM CLI](#comandos-sam-cli)
- [Desenvolvimento Local](#desenvolvimento-local)
- [Template SAM vs CloudFormation](#template-sam-vs-cloudformation)
- [Deployment](#deployment)
- [Políticas SAM](#políticas-sam)
- [SAR - Serverless Application Repository](#sar---serverless-application-repository)
- [Conceitos de Prova](#conceitos-de-prova)
- [Quiz](#quiz)
- [Referências](#books-referências)

<br />

## Introdução

AWS SAM (Serverless Application Model) é um framework open-source que **estende o CloudFormation** para simplificar o desenvolvimento de aplicações serverless. SAM usa uma **sintaxe declarativa** para definir funções Lambda, APIs, tabelas DynamoDB e mapeamentos de eventos.

**Pontos chave para a prova:**
- SAM é construído sobre CloudFormation
- Usa transform `AWS::Serverless-2016-10-31`
- Simplifica deployment de aplicações serverless
- Suporta desenvolvimento e teste local

<br />

## Transform Declaration

**CONCEITO FUNDAMENTAL:** Todo template SAM deve incluir a declaração Transform.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
```

**O que acontece:**
1. CloudFormation identifica que é um template SAM
2. Transforma recursos SAM em recursos CloudFormation nativos
3. Aplica configurações automáticas (IAM roles, etc.)

**Na prova:** Questões sobre templates que não funcionam geralmente faltam esta linha.

<br />

## Recursos SAM

### AWS::Serverless::Function

**Cria automaticamente:**
- Função Lambda
- IAM Role de execução
- CloudWatch Logs group
- Event source mappings

```yaml
MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    CodeUri: src/
    Handler: app.lambda_handler
    Runtime: python3.9
    Timeout: 30
    MemorySize: 128
    Environment:
      Variables:
        TABLE_NAME: !Ref MyTable
    Events:
      ApiEvent:
        Type: Api
        Properties:
          Path: /hello
          Method: get
      S3Event:
        Type: S3
        Properties:
          Bucket: !Ref MyBucket
          Events: s3:ObjectCreated:*
```

**Tipos de Events importantes para prova:**
- **Api**: Cria API Gateway REST API
- **HttpApi**: Cria API Gateway HTTP API (mais barato, HTTP/2)
- **S3**: Trigger em eventos S3
- **DynamoDB**: DynamoDB Streams
- **Kinesis**: Kinesis Streams
- **SQS**: SQS Queue
- **SNS**: SNS Topic
- **Schedule**: CloudWatch Events (cron/rate)

### AWS::Serverless::Api

**Cria automaticamente:**
- API Gateway REST API
- Deployment
- Stage

```yaml
MyApi:
  Type: AWS::Serverless::Api
  Properties:
    StageName: prod
    Auth:
      DefaultAuthorizer: MyCognitoAuth
      Authorizers:
        MyCognitoAuth:
          UserPoolArn: !GetAtt MyCognitoUserPool.Arn
    Cors:
      AllowMethods: "'GET,POST,PUT'"
      AllowHeaders: "'Content-Type,Authorization'"
      AllowOrigin: "'*'"
```

**Conceitos importantes:**
- **Auth**: Configuração de autorizadores
- **Cors**: Cross-Origin Resource Sharing
- **Stage**: Ambiente de deployment (dev, prod)

### AWS::Serverless::SimpleTable

**Cria automaticamente:**
- Tabela DynamoDB
- Com chave primária simples

```yaml
MyTable:
  Type: AWS::Serverless::SimpleTable
  Properties:
    PrimaryKey:
      Name: id
      Type: String
    ProvisionedThroughput:
      ReadCapacityUnits: 5
      WriteCapacityUnits: 5
```

**Limitações do SimpleTable:**
- Apenas uma chave primária
- Não suporta GSI/LSI
- Para tabelas complexas, usar `AWS::DynamoDB::Table`

<br />

## Comandos SAM CLI

**Comandos essenciais para a prova:**

```bash
sam init
sam build
sam local start-api
sam local invoke
sam deploy --guided
sam logs
sam validate
```

**Fluxo típico:**
1. `sam init` - Inicializa projeto
2. `sam build` - Compila aplicação
3. `sam local start-api` - Testa localmente
4. `sam deploy --guided` - Deploy inicial com prompts
5. `sam deploy` - Deploy subsequentes

<br />

## Desenvolvimento Local

**SAM Local simula:**
- Lambda execution environment
- API Gateway
- DynamoDB Local (com --docker-network)

```bash
sam local start-api --port 8080
sam local invoke MyFunction --event events/event.json
sam local generate-event apigateway aws-proxy > event.json
```

**Conceito importante:** SAM Local usa **Docker** para simular o ambiente Lambda.

<br />

## Template SAM vs CloudFormation

**Template SAM:**
```yaml
Transform: AWS::Serverless-2016-10-31
Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/
      Handler: app.handler
      Runtime: nodejs14.x
      Events:
        Api:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

**Equivalente CloudFormation (expandido):**
```yaml
Resources:
  MyFunction:
    Type: AWS::Lambda::Function
    Properties:
      Code:
        S3Bucket: !Ref Bucket
        S3Key: !Ref Key
      Handler: app.handler
      Runtime: nodejs14.x
      Role: !GetAtt MyFunctionRole.Arn
  
  MyFunctionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument: ...
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
  
  MyApi:
    Type: AWS::ApiGateway::RestApi
  
  MyApiDeployment:
    Type: AWS::ApiGateway::Deployment
  
  MyApiStage:
    Type: AWS::ApiGateway::Stage
```

**Na prova:** SAM reduz dramaticamente a complexidade do template.

<br />

## Deployment

**Processo de deployment SAM:**

1. **Build**: `sam build`
   - Compila código
   - Resolve dependências
   - Cria .aws-sam/build/

2. **Package**: `sam package` ou `sam deploy`
   - Upload código para S3
   - Atualiza template com S3 URLs

3. **Deploy**: CloudFormation stack deployment

**Configuração samconfig.toml:**
```toml
version = 0.1
[default.deploy.parameters]
stack_name = "my-sam-app"
s3_bucket = "my-deployment-bucket"
region = "us-east-1"
capabilities = "CAPABILITY_IAM"
parameter_overrides = "Environment=dev"
```

<br />

## Políticas SAM

**Políticas pré-definidas (Policy Templates):**

```yaml
MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    Policies:
      - DynamoDBCrudPolicy:
          TableName: !Ref MyTable
      - S3ReadPolicy:
          BucketName: !Ref MyBucket
      - SQSSendMessagePolicy:
          QueueName: !GetAtt MyQueue.QueueName
      - CloudWatchPutMetricPolicy: {}
```

**Principais Policy Templates:**
- **DynamoDBCrudPolicy**: Read/Write DynamoDB
- **DynamoDBReadPolicy**: Read-only DynamoDB
- **S3ReadPolicy**: Read S3 objects
- **S3CrudPolicy**: Full S3 access
- **SQSSendMessagePolicy**: Send SQS messages
- **SNSPublishMessagePolicy**: Publish SNS messages
- **CloudWatchPutMetricPolicy**: Put CloudWatch metrics

**Na prova:** Policy Templates são preferíveis a políticas IAM customizadas.

<br />

## SAR - Serverless Application Repository

**Conceitos importantes:**
- Repositório público de aplicações serverless
- Permite compartilhar e reutilizar aplicações SAM
- Aplicações podem ser deployed diretamente do SAR

**Metadata para publicação:**
```yaml
Metadata:
  AWS::ServerlessRepo::Application:
    Name: my-serverless-app
    Description: "Aplicação exemplo"
    Author: "Nome do Autor"
    SpdxLicenseId: MIT
    Labels: ['serverless', 'lambda']
    SemanticVersion: 1.0.0
```

**Deploy do SAR:**
```bash
sam deploy --guided --template-file <SAR-APP-URL>
```

<br />

## Conceitos de Prova

### 1. Quando usar SAM vs CloudFormation direto
**Use SAM quando:**
- Aplicações serverless (Lambda, API Gateway, DynamoDB)
- Desenvolvimento local necessário
- Templates simples e legíveis

**Use CloudFormation quando:**
- Recursos não-serverless complexos
- Controle granular sobre configurações
- Integrações avançadas

### 2. Globals Section
```yaml
Globals:
  Function:
    Runtime: python3.9
    Timeout: 30
    MemorySize: 256
    Environment:
      Variables:
        LOG_LEVEL: INFO
  Api:
    Cors:
      AllowOrigin: "'*'"
```

**Conceito:** Globals aplica configurações para todos os recursos do mesmo tipo.

### 3. Intrinsic Functions
SAM suporta todas as funções intrínsecas do CloudFormation:
- `!Ref`, `!GetAtt`, `!Sub`, `!Join`
- `!FindInMap`, `!Select`, `!Split`

### 4. Parameters e Outputs
```yaml
Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]

Outputs:
  ApiUrl:
    Description: "URL da API"
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/"
    Export:
      Name: !Sub "${AWS::StackName}-ApiUrl"
```

### 5. Conditional Resources
```yaml
Conditions:
  IsProd: !Equals [!Ref Environment, prod]

Resources:
  ProductionOnlyResource:
    Type: AWS::S3::Bucket
    Condition: IsProd
```

<br />

## Quiz

### 1. Qual declaração é obrigatória em todo template SAM?
**a)** `AWSTemplateFormatVersion: '2010-09-09'`  
**b)** `Transform: AWS::Serverless-2016-10-31`  
**c)** `Description: Template SAM`  
**d)** `Parameters: {}`

**Resposta:** b) `Transform: AWS::Serverless-2016-10-31`

**Explicação:** A declaração Transform é obrigatória e informa ao CloudFormation que o template usa a especificação SAM e deve ser transformado antes do deployment.

### 2. Qual comando executa uma função Lambda localmente?
**a)** `sam local start-lambda`  
**b)** `sam local invoke`  
**c)** `sam local run`  
**d)** `sam local execute`

**Resposta:** b) `sam local invoke`

**Explicação:** `sam local invoke` executa uma função Lambda específica localmente usando Docker, permitindo testes sem deployment.

### 3. Qual recurso SAM cria uma tabela DynamoDB simples?
**a)** `AWS::Serverless::Database`  
**b)** `AWS::Serverless::Table`  
**c)** `AWS::Serverless::SimpleTable`  
**d)** `AWS::Serverless::DynamoDB`

**Resposta:** c) `AWS::Serverless::SimpleTable`

**Explicação:** AWS::Serverless::SimpleTable cria uma tabela DynamoDB com configurações básicas e apenas chave primária simples.

### 4. Qual tipo de evento configura um trigger S3 para uma função Lambda?
**a)** `S3Trigger`  
**b)** `S3`  
**c)** `S3Event`  
**d)** `BucketEvent`

**Resposta:** b) `S3`

**Explicação:** O tipo de evento `S3` configura um trigger S3 que invoca a função Lambda quando eventos especificados ocorrem no bucket.

### 5. Qual policy template permite operações CRUD no DynamoDB?
**a)** `DynamoDBFullAccess`  
**b)** `DynamoDBCrudPolicy`  
**c)** `DynamoDBReadWritePolicy`  
**d)** `TableCrudPolicy`

**Resposta:** b) `DynamoDBCrudPolicy`

**Explicação:** DynamoDBCrudPolicy é um policy template SAM que concede permissões de Create, Read, Update e Delete para uma tabela DynamoDB específica.

### 6. O que o comando `sam build` faz?
**a)** Faz deploy da aplicação  
**b)** Compila e prepara a aplicação para deployment  
**c)** Executa testes unitários  
**d)** Valida o template SAM

**Resposta:** b) Compila e prepara a aplicação para deployment

**Explicação:** `sam build` compila o código, resolve dependências e transforma o template SAM em um template CloudFormation, preparando tudo para deployment.

### 7. Qual seção permite definir configurações globais para todos os recursos de um tipo?
**a)** `Parameters`  
**b)** `Globals`  
**c)** `Defaults`  
**d)** `Common`

**Resposta:** b) `Globals`

**Explicação:** A seção Globals permite definir propriedades que serão aplicadas a todos os recursos do mesmo tipo, evitando repetição no template.

### 8. O que é o SAR?
**a)** SAM Application Runtime  
**b)** Serverless Application Repository  
**c)** SAM Auto Resolver  
**d)** Serverless API Resource

**Resposta:** b) Serverless Application Repository

**Explicação:** SAR (Serverless Application Repository) é um repositório público da AWS onde desenvolvedores podem compartilhar e descobrir aplicações serverless prontas para uso.

### 9. Qual diferença entre API e HttpApi em eventos SAM?
**a)** API é para REST, HttpApi para GraphQL  
**b)** Não há diferença  
**c)** HttpApi é mais barato e suporta HTTP/2  
**d)** API é deprecated

**Resposta:** c) HttpApi é mais barato e suporta HTTP/2

**Explicação:** HttpApi cria um API Gateway HTTP API que é mais barato, mais rápido e suporta HTTP/2, enquanto Api cria um REST API com mais funcionalidades.

### 10. Como SAM Local simula o ambiente Lambda?
**a)** Usando máquinas virtuais  
**b)** Usando Docker containers  
**c)** Executando diretamente no sistema  
**d)** Conectando com AWS

**Resposta:** b) Usando Docker containers

**Explicação:** SAM Local usa Docker para criar containers que simulam o ambiente de execução Lambda, permitindo testes locais idênticos ao ambiente de produção.

### 11. Qual comando valida um template SAM?
**a)** `sam check`  
**b)** `sam validate`  
**c)** `sam verify`  
**d)** `sam test`

**Resposta:** b) `sam validate`

**Explicação:** `sam validate` verifica se o template SAM está sintaticamente correto e segue as especificações do SAM/CloudFormation.

### 12. O que acontece quando uma função SAM não especifica um IAM role?
**a)** Deployment falha  
**b)** SAM cria automaticamente um role básico  
**c)** Função executa sem permissões  
**d)** Usa role padrão da conta

**Resposta:** b) SAM cria automaticamente um role básico

**Explicação:** Quando não especificado, SAM cria automaticamente um IAM role com permissões básicas de execução Lambda (AWSLambdaBasicExecutionRole).

### 13. Qual é a limitação do AWS::Serverless::SimpleTable?
**a)** Não suporta encryption  
**b)** Máximo 100 itens  
**c)** Apenas chave primária simples  
**d)** Não suporta streams

**Resposta:** c) Apenas chave primária simples

**Explicação:** SimpleTable só suporta uma chave primária simples. Para tabelas com chave composta, GSI/LSI ou configurações avançadas, deve-se usar AWS::DynamoDB::Table.

### 14. Qual evento SAM configura execução periódica (cron)?
**a)** `Timer`  
**b)** `Schedule`  
**c)** `Cron`  
**d)** `CloudWatchEvent`

**Resposta:** b) `Schedule`

**Explicação:** O tipo de evento `Schedule` permite configurar execução periódica usando expressões rate() ou cron(), criando uma regra CloudWatch Events.

### 15. Como especificar dependências Python em uma função SAM?
**a)** No template.yaml  
**b)** Arquivo requirements.txt no CodeUri  
**c)** Seção Dependencies  
**d)** Variável de ambiente

**Resposta:** b) Arquivo requirements.txt no CodeUri

**Explicação:** Para Python, as dependências são especificadas em um arquivo requirements.txt dentro do diretório CodeUri. SAM automaticamente instala essas dependências durante o build.

<br />

## :books: Referências

- [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
- [SAM Template Specification](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification.html)
- [SAM Policy Templates](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-policy-templates.html)
- [SAM CLI Command Reference](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-command-reference.html)

<br />

---
