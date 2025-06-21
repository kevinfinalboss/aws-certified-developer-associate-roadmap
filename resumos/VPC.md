<p align="center">
	<img src="./img/aws-icons/aws-VPC.png" alt="aws-VPC-icon" style="height:150px; width:150px;" /> 
  <br />
	<h1 align="center">
    VPC - Guia Completo AWS Developer
  </h1>
</p>	
<br />

## :pushpin: Índice
- [Introdução](#introdução)
- [Componentes Fundamentais](#componentes-fundamentais)
- [Subredes](#subredes)
- [Internet Gateway e NAT Gateways](#internet-gateway-e-nat-gateways)
- [Route Tables](#route-tables)
- [Network ACL e Grupos de Segurança](#network-acl-e-grupos-de-segurança)
- [VPC Flow Logs](#vpc-flow-logs)
- [VPC Peering](#vpc-peering)
- [VPC Endpoints](#vpc-endpoints)
- [Site-to-Site VPN e Direct Connect](#site-to-site-vpn-e-direct-connect)
- [Transit Gateway](#transit-gateway)
- [DNS no VPC](#dns-no-vpc)
- [Elastic IP](#elastic-ip)
- [Bastion Hosts](#bastion-hosts)
- [Perguntas e Respostas](#perguntas-e-respostas)
- [Referências](#books-referências)

<br />

## Introdução
VPC é uma sigla para ***Virtual Private Cloud***, que te permite criar redes privadas na nuvem da AWS e implantar os seus recursos nela. Uma VPC é um recurso regional na AWS e abrange todas as zonas de disponibilidade daquela região.

**Características principais:**
- **Regional:** Abrange múltiplas AZs
- **Isolamento:** Logicamente isolada de outras VPCs
- **CIDR Block:** Define o range de IPs (ex: 10.0.0.0/16)
- **Máximo 5 VPCs** por região (limite padrão)
- **DNS resolution e DNS hostnames** habilitados por padrão

**VPC Padrão vs Personalizada:**
- **VPC Padrão:** Criada automaticamente, subnets públicas em cada AZ
- **VPC Personalizada:** Criada pelo usuário, controle total sobre configuração

<br />

## Componentes Fundamentais

### CIDR Blocks
- **IPv4 CIDR:** Obrigatório (ex: 10.0.0.0/16)
- **IPv6 CIDR:** Opcional
- **Range permitido:** /16 (65,536 IPs) até /28 (16 IPs)
- **IPs reservados:** 5 IPs por subnet (primeiro, segundo, terceiro, penúltimo, último)

### Availability Zones
- VPC abrange **múltiplas AZs**
- Subnets são **específicas de uma AZ**
- **High availability** através de múltiplas AZs

<br />

## Subredes
É uma espécie de partição de rede dentro de uma VPC. As subnets podem estar localizadas em diferentes zonas de disponibilidade dentro da região da VPC.

### Tipos de Subredes
**Subnet Pública:**
- Tem rota para Internet Gateway
- Recursos podem ter IPs públicos
- Acessível da internet

**Subnet Privada:**
- Sem rota direta para Internet Gateway
- Recursos só têm IPs privados
- Acesso à internet via NAT Gateway/Instance

**Subnet Isolada:**
- Sem acesso à internet
- Comunicação apenas dentro da VPC
- Máxima segurança

### IPs Reservados por Subnet
Em uma subnet 10.0.0.0/24:
- **10.0.0.0:** Network address
- **10.0.0.1:** VPC router
- **10.0.0.2:** DNS resolver
- **10.0.0.3:** Reservado para uso futuro
- **10.0.0.255:** Network broadcast

<br />

## Internet Gateway e NAT Gateways

### Internet Gateway (IGW)
- **Horizontalmente escalável** e redundante
- **Um IGW por VPC**
- Permite comunicação entre VPC e internet
- **Sem cobrança** adicional
- Realiza **NAT** para instâncias com IP público

### NAT Gateway
- **Managed service** pela AWS
- Permite que recursos em subnets privadas acessem a internet
- **Específico de AZ** (criar um por AZ para HA)
- **Cobrança por hora** e por GB processado
- **Mais robusto** que NAT Instance

### NAT Instance
- **EC2 instance** configurada manualmente
- **Você gerencia** (patching, scaling, HA)
- **Mais barato** que NAT Gateway
- Pode ser usado como **bastion host**
- **Disable source/destination check**

**Comparação NAT Gateway vs NAT Instance:**

| Aspecto | NAT Gateway | NAT Instance |
|---------|-------------|--------------|
| Gerenciamento | Managed pela AWS | Você gerencia |
| Disponibilidade | 99.99% SLA | Você configura |
| Largura de banda | Até 45 Gbps | Depende do tipo da instância |
| Segurança | Security Groups da subnet | Security Groups aplicados |
| Custo | Mais caro | Mais barato |

<br />

## Route Tables
Tabelas de roteamento determinam para onde o tráfego de rede é direcionado.

### Conceitos
- **Main Route Table:** Criada automaticamente com a VPC
- **Custom Route Tables:** Criadas pelo usuário
- **Subnet Association:** Cada subnet deve estar associada a uma route table
- **Route Priority:** Rota mais específica tem prioridade

### Tipos de Rotas
**Local Route:**
```
10.0.0.0/16 -> local
```

**Internet Gateway Route:**
```
0.0.0.0/0 -> igw-xxxxxxxxx
```

**NAT Gateway Route:**
```
0.0.0.0/0 -> nat-xxxxxxxxx
```

**VPC Peering Route:**
```
172.16.0.0/16 -> pcx-xxxxxxxxx
```

<br />

## Network ACL e Grupos de Segurança

### Network ACLs (NACLs)
**Características:**
- **Firewall de subnet**
- **Stateless** (tráfego de ida e volta deve ser explicitamente permitido)
- **Regras numeradas** (avaliadas em ordem)
- **Allow e Deny** rules
- **Padrão:** Allow all inbound e outbound

**NACL Padrão vs Custom:**
- **Padrão:** Permite todo tráfego
- **Custom:** Nega todo tráfego por padrão

### Security Groups
**Características:**
- **Firewall de instância**
- **Stateful** (se permitir entrada, saída é automática)
- **Apenas Allow rules**
- **Padrão:** Allow all outbound, deny all inbound

### Comparação NACLs vs Security Groups

| Aspecto | NACLs | Security Groups |
|---------|--------|-----------------|
| Nível | Subnet | Instância |
| Estado | Stateless | Stateful |
| Regras | Allow e Deny | Apenas Allow |
| Processamento | Em ordem numérica | Todas as regras |
| Padrão | Allow all | Deny inbound, Allow outbound |

<br />

## VPC Flow Logs
É a forma de monitoramento para ver logs sobre todo o tráfego no **VPC Flow Logs**, que permite que você capture informações sobre todo o tráfego de IP de entrada e saída das interfaces VPC.

### Níveis de Captura
- **VPC Level:** Todo tráfego da VPC
- **Subnet Level:** Tráfego de uma subnet específica
- **Network Interface Level:** Tráfego de uma ENI específica

### Destinos
- **CloudWatch Logs**
- **S3 Bucket**
- **Kinesis Data Firehose**

### Formato dos Logs
```
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes windowstart windowend action flowlogstatus
```

### Casos de Uso
- **Troubleshooting de conectividade**
- **Monitoramento de segurança**
- **Análise de tráfego**
- **Compliance e auditoria**

### Limitações
- **Não captura:**
  - Tráfego para Amazon DNS server
  - Tráfego para Amazon Time Sync Service
  - DHCP traffic
  - Tráfego para/from link-local addresses (169.254.x.x)
  - Tráfego para VPC router

<br />

## VPC Peering
VPC Peering é usado para estabelecer conectividade entre VPCs e outras estruturas. Conecta duas VPCs que estão em regiões diferentes ou de contas da AWS diferentes.

### Características
- **Conexão one-to-one** entre VPCs
- **Cross-region e cross-account** suportado
- **Não transitivo** (A-B, B-C ≠ A-C)
- **CIDR não pode se sobrepor**
- **Recursos podem se comunicar** como se estivessem na mesma rede

### Limitações
- **Sem transitividade**
- **Sem overlapping CIDR blocks**
- **Sem edge-to-edge routing**
- **DNS resolution** deve ser habilitado em ambas as VPCs

### Configuração
1. **Criar peering connection**
2. **Aceitar na VPC de destino**
3. **Atualizar route tables**
4. **Atualizar security groups/NACLs**

<br />

## VPC Endpoints
VPC Endpoints permite que você se conecte aos serviços da AWS usando uma rede privada ao invés de utilizar uma rede pública da internet. Isso oferece uma segurança aprimorada e menor latência para acessar serviços da AWS.

### Tipos de VPC Endpoints

#### Interface Endpoints (PrivateLink)
- **Elastic Network Interface** com IP privado
- **Suporta a maioria dos serviços** AWS
- **DNS privado** habilitado por padrão
- **Cobrança por hora** e por GB processado
- **Security Groups** aplicáveis

**Serviços suportados:** EC2, S3, DynamoDB, Lambda, SQS, SNS, ECS, etc.

#### Gateway Endpoints
- **Rota na route table**
- **Apenas S3 e DynamoDB**
- **Sem cobrança adicional**
- **Específico de região**

### Políticas de Endpoint
```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

<br />

## Site-to-Site VPN e Direct Connect

### Site-to-Site VPN
- **Virtual Private Gateway (VGW)** na AWS
- **Customer Gateway (CGW)** on-premises
- **Conexão criptografada** via internet
- **Redundância:** 2 túneis IPSec
- **Largura de banda:** Até 1.25 Gbps por túnel

### AWS Direct Connect (DX)
- **Conexão física dedicada**
- **Maior largura de banda** (1 Gbps a 100 Gbps)
- **Menor latência** e mais consistente
- **Redução de custos** para grandes volumes
- **Conexão privada** (não passa pela internet)

### Direct Connect Gateway
- Conecta **múltiplas VPCs** em diferentes regiões
- **Transit Gateway** para roteamento central

**Limitação importante:** VPC Endpoints são apenas para acessar serviços AWS dentro da VPC. Conexões Site-to-Site VPN e Direct Connect não conseguem acessar VPC Endpoints.

<br />

## Transit Gateway
Transit Gateway atua como um hub central que conecta VPCs, conexões VPN e Direct Connect.

### Características
- **Hub-and-spoke model**
- **Cross-region peering**
- **Route tables** para controle de tráfego
- **Multicast** suportado
- **Conexão com AWS RAM** para compartilhamento

### Benefícios
- **Simplifica conectividade**
- **Scaling centralizado**
- **Monitoramento unificado**
- **Roteamento flexível**

<br />

## DNS no VPC
### Configurações DNS
- **enableDnsResolution:** Resolve DNS queries
- **enableDnsHostnames:** Atribui hostnames DNS públicos

### Route 53 Private Hosted Zones
- **DNS privado** para recursos VPC
- **Split-view DNS** (interno vs externo)
- **Cross-VPC resolution** possível

<br />

## Elastic IP
### Características
- **IP público estático**
- **Reassociável** entre instâncias
- **Cobrança quando não usado**
- **Máximo 5 por região** (limite padrão)

### Casos de Uso
- **Failover** rápido
- **Whitelisting** de IPs
- **Serviços que requerem IP fixo**

<br />

## Bastion Hosts
### Conceito
- **Jump server** para acesso a recursos privados
- **Localizado em subnet pública**
- **SSH/RDP** para administração

### Melhores Práticas
- **Minimal security groups**
- **Key-based authentication**
- **Logging de sessões**
- **Auto Scaling** para alta disponibilidade

<br />

## Perguntas e Respostas

### Pergunta 1
**Você precisa permitir que instâncias EC2 em uma subnet privada acessem a internet para downloads de patches, mas não quer que sejam acessíveis da internet. Qual é a solução mais apropriada?**

**Resposta:**
A solução é usar **NAT Gateway**:

1. **Criar NAT Gateway** em uma subnet pública
2. **Atualizar route table** da subnet privada com rota 0.0.0.0/0 → NAT Gateway
3. **Instances mantêm IPs privados** apenas
4. **Tráfego outbound** é permitido, inbound é bloqueado

**Por que NAT Gateway e não NAT Instance?**
- **Managed service** (sem gerenciamento de SO)
- **Alta disponibilidade** automática
- **Melhor performance** (até 45 Gbps)
- **Automatic scaling**

### Pergunta 2
**Qual é a diferença entre Security Groups e NACLs e quando usar cada um?**

**Resposta:**
**Security Groups:**
- **Nível de instância** (firewall de instância)
- **Stateful:** Se permitir inbound, outbound é automático
- **Apenas Allow rules**
- **Primeira linha de defesa**

**Network ACLs:**
- **Nível de subnet** (firewall de subnet)
- **Stateless:** Deve configurar inbound E outbound
- **Allow e Deny rules**
- **Segunda linha de defesa**

**Quando usar:**
- **Security Groups:** Controle básico de acesso por instância
- **NACLs:** Controle adicional por subnet, regras de deny específicas

**Exemplo prático:**
- **Security Group:** Permitir HTTP (80) de qualquer lugar
- **NACL:** Bloquear IP específico malicioso

### Pergunta 3
**Como estabelecer comunicação entre duas VPCs em regiões diferentes?**

**Resposta:**
Para comunicação entre VPCs cross-region, use **VPC Peering**:

**Passos:**
1. **Criar VPC Peering Connection** na VPC origem
2. **Aceitar connection** na VPC destino (outra região)
3. **Atualizar Route Tables** em ambas as VPCs
4. **Configurar Security Groups** para permitir tráfego
5. **Habilitar DNS resolution** se necessário

**Exemplo de rotas:**
- **VPC A (us-east-1):** 10.0.0.0/16 → 172.16.0.0/16 via pcx-xxx
- **VPC B (us-west-2):** 172.16.0.0/16 → 10.0.0.0/16 via pcx-xxx

**Alternativa:** Transit Gateway com **cross-region peering**

### Pergunta 4
**Você quer que suas instâncias EC2 acessem S3 sem passar pela internet. Como configurar?**

**Resposta:**
Use **VPC Endpoint para S3**:

**Opções:**
1. **Gateway Endpoint** (recomendado para S3):
   - Sem custo adicional
   - Adiciona rota na route table
   - Apenas S3 e DynamoDB

2. **Interface Endpoint:**
   - Cria ENI com IP privado
   - Cobrança por hora + dados
   - Mais controle com security groups

**Configuração Gateway Endpoint:**
1. **Criar VPC Endpoint** tipo Gateway para S3
2. **Selecionar route tables** que devem ter acesso
3. **Configurar política** (opcional)
4. **Instances acessam** s3.amazonaws.com via rota privada

### Pergunta 5
**Como monitorar e fazer troubleshooting de conectividade na VPC?**

**Resposta:**
Use **VPC Flow Logs** para monitoramento:

**Configuração:**
1. **Habilitar Flow Logs** no nível desejado (VPC/Subnet/ENI)
2. **Escolher destino:** CloudWatch Logs ou S3
3. **Analisar logs** para padrões de tráfego

**Formato dos logs:**
```
2 123456789012 eni-abc123de 10.0.0.4 10.0.0.5 20641 22 6 1 52 1418530010 1418530070 ACCEPT OK
```

**Campos importantes:**
- **srcaddr/dstaddr:** IPs origem/destino
- **srcport/dstport:** Portas origem/destino  
- **action:** ACCEPT ou REJECT
- **protocol:** TCP (6), UDP (17), ICMP (1)

**Casos de uso:**
- **REJECT entries:** Tráfego bloqueado por Security Groups/NACLs
- **Patterns de tráfego:** Identificar comunicação não autorizada
- **Baseline de performance:** Volumes normais vs anômalos

### Pergunta 6
**Explique os diferentes tipos de IPs na AWS VPC.**

**Resposta:**
**Tipos de IPs:**

**Private IP:**
- **Atribuído automaticamente** à instância
- **Permanece** durante stop/start
- **Comunicação interna** VPC

**Public IP:**
- **Dinâmico:** Muda a cada stop/start  
- **Atribuído** se subnet tiver "Auto-assign public IP"
- **Acesso direto** à internet

**Elastic IP:**
- **IP público estático**
- **Persiste** entre stop/start
- **Reassociável** entre instâncias
- **Cobrança** quando não usado

**IPv6:**
- **Opcional** na VPC
- **Globalmente único**
- **Internet routable** por padrão

**Exemplo prático:**
- **Web servers:** Elastic IP para DNS fixo
- **App servers:** Apenas private IP + NAT Gateway
- **Database servers:** Apenas private IP

### Pergunta 7
**Como configurar alta disponibilidade para NAT Gateway?**

**Resposta:**
Para HA com NAT Gateway:

**Arquitetura Multi-AZ:**
1. **Criar NAT Gateway** em cada AZ onde há subnets privadas
2. **Route tables específicas** por AZ
3. **Falha de AZ** não afeta outras AZs

**Exemplo:**
- **AZ-1a:** NAT Gateway A, Route Table A
- **AZ-1b:** NAT Gateway B, Route Table B  
- **AZ-1c:** NAT Gateway C, Route Table C

**Vantagens:**
- **Redundância** automática
- **Performance** distribuída
- **Isolation de falhas**

**Custos:**
- **Multiple NAT Gateways** = maior custo
- **Trade-off** entre HA e economia

### Pergunta 8
**O que acontece se duas VPCs tiverem CIDR blocks sobrepostos e você tentar fazer peering?**

**Resposta:**
**VPC Peering com CIDR sobreposto NÃO é possível.**

**Problema:**
- **VPC A:** 10.0.0.0/16
- **VPC B:** 10.0.0.0/16  
- **Conflito:** Mesmo range de IPs

**Soluções:**
1. **Reconfigurar CIDR** de uma das VPCs (se possível)
2. **NAT Gateway** com port forwarding
3. **Application Load Balancer** como proxy
4. **Transit Gateway** com roteamento específico

**Prevenção:**
- **Planejar CIDR blocks** antes da criação
- **Usar ranges não sobrepostos:**
  - VPC A: 10.0.0.0/16
  - VPC B: 172.16.0.0/16
  - VPC C: 192.168.0.0/16

<br />

## :books: Referências
Para uma compreensão mais profunda sobre Amazon VPC recomendo a leitura da documentação oficial:

- [O que é Amazon VPC?](https://docs.aws.amazon.com/pt_br/vpc/latest/userguide/what-is-amazon-vpc.html)
- [Conceitos básicos da VPC](https://docs.aws.amazon.com/pt_br/vpc/latest/userguide/configure-your-vpc.html#vpc-subnet-basics)
- [Compare grupos de segurança e Network ACLs](https://docs.aws.amazon.com/pt_br/vpc/latest/userguide/VPC_Security.html)
- [Configurar tabelas de rotas](https://docs.aws.amazon.com/pt_br/vpc/latest/userguide/VPC_Route_Tables.html)
- [Monitoramento da sua VPC](https://docs.aws.amazon.com/pt_br/vpc/latest/userguide/monitoring.html)
- [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
- [VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/)
- [VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/)
- [AWS Transit Gateway](https://docs.aws.amazon.com/transit-gateway/)

<br />

---
