<p align="center">
	<img src="https://cdn.worldvectorlogo.com/logos/aws-elastic-beanstalk-1.svg" alt="aws-elastic-beanstalk-icon" style="height:150px; width:150px;" /> 
  <br />
	<h1 align="center">
    Elastic Beanstalk - Guia Completo AWS Developer
  </h1>
</p>	
<br />

## :pushpin: Índice
- [Introdução](#introdução)
- [Conceitos Fundamentais](#conceitos-fundamentais)
- [Plataformas Suportadas](#plataformas-suportadas)
- [Arquitetura do Beanstalk](#arquitetura-do-beanstalk)
- [Políticas de Implantação](#políticas-de-implantação)
- [Configuração de Ambiente](#configuração-de-ambiente)
- [Política de Ciclo de Vida](#política-de-ciclo-de-vida)
- [Monitoramento e Logs](#monitoramento-e-logs)
- [Configurações Avançadas](#configurações-avançadas)
- [Integração com Outros Serviços](#integração-com-outros-serviços)
- [Extensões (.ebextensions)](#extensões-ebextensions)
- [Blue/Green Deployments](#bluegreen-deployments)
- [Troubleshooting](#troubleshooting)
- [Perguntas e Respostas](#perguntas-e-respostas)
- [Referências](#books-referências)

<br />

## Introdução
AWS Elastic Beanstalk é um serviço que gerencia e realiza o deployment de aplicações web. É possível implantar e gerenciar aplicações sem necessitar conhecer a infraestrutura que executa esses aplicativos.

**Principais benefícios:**
- **Rápido para começar** - Upload do código e deploy automático
- **Sem custo adicional** - Paga apenas pelos recursos AWS utilizados
- **Controle completo** - Acesso total aos recursos subjacentes
- **Versioning** - Controle de versões de aplicação
- **Monitoramento integrado** - Health dashboard e alertas

**Não é recomendado para:**
- Aplicações que requerem controle muito específico da infraestrutura
- Arquiteturas complexas de microserviços
- Aplicações que precisam de customizações muito avançadas

<br />

## Conceitos Fundamentais

### Application
- **Container lógico** para componentes do Beanstalk
- **Coleção de versões** de aplicação
- **Política de ciclo de vida** para gerenciar versões

### Application Version
- **Específica versão** do código da aplicação
- **Deployment package** (WAR, ZIP, JAR)
- **Imutável** após criação

### Environment
- **Instância rodando** de uma versão da aplicação
- **Web Server Environment** - Atende requisições HTTP
- **Worker Environment** - Processa tarefas em background

### Environment Tier
- **Web Server Tier:** ELB + Auto Scaling + EC2
- **Worker Tier:** SQS + Auto Scaling + EC2

### Platform
- **Combinação** de OS, runtime, web server, application server
- **Managed platform updates** automatizadas

<br />

## Plataformas Suportadas

### Linguagens e Frameworks
- **Java** - Tomcat, Java SE
- **.NET** - Windows Server com IIS  
- **Node.js** - Nginx proxy reverso
- **PHP** - Apache ou Nginx
- **Python** - Apache ou Nginx
- **Ruby** - Passenger ou Puma
- **Go** - Nginx proxy reverso

### Containers
- **Docker** - Single container ou multi-container
- **Preconfigured Docker** - Glassfish, Python, etc.

### Platform Versions
- **Major versions** - Mudanças significativas
- **Minor versions** - Bug fixes e security patches
- **Patch versions** - Updates menores

<br />

## Arquitetura do Beanstalk

### Web Server Environment
```
Internet → ELB → Auto Scaling Group → EC2 Instances
                     ↓
                  CloudWatch
                     ↓
                  RDS (opcional)
```

### Worker Environment  
```
SQS Queue → Auto Scaling Group → EC2 Instances
                   ↓
               CloudWatch
                   ↓
               RDS (opcional)
```

### Recursos Criados Automaticamente
- **EC2 instances**
- **Auto Scaling Group**
- **Elastic Load Balancer**
- **Security Groups**
- **CloudWatch Alarms**
- **S3 Bucket** (logs e versões)

<br />

## Políticas de Implantação

### All at Once (Tudo de uma vez)
**Características:**
- **Mais rápido** método de deploy
- **Downtime** durante o deployment
- **Todas as instâncias** atualizadas simultaneamente
- **Rollback rápido** se necessário

**Quando usar:**
- Ambientes de desenvolvimento/teste
- Aplicações que podem ter downtime
- Deploys urgentes

**Processo:**
1. Para aplicação em todas as instâncias
2. Deploy nova versão em todas
3. Reinicia aplicação

### Rolling (Contínua)
**Características:**
- **Evita downtime** mas reduz capacidade
- **Deploy em lotes** (batch size configurável)
- **Tempo de deploy** mais longo
- **Duas versões** rodando simultaneamente

**Quando usar:**
- Aplicações que não podem ter downtime
- Ambientes de produção
- Quando capacidade reduzida é aceitável

**Processo:**
1. Remove lote de instâncias do load balancer
2. Deploy nova versão no lote
3. Adiciona lote de volta ao load balancer
4. Repete até todas as instâncias

### Rolling with Additional Batch (Contínua com lote adicional)
**Características:**
- **Mantém capacidade total** durante deploy
- **Cria instâncias adicionais** temporariamente
- **Custo adicional** durante deployment
- **Mais lento** que Rolling

**Quando usar:**
- Produção que não pode reduzir capacidade
- Aplicações críticas
- Quando custo adicional temporário é aceitável

**Processo:**
1. Cria lote adicional de instâncias
2. Deploy nova versão no lote adicional
3. Move tráfego para novas instâncias
4. Termina instâncias antigas
5. Repete processo

### Immutable (Imutável)
**Características:**
- **Zero downtime** e capacidade mantida
- **Novas instâncias** para nova versão
- **Rollback mais rápido** (troca DNS)
- **Maior custo** durante deployment

**Quando usar:**
- Aplicações críticas de produção
- Quando rollback rápido é essencial
- Ambientes que requerem alta disponibilidade

**Processo:**
1. Cria novo Auto Scaling Group
2. Deploy nova versão em novas instâncias
3. Gradualmente move tráfego
4. Termina ASG antigo

### Traffic Splitting (Divisão de tráfego)
**Características:**
- **Canary deployments**
- **Percentual de tráfego** para nova versão
- **Monitoring automático** de health
- **Rollback automático** se problemas

**Quando usar:**
- Validação de nova versão em produção
- A/B testing
- Deploys com baixo risco

**Processo:**
1. Deploy nova versão em novas instâncias
2. Direciona X% do tráfego para nova versão
3. Monitora métricas de health
4. Rollback automático ou promoção completa

<br />

## Configuração de Ambiente

### Environment Types
**Single Instance:**
- **Uma EC2 instance**
- **Elastic IP**
- **Sem load balancer**
- **Desenvolvimento/teste**

**Load Balanced:**
- **Multiple instances**
- **Auto Scaling**
- **Elastic Load Balancer**
- **Produção**

### Configuration Categories
- **Software** - Environment variables, log files
- **Instances** - Instance type, key pair
- **Capacity** - Auto Scaling settings
- **Load Balancer** - Health checks, listeners
- **Rolling Updates** - Deployment policies
- **Security** - Service role, instance profile
- **Monitoring** - Health reporting, alarms
- **Network** - VPC, subnets, security groups

### Environment Variables
```bash
# Configuração via console ou CLI
export RDS_HOSTNAME=mydb.region.rds.amazonaws.com
export RDS_PORT=3306
export RDS_DB_NAME=mydb
export RDS_USERNAME=admin
```

<br />

## Política de Ciclo de Vida
Elastic Beanstalk **pode armazenar no máximo 1.000 versões de aplicações**, portanto se você não remover as versões antigas não será possível fazer mais deploys. Para gerenciar isso, utiliza-se uma política de ciclo de vida.

### Tipos de Políticas
**Baseada em Tempo:**
- **Máximo de dias** para manter versões
- **Versões mais antigas** são removidas automaticamente
- **Exemplo:** Manter apenas versões dos últimos 30 dias

**Baseada em Contagem:**
- **Número máximo** de versões a manter
- **Versões mais antigas** removidas quando limite atingido
- **Exemplo:** Manter apenas as 50 versões mais recentes

### Configuração via Console
1. **Application** → **Application versions**
2. **Settings** → **Lifecycle**
3. **Configure policies**
4. **Apply** automaticamente

### Versionamento Source Bundle
- **S3 deletion** - Remove do S3 também
- **Retention** - Mantém no S3 mas remove do Beanstalk
- **Source bundle** pode ser reutilizado

<br />

## Monitoramento e Logs

### Health Monitoring
**Health Colors:**
- **Green** - Aplicação passa em todos os health checks
- **Yellow** - Aplicação falha em 1+ health checks recentemente
- **Red** - Aplicação falha em 3+ health checks recentemente  
- **Grey** - Informações insuficientes

### Enhanced Health Reporting
- **Detailed health information**
- **Root cause analysis**
- **Performance metrics**
- **Custom health checks**

### Log Files
**Tipos de logs:**
- **Application logs** - Logs da aplicação
- **Web server logs** - Apache/Nginx logs
- **Platform logs** - Beanstalk platform logs

**Coleta de logs:**
- **Request logs** - Via console ou CLI
- **Bundle logs** - Download completo
- **Tail logs** - Stream em tempo real

### CloudWatch Integration
- **Métricas básicas** automáticas
- **Custom metrics** via aplicação
- **Alarms** para notificações
- **Log streaming** para CloudWatch Logs

<br />

## Configurações Avançadas

### Configuration Files (.ebextensions)
Arquivos YAML/JSON no diretório `.ebextensions` para customizar ambiente.

```yaml
option_settings:
  aws:elasticbeanstalk:application:environment:
    DATABASE_URL: "mysql://username:password@host:port/database"
  aws:autoscaling:launchconfiguration:
    InstanceType: t3.medium
```

### Custom AMI
- **Pre-baked applications**
- **Faster deployment**
- **Consistent environment**
- **Custom software installation**

### Docker Support
**Single Container:**
```json
{
  "AWSEBDockerrunVersion": "1",
  "Image": {
    "Name": "my-app:latest",
    "Update": "true"
  },
  "Ports": [
    {
      "ContainerPort": "3000"
    }
  ]
}
```

**Multi-Container:**
```json
{
  "AWSEBDockerrunVersion": "2",
  "volumes": [],
  "containerDefinitions": [
    {
      "name": "web",
      "image": "my-web-app",
      "portMappings": [
        {
          "hostPort": 80,
          "containerPort": 3000
        }
      ]
    }
  ]
}
```

<br />

## Integração com Outros Serviços

### Database Integration
**RDS Integration:**
- **Managed database** instance
- **Environment variables** automáticas
- **Security groups** configurados
- **Backup e restore** integrados

**External Database:**
- **Connection strings** via environment variables
- **Security groups** manuais
- **Independente** do lifecycle do ambiente

### S3 Integration
- **Application versions** armazenadas
- **Log files** archivados
- **Static content** servido
- **Permissions** via IAM roles

### SQS Integration (Worker Tier)
- **Queue processing** automático
- **Dead letter queues**
- **Scaling based on queue depth**
- **Message handling** via HTTP POST

<br />

## Blue/Green Deployments

### Swap Environment URLs
**Processo:**
1. **Criar novo ambiente** (Green)
2. **Deploy nova versão** no Green
3. **Testar** ambiente Green
4. **Swap URLs** instantaneamente
5. **Monitorar** e fazer rollback se necessário

**Vantagens:**
- **Zero downtime**
- **Rollback instantâneo**
- **Teste completo** antes da troca
- **DNS TTL** pode causar delay

### Clone Environment
- **Copia configuração** exata
- **Mantém** todas as configurações
- **Ideal** para Blue/Green

<br />

## Troubleshooting

### Common Issues
**Deployment Failures:**
- **Insufficient permissions**
- **Invalid configuration**
- **Health check failures**
- **Resource limits**

**Environment Issues:**
- **Auto Scaling problems**
- **Load balancer misconfiguration**
- **Security group issues**
- **RDS connectivity**

### Debug Tools
**Log Analysis:**
- **eb logs** command
- **CloudWatch Logs**
- **S3 log bundles**

**Health Dashboard:**
- **Detailed health status**
- **Recent events**
- **Configuration changes**

**CLI Commands:**
```bash
eb logs
eb health
eb status  
eb config
eb deploy
```

<br />

## Perguntas e Respostas

### Pergunta 1
**Você precisa fazer um deployment de uma nova versão crítica em produção sem downtime e com capacidade para rollback rápido. Qual política de deployment é mais apropriada?**

**Resposta:**
A política **Immutable** é a mais apropriada:

**Vantagens para esse cenário:**
- **Zero downtime** - Novas instâncias servem o tráfego
- **Rollback rápido** - Apenas troca o DNS de volta
- **Capacidade mantida** - Dobra temporariamente as instâncias
- **Isolamento completo** - Nova versão em ambiente separado

**Processo:**
1. Cria novo Auto Scaling Group com nova versão
2. Gradualmente move tráfego do ELB
3. Se problemas, rollback instantâneo via DNS
4. Termina ASG antigo após confirmação

**Alternativa:** Traffic Splitting para validação gradual

### Pergunta 2
**Sua aplicação Beanstalk precisa acessar um banco RDS e um bucket S3. Como configurar as permissões de forma segura?**

**Resposta:**
Use **IAM Instance Profile** com políticas específicas:

**Configuração:**
1. **Criar IAM Role** para EC2 instances
2. **Attach policies** para RDS e S3
3. **Configurar no Beanstalk** via Security settings

**Exemplo de política:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "rds:DescribeDBInstances",
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:rds:region:account:db:mydb",
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```

**Environment Variables:**
```bash
RDS_HOSTNAME=mydb.region.rds.amazonaws.com
S3_BUCKET=my-app-bucket
```

### Pergunta 3
**Como implementar configurações específicas de ambiente (dev, staging, prod) no Beanstalk?**

**Resposta:**
Use **Environment Variables** e **Configuration Files**:

**Via Environment Variables:**
```bash
# Development
NODE_ENV=development
DATABASE_URL=dev-db-url

# Production  
NODE_ENV=production
DATABASE_URL=prod-db-url
```

**Via .ebextensions:**
```yaml
# .ebextensions/01-environment.config
option_settings:
  aws:elasticbeanstalk:application:environment:
    APP_ENV: production
    DEBUG: false
    LOG_LEVEL: info
```

**Conditional Configuration:**
```yaml
# .ebextensions/02-conditional.config
option_settings:
  - namespace: aws:elasticbeanstalk:application:environment
    option_name: DATABASE_POOL_SIZE
    value: 
      Fn::If:
        - IsProduction
        - 20
        - 5
```

### Pergunta 4
**Sua aplicação Worker Tier não está processando mensagens do SQS. Quais são as possíveis causas?**

**Resposta:**
**Possíveis causas e soluções:**

**1. IAM Permissions:**
- **Problema:** Worker não tem permissão para SQS
- **Solução:** Adicionar políticas SQS ao instance profile

**2. Application Endpoint:**
- **Problema:** App não responde em `/` com POST
- **Solução:** Configurar endpoint correto via HTTP path

**3. Queue Configuration:**
- **Problema:** Queue não existe ou inacessível
- **Solução:** Verificar nome da queue e permissions

**4. Message Format:**
- **Problema:** App não processa formato da mensagem
- **Solução:** Implementar parser correto

**Debug steps:**
```bash
eb logs
# Verificar logs de worker daemon
# Validar endpoint health
# Testar permissões SQS
```

### Pergunta 5
**Como configurar auto scaling baseado em métricas customizadas no Beanstalk?**

**Resposta:**
**Configure Auto Scaling triggers:**

**Via Console:**
1. **Configuration** → **Capacity**
2. **Auto Scaling** → **Triggers**
3. **Add trigger** com métrica customizada

**Métricas disponíveis:**
- **CPU Utilization**
- **Network In/Out**
- **Disk Read/Write**
- **Request Count**
- **Latency**

**Exemplo de trigger:**
```yaml
# .ebextensions/autoscaling.config
option_settings:
  aws:autoscaling:trigger:
    MeasureName: CPUUtilization
    Unit: Percent
    UpperThreshold: 80
    LowerThreshold: 20
    ScaleUpIncrement: 2
    ScaleDownIncrement: -1
```

**Custom CloudWatch Metrics:**
```python
import boto3
cloudwatch = boto3.client('cloudwatch')

cloudwatch.put_metric_data(
    Namespace='MyApp',
    MetricData=[
        {
            'MetricName': 'QueueDepth',
            'Value': queue_depth,
            'Unit': 'Count'
        }
    ]
)
```

### Pergunta 6
**Explique a diferença entre deploying via Beanstalk vs ECS/EKS.**

**Resposta:**
**AWS Elastic Beanstalk:**
- **Platform-as-a-Service (PaaS)**
- **Deploy código** diretamente
- **Managed infrastructure**
- **Ideal para:** Aplicações web tradicionais
- **Limitações:** Menos controle, plataformas específicas

**Amazon ECS:**
- **Container orchestration**
- **Deploy containers** Docker
- **Mais controle** sobre infraestrutura
- **Ideal para:** Microserviços, aplicações containerizadas
- **Complexidade:** Maior curva de aprendizado

**Amazon EKS:**
- **Kubernetes managed**
- **Máximo controle e flexibilidade**
- **Ideal para:** Arquiteturas complexas, multi-cloud
- **Complexidade:** Requer conhecimento Kubernetes

**Quando usar Beanstalk:**
- Rapid prototyping
- Aplicações monolíticas
- Times sem expertise DevOps
- Foco no desenvolvimento, não infraestrutura

### Pergunta 7
**Como fazer backup e restore de um ambiente Beanstalk?**

**Resposta:**
**Backup de Environment:**

**1. Saved Configurations:**
```bash
eb config save production-config
# Salva toda configuração do ambiente
```

**2. Application Versions:**
- **Automático:** Versões salvas no S3
- **Manual:** Download source bundle

**3. Database Backup:**
- **RDS integrado:** Snapshots automáticos
- **RDS externo:** Gerenciar snapshots separadamente

**Restore Process:**
```bash
# Criar novo ambiente com configuração salva
eb create new-env --cfg production-config

# Deploy versão específica
eb deploy --version-label v1.2.3
```

**Disaster Recovery:**
1. **Clone environment** em região diferente
2. **Replicate RDS** com cross-region snapshots
3. **S3 replication** para application versions
4. **Route 53** para failover automático

### Pergunta 8
**Quais são as melhores práticas para CI/CD com Beanstalk?**

**Resposta:**
**Pipeline CI/CD recomendado:**

**1. Source Control:**
```bash
# Git workflow
feature-branch → dev → staging → production
```

**2. Build Stage:**
```yaml
# CodeBuild buildspec.yml
version: 0.2
phases:
  install:
    runtime-versions:
      nodejs: 14
  build:
    commands:
      - npm install
      - npm run build
      - npm test
artifacts:
  files:
    - '**/*'
  name: my-app-$(date +%Y-%m-%d)
```

**3. Deploy Stages:**
```bash
# Automated deployment
eb deploy dev-environment
# Manual approval
eb deploy staging-environment  
# Manual approval
eb deploy production-environment
```

**4. Environment Strategy:**
- **Dev:** All at once (rápido)
- **Staging:** Rolling (testa processo)
- **Production:** Immutable (seguro)

**5. Monitoring:**
- **CloudWatch alarms** em produção
- **Health checks** automáticos
- **Rollback** triggers automáticos

**Tools Integration:**
- **CodeCommit/GitHub** para source
- **CodeBuild** para build
- **CodePipeline** para orchestration
- **CodeDeploy** integração opcional

<br />

## :books: Referências
Para uma compreensão mais profunda sobre Elastic Beanstalk recomendo a leitura da documentação oficial:

- [O que é o AWS Elastic Beanstalk?](https://docs.aws.amazon.com/pt_br/elasticbeanstalk/latest/dg/Welcome.html)
- [Como escolher uma política de implantação](https://docs.aws.amazon.com/pt_br/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html#deployments-scenarios)
- [Definir as configurações de ciclo de vida da versão do aplicativo](https://docs.aws.amazon.com/pt_br/elasticbeanstalk/latest/dg/applications-lifecycle.html)
- [Elastic Beanstalk Developer Guide](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/)
- [EB CLI Command Reference](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3.html)
- [Configuration Files (.ebextensions)](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions.html)
- [Platform Versions](https://docs.aws.amazon.com/elasticbeanstalk/latest/platforms/)

<br />

---
