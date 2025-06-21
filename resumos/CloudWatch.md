<p align="center">
	<img src="./img/aws-icons/aws-CloudWatch.png" alt="aws-cloudwatch-icon" style="height:150px; width:150px;" /> 
  <br />
	<h1 align="center">
    CloudWatch - Guia Completo AWS Developer
  </h1>
</p>	

<br />

## :pushpin: Índice

- [Introdução](#introdução)
- [Métricas](#métricas)
- [Métricas personalizadas](#métricas-personalizadas)
- [Logs](#logs)
  - [Filtros de métricas](#filtros-de-métricas)
  - [CloudWatch Logs Insights](#cloudwatch-logs-insights)
  - [Exportação de logs](#exportação-de-logs)
- [Alarmes](#alarmes)
- [CloudWatch Logs para EC2](#cloudwatch-logs-para-ec2)
- [Eventos e EventBridge](#eventos-e-eventbridge)
- [CloudWatch Dashboards](#cloudwatch-dashboards)
- [CloudWatch Synthetics](#cloudwatch-synthetics)
- [Container Insights](#container-insights)
- [CloudWatch Agent](#cloudwatch-agent)
- [X-Ray Integration](#x-ray-integration)
- [Billing e Custos](#billing-e-custos)
- [Perguntas e Respostas](#perguntas-e-respostas)
- [Referências](#books-referências)

<br />

## Introdução

Amazon **CloudWatch** é um serviço de monitoramento e observabilidade que fornece dados e insights acionáveis para monitorar aplicações, responder a mudanças no desempenho do sistema, otimizar o uso de recursos e obter uma visão unificada da saúde operacional.

**Conceitos chave:**
- **Métricas**: Dados numéricos sobre recursos AWS
- **Logs**: Dados de texto estruturados e não estruturados
- **Eventos**: Mudanças no ambiente AWS
- **Alarmes**: Notificações baseadas em limites de métricas
- **Dashboards**: Visualizações personalizadas

<br />

## Métricas

### Métricas Básicas

CloudWatch coleta métricas automaticamente para serviços AWS. Cada métrica tem:
- **Namespace**: Agrupa métricas relacionadas (ex: AWS/EC2)
- **Dimensões**: Pares chave-valor que identificam unicamente uma métrica
- **Timestamp**: Quando a métrica foi coletada
- **Valor**: Dados numéricos da métrica

### Métricas Detalhadas vs Básicas

**Métricas Básicas (Gratuitas):**
- Intervalo de 5 minutos para EC2
- Intervalo de 1 minuto para ELB, RDS, Lambda

**Métricas Detalhadas (Pagas):**
- Intervalo de 1 minuto para EC2
- Maior granularidade de dados

### Retenção de Métricas

- **Dados de 1 minuto**: 15 dias
- **Dados de 5 minutos**: 63 dias
- **Dados de 1 hora**: 455 dias (15 meses)

<br />

## Métricas Personalizadas

### Publicando Métricas Personalizadas

Você pode enviar suas próprias métricas usando a API PutMetricData:

```bash
aws cloudwatch put-metric-data \
  --namespace "MyApp/Performance" \
  --metric-data MetricName=ResponseTime,Value=120.5,Unit=Milliseconds,Dimensions=Environment=Production
```

### Resolução de Métricas

- **Resolução Padrão**: 1 minuto
- **Alta Resolução**: 1 segundo (custo adicional)

### Limitações

- Máximo de 10 dimensões por métrica
- Máximo de 1000 métricas por chamada PutMetricData
- Timestamp: até 2 semanas no passado e 2 horas no futuro

<br />

## Logs

### Conceitos de Logs

- **Log Groups**: Contêiner para log streams
- **Log Streams**: Sequência de eventos de log da mesma fonte
- **Log Events**: Registro individual com timestamp e mensagem

### Fontes de Logs

- **EC2**: Através do CloudWatch Agent
- **Lambda**: Automaticamente
- **ECS**: Através do log driver
- **API Gateway**: Logs de execução e acesso
- **VPC Flow Logs**: Tráfego de rede

### Filtros de Métricas

Permitem extrair métricas de logs baseadas em padrões:

```json
{
  "filterName": "ErrorCount",
  "filterPattern": "[timestamp, request_id, ERROR]",
  "metricTransformations": [{
    "metricName": "ErrorCount",
    "metricNamespace": "MyApp/Errors",
    "metricValue": "1"
  }]
}
```

### CloudWatch Logs Insights

Serviço de análise de logs que permite consultas interativas:

```sql
fields @timestamp, @message
| filter @message like /ERROR/
| stats count() by bin(5m)
```

### Exportação de Logs

- **Exportação para S3**: Para arquivamento de longo prazo
- **Streaming para Kinesis**: Para processamento em tempo real
- **Subscription Filters**: Para Lambda, Kinesis, ou Kinesis Data Firehose

<br />

## Alarmes

### Tipos de Alarmes

1. **Alarmes de Métrica**: Baseados em métricas do CloudWatch
2. **Alarmes Compostos**: Combinam múltiplos alarmes
3. **Alarmes de Anomalia**: Detectam comportamentos anômalos

### Estados do Alarme

- **OK**: Métrica dentro do limite
- **ALARM**: Métrica violou o limite
- **INSUFFICIENT_DATA**: Dados insuficientes

### Ações do Alarme

- **Auto Scaling**: Escalar recursos automaticamente
- **EC2 Actions**: Parar, iniciar, reiniciar instâncias
- **SNS**: Enviar notificações
- **Systems Manager**: Executar documentos

### Período de Avaliação

- **Período**: Intervalo de tempo para agregação
- **Pontos de Dados**: Quantos períodos considerar
- **Falta de Dados**: Como tratar dados ausentes

<br />

## CloudWatch Logs para EC2

### Configuração do Agent

1. **Instalar CloudWatch Agent**
2. **Configurar IAM Role** com CloudWatchAgentServerPolicy
3. **Configurar arquivo de configuração**:

```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/messages",
            "log_group_name": "/aws/ec2/system",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  }
}
```

### Logs Unificados

O CloudWatch Agent pode coletar:
- Logs do sistema
- Logs de aplicação
- Métricas do sistema
- Métricas personalizadas

<br />

## Eventos e EventBridge

### CloudWatch Events (Legacy)

- Monitora mudanças no ambiente AWS
- Integração com CloudTrail para APIs
- Padrões de eventos para filtragem

### Amazon EventBridge

Substituto do CloudWatch Events com recursos adicionais:
- **Event Buses**: Canais de eventos customizados
- **Schema Registry**: Descoberta de esquemas de eventos
- **Parceiros SaaS**: Integração com terceiros

### Padrões de Eventos

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["running"]
  }
}
```

<br />

## CloudWatch Dashboards

### Recursos dos Dashboards

- **Widgets**: Gráficos, números, textos
- **Métricas**: Visualização de múltiplas métricas
- **Logs**: Integração com Logs Insights
- **Compartilhamento**: URLs públicas e privadas

### Tipos de Widgets

- **Line Chart**: Métricas ao longo do tempo
- **Number**: Valor único
- **Gauge**: Medidor visual
- **Log Table**: Resultados de consultas

<br />

## CloudWatch Synthetics

### Canários (Canaries)

Scripts que monitoram aplicações e endpoints:
- **Heartbeat**: Verificação de disponibilidade
- **API Canary**: Testes de API
- **Broken Link Checker**: Verificação de links
- **Visual Monitoring**: Screenshots e comparações

### Linguagens Suportadas

- **Node.js**: Para scripts personalizados
- **Python**: Para automação avançada

<br />

## Container Insights

### Recursos

- **Métricas de Performance**: CPU, memória, rede, disco
- **Logs de Aplicação**: Coleta automática
- **Mapas de Serviço**: Visualização de dependências

### Plataformas Suportadas

- **Amazon ECS**: Clusters e serviços
- **Amazon EKS**: Clusters Kubernetes
- **Kubernetes**: Cluster auto-gerenciado

<br />

## CloudWatch Agent

### Instalação e Configuração

1. **Download do Agent**
2. **Criação de arquivo de configuração**
3. **Execução com Systems Manager**

### Métricas Coletadas

- **Métricas de Sistema**: CPU, memória, disco
- **Métricas de Processo**: Por processo específico
- **Métricas de Rede**: Conexões, pacotes

<br />

## X-Ray Integration

### Distributed Tracing

- **Traces**: Caminho completo de requisições
- **Segments**: Componentes individuais
- **Annotations**: Dados indexáveis
- **Metadata**: Dados adicionais não indexáveis

### Service Map

Visualização de arquitetura de serviços e dependências.

<br />

## Billing e Custos

### Métricas de Billing

- **EstimatedCharges**: Custos estimados
- **Por Serviço**: Detalhamento por serviço AWS
- **Por Região**: Custos regionais

### Alertas de Billing

Configuração de alarmes para controle de custos.

<br />

## Perguntas e Respostas

### 1. Qual é a resolução padrão das métricas personalizadas no CloudWatch?

**Resposta**: 1 minuto

**Explicação**: Por padrão, as métricas personalizadas no CloudWatch têm resolução de 1 minuto. Você pode optar por alta resolução (1 segundo) pagando um custo adicional.

### 2. Quantas dimensões uma métrica pode ter no CloudWatch?

**Resposta**: Máximo de 10 dimensões

**Explicação**: Cada métrica no CloudWatch pode ter até 10 dimensões, que são pares chave-valor que identificam unicamente uma métrica.

### 3. Qual é o período máximo que dados de timestamp podem estar no passado para métricas personalizadas?

**Resposta**: 2 semanas (14 dias)

**Explicação**: O CloudWatch aceita timestamps de até 2 semanas no passado e até 2 horas no futuro. Timestamps fora desse intervalo são rejeitados.

### 4. Quais são os três estados possíveis de um alarme do CloudWatch?

**Resposta**: OK, ALARM, INSUFFICIENT_DATA

**Explicação**: 
- **OK**: A métrica está dentro do limite definido
- **ALARM**: A métrica violou o limite definido
- **INSUFFICIENT_DATA**: Não há dados suficientes para avaliar o alarme

### 5. Como você pode exportar logs do CloudWatch para análise de longo prazo?

**Resposta**: Exportação para S3 ou streaming para Kinesis

**Explicação**: Logs podem ser exportados para S3 para arquivamento ou enviados para Kinesis Data Streams/Firehose para processamento em tempo real.

### 6. Qual é a diferença entre CloudWatch Events e EventBridge?

**Resposta**: EventBridge é a evolução do CloudWatch Events com recursos adicionais

**Explicação**: EventBridge inclui todos os recursos do CloudWatch Events, além de event buses customizados, schema registry e integração com parceiros SaaS.

### 7. Qual permissão IAM é necessária para o CloudWatch Agent em instâncias EC2?

**Resposta**: CloudWatchAgentServerPolicy

**Explicação**: Esta política gerenciada pela AWS fornece as permissões necessárias para o CloudWatch Agent coletar métricas e logs de instâncias EC2.

### 8. Por quanto tempo as métricas de 1 minuto são retidas no CloudWatch?

**Resposta**: 15 dias

**Explicação**: As métricas têm diferentes períodos de retenção:
- 1 minuto: 15 dias
- 5 minutos: 63 dias
- 1 hora: 455 dias

### 9. Qual é o comando AWS CLI para definir manualmente o estado de um alarme?

**Resposta**: `aws cloudwatch set-alarm-state`

**Explicação**: Este comando permite definir manualmente o estado de um alarme para testes, usando parâmetros como --alarm-name, --state-value e --state-reason.

### 10. Qual recurso do CloudWatch permite monitorar aplicações através de scripts automatizados?

**Resposta**: CloudWatch Synthetics (Canaries)

**Explicação**: Os Canaries são scripts que executam continuamente para monitorar endpoints, APIs e funcionalidades de aplicações, simulando ações do usuário.

### 11. Como você pode filtrar logs no CloudWatch Logs Insights?

**Resposta**: Usando a palavra-chave `filter` em consultas

**Explicação**: Exemplo: `fields @timestamp, @message | filter @message like /ERROR/` para filtrar logs contendo "ERROR".

### 12. Qual é a diferença entre métricas básicas e detalhadas no EC2?

**Resposta**: Métricas básicas são de 5 minutos (gratuitas), detalhadas são de 1 minuto (pagas)

**Explicação**: Métricas básicas são coletadas automaticamente a cada 5 minutos sem custo adicional. Métricas detalhadas oferecem granularidade de 1 minuto mediante pagamento.

### 13. Qual recurso permite visualizar a arquitetura de serviços e suas dependências?

**Resposta**: X-Ray Service Map

**Explicação**: O Service Map do X-Ray fornece uma visualização da arquitetura da aplicação, mostrando como os serviços se comunicam e suas dependências.

### 14. Como você pode configurar alertas de billing no CloudWatch?

**Resposta**: Criando alarmes baseados na métrica EstimatedCharges

**Explicação**: Você pode criar alarmes para a métrica EstimatedCharges no namespace AWS/Billing para receber notificações quando os custos excedem um limite.

### 15. Qual é a linguagem de consulta usada no CloudWatch Logs Insights?

**Resposta**: Uma linguagem de consulta similar ao SQL específica do CloudWatch

**Explicação**: O CloudWatch Logs Insights usa uma linguagem de consulta própria com comandos como `fields`, `filter`, `stats`, `sort` para analisar logs.

<br />

## 📚 Referências

- [O que é o Amazon CloudWatch?](https://docs.aws.amazon.com/pt_br/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
- [Conceitos do Amazon CloudWatch](https://docs.aws.amazon.com/pt_br/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html)
- [CloudWatch Logs User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/)
- [CloudWatch Events User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/)
- [CloudWatch Synthetics User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Synthetics_Canaries.html)

<br />

---
