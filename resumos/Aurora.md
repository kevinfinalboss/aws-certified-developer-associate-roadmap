# Amazon Aurora

<p align="center">
  <img src="https://dbdb.io/media/logos/amazon-aurora_IGQMXko.png" alt="amazon-aurora-logo" style="height:150px; width:150px;" />
  <br />
  <h1 align="center">
    Amazon Aurora
  </h1>
</p>

---

## 📌 Índice

- [Introdução](#introdução)
- [Compatibilidade e Performance](#compatibilidade-e-performance)
- [Replicação e Alta Disponibilidade](#replicação-e-alta-disponibilidade)
- [Endpoints de Aurora](#endpoints-de-aurora)
- [Escalabilidade e Armazenamento](#escalabilidade-e-armazenamento)
- [Failover](#failover)
- [Custo e Free Tier](#custo-e-free-tier)
- [Principais Diferenças Aurora x RDS](#principais-diferenças-aurora-x-rds)
- [Considerações para Prova](#considerações-para-prova)
- [Questões Frequentes de Prova](#questões-frequentes-de-prova)
- [📚 Referências](#-referências)

---

## Introdução

O **Amazon Aurora** é um banco de dados relacional gerenciado pela AWS que oferece compatibilidade total com MySQL e PostgreSQL, porém com performance e disponibilidade superiores, pois é uma tecnologia proprietária e otimizada para a nuvem AWS.

---

## Compatibilidade e Performance

- Compatível com **MySQL 5.6, 5.7 e 8.0** e com **PostgreSQL 9.6, 10, 11, 12, 13**.
- Performance até **5x superior ao MySQL** tradicional no RDS.
- Performance até **3x superior ao PostgreSQL** tradicional no RDS.
- Essa performance é obtida com otimizações no armazenamento distribuído e replicação.

---

## Replicação e Alta Disponibilidade

- Suporta até **15 réplicas de leitura**, muito mais do que o limite de 5 réplicas no MySQL padrão.
- Réplicas são atualizadas quase em tempo real, com replicação **quase síncrona**.
- Os dados são armazenados de forma redundante com **6 cópias em 3 zonas de disponibilidade (AZs)**.
- Isso garante alta durabilidade e tolerância a falhas.

---

## Endpoints de Aurora

Aurora utiliza um sistema de **endpoints distintos** para facilitar a conexão e otimizar o tráfego:

### Endpoint de Cluster (Cluster Endpoint)

- Aponta para a **instância principal (writer)**, usada para operações de escrita.
- É o endpoint padrão para conexões que fazem inserts, updates e deletes.
- Se ocorrer failover, o cluster endpoint automaticamente aponta para a nova instância principal.

### Endpoint de Leitura (Reader Endpoint)

- Distribui o tráfego de leitura entre todas as réplicas de leitura disponíveis (até 15).
- Usado para consultas e operações somente leitura.
- Ajuda a balancear carga e melhorar desempenho em ambientes com alta demanda de leitura.

### Endpoints de Instância (Instance Endpoints)

- Apontam diretamente para instâncias específicas, seja principal ou réplicas.
- Útil para conexões específicas que requerem apontar para uma instância fixa, por exemplo, para manutenção ou monitoramento.

---

## Escalabilidade e Armazenamento

- O armazenamento do cluster Aurora é **auto escalável** e pode crescer automaticamente conforme a necessidade, até um limite máximo de **128 tebibytes (TiB)**.
- Essa escalabilidade elimina a necessidade de provisionamento manual de armazenamento.
- Aurora gerencia a capacidade sem interrupção das operações.

---

## Failover

- O failover no Aurora é **quase instantâneo**, graças à arquitetura nativa para nuvem e replicação rápida.
- Em caso de falha da instância primária, uma das réplicas de leitura é promovida automaticamente a instância principal.
- Essa promoção acontece em segundos, garantindo alta disponibilidade.
- Não há necessidade de intervenção manual para failover.

---

## Custo e Free Tier

- O Aurora possui custo **aproximadamente 20% a 30% maior que o RDS MySQL/PostgreSQL padrão** devido à performance e disponibilidade superiores.
- Não existe **camada gratuita (free tier)** para Aurora.
- O custo depende do tipo e número de instâncias, armazenamento utilizado, I/O e transferência de dados.

---

## Principais Diferenças Aurora x RDS

| Característica          | Aurora                                    | RDS MySQL/PostgreSQL                    |
|------------------------|------------------------------------------|----------------------------------------|
| **Performance**        | Até 5x (MySQL) e 3x (PostgreSQL) maior  | Performance padrão dos bancos opensource|
| **Número de Réplicas** | Até 15 réplicas de leitura                | Até 5 réplicas                         |
| **Replicação**         | Quase síncrona e rápida                   | Replicação assíncrona mais lenta       |
| **Failover**           | Automático e quase instantâneo            | Pode levar mais tempo e ser manual     |
| **Armazenamento**      | Escalável até 128 TiB automaticamente     | Limitado e precisa ajuste manual       |
| **Alta Disponibilidade**| 6 cópias em 3 AZs                         | 2 cópias em 2 AZs (Multi-AZ opcional) |
| **Free Tier**          | Não possui                               | Possui opção gratuita para RDS         |
| **Compatibilidade**    | MySQL e PostgreSQL                       | MySQL, PostgreSQL, Oracle, SQL Server  |

---

## Considerações para Prova

- Entenda que Aurora é uma tecnologia AWS proprietária, não open-source.
- Saiba que Aurora suporta até 15 réplicas e replicação quase síncrona, melhorando performance e disponibilidade.
- Conheça os tipos de endpoints: cluster (escrita), leitura (balanceamento) e instância (conexão direta).
- Falha da instância principal dispara failover rápido e automático para uma réplica.
- O armazenamento é elástico e pode crescer até 128 TiB sem downtime.
- Aurora custa mais que RDS, mas oferece vantagens em performance e alta disponibilidade.
- Aurora não possui free tier, diferente do RDS.
- Familiarize-se com os casos de uso e vantagens para responder questões de prova focadas em escalabilidade, replicação e disponibilidade.

---

## Questões Frequentes de Prova

### Questão 1  
**Qual a diferença principal entre os endpoints do Amazon Aurora?**  
✅ *Resposta:*  
- O **endpoint de cluster** aponta para a instância principal para escrita.  
- O **endpoint de leitura** distribui as consultas entre as réplicas de leitura.  
- Os **endpoints de instância** conectam diretamente a instâncias específicas.

---

### Questão 2  
**Quantas réplicas de leitura Aurora suporta em comparação ao MySQL no RDS?**  
✅ *Resposta:* Aurora suporta até **15 réplicas de leitura**, enquanto MySQL no RDS suporta até 5.

---

### Questão 3  
**Como o Amazon Aurora garante alta disponibilidade dos dados?**  
✅ *Resposta:* Armazena **6 cópias dos dados distribuídas em 3 zonas de disponibilidade**, com replicação quase síncrona e failover automático.

---

### Questão 4  
**Qual a vantagem do failover no Aurora em comparação ao RDS padrão?**  
✅ *Resposta:* O failover no Aurora é **quase instantâneo e automático**, enquanto no RDS tradicional pode ser mais lento e requer intervenção.

---

### Questão 5  
**O que acontece quando o armazenamento do cluster Aurora chega ao limite atual?**  
✅ *Resposta:* O armazenamento **auto escala automaticamente** até o limite máximo de 128 TiB, sem downtime.

---

### Questão 6  
**Aurora possui camada gratuita (free tier)?**  
✅ *Resposta:* **Não**, Aurora não possui free tier, diferente do RDS.

---

## 📚 Referências

- [Amazon Aurora – Documentação Oficial AWS](https://docs.aws.amazon.com/pt_br/AmazonRDS/latest/AuroraUserGuide/CHAP_AuroraOverview.html)  
- [Aurora Endpoints](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.Endpoints.html)  
- [Comparação Aurora e RDS](https://aws.amazon.com/rds/aurora/faqs/)
