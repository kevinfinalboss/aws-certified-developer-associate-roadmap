<p align="center">
	<img src="./img/aws-icons/aws-EC2.png" alt="aws-ec2-icon" style="height:150px; width:150px;" /> 
  <br />
	<h1 align="center">
    EC2 - Guia Completo AWS Developer Associate
  </h1>
</p>	

<br />

## :pushpin: Índice

- [Introdução](#introdução)
- [Tipos de Instâncias EC2](#tipos-de-instâncias-ec2)
- [Opções de Compra](#opções-de-compra)
- [Ciclo de Vida da Instância](#ciclo-de-vida-da-instância)
- [User Data](#user-data)
- [Metadados da Instância](#metadados-da-instância)
- [Placement Groups](#placement-groups)
- [Elastic Network Interface (ENI)](#elastic-network-interface-eni)
- [Storage no EC2](#storage-no-ec2)
  - [EBS (Elastic Block Store)](#ebs-elastic-block-store)
  - [EFS (Elastic File System)](#efs-elastic-file-system)
  - [FSx](#fsx)
  - [Instance Store](#instance-store)
  - [Comparação de Storage](#comparação-de-storage)
- [AMI (Amazon Machine Image)](#ami-amazon-machine-image)
- [Auto Scaling Group (ASG)](#auto-scaling-group-asg)
- [Load Balancers](#load-balancers)
- [Security Groups](#security-groups)
- [Key Pairs](#key-pairs)
- [Hibernation](#hibernation)
- [Monitoring](#monitoring)
- [Pricing](#pricing)
- [Best Practices](#best-practices)
- [Questões de Prova](#questões-de-prova)
- [Referências](#books-referências)

<br />

## Introdução

**Amazon Elastic Compute Cloud (EC2)** é o serviço fundamental de computação da AWS que fornece capacidade de computação redimensionável na nuvem. É um dos serviços mais importantes para a certificação AWS Developer Associate.

**Características principais:**
- Instâncias virtuais escaláveis
- Controle completo do sistema operacional
- Integração com outros serviços AWS
- Modelo de pagamento por uso
- Disponibilidade em múltiplas regiões e AZs

**Casos de uso:**
- Servidores web e aplicações
- Processamento de dados
- Desenvolvimento e testes
- Backup e disaster recovery
- High Performance Computing (HPC)

<br />

## Tipos de Instâncias EC2

### Famílias de Instâncias

**Nomenclatura:** m5.large
- **m:** Família da instância
- **5:** Geração
- **large:** Tamanho

#### 1. General Purpose (Uso Geral)
**Famílias:** A1, T4g, T3, T3a, T2, M6i, M6a, M5, M5a, M5n, M4

**Características:**
- Balanceamento entre compute, memória e rede
- Burstable performance (T-series)
- CPU Credits (T2/T3)

**Casos de uso:**
- Servidores web
- Aplicações pequenas e médias
- Desenvolvimento e testes
- Microserviços

#### 2. Compute Optimized (Computação Otimizada)
**Famílias:** C6i, C6a, C5, C5n, C4

**Características:**
- Processadores de alta performance
- Ratio alto de CPU para memória
- Enhanced networking

**Casos de uso:**
- Processamento científico
- Gaming servers
- Web servers de alta performance
- Aplicações CPU-intensive

#### 3. Memory Optimized (Memória Otimizada)
**Famílias:** R6i, R5, R5a, R5n, R4, X1e, X1, High Memory, z1d

**Características:**
- Alta quantidade de RAM
- Processamento rápido de grandes datasets
- Armazenamento NVMe SSD

**Casos de uso:**
- Bancos de dados in-memory
- Real-time analytics
- Caches distribuídos
- Aplicações SAP

#### 4. Storage Optimized (Armazenamento Otimizado)
**Famílias:** I4i, I3, I3en, D2, D3, H1

**Características:**
- Alto IOPS
- Armazenamento NVMe SSD local
- Alto throughput sequencial

**Casos de uso:**
- Bancos de dados distribuídos
- Data warehousing
- Sistemas de arquivos distribuídos
- Aplicações que requerem baixa latência

#### 5. Accelerated Computing (Computação Acelerada)
**Famílias:** P4, P3, P2, Inf1, G4, G3, F1, VT1

**Características:**
- GPUs ou FPGAs
- Processamento paralelo
- Machine learning optimized

**Casos de uso:**
- Machine learning
- High performance computing
- Rendering 3D
- Video processing

### Burstable Performance (T-series)

**CPU Credits:**
- Instâncias T acumulam créditos quando CPU está abaixo da baseline
- Gastam créditos quando CPU está acima da baseline
- T2: Unlimited mode disponível
- T3/T4g: Unlimited mode por padrão

<br />

## Opções de Compra

### On-Demand Instances
**Características:**
- Pagamento por segundo (Linux) ou por hora (Windows)
- Sem compromisso de longo prazo
- Controle total sobre ciclo de vida

**Casos de uso:**
- Desenvolvimento e testes
- Workloads imprevisíveis
- Aplicações de curto prazo

### Reserved Instances (RI)
**Características:**
- Desconto de até 72% vs On-Demand
- Compromisso de 1 ou 3 anos
- Opções de pagamento: No Upfront, Partial Upfront, All Upfront

**Tipos:**
- **Standard RI:** Maior desconto, menos flexibilidade
- **Convertible RI:** Menor desconto, mais flexibilidade

### Spot Instances
**Características:**
- Desconto de até 90% vs On-Demand
- Preço varia baseado em oferta/demanda
- Podem ser interrompidas com 2 minutos de aviso

**Casos de uso:**
- Processamento batch
- Análise de dados
- Workloads tolerantes a falhas

### Dedicated Hosts
**Características:**
- Servidor físico dedicado
- Controle sobre placement de instâncias
- Licenciamento por socket/core

**Casos de uso:**
- Compliance regulatório
- Licenças BYOL (Bring Your Own License)

### Dedicated Instances
**Características:**
- Instâncias em hardware dedicado
- Isolamento no nível de hardware
- Sem controle sobre placement

### Savings Plans
**Características:**
- Desconto de até 72%
- Compromisso de uso por hora ($/hour)
- Flexibilidade de família, tamanho, AZ, região

<br />

## Ciclo de Vida da Instância

### Estados da Instância

**Pending:** Instância sendo inicializada
**Running:** Instância em execução (cobrança ativa)
**Stopping:** Instância sendo parada
**Stopped:** Instância parada (sem cobrança de compute)
**Shutting-down:** Instância sendo terminada
**Terminated:** Instância terminada (não pode ser reiniciada)

### Operações Importantes

**Start/Stop:**
- Preserva Instance Store data apenas durante execução
- EBS root volume persiste (por padrão)
- Pode mudar de host físico

**Reboot:**
- Preserva Instance Store data
- Permanece no mesmo host físico

**Terminate:**
- Deleta Instance Store data
- EBS root volume deletado (por padrão, configurável)

<br />

## User Data

**Funcionalidade:** Script executado na primeira inicialização da instância.

**Características:**
- Executa como root
- Executado apenas no primeiro boot
- Máximo de 16KB
- Base64 encoded quando passado via API

**Casos de uso:**
- Instalação de software
- Configuração inicial
- Download de arquivos
- Configuração de serviços

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "Hello World" > /var/www/html/index.html
```

<br />

## Metadados da Instância

**Endpoint:** http://169.254.169.254/latest/meta-data/

**Informações disponíveis:**
- Instance ID
- Instance type
- Security groups
- IAM role credentials
- Network information

**Exemplo de uso:**
```bash
curl http://169.254.169.254/latest/meta-data/instance-id
curl http://169.254.169.254/latest/meta-data/placement/availability-zone
```

**IMDSv2 (Instance Metadata Service v2):**
- Mais seguro que v1
- Requer token de sessão
- Proteção contra SSRF

<br />

## Placement Groups

### Cluster Placement Group
**Características:**
- Instâncias em close proximity
- Mesma AZ
- Enhanced networking recomendado

**Casos de uso:**
- HPC applications
- Baixa latência de rede

### Spread Placement Group
**Características:**
- Instâncias em hardware diferente
- Máximo 7 instâncias por AZ
- Cross-AZ suportado

**Casos de uso:**
- Alta disponibilidade
- Aplicações críticas

### Partition Placement Group
**Características:**
- Instâncias distribuídas em partições
- Até 7 partições por AZ
- Isolamento de falhas

**Casos de uso:**
- Hadoop
- Cassandra
- Kafka

<br />

## Elastic Network Interface (ENI)

**Funcionalidades:**
- Virtual network card
- Múltiplos ENIs por instância
- Portable entre instâncias

**Atributos:**
- Primary private IPv4
- Secondary private IPv4s
- Elastic IP (IPv4)
- Public IPv4
- Security groups
- MAC address

**Casos de uso:**
- Failover scenarios
- Management networks
- Dual-homed instances

<br />

## Storage no EC2

### EBS (Elastic Block Store)

**Características:**
- Block-level storage
- Persistent storage
- Replicação automática dentro da AZ
- Snapshots para S3

#### Tipos de Volume EBS

##### GP3 (General Purpose SSD)
- **Performance:** 3,000 IOPS baseline, até 16,000 IOPS
- **Throughput:** 125 MiB/s baseline, até 1,000 MiB/s
- **Tamanho:** 1 GiB - 16 TiB
- **Casos de uso:** Boot volumes, aplicações de uso geral

##### GP2 (General Purpose SSD - Previous Generation)
- **Performance:** 3 IOPS/GiB, máximo 16,000 IOPS
- **Burst:** Até 3,000 IOPS para volumes < 1 TiB
- **Tamanho:** 1 GiB - 16 TiB
- **Casos de uso:** Boot volumes, aplicações de uso geral

##### IO2 (Provisioned IOPS SSD)
- **Performance:** Até 64,000 IOPS
- **Durabilidade:** 99.999%
- **Tamanho:** 4 GiB - 16 TiB
- **Casos de uso:** Bancos de dados críticos

##### IO1 (Provisioned IOPS SSD - Previous Generation)
- **Performance:** Até 64,000 IOPS
- **Durabilidade:** 99.999%
- **Tamanho:** 4 GiB - 16 TiB
- **Casos de uso:** Bancos de dados críticos

##### ST1 (Throughput Optimized HDD)
- **Performance:** Até 500 MiB/s
- **Baseline:** 40 MiB/s/TiB
- **Tamanho:** 125 GiB - 16 TiB
- **Casos de uso:** Big data, data warehousing

##### SC1 (Cold HDD)
- **Performance:** Até 250 MiB/s
- **Baseline:** 12 MiB/s/TiB
- **Tamanho:** 125 GiB - 16 TiB
- **Casos de uso:** Armazenamento infrequente

#### EBS Multi-Attach
**Características:**
- Disponível apenas para IO1/IO2
- Até 16 instâncias por volume
- Mesma AZ
- Cluster-aware file systems necessários

#### EBS Snapshots
**Características:**
- Incremental backups para S3
- Cross-region copy
- Snapshot scheduling via Data Lifecycle Manager

### EFS (Elastic File System)

**Características:**
- NFSv4.1 protocol
- Fully managed
- Concurrent access (1000s of instances)
- Cross-AZ access
- POSIX compliant

#### Performance Modes
**General Purpose:**
- Latência mais baixa
- Até 7,000 file operations/sec

**Max I/O:**
- Maior latência
- Mais de 7,000 file operations/sec

#### Throughput Modes
**Provisioned:**
- Throughput independente do tamanho
- Até 4 GiB/s

**Bursting:**
- Throughput proporcional ao tamanho
- 100 MiB/s per TiB

#### Storage Classes
**Standard:** Acesso frequente
**Infrequent Access (IA):** Acesso menos frequente, custo menor

**Casos de uso:**
- Content management
- Web serving
- Data analytics
- WordPress

### FSx

#### FSx for Windows File Server
**Características:**
- Windows-native shared file system
- SMB protocol
- Active Directory integration
- NTFS, DFS, ACLs

**Casos de uso:**
- Windows applications
- SQL Server
- SharePoint

#### FSx for Lustre
**Características:**
- High-performance computing
- Parallel file system
- S3 integration
- Scratch and persistent storage

**Casos de uso:**
- Machine learning
- Video processing
- Financial modeling

#### FSx for NetApp ONTAP
**Características:**
- NetApp ONTAP features
- NFS, SMB, iSCSI protocols
- Snapshots, cloning
- Deduplication

#### FSx for OpenZFS
**Características:**
- OpenZFS features
- NFS protocol
- Point-in-time snapshots
- Data compression

### Instance Store

**Características:**
- Temporary storage
- Fisicamente attached ao host
- Alto IOPS e throughput
- Dados perdidos se instância parar/terminar

**Casos de uso:**
- Buffers e caches
- Temporary processing
- Replicated data

### Comparação de Storage

| Característica | EBS | EFS | FSx | Instance Store |
|:-------------- |:--- |:--- |:--- |:-------------- |
| **Tipo** | Block | File | File | Block |
| **Protocolo** | - | NFS | SMB/NFS/iSCSI | - |
| **Persistência** | Sim | Sim | Sim | Não |
| **Multi-instância** | Não* | Sim | Sim | Não |
| **Cross-AZ** | Não | Sim | Sim | Não |
| **Backup** | Snapshots | Backup | Backup | Não |
| **Custo** | Médio | Alto | Alto | Incluído |

*Exceto Multi-Attach para IO1/IO2

<br />

## AMI (Amazon Machine Image)

**Funcionalidades:**
- Template para instâncias EC2
- Inclui OS, aplicações, configurações
- Regional (pode ser copiada entre regiões)

### Tipos de AMI

**AWS Managed:**
- Mantidas pela AWS
- Atualizações regulares
- Amazon Linux, Ubuntu, Windows

**AWS Marketplace:**
- Vendidas por terceiros
- Software pré-instalado
- Licenciamento incluído

**Community:**
- Compartilhadas por usuários
- Gratuitas
- Usar com cuidado

**Custom:**
- Criadas por você
- Configurações específicas
- Tempo de boot reduzido

### Criação de AMI

**Processo:**
1. Configure instância EC2
2. Pare a instância (recomendado)
3. Crie AMI
4. AMI criada com snapshots dos volumes

**Considerações:**
- Instância pode permanecer rodando
- Snapshots automáticos dos volumes EBS
- Instance store volumes não incluídos

<br />

## Auto Scaling Group (ASG)

**Funcionalidades:**
- Escalabilidade automática
- Health checks
- Multiple AZs
- Integration com Load Balancers

### Configuração do ASG

**Launch Configuration/Template:**
- AMI ID
- Instance type
- Security groups
- User data
- Key pair

**ASG Settings:**
- Min/Max/Desired capacity
- Availability Zones
- Health check type
- Health check grace period

### Scaling Policies

#### Target Tracking
- Mantém métrica específica
- Exemplo: CPU utilization = 70%
- Mais simples de configurar

#### Simple Scaling
- Baseado em CloudWatch alarms
- Um step por vez
- Cooldown period

#### Step Scaling
- Múltiplos steps baseados em alarm
- Sem cooldown period
- Mais responsivo

#### Predictive Scaling
- Machine learning
- Antecipa mudanças
- Provisiona recursos antecipadamente

### Warm Pools
- Instâncias pré-inicializadas
- Reduz tempo de scaling
- Custo reduzido

### Lifecycle Hooks
- Customização durante launch/terminate
- Execução de scripts
- Integração com outros serviços

<br />

## Load Balancers

### Application Load Balancer (ALB)
**Características:**
- Layer 7 (HTTP/HTTPS)
- Path-based routing
- Host-based routing
- WebSocket support

### Network Load Balancer (NLB)
**Características:**
- Layer 4 (TCP/UDP)
- Ultra-low latency
- Static IP addresses
- Extreme performance

### Gateway Load Balancer (GLB)
**Características:**
- Layer 3 (IP)
- Third-party appliances
- GENEVE protocol

### Classic Load Balancer (CLB)
**Características:**
- Layer 4 & 7
- Legacy
- Não recomendado para novos projetos

<br />

## Security Groups

**Funcionalidades:**
- Virtual firewall
- Stateful
- Instance level
- Allow rules apenas

**Características:**
- Default: All outbound allowed, inbound blocked
- Pode referenciar outros security groups
- Múltiplos security groups por instância
- Mudanças aplicam imediatamente

**Regras:**
- Protocol (TCP/UDP/ICMP)
- Port range
- Source (IP/CIDR/SG)

<br />

## Key Pairs

**Funcionalidades:**
- Acesso SSH/RDP
- Public/private key pair
- AWS armazena public key
- Você mantém private key

**Tipos:**
- **RSA:** 2048-bit
- **ED25519:** Mais seguro

**Importante:**
- Sem key pair = sem acesso
- Perda da private key = perda de acesso
- Backup da private key essencial

<br />

## Hibernation

**Funcionalidades:**
- Preserva in-memory state
- Faster boot times
- EBS root volume preservado

**Limitações:**
- Não disponível para todas as instâncias
- Máximo 60 dias hibernação
- RAM deve ser menor que 150GB

<br />

## Monitoring

### CloudWatch Metrics

**Basic Monitoring (grátis):**
- Métricas cada 5 minutos
- CPU, Network, Disk

**Detailed Monitoring (pago):**
- Métricas cada 1 minuto
- Mais granularidade

**Custom Metrics:**
- Métricas específicas da aplicação
- CloudWatch Agent

### CloudTrail
- API calls logging
- Governance e compliance

<br />

## Pricing

### Fatores de Cobrança

**Compute:**
- Instance type
- Duration
- Pricing model

**Storage:**
- EBS volumes
- EBS snapshots
- Instance store (incluído)

**Network:**
- Data transfer out
- Data transfer between AZs
- Elastic IPs

**Outros:**
- Load Balancers
- Dedicated Hosts
- Reserved IP addresses

<br />

## Best Practices

### Performance
- Escolha o instance type apropriado
- Use Enhanced Networking
- Optimize EBS performance
- Monitor CloudWatch metrics

### Security
- Use security groups apropriados
- Mantenha OS atualizado
- Use IAM roles ao invés de access keys
- Enable detailed logging

### Cost Optimization
- Use Reserved Instances para workloads previsíveis
- Considere Spot Instances para workloads fault-tolerant
- Right-size instances regularmente
- Use Auto Scaling para otimizar custos

### Reliability
- Deploy em múltiplas AZs
- Use Auto Scaling Groups
- Implement health checks
- Regular backups (snapshots)

<br />

## Questões de Prova

### Questão 1
**Cenário:** Uma aplicação web precisa de alta disponibilidade e tolerância a falhas. A aplicação deve ser distribuída em múltiplas AZs e escalar automaticamente baseado na demanda.

**Qual é a melhor arquitetura?**

A) EC2 instance em uma AZ com EBS volume  
B) Multiple EC2 instances em uma AZ com ALB  
C) Auto Scaling Group em múltiplas AZs com ALB  
D) Single EC2 instance com múltiplos EBS volumes  

**Resposta:** C) Auto Scaling Group em múltiplas AZs com ALB
**Explicação:** ASG fornece escalabilidade automática e high availability, múltiplas AZs garantem fault tolerance, ALB distribui tráfego.

### Questão 2
**Cenário:** Uma aplicação HPC precisa de baixa latência de rede entre instâncias para processamento paralelo.

**Qual Placement Group usar?**

A) Spread Placement Group  
B) Cluster Placement Group  
C) Partition Placement Group  
D) Nenhum Placement Group  

**Resposta:** B) Cluster Placement Group
**Explicação:** Cluster PG coloca instâncias em close proximity na mesma AZ para baixa latência de rede.

### Questão 3
**Cenário:** Uma aplicação precisa de armazenamento persistente que pode ser compartilhado entre múltiplas instâncias EC2 em diferentes AZs.

**Qual storage usar?**

A) EBS General Purpose (GP3)  
B) Instance Store  
C) EFS  
D) EBS Provisioned IOPS (IO2)  

**Resposta:** C) EFS
**Explicação:** EFS é o único que suporta acesso concorrente de múltiplas instâncias cross-AZ.

### Questão 4
**Cenário:** Uma aplicação batch pode ser interrompida e possui tolerância a falhas. O foco é minimizar custos.

**Qual opção de compra usar?**

A) On-Demand Instances  
B) Reserved Instances  
C) Spot Instances  
D) Dedicated Hosts  

**Resposta:** C) Spot Instances
**Explicação:** Spot Instances oferecem até 90% de desconto e são ideais para workloads fault-tolerant.

### Questão 5
**Cenário:** Uma aplicação precisa de dados de configuração específicos da instância durante a inicialização.

**Onde buscar essas informações?**

A) User Data  
B) Instance Metadata  
C) CloudFormation  
D) Systems Manager Parameter Store  

**Resposta:** B) Instance Metadata
**Explicação:** Instance Metadata fornece informações sobre a instância como instance-id, AZ, security groups, etc.

### Questão 6
**Cenário:** Uma aplicação de banco de dados precisa de armazenamento com máximo IOPS e mínima latência.

**Qual storage usar?**

A) EBS GP3  
B) EBS IO2  
C) Instance Store  
D) EFS  

**Resposta:** C) Instance Store
**Explicação:** Instance Store oferece o maior IOPS e menor latência por estar fisicamente attached ao host.

### Questão 7
**Cenário:** Uma aplicação Windows precisa de shared file system com Active Directory integration.

**Qual solução usar?**

A) EFS  
B) EBS Multi-Attach  
C) FSx for Windows File Server  
D) S3  

**Resposta:** C) FSx for Windows File Server
**Explicação:** FSx for Windows é otimizado para aplicações Windows com AD integration.

### Questão 8
**Cenário:** Uma aplicação precisa manter estado de memória durante manutenção programada.

**Qual funcionalidade usar?**

A) Stop/Start  
B) Hibernation  
C) Reboot  
D) Terminate/Launch  

**Resposta:** B) Hibernation
**Explicação:** Hibernation preserva o estado da memória RAM no EBS root volume.

### Questão 9
**Cenário:** Uma aplicação crítica precisa garantir que instâncias sejam executadas em hardware físico separado.

**Qual solução usar?**

A) Cluster Placement Group  
B) Spread Placement Group  
C) Multiple Security Groups  
D) Different Instance Types  

**Resposta:** B) Spread Placement Group
**Explicação:** Spread PG garante que instâncias sejam executadas em hardware físico diferente.

### Questão 10
**Cenário:** Um volume EBS precisa ser compartilhado entre múltiplas instâncias EC2 na mesma AZ para um cluster de banco de dados.

**Qual tipo de volume usar?**

A) GP3  
B) IO2 com Multi-Attach  
C) ST1  
D) SC1  

**Resposta:** B) IO2 com Multi-Attach
**Explicação:** Apenas IO1/IO2 suportam Multi-Attach para compartilhamento entre instâncias.

<br />

## :books: Referências

- [Amazon EC2 User Guide](https://docs.aws.amazon.com/ec2/)
- [Amazon EBS User Guide](https://docs.aws.amazon.com/ebs/)
- [Amazon EFS User Guide](https://docs.aws.amazon.com/efs/)
- [Amazon FSx User Guide](https://docs.aws.amazon.com/fsx/)
- [Auto Scaling User Guide](https://docs.aws.amazon.com/autoscaling/)
- [Elastic Load Balancing User Guide](https://docs.aws.amazon.com/elasticloadbalancing/)
- [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [EC2 Pricing](https://aws.amazon.com/ec2/pricing/)
