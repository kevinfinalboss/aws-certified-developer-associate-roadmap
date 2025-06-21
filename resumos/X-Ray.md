<p align="center">
	<img src="./img/aws-icons/aws-X-Ray.png" alt="aws-x-ray-icon" style="height:150px; width:150px;" /> 
  <br />
	<h1 align="center">
    X-Ray - Guia Completo AWS Developer
  </h1>
</p>	
<br />

## :pushpin: Índice
- [Introdução](#introdução)
- [Conceitos Fundamentais](#conceitos-fundamentais)
- [Arquitetura do X-Ray](#arquitetura-do-x-ray)
- [Compatibilidade do X-Ray](#compatibilidade-do-x-ray)
- [Como habilitar o X-Ray](#como-habilitar-o-x-ray)
- [Configuração por Serviço](#configuração-por-serviço)
- [Amostragem (Sampling)](#amostragem-sampling)
- [Annotations vs Metadata](#annotations-vs-metadata)
- [Segurança e IAM](#segurança-e-iam)
- [Troubleshooting](#troubleshooting)
- [Perguntas e Respostas](#perguntas-e-respostas)
- [Referências](#books-referências)

<br />

## Introdução
Amazon **X-Ray** é utilizada para realizar o debug das suas aplicações. Ele fornece uma análise visual da sua aplicação, resolução de problemas de performance, entender dependências em arquiteturas de micro-serviços, como cada solicitação(*request*) está se comportando e a sua resposta, qual serviço está causando problema, etc.

Debugar monolitos é muito mais fácil se comparado a serviços distribuídos e micro-serviços.

**Principais benefícios:**
- Resolução de problemas de performance e erros
- Rastreamento distribuído de microsserviços
- Análise de latência end-to-end
- Identificação de gargalos
- Mapa visual de arquitetura

**Disponível nas linguagens:**
- C# .NET
- Go
- Java
- Node.js
- Python
- Ruby

O Amazon X-Ray recebe os dados dos serviços como *segmentos*.

<br />

## Conceitos Fundamentais

### Traces
Um **trace** coleta todos os segmentos gerados por uma única solicitação. Representa o caminho completo da requisição através da aplicação.

### Segments
Um **segment** fornece o nome do recurso, detalhes sobre a solicitação e detalhes sobre o trabalho realizado.

### Subsegments
**Subsegments** fornecem informações mais granulares sobre uma chamada para um serviço downstream que sua aplicação fez para atender à solicitação original.

### Service Map
Representação visual JSON que detalha os serviços que compõem sua aplicação e como esses serviços se conectam.

### Sampling
Determina quantas solicitações são registradas pelo X-Ray. Por padrão, o X-Ray registra a primeira solicitação a cada segundo e 5% das solicitações adicionais.

<br />

## Arquitetura do X-Ray

1. **X-Ray SDK** - Captura dados de trace da aplicação
2. **X-Ray Daemon** - Escuta o tráfego UDP na porta 2000 e encaminha para a API do X-Ray
3. **X-Ray API** - Recebe trace data do daemon
4. **X-Ray Console** - Interface visual para análise

<br />

## Compatibilidade do X-Ray

### Serviços AWS com Integração Nativa
- **API Gateway** - Tracing automático habilitado com um clique
- **AWS Lambda** - Configuração via console ou variável de ambiente
- **Elastic Beanstalk** - Configuração via console
- **ECS** - Daemon como sidecar container
- **EKS** - Daemon como DaemonSet
- **SNS** - Tracing de mensagens
- **SQS** - Tracing de mensagens

### Serviços de Compute
- **EC2** - Instalar X-Ray daemon
- **On-premise** - Instalar X-Ray daemon
- **ECS** - X-Ray daemon container
- **EKS** - X-Ray daemon pod

### Load Balancers
- **Application Load Balancer (ALB)** - Adiciona trace ID nos headers
- **Network Load Balancer (NLB)** - Suporte limitado

<br />

## Como habilitar o X-Ray

### Requisitos Básicos
1. **Importar a SDK do X-Ray** no código da aplicação
2. **Executar o X-Ray daemon** no ambiente
3. **Configurar IAM permissions** apropriadas

### Configuração do Daemon
```bash
# Download do daemon
wget https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-3.x.x.zip

# Executar o daemon
./xray -o -n us-east-1
```

### Variáveis de Ambiente Importantes
- `_X_AMZN_TRACE_ID` - Contém o trace ID
- `AWS_XRAY_TRACING_NAME` - Nome do serviço
- `AWS_XRAY_DAEMON_ADDRESS` - Endereço do daemon
- `AWS_XRAY_CONTEXT_MISSING` - Comportamento quando contexto está faltando (LOG_ERROR ou IGNORE_ERROR)

<br />

## Configuração por Serviço

### Lambda
```python
import aws_xray_sdk.core
from aws_xray_sdk.core import xray_recorder

@xray_recorder.capture('lambda_handler')
def lambda_handler(event, context):
    return 'Hello World'
```

**Habilitação:**
- Console: Configuration → Monitoring → Enable X-Ray tracing
- Environment variable: `_X_AMZN_TRACE_ID`

### API Gateway
- **Habilitação:** Stage settings → X-Ray Tracing
- **Automatic:** Traces são criados automaticamente
- **Headers:** API Gateway adiciona `X-Amzn-Trace-Id` header

### ECS
```yaml
# Task definition
{
  "name": "xray-daemon",
  "image": "amazon/aws-xray-daemon",
  "cpu": 32,
  "memoryReservation": 256,
  "portMappings": [
    {
      "containerPort": 2000,
      "protocol": "udp"
    }
  ]
}
```

### Elastic Beanstalk
- **Configuração:** Configuration → Software → X-Ray daemon
- **Automatic:** Daemon é instalado automaticamente
- **.ebextensions:** Configuração personalizada via arquivos de configuração

<br />

## Amostragem (Sampling)

### Regras de Amostragem Padrão
- **1 requisição por segundo** (reservoir)
- **5% das requisições adicionais** (rate)

### Regras Personalizadas
```json
{
  "version": 2,
  "default": {
    "fixed_target": 1,
    "rate": 0.1
  },
  "rules": [
    {
      "description": "Player moves",
      "service_name": "Scorekeep",
      "http_method": "POST",
      "url_path": "/api/move/*",
      "fixed_target": 2,
      "rate": 0.05
    }
  ]
}
```

### Configuração via Console
- **X-Ray Console** → Sampling rules
- **Criar regras** baseadas em service name, HTTP method, URL path
- **Prioridade** determina ordem de avaliação

<br />

## Annotations vs Metadata

### Annotations
- **Indexadas** e **pesquisáveis**
- **Limitadas** a string, number, boolean
- **Filtros** no console
- **Máximo 50 annotations** por segment

```python
from aws_xray_sdk.core import xray_recorder

subsegment = xray_recorder.begin_subsegment('annotations')
subsegment.put_annotation('game_name', 'TicTacToe')
subsegment.put_annotation('game_id', 12345)
xray_recorder.end_subsegment()
```

### Metadata
- **NÃO indexadas** nem pesquisáveis
- **Qualquer tipo** de objeto
- **Informações adicionais** para debugging
- **Sem limite** específico

```python
from aws_xray_sdk.core import xray_recorder

subsegment = xray_recorder.begin_subsegment('metadata')
subsegment.put_metadata('game_result', {
    'winner': 'Player 1',
    'moves': 7,
    'time': 234
})
xray_recorder.end_subsegment()
```

<br />

## Segurança e IAM

### Permissions Necessárias

#### Para X-Ray Daemon
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "xray:PutTraceSegments",
        "xray:PutTelemetryRecords"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Para Aplicação (leitura)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "xray:GetSamplingRules",
        "xray:GetSamplingTargets",
        "xray:GetSamplingStatisticSummaries"
      ],
      "Resource": "*"
    }
  ]
}
```

### Service-Linked Role
- **AWSXRayServiceLinkedRole** criada automaticamente
- **Permissions mínimas** para operação do serviço

<br />

## Troubleshooting

### Traces não aparecem
1. **Verificar IAM permissions**
2. **X-Ray daemon executando** na porta 2000
3. **SDK configurada** corretamente
4. **Sampling rules** muito restritivas
5. **Clock synchronization** entre serviços

### Missing Segments
- **CONTEXT_MISSING** environment variable
- **Async operations** não instrumentadas
- **Cold starts** no Lambda

### Performance Impact
- **Overhead mínimo** < 1% CPU
- **Sampling** reduz impacto
- **Async sending** não bloqueia aplicação

<br />

## Perguntas e Respostas

### Pergunta 1
**Você está desenvolvendo uma aplicação Lambda que faz chamadas para DynamoDB e S3. Como você habilitaria o X-Ray tracing para monitorar toda a cadeia de requests?**

**Resposta:** 
Para habilitar X-Ray tracing em Lambda:
1. **Habilitar tracing** no console Lambda (Configuration → Monitoring → Enable X-Ray tracing)
2. **Importar SDK** do X-Ray no código
3. **Instrumentar AWS SDK** para capturar chamadas para DynamoDB e S3
4. **Configurar IAM role** com permissões xray:PutTraceSegments

**Código exemplo:**
```python
import aws_xray_sdk.core
from aws_xray_sdk.core import patch_all

patch_all()  # Instrumenta automaticamente AWS SDK

def lambda_handler(event, context):
    # Operações com DynamoDB e S3 serão automaticamente traced
    pass
```

### Pergunta 2
**Qual é a diferença entre annotations e metadata no X-Ray e quando usar cada um?**

**Resposta:**
- **Annotations:** São indexadas e pesquisáveis no console. Use para dados que você quer filtrar (user_id, error_type, region). Limitadas a 50 por segment e apenas string/number/boolean.
- **Metadata:** Não são indexadas nem pesquisáveis. Use para informações detalhadas de debugging (request payload, response data). Sem limite de tipo ou quantidade.

**Quando usar:**
- **Annotations:** IDs, status codes, categorias para filtros
- **Metadata:** Dados complexos, payloads, informações detalhadas

### Pergunta 3
**Uma aplicação ECS não está enviando traces para X-Ray. Quais são as possíveis causas?**

**Resposta:**
Possíveis causas e soluções:
1. **X-Ray daemon não está rodando:** Configurar daemon como sidecar container
2. **Task Role sem permissões:** Adicionar políticas xray:PutTraceSegments e xray:PutTelemetryRecords
3. **Network configuration:** Daemon deve escutar na porta 2000 (UDP)
4. **SDK não configurada:** Aplicação deve usar AWS X-Ray SDK
5. **Sampling rules muito restritivas:** Verificar regras de amostragem

### Pergunta 4
**Como configurar sampling rules personalizadas para diferentes tipos de requests?**

**Resposta:**
Sampling rules permitem controlar quais requests são traced:

**Via Console:**
1. X-Ray Console → Sampling rules → Create rule
2. Definir service name, HTTP method, URL path
3. Configurar fixed target (requests/segundo) e rate (% adicional)

**Exemplo de regra:**
- Service: "user-service"  
- HTTP Method: "POST"
- URL Path: "/api/users/*"
- Fixed target: 2 (2 requests por segundo sempre traced)
- Rate: 0.1 (10% dos requests adicionais)

### Pergunta 5
**O que acontece quando o X-Ray daemon não está disponível?**

**Resposta:**
Quando o daemon não está disponível:
1. **Aplicação continua funcionando** normalmente
2. **Traces são perdidos** (não enviados)
3. **Sem impacto na performance** da aplicação
4. **Logs de erro** podem aparecer (configurável via AWS_XRAY_CONTEXT_MISSING)

**Configuração do comportamento:**
- `AWS_XRAY_CONTEXT_MISSING=LOG_ERROR` - Log errors (padrão)
- `AWS_XRAY_CONTEXT_MISSING=IGNORE_ERROR` - Ignora silenciosamente

### Pergunta 6
**Como o API Gateway integra com X-Ray?**

**Resposta:**
API Gateway oferece integração nativa com X-Ray:

**Habilitação:**
- Stage Settings → X-Ray Tracing → Enable

**Funcionalidades:**
- **Automatic tracing:** Traces criados automaticamente para cada request
- **Trace ID header:** Adiciona X-Amzn-Trace-Id header downstream
- **Integration tracing:** Traces incluem chamadas para Lambda, HTTP endpoints
- **Error tracking:** Captura 4xx e 5xx errors automaticamente

**Limitações:**
- Apenas stages podem ser habilitados (não resources individuais)
- Adiciona latência mínima (~1ms)

### Pergunta 7
**Quais são as melhores práticas para instrumentação manual com X-Ray SDK?**

**Resposta:**
Melhores práticas para instrumentação manual:

1. **Patch AWS SDK automaticamente:**
```python
from aws_xray_sdk.core import patch_all
patch_all()
```

2. **Usar decorators para funções:**
```python
@xray_recorder.capture('custom_function')
def my_function():
    pass
```

3. **Subsegments para operações específicas:**
```python
subsegment = xray_recorder.begin_subsegment('database_query')
# operação
xray_recorder.end_subsegment()
```

4. **Exception handling:**
```python
try:
    # operação
except Exception as e:
    subsegment.add_exception(e)
    raise
```

### Pergunta 8
**Como o X-Ray funciona com containers em EKS?**

**Resposta:**
Para usar X-Ray com EKS:

1. **DaemonSet deployment:** Deploy X-Ray daemon como DaemonSet
2. **Service account:** Configurar RBAC apropriado
3. **IAM for Service Accounts (IRSA):** Associar IAM role ao service account
4. **Environment variables:** Configurar aplicações para usar daemon local

**Exemplo de DaemonSet:**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: xray-daemon
spec:
  template:
    spec:
      containers:
      - name: xray-daemon
        image: amazon/aws-xray-daemon
        ports:
        - containerPort: 2000
          protocol: UDP
```

<br />

## :books: Referências
Para uma compreensão mais profunda sobre Amazon X-Ray recomendo a leitura da documentação oficial:

- [O que é o Amazon X-Ray?](https://docs.aws.amazon.com/pt_br/xray/latest/devguide/aws-xray.html)
- [AWS X-Ray Developer Guide](https://docs.aws.amazon.com/xray/latest/devguide/)
- [X-Ray API Reference](https://docs.aws.amazon.com/xray/latest/api/)
- [AWS X-Ray Pricing](https://aws.amazon.com/xray/pricing/)
- [X-Ray Sample Applications](https://docs.aws.amazon.com/xray/latest/devguide/xray-sample-apps.html)

<br />

---
Feito com ♥ by :man_astronaut: Guilherme Bezerra :wave: [Entrar em contato!](https://www.linkedin.com/in/gbdsantos/)
