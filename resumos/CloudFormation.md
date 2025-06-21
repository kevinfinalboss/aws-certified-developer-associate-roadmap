<p align="center">
	<img src="./img/aws-icons/aws-CloudFormation.png" alt="aws-cloudFormation-icon" style="height:150px; width:150px;" /> 
  <br />
	<h1 align="center">
    CloudFormation - AWS Developer Associate
  </h1>
</p>	

<br />

## :pushpin: Índice

- [Introdução](#introdução)
- [Benefícios de utilizar o CloudFormation](#benefícios-de-utilizar-o-cloudformation)
- [Como o CloudFormation funciona](#como-o-cloudformation-funciona)
- [Implantando modelos do CloudFormation](#implantando-modelos-do-cloudformation)
- [Anatomia dos Templates](#anatomia-dos-templates)
- [Introdução ao YAML](#introdução-ao-yaml)
- [Recursos (Resources)](#recursos-resources)
- [Parâmetros (Parameters)](#parâmetros-parameters)
- [Mapeamentos (Mappings)](#mapeamentos-mappings)
- [Saídas (Outputs)](#saídas-outputs)
- [Condicionais (Conditions)](#condicionais-conditions)
- [Funções Intrínsecas](#funções-intrínsecas)
- [Pseudo Parameters](#pseudo-parameters)
- [Rollbacks e Troubleshooting](#rollbacks-e-troubleshooting)
- [Cross vs Pilhas Aninhadas](#cross-vs-pilhas-aninhadas)
- [StackSets](#stacksets)
- [CloudFormation Drift](#cloudformation-drift)
- [Helper Scripts](#helper-scripts)
- [ChangeSets](#changesets)
- [CloudFormation Stack Policies](#cloudformation-stack-policies)
- [Terminação de Stack](#terminação-de-stack)
- [Best Practices](#best-practices)
- [Perguntas e Respostas](#perguntas-e-respostas)
- [Referências](#books-referências)

<br />

## Introdução

Amazon **CloudFormation** é um serviço de Infrastructure as Code (IaC) que permite modelar e provisionar recursos da AWS usando templates declarativos. Com o CloudFormation, você pode criar, atualizar e deletar recursos de forma automática, garantindo consistência e repetibilidade em suas implantações.

O CloudFormation funciona através de **templates** (arquivos JSON ou YAML) que descrevem a infraestrutura desejada, e **stacks** que são instâncias desses templates executando na AWS.

> ⚠️ O Amazon CloudFormation é gratuito, porém os recursos da AWS criados pelo CloudFormation são cobrados normalmente.

<br />

## Benefícios de utilizar o CloudFormation

- **Infrastructure as Code**: Versionar e tratar infraestrutura como código
- **Replicação**: Replicar infraestrutura em múltiplas regiões e contas
- **Controle de alterações**: Rastrear todas as mudanças na infraestrutura
- **Rollback automático**: Reverter automaticamente em caso de falhas
- **Dependências automáticas**: CloudFormation resolve dependências entre recursos
- **Custo-efetivo**: Deletar toda a infraestrutura facilmente para economizar custos
- **Produtividade**: Reutilizar templates para criar ambientes consistentes

<br />

## Como o CloudFormation funciona

1. **Design**: Criar template JSON/YAML descrevendo os recursos
2. **Upload**: Fazer upload do template para o CloudFormation
3. **Deploy**: CloudFormation analisa o template e cria os recursos na ordem correta
4. **Manage**: Atualizar ou deletar a stack conforme necessário

O CloudFormation precisa das permissões IAM necessárias para criar os recursos especificados no template.

<br />

## Implantando modelos do CloudFormation

- **Manual**: 
  - AWS Console
  - CloudFormation Designer (GUI)
- **Automatizado**: 
  - AWS CLI
  - AWS SDK
  - CI/CD pipelines

<br />

## Anatomia dos Templates

Um template CloudFormation possui a seguinte estrutura:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "Descrição do template"

Metadata:
  # Informações adicionais sobre o template

Parameters:
  # Parâmetros de entrada

Mappings:
  # Mapeamentos estáticos

Conditions:
  # Condições lógicas

Transform:
  # Transformações (ex: SAM)

Resources:
  # Recursos AWS (OBRIGATÓRIO)

Outputs:
  # Valores de saída
```

<br />

## Introdução ao YAML

YAML é a linguagem preferida para CloudFormation devido à sua legibilidade:

- **Chave/Valor**: `chave: valor`
- **Listas**: 
  ```yaml
  lista:
    - item1
    - item2
  ```
- **Objetos aninhados**:
  ```yaml
  objeto:
    propriedade1: valor1
    propriedade2: valor2
  ```
- **Comentários**: `# Isto é um comentário`
- **Strings multilinhas**:
  ```yaml
  script: |
    #!/bin/bash
    echo "Hello World"
  ```

<br />

## Recursos (Resources)

Os recursos são o núcleo do template e representam componentes AWS. Sintaxe: `AWS::Service::ResourceType`

```yaml
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c02fb55956c7d316
      InstanceType: t2.micro
      KeyName: my-key-pair
      SecurityGroupIds:
        - !Ref MySecurityGroup
      Tags:
        - Key: Name
          Value: MyInstance
  
  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow SSH access
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
```

### Atributos dos Recursos

- **Type**: Tipo do recurso (obrigatório)
- **Properties**: Propriedades específicas do recurso
- **DependsOn**: Dependências explícitas
- **Metadata**: Metadados adicionais
- **CreationPolicy**: Política de criação
- **UpdatePolicy**: Política de atualização
- **DeletionPolicy**: Política de exclusão (Retain, Delete, Snapshot)

<br />

## Parâmetros (Parameters)

Parâmetros permitem personalizar templates durante a criação da stack:

```yaml
Parameters:
  EnvironmentName:
    Type: String
    Default: Development
    Description: Nome do ambiente
    AllowedValues:
      - Development
      - Testing
      - Production
  
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t2.medium
    ConstraintDescription: Deve ser um tipo de instância válido
  
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Nome do Key Pair para acesso SSH
  
  DBPassword:
    Type: String
    NoEcho: true
    MinLength: 8
    Description: Senha do banco de dados
```

### Tipos de Parâmetros

- **String**: Texto simples
- **Number**: Valores numéricos
- **List<Number>**: Lista de números
- **CommaDelimitedList**: Lista separada por vírgulas
- **AWS-Specific Types**: AWS::EC2::KeyPair::KeyName, AWS::EC2::VPC::Id, etc.

<br />

## Mapeamentos (Mappings)

Mapeamentos são valores estáticos condicionais organizados em chave-valor:

```yaml
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0c02fb55956c7d316
    us-west-2:
      AMI: ami-0c02fb55956c7d316
    eu-west-1:
      AMI: ami-0c02fb55956c7d316
  
  EnvironmentMap:
    Development:
      InstanceType: t2.micro
    Production:
      InstanceType: t2.large

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", AMI]
      InstanceType: !FindInMap [EnvironmentMap, !Ref EnvironmentName, InstanceType]
```

<br />

## Saídas (Outputs)

Outputs retornam valores após a criação da stack e podem ser exportados para outras stacks:

```yaml
Outputs:
  VPCId:
    Description: ID da VPC criada
    Value: !Ref MyVPC
    Export:
      Name: !Sub "${AWS::StackName}-VPC-ID"
  
  WebsiteURL:
    Description: URL do website
    Value: !Sub "http://${MyInstance.PublicDnsName}"
  
  SecurityGroupId:
    Description: ID do Security Group
    Value: !Ref MySecurityGroup
    Export:
      Name: MySecurityGroup-ID
```

### Cross Stack Reference

Para importar valores de outras stacks:

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      SecurityGroupIds:
        - !ImportValue MySecurityGroup-ID
```

<br />

## Condicionais (Conditions)

Condições controlam a criação de recursos baseado em lógica:

```yaml
Conditions:
  CreateProdResources: !Equals [!Ref EnvironmentName, Production]
  CreateDevResources: !Equals [!Ref EnvironmentName, Development]
  CreateDBSubnetGroup: !And
    - !Equals [!Ref EnvironmentName, Production]
    - !Not [!Equals [!Ref DBSubnetGroupName, ""]]

Resources:
  ProdInstance:
    Type: AWS::EC2::Instance
    Condition: CreateProdResources
    Properties:
      InstanceType: t2.large
  
  DevInstance:
    Type: AWS::EC2::Instance
    Condition: CreateDevResources
    Properties:
      InstanceType: t2.micro
```

<br />

## Funções Intrínsecas

Funções built-in para manipular dados nos templates:

### Principais Funções

- **!Ref**: Referencia parâmetros ou recursos
- **!GetAtt**: Obtém atributos de recursos
- **!FindInMap**: Busca valores em mappings
- **!ImportValue**: Importa valores exportados
- **!Join**: Concatena strings
- **!Sub**: Substitui variáveis em strings
- **!Base64**: Codifica em Base64
- **!Select**: Seleciona item de lista
- **!Split**: Divide string em lista

```yaml
UserData: !Base64 !Sub |
  #!/bin/bash
  yum update -y
  echo "Hello from ${AWS::StackName}" > /var/www/html/index.html
  
DatabaseURL: !Sub 
  - "jdbc:mysql://${Domain}:${Port}/${DBName}"
  - Domain: !GetAtt MyDB.Endpoint.Address
    Port: !GetAtt MyDB.Endpoint.Port
    DBName: !Ref DatabaseName
```

<br />

## Pseudo Parameters

Parâmetros automáticos fornecidos pela AWS:

- **AWS::AccountId**: ID da conta AWS
- **AWS::Region**: Região atual
- **AWS::StackId**: ID da stack
- **AWS::StackName**: Nome da stack
- **AWS::NoValue**: Remove propriedade condicionalmente

```yaml
Tags:
  - Key: AccountId
    Value: !Ref AWS::AccountId
  - Key: Region
    Value: !Ref AWS::Region
  - Key: StackName
    Value: !Ref AWS::StackName
```

<br />

## Rollbacks e Troubleshooting

### Comportamento de Rollback

- **CREATE_FAILED**: Stack é deletada automaticamente
- **UPDATE_FAILED**: Stack volta ao estado anterior
- **DELETE_FAILED**: Stack fica em estado de falha

### Troubleshooting

1. **CloudFormation Events**: Verificar log de eventos da stack
2. **CloudTrail**: Analisar chamadas de API
3. **Resource Status**: Verificar status individual dos recursos
4. **Stack Policies**: Podem estar bloqueando operações

### Disable Rollback

```bash
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --disable-rollback
```

<br />

## Cross vs Pilhas Aninhadas

### Cross Stack (Pilhas Cruzadas)

- **Quando usar**: Recursos compartilhados entre diferentes lifecycles
- **Implementação**: Outputs + ImportValue
- **Exemplo**: VPC compartilhada entre múltiplas aplicações

### Nested Stacks (Pilhas Aninhadas)

- **Quando usar**: Componentes reutilizáveis dentro da mesma aplicação
- **Implementação**: Recurso AWS::CloudFormation::Stack
- **Exemplo**: Template de Load Balancer reutilizado

```yaml
NestedStack:
  Type: AWS::CloudFormation::Stack
  Properties:
    TemplateURL: https://s3.amazonaws.com/my-bucket/nested-template.yaml
    Parameters:
      VPCId: !Ref MyVPC
      SubnetId: !Ref MySubnet
```

<br />

## StackSets

StackSets permite implantar stacks em múltiplas contas e regiões:

### Conceitos

- **Stack Set**: Template principal
- **Stack Instances**: Instâncias do template em contas/regiões específicas
- **Administration Account**: Conta que gerencia o StackSet
- **Target Accounts**: Contas onde as stacks são criadas

### Casos de Uso

- Configurações de segurança organizacionais
- Recursos de compliance
- Baseline de conta

<br />

## CloudFormation Drift

Detecta alterações manuais nos recursos após a criação:

### Tipos de Drift

- **IN_SYNC**: Sem alterações
- **DRIFTED**: Recursos modificados
- **NOT_CHECKED**: Drift não verificado
- **UNKNOWN**: Não foi possível determinar

### Comandos

```bash
aws cloudformation detect-stack-drift --stack-name my-stack
aws cloudformation describe-stack-drift-detection-status --stack-drift-detection-id <id>
```

<br />

## Helper Scripts

Scripts para configurar instâncias EC2:

### cfn-init

Processa metadados AWS::CloudFormation::Init:

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Metadata:
      AWS::CloudFormation::Init:
        config:
          packages:
            yum:
              httpd: []
          files:
            /var/www/html/index.html:
              content: !Sub |
                <h1>Hello from ${AWS::StackName}</h1>
              mode: '000644'
          services:
            sysvinit:
              httpd:
                enabled: true
                ensureRunning: true
    Properties:
      UserData: !Base64 !Sub |
        #!/bin/bash
        /opt/aws/bin/cfn-init -v --stack ${AWS::StackName} --resource MyInstance --region ${AWS::Region}
        /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource MyInstance --region ${AWS::Region}
```

### cfn-signal

Sinaliza o status de configuração da instância:

```yaml
CreationPolicy:
  ResourceSignal:
    Count: 1
    Timeout: PT10M
```

### cfn-get-metadata

Recupera metadados de recursos:

```bash
cfn-get-metadata --stack mystack --resource MyInstance
```

### cfn-hup

Monitora mudanças em metadados:

```yaml
files:
  /etc/cfn/cfn-hup.conf:
    content: !Sub |
      [main]
      stack=${AWS::StackName}
      region=${AWS::Region}
      interval=1
  /etc/cfn/hooks.d/cfn-auto-reloader.conf:
    content: !Sub |
      [cfn-auto-reloader-hook]
      triggers=post.update
      path=Resources.MyInstance.Metadata.AWS::CloudFormation::Init
      action=/opt/aws/bin/cfn-init -v --stack ${AWS::StackName} --resource MyInstance --region ${AWS::Region}
```

<br />

## ChangeSets

ChangeSets permitem visualizar mudanças antes de aplicá-las:

### Processo

1. **Criar ChangeSet**: Analisa diferenças
2. **Revisar**: Verificar mudanças propostas
3. **Executar**: Aplicar mudanças

### Comandos

```bash
aws cloudformation create-change-set \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --change-set-name my-changeset

aws cloudformation describe-change-set \
  --change-set-name my-changeset \
  --stack-name my-stack

aws cloudformation execute-change-set \
  --change-set-name my-changeset \
  --stack-name my-stack
```

<br />

## CloudFormation Stack Policies

Protegem recursos críticos durante atualizações:

```json
{
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "Update:Delete",
      "Principal": "*",
      "Resource": "LogicalResourceId/ProductionDatabase"
    },
    {
      "Effect": "Allow",
      "Action": "Update:*",
      "Principal": "*",
      "Resource": "*"
    }
  ]
}
```

<br />

## Terminação de Stack

### Termination Protection

Previne exclusão acidental:

```bash
aws cloudformation update-termination-protection \
  --enable-termination-protection \
  --stack-name my-stack
```

### Deletion Policies

- **Delete**: Comportamento padrão
- **Retain**: Manter recurso após exclusão da stack
- **Snapshot**: Criar snapshot antes de deletar (EBS, RDS)

```yaml
Resources:
  MyDB:
    Type: AWS::RDS::DBInstance
    DeletionPolicy: Snapshot
    Properties:
      DBInstanceIdentifier: mydb
```

<br />

## Best Practices

### Organização

- Usar templates modulares e reutilizáveis
- Separar recursos por lifecycle
- Usar naming conventions consistentes
- Versionar templates

### Segurança

- Usar IAM roles específicas
- Não hardcode secrets nos templates
- Usar AWS Secrets Manager ou Parameter Store
- Implementar stack policies para recursos críticos

### Monitoramento

- Habilitar CloudTrail para auditoria
- Usar tags para organização e billing
- Implementar drift detection
- Monitorar events e logs

### Performance

- Usar nested stacks para templates grandes
- Implementar parallel creation quando possível
- Usar creation policies para recursos dependentes

<br />

## Perguntas e Respostas

### 1. Qual é a diferença entre parâmetros e mapeamentos no CloudFormation?

**Resposta**: Parâmetros são valores dinâmicos fornecidos durante a criação da stack, permitindo personalização. Mapeamentos são valores estáticos definidos no template, organizados em estrutura chave-valor, ideais para configurações condicionais baseadas em região ou ambiente.

### 2. Como funciona o rollback automático no CloudFormation?

**Resposta**: Se uma stack falha durante CREATE, ela é automaticamente deletada. Se falha durante UPDATE, retorna ao estado anterior. Durante DELETE, se falha, a stack fica em estado de falha até ser resolvida manualmente.

### 3. Qual a diferença entre !Ref e !GetAtt?

**Resposta**: !Ref retorna o valor padrão do recurso (ex: Instance ID para EC2). !GetAtt retorna atributos específicos do recurso (ex: PublicDnsName de uma instância EC2).

### 4. Como implementar dependências entre recursos?

**Resposta**: CloudFormation resolve dependências automaticamente através de !Ref e !GetAtt. Para dependências explícitas, use o atributo DependsOn.

### 5. Quando usar Cross Stack vs Nested Stack?

**Resposta**: Use Cross Stack para recursos compartilhados entre diferentes lifecycles (ex: VPC compartilhada). Use Nested Stack para componentes reutilizáveis dentro da mesma aplicação (ex: template de Load Balancer).

### 6. Como proteger recursos críticos durante atualizações?

**Resposta**: Use Stack Policies para definir permissões de atualização, DeletionPolicy: Retain para preservar recursos, e Termination Protection para prevenir exclusão da stack.

### 7. Qual a finalidade do cfn-init?

**Resposta**: cfn-init processa metadados AWS::CloudFormation::Init para configurar instâncias EC2, incluindo instalação de pacotes, criação de arquivos e configuração de serviços.

### 8. Como funciona o CloudFormation Drift?

**Resposta**: Drift detection compara o estado atual dos recursos com a configuração definida no template, identificando modificações manuais que possam ter ocorrido fora do CloudFormation.

### 9. O que são ChangeSets e quando usá-los?

**Resposta**: ChangeSets permitem visualizar e revisar mudanças antes de aplicá-las na stack, úteis para ambientes de produção onde é necessário aprovar alterações antes da execução.

### 10. Como implementar configuração condicional de recursos?

**Resposta**: Use a seção Conditions com funções como !Equals, !And, !Or, !Not para definir lógica, e aplique condições aos recursos usando o atributo Condition.

### 11. Qual é o limite de recursos por stack?

**Resposta**: Uma stack pode ter até 500 recursos. Para infraestruturas maiores, use nested stacks ou divida em múltiplas stacks relacionadas.

### 12. Como gerenciar secrets em templates CloudFormation?

**Resposta**: Nunca coloque secrets diretamente no template. Use AWS Secrets Manager, Systems Manager Parameter Store (SecureString), ou parâmetros com NoEcho: true.

### 13. O que acontece quando uma stack fica em estado UPDATE_ROLLBACK_FAILED?

**Resposta**: A stack não pode ser atualizada nem revertida. Soluções: continue o rollback pulando recursos problemáticos, ou contate o AWS Support para casos complexos.

### 14. Como implementar blue-green deployments com CloudFormation?

**Resposta**: Crie uma nova stack (green) paralela à existente (blue), teste, e depois redirecione o tráfego (via Route 53 ou ALB) antes de deletar a stack antiga.

### 15. Qual a diferença entre DeletionPolicy e DependsOn?

**Resposta**: DeletionPolicy controla o que acontece com o recurso quando a stack é deletada (Delete, Retain, Snapshot). DependsOn controla a ordem de criação/deleção de recursos.

### 16. Como debugar falhas de criação de stack?

**Resposta**: Verifique CloudFormation Events na console, use CloudTrail para logs de API, analise logs específicos do serviço, e considere usar --disable-rollback para investigar recursos que falharam.

### 17. O que são Transform sections e quando usar?

**Resposta**: Transform aplica macros ao template, sendo AWS::Serverless-2016-10-31 (SAM) o mais comum, que simplifica a definição de recursos serverless expandindo automaticamente para CloudFormation padrão.

### 18. Como implementar approval workflows com CloudFormation?

**Resposta**: Use ChangeSets com CodePipeline, implemente manual approval actions, ou use AWS Service Catalog para templates pré-aprovados com workflows de governança.

### 19. Qual a diferença entre Stack failure e Resource failure?

**Resposta**: Resource failure é quando um recurso específico falha na criação/atualização. Stack failure ocorre quando falhas de recursos impedem a conclusão da operação da stack inteira.

### 20. Como otimizar performance de criação de stacks grandes?

**Resposta**: Use nested stacks para paralelização, minimize dependências desnecessárias, agrupe recursos relacionados, e considere usar StackSets para deployment em múltiplas regiões simultaneamente.

<br />

## 📚 Referências

- [AWS CloudFormation User Guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)
- [CloudFormation Template Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-reference.html)
- [CloudFormation Intrinsic Functions](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)
- [CloudFormation Best Practices](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html)
- [AWS CloudFormation CLI Reference](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/index.html)
- [CloudFormation Helper Scripts](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-helper-scripts-reference.html)

<br />

---
