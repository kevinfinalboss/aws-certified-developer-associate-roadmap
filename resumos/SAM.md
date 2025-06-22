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
- [Conceitos Fundamentais](#conceitos-fundamentais)
- [Instalação e Configuração](#instalação-e-configuração)
- [Estrutura do Template SAM](#estrutura-do-template-sam)
- [Recursos Principais](#recursos-principais)
- [Comandos Essenciais](#comandos-essenciais)
- [Desenvolvimento Local](#desenvolvimento-local)
- [Testando Localmente](#testando-localmente)
- [Pipeline de Deployment](#pipeline-de-deployment)
- [Políticas e Permissões](#políticas-e-permissões)
- [Monitoramento e Debug](#monitoramento-e-debug)
- [SAR - Serverless Application Repository](#sar---serverless-application-repository)
- [Melhores Práticas](#melhores-práticas)
- [Exemplos Práticos](#exemplos-práticos)
- [Quiz](#quiz)
- [Referências](#books-referências)

<br />

## Introdução

Amazon SAM (Serverless Application Model) é um framework open-source da AWS para desenvolvimento e deployment de aplicações serverless. SAM estende o AWS CloudFormation com uma sintaxe simplificada especificamente otimizada para funções Lambda, APIs, bancos de dados e mapeamentos de origem de eventos.

**Características principais:**
- Sintaxe simplificada baseada em YAML/JSON
- Transformação automática para CloudFormation
- Desenvolvimento e teste local
- Deployment automatizado
- Integração nativa com serviços AWS

<br />

## Conceitos Fundamentais

### Transform Declaration
```yaml
Transform: AWS::Serverless-2016-10-31
```
Esta declaração indica ao CloudFormation que o template utiliza a especificação SAM e deve ser transformado antes do deployment.

### Recursos SAM vs CloudFormation
SAM oferece recursos simplificados que são expandidos para múltiplos recursos CloudFormation:
- `AWS::Serverless::Function` → Lambda Function + IAM Role + Event Sources
- `AWS::Serverless::Api` → API Gateway + Deployment + Stage
- `AWS::Serverless::SimpleTable` → DynamoDB Table

<br />

## Instalação e Configuração

### Pré-requisitos
- AWS CLI configurado
- Docker (para desenvolvimento local)
- Python 3.7+ ou runtime desejado

### Instalação SAM CLI
```bash
pip install aws-sam-cli
sam --version
```

### Inicialização de projeto
```bash
sam init
sam init --runtime python3.9 --name my-sam-app
sam init --app-template hello-world --runtime nodejs14.x
```

<br />

## Estrutura do Template SAM

### Template Básico
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: 'Aplicação SAM exemplo'

Globals:
  Function:
    Timeout: 30
    MemorySize: 512
    Runtime: python3.9

Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]

Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/
      Handler: app.lambda_handler
      Environment:
        Variables:
          ENV: !Ref Environment

Outputs:
  MyFunctionArn:
    Description: "ARN da função Lambda"
    Value: !GetAtt MyFunction.Arn
    Export:
      Name: !Sub "${AWS::StackName}-MyFunctionArn"
```

<br />

## Recursos Principais

### AWS::Serverless::Function
```yaml
MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    CodeUri: src/
    Handler: index.handler
    Runtime: nodejs14.x
    Timeout: 30
    MemorySize: 256
    ReservedConcurrencyLimit: 100
    Environment:
      Variables:
        TABLE_NAME: !Ref MyTable
    Events:
      Api:
        Type: Api
        Properties:
          Path: /users
          Method: get
      Schedule:
        Type: Schedule
        Properties:
          Schedule: rate(5 minutes)
    Layers:
      - !Ref MyLayer
    DeadLetterQueue:
      Type: SQS
      TargetArn: !GetAtt MyDeadLetterQueue.Arn
```

### AWS::Serverless::Api
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
    GatewayResponses:
      DEFAULT_4XX:
        ResponseParameters:
          Headers:
            Access-Control-Allow-Origin: "'*'"
    Cors:
      AllowMethods: "'*'"
      AllowHeaders: "'*'"
      AllowOrigin: "'*'"
```

### AWS::Serverless::SimpleTable
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
    SSESpecification:
      SSEEnabled: true
    StreamSpecification:
      StreamViewType: NEW_AND_OLD_IMAGES
```

### AWS::Serverless::LayerVersion
```yaml
MyLayer:
  Type: AWS::Serverless::LayerVersion
  Properties:
    LayerName: my-dependencies
    Description: Dependências compartilhadas
    ContentUri: dependencies/
    CompatibleRuntimes:
      - python3.9
      - python3.8
    RetentionPolicy: Delete
```

<br />

## Comandos Essenciais

### Build e Package
```bash
sam build
sam build --use-container
sam build --build-dir custom-build-dir
sam package --s3-bucket my-bucket --output-template-file packaged.yaml
```

### Deploy
```bash
sam deploy --guided
sam deploy --stack-name my-stack --s3-bucket my-bucket --capabilities CAPABILITY_IAM
sam deploy --parameter-overrides Environment=prod
```

### Validação
```bash
sam validate
sam validate --template template.yaml
```

### Logs
```bash
sam logs -n MyFunction --stack-name my-stack --tail
sam logs -n MyFunction --start-time '10min ago' --end-time '2min ago'
```

<br />

## Desenvolvimento Local

### Executando API localmente
```bash
sam local start-api
sam local start-api --port 8080
sam local start-api --env-vars env.json
```

### Executando Lambda localmente
```bash
sam local invoke MyFunction
sam local invoke MyFunction --event events/event.json
sam local invoke MyFunction --env-vars env.json
```

### Gerando eventos de teste
```bash
sam local generate-event apigateway aws-proxy > event.json
sam local generate-event s3 put > s3-event.json
sam local generate-event dynamodb update > dynamodb-event.json
```

<br />

## Testando Localmente

### Estrutura de testes
```yaml
MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    CodeUri: src/
    Handler: app.lambda_handler
    Events:
      TestEvent:
        Type: Api
        Properties:
          Path: /test
          Method: post
```

### Arquivo de eventos personalizados
```json
{
  "httpMethod": "POST",
  "body": "{\"message\": \"hello world\"}",
  "headers": {
    "Content-Type": "application/json"
  },
  "pathParameters": {
    "id": "123"
  },
  "queryStringParameters": {
    "filter": "active"
  }
}
```

### Variáveis de ambiente para testes
```json
{
  "MyFunction": {
    "TABLE_NAME": "local-table",
    "DEBUG": "true",
    "AWS_DEFAULT_REGION": "us-east-1"
  }
}
```

<br />

## Pipeline de Deployment

### Template com múltiplos estágios
```yaml
Parameters:
  Stage:
    Type: String
    Default: dev

Mappings:
  StageMap:
    dev:
      MemorySize: 128
      Timeout: 30
    prod:
      MemorySize: 512
      Timeout: 60

Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      MemorySize: !FindInMap [StageMap, !Ref Stage, MemorySize]
      Timeout: !FindInMap [StageMap, !Ref Stage, Timeout]
```

### Script de deployment automatizado
```bash
#!/bin/bash
STACK_NAME="my-app-${STAGE}"
S3_BUCKET="my-deployment-bucket-${STAGE}"

sam build
sam package --s3-bucket $S3_BUCKET --output-template-file packaged.yaml
sam deploy --template-file packaged.yaml --stack-name $STACK_NAME --parameter-overrides Stage=$STAGE --capabilities CAPABILITY_IAM
```

<br />

## Políticas e Permissões

### Políticas SAM pré-definidas
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

### Políticas customizadas
```yaml
MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    Policies:
      - Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Action:
              - secretsmanager:GetSecretValue
            Resource: !Ref MySecret
```

<br />

## Monitoramento e Debug

### CloudWatch Insights
```yaml
MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    Tracing: Active
    Environment:
      Variables:
        _X_AMZN_TRACE_ID: !Ref AWS::NoValue
```

### Configuração de alarmes
```yaml
HighErrorRateAlarm:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmDescription: "Alta taxa de erros na função"
    MetricName: Errors
    Namespace: AWS/Lambda
    Statistic: Sum
    Period: 300
    EvaluationPeriods: 2
    Threshold: 5
    ComparisonOperator: GreaterThanThreshold
    Dimensions:
      - Name: FunctionName
        Value: !Ref MyFunction
```

<br />

## SAR - Serverless Application Repository

### Publicando aplicação no SAR
```yaml
Metadata:
  AWS::ServerlessRepo::Application:
    Name: my-serverless-app
    Description: "Aplicação serverless de exemplo"
    Author: "Seu Nome"
    SpdxLicenseId: MIT
    LicenseUrl: LICENSE.txt
    ReadmeUrl: README.md
    Labels: ['serverless', 'lambda', 'api']
    HomePageUrl: https://github.com/user/repo
    SemanticVersion: 1.0.0
    SourceCodeUrl: https://github.com/user/repo
```

### Deployment via SAR
```bash
sam publish --template packaged.yaml --region us-east-1
sam deploy --guided --template-file https://serverlessrepo.aws.amazon.com/applications/...
```

<br />

## Melhores Práticas

### Organização de código
```
my-sam-app/
├── template.yaml
├── samconfig.toml
├── src/
│   ├── functions/
│   │   ├── user/
│   │   │   ├── app.py
│   │   │   └── requirements.txt
│   │   └── order/
│   │       ├── app.py
│   │       └── requirements.txt
│   └── layers/
│       └── common/
│           └── python/
│               └── lib/
├── events/
│   ├── user-event.json
│   └── order-event.json
└── tests/
    ├── unit/
    └── integration/
```

### Configuração por ambiente
```toml
# samconfig.toml
version = 0.1

[default.deploy.parameters]
stack_name = "my-app-dev"
s3_bucket = "my-deployment-bucket-dev"
region = "us-east-1"
capabilities = "CAPABILITY_IAM"

[prod.deploy.parameters]
stack_name = "my-app-prod"
s3_bucket = "my-deployment-bucket-prod"
parameter_overrides = "Environment=prod"
```

<br />

## Exemplos Práticos

### API REST completa
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Globals:
  Function:
    Runtime: python3.9
    Timeout: 30
    Environment:
      Variables:
        TABLE_NAME: !Ref UsersTable

Resources:
  UsersApi:
    Type: AWS::Serverless::Api
    Properties:
      StageName: prod
      Cors:
        AllowMethods: "'*'"
        AllowHeaders: "'*'"
        AllowOrigin: "'*'"

  CreateUserFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/create_user/
      Handler: app.lambda_handler
      Events:
        CreateUser:
          Type: Api
          Properties:
            RestApiId: !Ref UsersApi
            Path: /users
            Method: post
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref UsersTable

  GetUserFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/get_user/
      Handler: app.lambda_handler
      Events:
        GetUser:
          Type: Api
          Properties:
            RestApiId: !Ref UsersApi
            Path: /users/{id}
            Method: get
      Policies:
        - DynamoDBReadPolicy:
            TableName: !Ref UsersTable

  UsersTable:
    Type: AWS::Serverless::SimpleTable
    Properties:
      PrimaryKey:
        Name: id
        Type: String
```

### Processamento de eventos S3
```yaml
ProcessS3Function:
  Type: AWS::Serverless::Function
  Properties:
    CodeUri: src/process_s3/
    Handler: app.lambda_handler
    Events:
      S3Event:
        Type: S3
        Properties:
          Bucket: !Ref ProcessingBucket
          Events: s3:ObjectCreated:*
          Filter:
            S3Key:
              Rules:
                - Name: prefix
                  Value: uploads/
                - Name: suffix
                  Value: .json
    Policies:
      - S3ReadPolicy:
          BucketName: !Ref ProcessingBucket

ProcessingBucket:
  Type: AWS::S3::Bucket
  Properties:
    NotificationConfiguration:
      LambdaConfigurations:
        - Event: s3:ObjectCreated:*
          Function: !GetAtt ProcessS3Function.Arn
```

<br />

## Quiz

### 1. O que significa o Transform header em um template SAM?
**Resposta:** `Transform: AWS::Serverless-2016-10-31`

**Explicação:** O Transform header indica ao AWS CloudFormation que este é um template SAM que precisa ser transformado de recursos SAM simplificados para recursos CloudFormation completos antes do deployment. Esta declaração é obrigatória em todos os templates SAM.

### 2. Qual comando é usado para testar uma função Lambda localmente?
**Resposta:** `sam local invoke`

**Explicação:** O comando `sam local invoke` permite executar uma função Lambda específica localmente usando Docker, simulando o ambiente de execução da AWS. Pode ser usado com eventos personalizados através do parâmetro `--event`.

### 3. Qual recurso SAM é usado para criar uma tabela DynamoDB simples?
**Resposta:** `AWS::Serverless::SimpleTable`

**Explicação:** AWS::Serverless::SimpleTable é um recurso SAM que cria uma tabela DynamoDB com configurações básicas. Para tabelas mais complexas com múltiplos índices e configurações avançadas, deve-se usar AWS::DynamoDB::Table.

### 4. Como você especifica variáveis de ambiente em uma função SAM?
**Resposta:** 
```yaml
Environment:
  Variables:
    VARIABLE_NAME: value
```

**Explicação:** As variáveis de ambiente são definidas na seção Environment.Variables da função. Elas ficam disponíveis no runtime da função Lambda e podem referenciar outros recursos usando funções intrínsecas como !Ref.

### 5. Qual é a diferença entre sam build e sam package?
**Resposta:** 
- `sam build`: Compila a aplicação localmente e converte template SAM para CloudFormation
- `sam package`: Empacota o código em ZIP e faz upload para S3, gerando template com referências S3

**Explicação:** sam build prepara a aplicação para deployment criando artefatos locais, enquanto sam package carrega esses artefatos para S3 e atualiza o template com as URLs S3, necessário para deployment.

### 6. O que são as políticas SAM pré-definidas?
**Resposta:** Políticas IAM simplificadas fornecidas pelo SAM para permissões comuns

**Explicação:** SAM oferece políticas como DynamoDBCrudPolicy, S3ReadPolicy, SQSSendMessagePolicy que são expandidas automaticamente para permissões IAM completas, simplificando a configuração de segurança.

### 7. Como você configura CORS em uma API SAM?
**Resposta:**
```yaml
Cors:
  AllowMethods: "'*'"
  AllowHeaders: "'*'"
  AllowOrigin: "'*'"
```

**Explicação:** CORS é configurado no recurso AWS::Serverless::Api ou globalmente. Os valores devem estar entre aspas simples dentro de aspas duplas devido à sintaxe YAML/CloudFormation.

### 8. Qual comando inicia uma API Gateway local para desenvolvimento?
**Resposta:** `sam local start-api`

**Explicação:** Este comando inicia um servidor local que simula API Gateway, permitindo testar endpoints HTTP localmente. A API fica disponível por padrão em http://127.0.0.1:3000.

### 9. O que é o SAR (Serverless Application Repository)?
**Resposta:** Repositório público da AWS para compartilhar e descobrir aplicações serverless

**Explicação:** SAR permite que desenvolvedores publiquem aplicações SAM completas para reuso pela comunidade, incluindo metadados como descrição, licença e parâmetros de configuração.

### 10. Como você especifica dependências de uma função Lambda em SAM?
**Resposta:** Através da propriedade Layers ou incluindo requirements.txt/package.json no CodeUri

**Explicação:** Dependências podem ser incluídas diretamente no código da função ou organizadas em Layers reutilizáveis. Layers são recomendados para dependências compartilhadas entre múltiplas funções.

<br />

## :books: Referências

Para uma compreensão mais profunda sobre SAM recomendo a leitura da documentação oficial:

- [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
- [AWS SAM CLI Reference](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-command-reference.html)
- [SAM Template Specification](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification.html)
- [Serverless Application Repository](https://docs.aws.amazon.com/serverlessrepo/latest/devguide/what-is-serverlessrepo.html)
- [AWS SAM Examples](https://github.com/aws/serverless-application-model/tree/master/examples)

<br />

---
Feito com ♥ by :man_astronaut: Guilherme Bezerra :wave: [Entrar em contato!](https://www.linkedin.com/in/gbdsantos/)
