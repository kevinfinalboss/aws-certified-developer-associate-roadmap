# Amazon CodeBuild

## 📌 Índice

- [Introdução](#introdução)
- [Buildspec.yml](#buildspacyml)
  - [Estrutura do buildspec.yml](#estrutura-do-buildspecyml)
  - [Fases de Build](#fases-de-build)
  - [Configurações Avançadas](#configurações-avançadas)
- [Ambientes de Build](#ambientes-de-build)
- [Cache](#cache)
- [Artefatos](#artefatos)
- [Variáveis de Ambiente](#variáveis-de-ambiente)
- [Logs e Monitoramento](#logs-e-monitoramento)
- [Integração com Outros Serviços](#integração-com-outros-serviços)
- [VPC Configuration](#vpc-configuration)
- [Build Batches](#build-batches)
- [Triggers e Webhooks](#triggers-e-webhooks)
- [Segurança](#segurança)
- [Troubleshooting](#troubleshooting)
- [Exemplos de Questões de Prova](#exemplos-de-questões-de-prova)
- [📚 Referências](#📚-referências)

---

## Introdução

Amazon CodeBuild é um serviço de build na nuvem totalmente gerenciado que compila código, executa testes e produz artefatos de software prontos para implantação. O CodeBuild elimina a necessidade de provisionar, gerenciar e dimensionar seus próprios servidores de build.

### Principais características:

- **Totalmente gerenciado**: sem necessidade de gerenciar infraestrutura
- **Escalável**: dimensiona automaticamente para processar múltiplos builds
- **Pay-per-use**: pague apenas pelo tempo de build utilizado
- **Seguro**: builds executados em ambientes isolados
- **Integrado**: funciona com CodeCommit, GitHub, Bitbucket, S3

---

## Buildspec.yml

### O que é?

O arquivo `buildspec.yml` contém as instruções de build que o CodeBuild executa. Deve estar localizado no **diretório raiz** do código fonte.

### Estrutura do buildspec.yml

```yaml
version: 0.2

run-as: root

env:
  variables:
    key: value
  parameter-store:
    key: value
  secrets-manager:
    key: value

phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - echo "Install phase"
  pre_build:
    commands:
      - echo "Pre-build phase"
  build:
    commands:
      - echo "Build phase"
  post_build:
    commands:
      - echo "Post-build phase"

artifacts:
  files:
    - '**/*'
  base-directory: build

cache:
  paths:
    - node_modules/**/*
```

### Fases de Build

| Fase | Descrição | Casos de Uso |
|---|---|---|
| **install** | Instalar dependências e runtime | Instalar Node.js, Python, packages |
| **pre_build** | Comandos antes do build | Login em registries, preparar ambiente |
| **build** | Compilação principal | Compilar código, executar testes |
| **post_build** | Comandos após build | Build de imagens Docker, upload artefatos |

### Configurações Avançadas

#### Runtime Versions

```yaml
phases:
  install:
    runtime-versions:
      python: 3.9
      nodejs: 18
      java: corretto17
      docker: 20
```

#### Commands com tratamento de erro

```yaml
phases:
  build:
    commands:
      - echo "Executando testes"
      - npm test || exit 1
    finally:
      - echo "Limpeza sempre executada"
```

#### Buildspec alternativo

```yaml
version: 0.2
phases:
  build:
    commands:
      - aws codebuild start-build --project-name my-other-project
```

---

## Ambientes de Build

### Tipos de Ambiente

| Tipo | Descrição | Quando Usar |
|---|---|---|
| **Managed Image** | Imagens pré-configuradas pela AWS | Casos comuns, linguagens populares |
| **Custom Image** | Sua própria imagem Docker | Dependências específicas, ferramentas customizadas |

### Managed Images Populares

- **Amazon Linux 2**: `aws/codebuild/amazonlinux2-x86_64-standard:4.0`
- **Ubuntu**: `aws/codebuild/standard:6.0`
- **Windows**: `aws/codebuild/windows-base:2019-1.0`

### Especificações de Computação

| Tipo | vCPU | Memória | Disco | Quando Usar |
|---|---|---|---|---|
| **BUILD_GENERAL1_SMALL** | 1 | 3 GB | 64 GB | Builds leves, testes simples |
| **BUILD_GENERAL1_MEDIUM** | 2 | 7 GB | 128 GB | Builds médios, aplicações normais |
| **BUILD_GENERAL1_LARGE** | 4 | 15 GB | 128 GB | Builds pesados, compilação complexa |
| **BUILD_GENERAL1_2XLARGE** | 8 | 29 GB | 128 GB | Builds muito pesados, paralelização |

---

## Cache

### Tipos de Cache

| Tipo | Descrição | Casos de Uso |
|---|---|---|
| **S3 Cache** | Armazena cache no S3 | Cache de dependências, artefatos |
| **Local Cache** | Cache no próprio ambiente | Source cache, Docker layers |

### Configuração de Cache

```yaml
cache:
  paths:
    - '/root/.npm/**/*'
    - '/root/.cache/pip/**/*'
    - 'node_modules/**/*'
    - '.gradle/caches/**/*'
```

### Local Cache Types

```yaml
cache:
  type: LOCAL
  modes:
    - LOCAL_DOCKER_LAYER_CACHE
    - LOCAL_SOURCE_CACHE
    - LOCAL_CUSTOM_CACHE
```

### Benefícios do Cache

- Reduz tempo de build significativamente
- Diminui custos evitando downloads repetidos
- Melhora performance de builds subsequentes
- Especialmente útil para dependências grandes

---

## Artefatos

### Configuração de Artefatos

```yaml
artifacts:
  files:
    - '**/*'
  name: MyAppBuild
  base-directory: dist
  exclude-paths:
    - node_modules/**/*
    - '**/*.tmp'
```

### Tipos de Output

- **S3**: armazenar artefatos no S3
- **CodePipeline**: passar para próximo estágio
- **Multiple artifacts**: diferentes saídas por build

### Artefatos Secundários

```yaml
artifacts:
  secondary-artifacts:
    artifact1:
      files:
        - '**/*'
      base-directory: build1
    artifact2:
      files:
        - '**/*'
      base-directory: build2
```

---

## Variáveis de Ambiente

### Tipos de Variáveis

| Tipo | Descrição | Configuração |
|---|---|---|
| **Plaintext** | Variáveis simples em texto | No console ou buildspec |
| **Parameter Store** | SSM Parameter Store | Referência ao parâmetro |
| **Secrets Manager** | AWS Secrets Manager | Referência ao secret |

### Configuração no buildspec.yml

```yaml
env:
  variables:
    NODE_ENV: production
    API_URL: https://api.example.com
  parameter-store:
    DATABASE_URL: /myapp/database/url
    API_KEY: /myapp/api/key
  secrets-manager:
    DB_PASSWORD: prod/myapp/db:password
    JWT_SECRET: prod/myapp/jwt:secret
```

### Variáveis Automáticas

O CodeBuild fornece variáveis automáticas:

- `AWS_DEFAULT_REGION`
- `AWS_ACCOUNT_ID`
- `CODEBUILD_BUILD_ID`
- `CODEBUILD_BUILD_NUMBER`
- `CODEBUILD_PROJECT_NAME`
- `CODEBUILD_SOURCE_VERSION`

---

## Logs e Monitoramento

### CloudWatch Logs

- Logs detalhados de cada fase do build
- Configurável por projeto
- Pode ser enviado para S3 para arquivamento
- Stream em tempo real disponível

### Métricas CloudWatch

| Métrica | Descrição |
|---|---|
| **Builds** | Número total de builds |
| **Duration** | Duração dos builds |
| **SucceededBuilds** | Builds bem-sucedidos |
| **FailedBuilds** | Builds que falharam |

### Build Status

- **SUCCEEDED**: build completado com sucesso
- **FAILED**: build falhou
- **FAULT**: erro interno do CodeBuild
- **STOPPED**: build cancelado
- **TIMED_OUT**: build excedeu timeout

---

## Integração com Outros Serviços

### CodeCommit

- Trigger automático em commits
- Suporte a branches específicos
- Webhook integration

### GitHub

- OAuth ou Personal Access Token
- Webhook para builds automáticos
- Suporte a Pull Requests

### CodePipeline

- Build stage no pipeline
- Passagem automática de artefatos
- Integração com deploy stages

### ECR (Elastic Container Registry)

- Build e push de imagens Docker
- Autenticação automática
- Versionamento de imagens

```yaml
phases:
  pre_build:
    commands:
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
  build:
    commands:
      - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
  post_build:
    commands:
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
```

---

## VPC Configuration

### Quando usar VPC?

- Acessar recursos privados (RDS, ElastiCache)
- Compliance requirements
- Isolamento adicional de rede

### Configuração

- Especificar VPC ID
- Definir subnets (privadas recomendadas)
- Configurar security groups
- Garantir NAT Gateway para acesso à internet

### Considerações

- Builds podem ser mais lentos devido à configuração de rede
- Custos adicionais de NAT Gateway
- Necessário configurar corretamente security groups

---

## Build Batches

### O que são?

Permite executar múltiplos builds relacionados como um único batch, útil para:

- Builds paralelos de diferentes componentes
- Matrix builds (diferentes versões/ambientes)
- Builds com dependências entre si

### Tipos de Batch

- **BUILD_GRAPH**: builds com dependências
- **BUILD_LIST**: builds paralelos independentes
- **BUILD_MATRIX**: combinações de variáveis

### Exemplo de Build Matrix

```yaml
batch:
  build-matrix:
    static:
      variables:
        NODE_VERSION: ["14", "16", "18"]
        ENVIRONMENT: ["test", "staging"]
```

---

## Triggers e Webhooks

### Tipos de Triggers

| Trigger | Descrição | Quando Usar |
|---|---|---|
| **Webhook** | HTTP callback automático | GitHub, Bitbucket pushes |
| **CloudWatch Events** | Eventos baseados em schedule | Builds periódicos |
| **CodePipeline** | Triggered por pipeline | Integração CI/CD |

### Configuração de Webhook

- Suporte a filtros por branch
- Filtros por tipo de evento (push, pull request)
- Configuração de secret para segurança

### Event Filters

```json
{
  "type": "EVENT",
  "pattern": "PUSH",
  "excludeMatchedPattern": false
}
```

---

## Segurança

### IAM Roles

#### Service Role

Permissões que o CodeBuild precisa:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Permissions Boundaries

- Limite máximo de permissões
- Controle adicional de segurança
- Compliance requirements

### Encryption

- **Artifacts**: encriptados no S3
- **Environment variables**: encriptação para secrets
- **Cache**: encriptado por padrão

### Network Security

- VPC isolation quando necessário
- Security groups para controle de tráfego
- Private subnets para recursos sensíveis

---

## Troubleshooting

### Problemas Comuns

| Problema | Possível Causa | Solução |
|---|---|---|
| **BUILD_FAILED** | Erro no código ou teste | Verificar logs detalhados |
| **PROVISIONING_FAULT** | Problema de recursos | Tentar novamente, verificar limites |
| **CLIENT_ERROR** | Configuração incorreta | Verificar buildspec e permissões |
| **TIMED_OUT** | Build muito longo | Otimizar build ou aumentar timeout |

### Debugging Tips

1. **Verificar logs detalhados** no CloudWatch
2. **Testar buildspec localmente** usando CodeBuild Local
3. **Usar echo statements** para debug
4. **Verificar permissões IAM**
5. **Validar variáveis de ambiente**

### Performance Optimization

- Use cache apropriadamente
- Paralelizar builds quando possível
- Escolher compute type adequado
- Otimizar imagens Docker
- Minimizar transferência de dados

---

## Exemplos de Questões de Prova

### Questão 1: Buildspec Location
**Onde deve estar localizado o arquivo buildspec.yml?**

✅ **Resposta:** No diretório raiz (root directory) do código fonte.

---

### Questão 2: Build Failure Investigation
**Build está falhando constantemente. Onde investigar primeiro?**

✅ **Resposta:** CloudWatch Logs do projeto CodeBuild - contém logs detalhados de cada fase.

---

### Questão 3: Environment Variables Security
**Como passar senhas de banco seguramente para o build?**

✅ **Resposta:** Usar AWS Secrets Manager ou SSM Parameter Store (SecureString) e referenciar no buildspec.

---

### Questão 4: Cache Configuration
**Build demora muito baixando dependências. Como otimizar?**

✅ **Resposta:** Configurar cache no buildspec.yml para armazenar dependências (node_modules, .gradle, etc.).

---

### Questão 5: Docker Image Build
**Precisa fazer build de imagem Docker e enviar para ECR. Qual fase usar?**

✅ **Resposta:** pre_build para login ECR, build para docker build, post_build para docker push.

---

### Questão 6: VPC Access
**Build precisa acessar RDS privado. Como configurar?**

✅ **Resposta:** Configurar CodeBuild para executar dentro da VPC com subnets privadas e security groups apropriados.

---

### Questão 7: Parallel Builds
**Precisa buildar múltiplos componentes simultaneamente. Qual recurso usar?**

✅ **Resposta:** Build Batches com BUILD_LIST ou BUILD_MATRIX para builds paralelos.

---

### Questão 8: Build Timeout
**Build retorna TIMED_OUT após 1 hora. Qual o problema?**

✅ **Resposta:** Timeout padrão é 60 minutos. Aumentar timeout do projeto ou otimizar o build.

---

### Questão 9: Artifact Storage
**Onde os artefatos de build são armazenados por padrão?**

✅ **Resposta:** Amazon S3 - especificado na configuração de artifacts do projeto.

---

### Questão 10: Environment Selection
**Build simples de Node.js. Qual environment escolher?**

✅ **Resposta:** Managed image (aws/codebuild/standard) com BUILD_GENERAL1_SMALL para economia.

---

### Questão 11: GitHub Integration
**Como configurar build automático no push para GitHub?**

✅ **Resposta:** Configurar webhook no CodeBuild com GitHub OAuth e filtros por branch/evento.

---

### Questão 12: Build Variables
**Como acessar informações do build atual no buildspec?**

✅ **Resposta:** Usar variáveis automáticas como $CODEBUILD_BUILD_ID, $CODEBUILD_PROJECT_NAME.

---

### Questão 13: Custom Dependencies
**Build precisa de ferramentas específicas não disponíveis na managed image.**

✅ **Resposta:** Usar custom Docker image com as ferramentas pré-instaladas.

---

### Questão 14: Multi-Stage Build
**Diferentes artefatos para diferentes ambientes no mesmo build.**

✅ **Resposta:** Usar secondary-artifacts no buildspec para múltiplas saídas.

---

### Questão 15: Build Permissions
**CodeBuild falha ao acessar S3. Qual verificar primeiro?**

✅ **Resposta:** IAM Service Role do CodeBuild - deve ter permissões s3:GetObject e s3:PutObject.

---

## 📚 Referências

- [AWS CodeBuild User Guide](https://docs.aws.amazon.com/pt_br/codebuild/latest/userguide/welcome.html)
- [Buildspec File Reference](https://docs.aws.amazon.com/pt_br/codebuild/latest/userguide/build-spec-ref.html)
- [Docker Images for CodeBuild](https://docs.aws.amazon.com/pt_br/codebuild/latest/userguide/build-env-ref-available.html)
- [Environment Variables](https://docs.aws.amazon.com/pt_br/codebuild/latest/userguide/build-env-ref-env-vars.html)
- [VPC Support](https://docs.aws.amazon.com/pt_br/codebuild/latest/userguide/vpc-support.html)
- [Build Batches](https://docs.aws.amazon.com/pt_br/codebuild/latest/userguide/batch-build.html)
- [Troubleshooting](https://docs.aws.amazon.com/pt_br/codebuild/latest/userguide/troubleshooting.html)
