<p align="center">
	<img src="./img/aws-icons/aws-CodeArtifact.png" alt="aws-codeartifact-icon" style="height:150px; width:150px;" /> 
  <br />
	<h1 align="center">
    CodeArtifact - Guia Completo AWS Developer
  </h1>
</p>	

<br />

## :pushpin: Índice

- [Introdução](#introdução)
- [Conceitos Fundamentais](#conceitos-fundamentais)
- [Domínios e Repositórios](#domínios-e-repositórios)
- [Gerenciadores de Pacotes Suportados](#gerenciadores-de-pacotes-suportados)
- [Configuração e Autenticação](#configuração-e-autenticação)
- [Upstream Repositories](#upstream-repositories)
- [Controle de Acesso e Segurança](#controle-de-acesso-e-segurança)
- [Versionamento e Tags](#versionamento-e-tags)
- [Integração com CI/CD](#integração-com-cicd)
- [Monitoramento e Logs](#monitoramento-e-logs)
- [Custos e Limites](#custos-e-limites)
- [Casos de Uso](#casos-de-uso)
- [Troubleshooting](#troubleshooting)
- [Perguntas e Respostas](#perguntas-e-respostas)
- [Referências](#books-referências)

<br />

## Introdução

Amazon **CodeArtifact** é um serviço gerenciado de repositório de artefatos que facilita o armazenamento, publicação e compartilhamento de pacotes de software. Suporta gerenciadores de pacotes populares como Maven, Gradle, npm, yarn, pip, twine e NuGet.

**Benefícios principais:**
- **Segurança**: Integração com IAM e VPC
- **Escalabilidade**: Totalmente gerenciado pela AWS
- **Integração**: Funciona com ferramentas existentes
- **Controle**: Políticas granulares de acesso
- **Proxy**: Acesso a repositórios públicos através de upstream

<br />

## Conceitos Fundamentais

### Hierarquia do CodeArtifact

1. **Domínio**: Container de alto nível para repositórios
2. **Repositório**: Armazena pacotes de um tipo específico
3. **Pacote**: Unidade de software (ex: biblioteca, módulo)
4. **Versão do Pacote**: Versão específica de um pacote

### Formatos de Pacotes Suportados

- **Maven**: Projetos Java (JAR, WAR, POM)
- **npm**: Pacotes Node.js
- **PyPI**: Pacotes Python
- **NuGet**: Pacotes .NET
- **Generic**: Arquivos genéricos

<br />

## Domínios e Repositórios

### Domínios

Domínios fornecem uma unidade de gerenciamento para repositórios:
- Único por região e conta AWS
- Controle de acesso centralizado
- Política de recursos aplicada ao domínio

### Tipos de Repositórios

1. **Repositório Principal**: Armazena pacotes próprios
2. **Repositório Upstream**: Conecta a repositórios externos
3. **Repositório Virtual**: Agrega múltiplos repositórios

### Criando um Domínio

```bash
aws codeartifact create-domain \
  --domain my-company-domain \
  --encryption-key alias/aws/s3
```

### Criando um Repositório

```bash
aws codeartifact create-repository \
  --domain my-company-domain \
  --repository my-app-repo \
  --description "Repositório para aplicação principal"
```

<br />

## Gerenciadores de Pacotes Suportados

### Maven/Gradle (Java)

**Configuração do settings.xml:**
```xml
<settings>
  <servers>
    <server>
      <id>codeartifact</id>
      <username>aws</username>
      <password>${env.CODEARTIFACT_AUTH_TOKEN}</password>
    </server>
  </servers>
  <mirrors>
    <mirror>
      <id>codeartifact</id>
      <name>codeartifact</name>
      <url>https://my-domain-123456789012.d.codeartifact.us-east-1.amazonaws.com/maven/my-repo/</url>
      <mirrorOf>*</mirrorOf>
    </mirror>
  </mirrors>
</settings>
```

### npm (Node.js)

**Configuração:**
```bash
npm config set registry https://my-domain-123456789012.d.codeartifact.us-east-1.amazonaws.com/npm/my-repo/
npm config set //my-domain-123456789012.d.codeartifact.us-east-1.amazonaws.com/npm/my-repo/:_authToken=${CODEARTIFACT_AUTH_TOKEN}
```

### pip (Python)

**Configuração do pip.conf:**
```ini
[global]
index-url = https://aws:${CODEARTIFACT_AUTH_TOKEN}@my-domain-123456789012.d.codeartifact.us-east-1.amazonaws.com/pypi/my-repo/simple/
```

### NuGet (.NET)

**Configuração:**
```bash
dotnet nuget add source https://my-domain-123456789012.d.codeartifact.us-east-1.amazonaws.com/nuget/my-repo/v3/index.json \
  --name codeartifact \
  --username aws \
  --password ${CODEARTIFACT_AUTH_TOKEN}
```

<br />

## Configuração e Autenticação

### Obtendo Token de Autenticação

```bash
export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token \
  --domain my-company-domain \
  --domain-owner 123456789012 \
  --query authorizationToken \
  --output text`
```

### Login Automático

```bash
aws codeartifact login \
  --tool npm \
  --domain my-company-domain \
  --repository my-repo
```

### Duração do Token

- **Padrão**: 12 horas
- **Máximo**: 12 horas
- **Mínimo**: 15 minutos

<br />

## Upstream Repositories

### Conectando Repositórios Públicos

CodeArtifact pode funcionar como proxy para repositórios públicos:

- **npm**: registry.npmjs.org
- **PyPI**: pypi.org
- **Maven Central**: repo1.maven.org/maven2

### Configurando Upstream

```bash
aws codeartifact associate-external-connection \
  --domain my-company-domain \
  --repository my-repo \
  --external-connection public:npmjs
```

### Vantagens do Upstream

1. **Cache Local**: Pacotes são armazenados localmente
2. **Alta Disponibilidade**: Menos dependência de repositórios externos
3. **Controle**: Políticas de acesso unificadas
4. **Performance**: Menor latência para pacotes frequentes

<br />

## Controle de Acesso e Segurança

### Políticas IAM

**Política de Leitura:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "codeartifact:GetAuthorizationToken",
        "codeartifact:ReadFromRepository"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "sts:GetServiceBearerToken",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "sts:AWSServiceName": "codeartifact.amazonaws.com"
        }
      }
    }
  ]
}
```

### Políticas de Recurso

**Política de Domínio:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReadAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/CodeArtifactReadRole"
      },
      "Action": [
        "codeartifact:ReadFromRepository",
        "codeartifact:GetRepositoryEndpoint"
      ],
      "Resource": "*"
    }
  ]
}
```

### Integração com VPC

- **VPC Endpoints**: Acesso privado sem internet
- **Security Groups**: Controle de tráfego de rede
- **NACLs**: Camada adicional de segurança

<br />

## Versionamento e Tags

### Estratégias de Versionamento

1. **Semantic Versioning**: MAJOR.MINOR.PATCH
2. **Snapshot Versions**: Para desenvolvimento
3. **Release Versions**: Para produção

### Tags de Pacotes

```bash
aws codeartifact tag-resource \
  --resource-arn arn:aws:codeartifact:us-east-1:123456789012:package/my-domain/my-repo/npm/my-package \
  --tags Key=Environment,Value=Production
```

### Políticas de Retenção

Configure políticas para gerenciar automaticamente versões antigas:
- Manter apenas N versões mais recentes
- Remover versões após X dias
- Preservar versões com tags específicas

<br />

## Integração com CI/CD

### CodeBuild

**buildspec.yml:**
```yaml
version: 0.2
phases:
  pre_build:
    commands:
      - export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token --domain my-domain --domain-owner 123456789012 --query authorizationToken --output text`
      - aws codeartifact login --tool npm --domain my-domain --repository my-repo
  build:
    commands:
      - npm install
      - npm run build
  post_build:
    commands:
      - npm publish
```

### CodePipeline

Integration através de actions personalizadas e CodeBuild.

### Jenkins

```groovy
pipeline {
    agent any
    stages {
        stage('Setup') {
            steps {
                script {
                    env.CODEARTIFACT_AUTH_TOKEN = sh(
                        script: 'aws codeartifact get-authorization-token --domain my-domain --domain-owner 123456789012 --query authorizationToken --output text',
                        returnStdout: true
                    ).trim()
                }
            }
        }
    }
}
```

<br />

## Monitoramento e Logs

### CloudWatch Metrics

- **PackageDownloads**: Número de downloads
- **PackageUploads**: Número de uploads
- **StorageUsed**: Espaço utilizado
- **RequestCount**: Total de requisições

### CloudTrail

Monitora atividades da API:
- Criação/exclusão de domínios
- Modificações de políticas
- Operações de pacotes

### Eventos

- **PackageVersionPublished**: Quando nova versão é publicada
- **PackageVersionDeleted**: Quando versão é removida

<br />

## Custos e Limites

### Modelo de Preços

1. **Armazenamento**: Por GB/mês
2. **Requisições**: Por 10.000 requisições
3. **Transferência de Dados**: Saída para internet

### Limites de Serviço

- **Domínios**: 10 por conta
- **Repositórios**: 100 por domínio
- **Tamanho do Pacote**: 1 GB máximo
- **Upstream Connections**: 10 por repositório

### Otimização de Custos

1. **Políticas de Retenção**: Remove versões antigas
2. **Upstream Cache**: Reduz transferência de dados
3. **Compressão**: Para pacotes maiores

<br />

## Casos de Uso

### Desenvolvimento Enterprise

- **Bibliotecas Internas**: Compartilhamento entre equipes
- **Compliance**: Controle de dependências
- **Segurança**: Scan de vulnerabilidades

### CI/CD Pipeline

- **Build Reproducible**: Versões fixas de dependências
- **Rollback**: Capacidade de reverter versões
- **Staging**: Diferentes repositórios por ambiente

### Microserviços

- **Shared Libraries**: Código comum entre serviços
- **API Clients**: SDKs gerados automaticamente
- **Configuration**: Pacotes de configuração

<br />

## Troubleshooting

### Problemas Comuns

1. **Token Expirado**
   - Sintoma: 401 Unauthorized
   - Solução: Renovar token de autenticação

2. **Permissões Insuficientes**
   - Sintoma: 403 Forbidden
   - Solução: Verificar políticas IAM

3. **Configuração Incorreta**
   - Sintoma: Pacotes não encontrados
   - Solução: Verificar URLs e configurações

### Comandos de Diagnóstico

```bash
aws codeartifact describe-domain --domain my-domain
aws codeartifact list-repositories --domain my-domain
aws codeartifact describe-repository --domain my-domain --repository my-repo
```

<br />

## Perguntas e Respostas

### 1. Qual é a duração máxima de um token de autorização do CodeArtifact?

**Resposta**: 12 horas

**Explicação**: Os tokens de autorização do CodeArtifact têm duração máxima de 12 horas e mínima de 15 minutos. Após a expiração, é necessário gerar um novo token.

### 2. Quais gerenciadores de pacotes são suportados pelo CodeArtifact?

**Resposta**: Maven, Gradle, npm, yarn, pip, twine, NuGet e Generic

**Explicação**: O CodeArtifact suporta os principais gerenciadores de pacotes para diferentes linguagens: Java (Maven/Gradle), Node.js (npm/yarn), Python (pip/twine), .NET (NuGet) e arquivos genéricos.

### 3. Quantos domínios você pode criar por conta AWS?

**Resposta**: 10 domínios por conta

**Explicação**: Por padrão, cada conta AWS pode criar até 10 domínios CodeArtifact. Este limite pode ser aumentado através de solicitação ao suporte AWS.

### 4. O que são upstream repositories no CodeArtifact?

**Resposta**: Conexões com repositórios externos que funcionam como proxy

**Explicação**: Upstream repositories permitem ao CodeArtifact funcionar como proxy para repositórios públicos (npmjs, PyPI, Maven Central), fazendo cache local dos pacotes para melhor performance e disponibilidade.

### 5. Qual comando AWS CLI é usado para obter um token de autorização?

**Resposta**: `aws codeartifact get-authorization-token`

**Explicação**: Este comando retorna um token temporário usado para autenticação com repositórios CodeArtifact. O token deve ser usado em configurações de gerenciadores de pacotes.

### 6. Qual é o tamanho máximo de um pacote no CodeArtifact?

**Resposta**: 1 GB

**Explicação**: Cada pacote individual pode ter no máximo 1 GB de tamanho. Para arquivos maiores, considere dividir em múltiplos pacotes ou usar outras soluções de armazenamento.

### 7. Como você pode automatizar a configuração de um gerenciador de pacotes com CodeArtifact?

**Resposta**: Usando o comando `aws codeartifact login`

**Explicação**: Este comando configura automaticamente o gerenciador de pacotes especificado (npm, pip, etc.) com as URLs e credenciais corretas para o repositório CodeArtifact.

### 8. Qual permissão IAM é necessária para obter tokens de autorização?

**Resposta**: `codeartifact:GetAuthorizationToken` e `sts:GetServiceBearerToken`

**Explicação**: Ambas as permissões são necessárias: a primeira para gerar o token CodeArtifact e a segunda para que o serviço STS possa emitir o token bearer.

### 9. Quantos repositórios upstream podem ser configurados por repositório?

**Resposta**: 10 conexões upstream por repositório

**Explicação**: Cada repositório CodeArtifact pode ter até 10 conexões upstream, permitindo agregar pacotes de múltiplas fontes externas.

### 10. Qual é a estrutura hierárquica do CodeArtifact?

**Resposta**: Domínio > Repositório > Pacote > Versão

**Explicação**: A hierarquia é: Domínio (container de alto nível) contém Repositórios, que armazenam Pacotes, que têm múltiplas Versões.

### 11. Como você pode implementar diferentes ambientes (dev, staging, prod) com CodeArtifact?

**Resposta**: Usando repositórios separados ou domínios separados

**Explicação**: Você pode criar repositórios diferentes dentro do mesmo domínio ou usar domínios separados para cada ambiente, aplicando políticas de acesso específicas para cada um.

### 12. Qual serviço AWS monitora as atividades da API do CodeArtifact?

**Resposta**: AWS CloudTrail

**Explicação**: CloudTrail registra todas as chamadas de API do CodeArtifact, incluindo criação/exclusão de recursos e operações de pacotes, fornecendo auditoria completa.

### 13. Como você pode controlar quais pacotes upstream são permitidos?

**Resposta**: Através de políticas de domínio e repositório

**Explicação**: Você pode criar políticas que especificam quais pacotes podem ser baixados de repositórios upstream, implementando whitelist ou blacklist de dependências.

### 14. Qual é o formato de URL de um repositório CodeArtifact para npm?

**Resposta**: `https://DOMAIN-OWNER.d.codeartifact.REGION.amazonaws.com/npm/REPOSITORY/`

**Explicação**: A URL segue o padrão com domain-owner (número da conta), região AWS e nome do repositório, específica para cada tipo de gerenciador de pacotes.

### 15. Como você pode implementar versionamento semântico no CodeArtifact?

**Resposta**: Usando tags e políticas de retenção baseadas em padrões de versão

**Explicação**: Você pode configurar políticas que preservem versões baseadas em padrões semânticos (major.minor.patch) e aplicar tags para identificar releases estáveis versus snapshots de desenvolvimento.

<br />

## :books: Referências

- [O que é o AWS CodeArtifact?](https://docs.aws.amazon.com/pt_br/codeartifact/latest/ug/welcome.html)
- [CodeArtifact User Guide](https://docs.aws.amazon.com/codeartifact/latest/ug/)
- [CodeArtifact API Reference](https://docs.aws.amazon.com/codeartifact/latest/APIReference/)
- [Using CodeArtifact with npm](https://docs.aws.amazon.com/codeartifact/latest/ug/using-npm.html)
- [Using CodeArtifact with Maven](https://docs.aws.amazon.com/codeartifact/latest/ug/using-maven.html)
- [Using CodeArtifact with Python](https://docs.aws.amazon.com/codeartifact/latest/ug/using-python.html)

<br />

---
Feito com ♥ by :man_astronaut: Guilherme Bezerra :wave: [Entrar em contato!](https://www.linkedin.com/in/gbdsantos/)
