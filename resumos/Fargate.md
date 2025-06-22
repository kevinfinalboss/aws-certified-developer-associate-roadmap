<p align="center">
  <img src="https://miro.medium.com/v2/resize:fit:512/1*gsV86jnibrpuRvMQAEbZhQ.png" alt="aws-fargate-icon" style="height:150px; width:150px;" /> 
  <br />
  <h1 align="center">
    AWS Fargate
  </h1>
</p>

<br />

## Índice

- [O que é AWS Fargate?](#o-que-é-aws-fargate)
- [Como funciona o Fargate](#como-funciona-o-fargate)
- [Fargate vs EC2](#fargate-vs-ec2)
- [Tipos de launch](#tipos-de-launch)
- [Configurações de CPU e memória](#configurações-de-cpu-e-memória)
- [Benefícios do Fargate](#benefícios-do-fargate)
- [Integração com outros serviços AWS](#integração-com-outros-serviços-aws)
- [Rede e segurança](#rede-e-segurança)
- [Limitações e custos](#limitações-e-custos)
- [Questões para prova AWS Developer Associate](#questões-para-prova-aws-developer-associate)
- [Referências](#referências)

<br />

---

## O que é AWS Fargate?

AWS Fargate é um **mecanismo de computação serverless** para containers que permite executar containers sem gerenciar servidores ou clusters. É compatível com **Amazon ECS** (Elastic Container Service) e **Amazon EKS** (Elastic Kubernetes Service).

---

## Como funciona o Fargate

O Fargate abstrai a infraestrutura subjacente, permitindo que você foque apenas na definição de containers, CPU, memória e rede. A AWS gerencia automaticamente o provisionamento, patches e escalabilidade da infraestrutura.

---

## Fargate vs EC2

**Fargate:**
- Serverless, sem gerenciamento de instâncias
- Cobrança por recursos utilizados (CPU/memória)
- Menor controle sobre infraestrutura
- Ideal para workloads variáveis

**EC2:**
- Controle total sobre instâncias
- Cobrança por instância completa
- Maior flexibilidade de configuração
- Melhor para workloads previsíveis

---

## Tipos de launch

- **Fargate:** Para containers serverless
- **EC2:** Para containers em instâncias gerenciadas
- **External:** Para containers executados fora da AWS (ECS Anywhere)

---

## Configurações de CPU e memória

**Combinações válidas:**
- 0.25 vCPU: 512 MB a 2 GB
- 0.5 vCPU: 1 GB a 4 GB
- 1 vCPU: 2 GB a 8 GB
- 2 vCPU: 4 GB a 16 GB
- 4 vCPU: 8 GB a 30 GB

---

## Benefícios do Fargate

- **Serverless:** Sem gerenciamento de infraestrutura
- **Escalabilidade automática:** Baseada na demanda
- **Segurança:** Isolamento completo por task
- **Simplicidade:** Foco apenas no container
- **Flexibilidade:** Suporte a ECS e EKS

---

## Integração com outros serviços AWS

- **Amazon ECS:** Para orquestração de containers
- **Amazon EKS:** Para Kubernetes gerenciado
- **Application Load Balancer:** Para distribuição de tráfego
- **AWS VPC:** Para configuração de rede
- **CloudWatch:** Para monitoramento e logs

---

## Rede e segurança

- **Task networking:** Cada task recebe ENI própria
- **Security Groups:** Aplicados no nível da task
- **Subnets:** Tasks executam em subnets específicas
- **IAM Roles:** Para permissões granulares
- **Secrets Manager:** Para gerenciar credenciais

---

## Limitações e custos

**Limitações:**
- Não suporta volumes persistentes (apenas efêmero)
- Sem acesso privilegiado ao host
- Limitações de configuração de CPU/memória

**Custos:**
- Cobrança por vCPU-segundo e GB-segundo
- Mais caro que EC2 para workloads constantes
- Sem custos de infraestrutura ociosa

---

## Questões para prova AWS Developer Associate

### 1. O que é AWS Fargate e qual sua principal vantagem?

**Resposta:**  
AWS Fargate é um mecanismo de computação serverless para containers que elimina a necessidade de gerenciar servidores ou clusters. Sua principal vantagem é permitir foco total na aplicação sem se preocupar com infraestrutura.

**Explicação:**  
Fargate abstrai toda a complexidade de gerenciamento de infraestrutura, permitindo que desenvolvedores se concentrem apenas no código.

---

### 2. Quais serviços AWS são compatíveis com Fargate?

**Resposta:**  
Amazon ECS (Elastic Container Service) e Amazon EKS (Elastic Kubernetes Service).

**Explicação:**  
Fargate pode ser usado como launch type tanto em ECS quanto em EKS para executar containers sem gerenciar instâncias EC2.

---

### 3. Qual é a diferença principal entre Fargate e EC2 launch types?

**Resposta:**  
Fargate é serverless (sem gerenciamento de instâncias) enquanto EC2 requer gerenciamento de instâncias. Fargate cobra por recursos utilizados, EC2 cobra por instância completa.

**Explicação:**  
A escolha depende do nível de controle desejado e padrão de uso da aplicação.

---

### 4. Como funciona a rede no Fargate?

**Resposta:**  
Cada task no Fargate recebe sua própria ENI (Elastic Network Interface) com IP privado, e Security Groups são aplicados diretamente na task, não na instância.

**Explicação:**  
Isso proporciona isolamento de rede superior e controle granular de segurança por task.

---

### 5. Quais são as configurações válidas de CPU e memória no Fargate?

**Resposta:**  
As combinações seguem proporções específicas: 0.25 vCPU (512MB-2GB), 0.5 vCPU (1-4GB), 1 vCPU (2-8GB), 2 vCPU (4-16GB), 4 vCPU (8-30GB).

**Explicação:**  
Essas combinações são pré-definidas pela AWS para otimizar performance e custos.

---

### 6. Como o Fargate lida com armazenamento?

**Resposta:**  
Fargate oferece apenas armazenamento efêmero (temporário) de até 20GB. Não suporta volumes persistentes como EBS.

**Explicação:**  
Para dados persistentes, é necessário usar serviços externos como S3, EFS ou RDS.

---

### 7. Como funciona a cobrança no Fargate?

**Resposta:**  
Cobrança baseada em vCPU-segundo e GB-segundo utilizados, iniciando quando a task inicia e parando quando termina.

**Explicação:**  
Você paga apenas pelos recursos que realmente utiliza, sem custos de infraestrutura ociosa.

---

### 8. Qual é a vantagem de usar Fargate com Application Load Balancer?

**Resposta:**  
ALB pode distribuir tráfego diretamente para tasks Fargate usando target groups, permitindo alta disponibilidade e balanceamento de carga automático.

**Explicação:**  
Como cada task tem sua própria ENI, o ALB pode rotear tráfego diretamente para elas.

---

### 9. Como configurar logs no Fargate?

**Resposta:**  
Usando CloudWatch Logs driver na definição da task, especificando log group e região. Logs são enviados automaticamente para CloudWatch.

**Explicação:**  
Fargate integra nativamente com CloudWatch para coleta e armazenamento de logs.

---

### 10. Quando escolher Fargate ao invés de EC2 para containers?

**Resposta:**  
Escolha Fargate para workloads variáveis, aplicações serverless, quando quiser reduzir overhead operacional ou para desenvolvimento/teste onde simplicidade é prioridade.

**Explicação:**  
Fargate é ideal quando o foco é na aplicação e não na infraestrutura, especialmente para cargas de trabalho imprevisíveis.

---

## Referências

- [O que é AWS Fargate?](https://aws.amazon.com/fargate/)  
- [Documentação oficial do AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/userguide/what-is-fargate.html)  
- [Amazon ECS com Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)  
- [Amazon EKS com Fargate](https://docs.aws.amazon.com/eks/latest/userguide/fargate.html)

---
