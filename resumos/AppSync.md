<p align="center">
  <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_App-Integration/Arch_64/Arch_AWS-AppSync_64@5x.png" alt="aws-appsync-icon" style="height:150px; width:150px;" /> 
  <br />
  <h1 align="center">
    AWS AppSync
  </h1>
</p>

<br />

## Índice

- [O que é AWS AppSync?](#o-que-é-aws-appsync)
- [Componentes principais](#componentes-principais)
- [GraphQL e AppSync](#graphql-e-appsync)
- [Resolvers](#resolvers)
- [Data Sources](#data-sources)
- [Autenticação e autorização](#autenticação-e-autorização)
- [Subscriptions em tempo real](#subscriptions-em-tempo-real)
- [Caching](#caching)
- [Integração com outros serviços AWS](#integração-com-outros-serviços-aws)
- [Limitações e custos](#limitações-e-custos)
- [Questões para prova AWS Developer Associate](#questões-para-prova-aws-developer-associate)
- [Referências](#referências)

<br />

---

## O que é AWS AppSync?

AWS AppSync é um **serviço gerenciado de GraphQL** que simplifica o desenvolvimento de aplicações permitindo que você crie APIs GraphQL seguras e em tempo real. Conecta automaticamente suas aplicações a múltiplas fontes de dados como DynamoDB, Lambda, Elasticsearch e HTTP APIs.

---

## Componentes principais

**GraphQL Schema:** Define a estrutura da API e operações disponíveis  
**Resolvers:** Conectam campos do schema às fontes de dados  
**Data Sources:** Conectores para DynamoDB, Lambda, RDS, HTTP endpoints  
**Subscriptions:** Comunicação em tempo real via WebSockets  
**Caching:** Cache automático para melhorar performance

---

## GraphQL e AppSync

**Características do GraphQL:**
- Query language para APIs
- Permite solicitar dados específicos (evita over/under-fetching)
- Single endpoint para todas operações
- Type system forte
- Introspection automática

**AppSync adiciona:**
- Infraestrutura gerenciada
- Resolvers automáticos
- Subscriptions em tempo real
- Autenticação integrada
- Offline sync capabilities

---

## Resolvers

**Tipos de Resolvers:**

**Unit Resolvers:**
- Conectam diretamente um campo a uma data source
- Mais simples e performáticos
- Recomendados para operações básicas

**Pipeline Resolvers:**
- Executam múltiplas functions em sequência
- Permitem lógica complexa e transformações
- Úteis para operações que envolvem múltiplas data sources

**JavaScript Resolvers:**
- Permitem lógica customizada em JavaScript
- Maior flexibilidade para transformações
- Suporte a bibliotecas JavaScript

---

## Data Sources

**DynamoDB:**
- Conexão direta com tabelas
- Resolvers automáticos para CRUD
- Suporte a batch operations

**Lambda:**
- Lógica customizada serverless
- Útil para business logic complexa
- Pode conectar a qualquer sistema

**Elasticsearch:**
- Busca e analytics
- Queries complexas e agregações
- Full-text search

**RDS (Aurora Serverless):**
- Bancos relacionais
- Queries SQL através de Data API
- Transações ACID

**HTTP Data Sources:**
- APIs REST existentes
- Integração com sistemas legados
- Transformação de dados

---

## Autenticação e autorização

**Métodos de autenticação:**

**API Key:**
- Para desenvolvimento e testing
- Não recomendado para produção
- Controle básico de acesso

**Amazon Cognito User Pools:**
- Autenticação de usuários
- JWT tokens
- Grupos e roles

**AWS IAM:**
- Para aplicações AWS
- Roles e policies
- Integração com outros serviços

**OIDC (OpenID Connect):**
- Provedores externos
- Tokens JWT
- Federação de identidade

**Autorização granular:**
- Field-level security
- @auth directives no schema
- Dynamic authorization

---

## Subscriptions em tempo real

**Características:**
- WebSockets gerenciados
- Atualizações automáticas
- Filtros em subscriptions
- Scaling automático

**Casos de uso:**
- Chat applications
- Live dashboards
- Real-time notifications
- Collaborative editing

**Implementação:**
- Triggadas por mutations
- Filtros baseados em argumentos
- Multiplexing automático

---

## Caching

**Tipos de cache:**

**Server-side caching:**
- Cache automático por TTL
- Invalidação manual
- Melhor performance

**Client-side caching:**
- Apollo Client integration
- Normalized cache
- Offline capabilities

**Configuração:**
- TTL por resolver
- Cache keys customizadas
- Invalidation patterns

---

## Integração com outros serviços AWS

**AWS Amplify:**
- Geração automática de código client
- Deploy integrado
- CLI tools

**Amazon CloudWatch:**
- Métricas e logs
- Monitoramento de performance
- Alertas automáticos

**AWS X-Ray:**
- Tracing distribuído
- Análise de latência
- Debug de resolvers

**Amazon CloudFront:**
- Edge caching
- Distribuição global
- Reduced latency

---

## Limitações e custos

**Limitações:**
- Maximum de 100 data sources por API
- Schema size limit (1MB)
- Query complexity limits
- Rate limiting per connection

**Custos:**
- Por query/mutation/subscription executada
- Data transfer charges
- Real-time subscriptions têm custo adicional
- Caching opcional com custo extra

---

## Questões para prova AWS Developer Associate

### 1. O que é AWS AppSync e qual seu principal benefício?

**Resposta:**  
AWS AppSync é um serviço gerenciado de GraphQL que permite criar APIs seguras conectando múltiplas fontes de dados. O principal benefício é simplificar o desenvolvimento de aplicações com dados em tempo real.

**Explicação:**  
AppSync elimina a necessidade de gerenciar infraestrutura GraphQL e oferece features como subscriptions automáticas.

---

### 2. Qual a diferença entre Unit Resolvers e Pipeline Resolvers?

**Resposta:**  
Unit Resolvers conectam diretamente um campo a uma data source e são mais simples. Pipeline Resolvers executam múltiplas functions em sequência permitindo lógica mais complexa.

**Explicação:**  
Pipeline Resolvers são úteis quando você precisa combinar dados de múltiplas sources ou aplicar transformações complexas.

---

### 3. Quais data sources o AppSync suporta nativamente?

**Resposta:**  
DynamoDB, Lambda, Elasticsearch, RDS (Aurora Serverless), HTTP endpoints e None (para resolvers locais).

**Explicação:**  
Cada data source tem conectores otimizados que facilitam a integração sem código adicional.

---

### 4. Como funciona a autenticação no AppSync?

**Resposta:**  
AppSync suporta API Key, Cognito User Pools, IAM e OIDC. Você pode usar múltiplos métodos simultaneamente e aplicar autorização granular com @auth directives.

**Explicação:**  
A flexibilidade de autenticação permite diferentes cenários desde desenvolvimento até produção enterprise.

---

### 5. O que são subscriptions no AppSync e como funcionam?

**Resposta:**  
Subscriptions permitem comunicação em tempo real via WebSockets. São triggadas por mutations e permitem que clientes recebam atualizações automáticas quando dados mudam.

**Explicação:**  
AppSync gerencia toda infraestrutura WebSocket e scaling automático para subscriptions.

---

### 6. Como o caching funciona no AppSync?

**Resposta:**  
AppSync oferece server-side caching com TTL configurável por resolver e integração com client-side caching (Apollo Client). Também suporta edge caching via CloudFront.

**Explicação:**  
O caching melhora performance reduzindo calls desnecessárias às data sources.

---

### 7. Qual a vantagem do GraphQL sobre REST APIs?

**Resposta:**  
GraphQL permite solicitar dados específicos evitando over/under-fetching, tem single endpoint, type system forte e introspection automática.

**Explicação:**  
Isso resulta em aplicações mais eficientes e desenvolvimento mais rápido.

---

### 8. Como implementar business logic complexa no AppSync?

**Resposta:**  
Usando Lambda data sources para lógica customizada, Pipeline Resolvers para workflows complexos, ou JavaScript Resolvers para transformações inline.

**Explicação:**  
A escolha depende da complexidade e performance requirements da aplicação.

---

### 9. Como monitorar performance de uma API AppSync?

**Resposta:**  
Usando CloudWatch para métricas e logs, X-Ray para tracing distribuído, e métricas built-in do AppSync como latência por resolver.

**Explicação:**  
O monitoramento é essencial para identificar gargalos e otimizar performance.

---

### 10. Quando usar AppSync vs API Gateway + Lambda?

**Resposta:**  
Use AppSync para aplicações que precisam de real-time features, GraphQL benefits, múltiplas data sources, ou desenvolvimento rápido. Use API Gateway + Lambda para APIs REST simples ou maior controle sobre infraestrutura.

**Explicação:**  
AppSync é ideal para aplicações modernas com requirements de real-time e múltiplas fontes de dados.

---

## Referências

- [O que é AWS AppSync?](https://aws.amazon.com/appsync/)  
- [Documentação oficial do AWS AppSync](https://docs.aws.amazon.com/appsync/)  
- [GraphQL Schema Design](https://docs.aws.amazon.com/appsync/latest/devguide/designing-a-graphql-api.html)  
- [AppSync Resolvers](https://docs.aws.amazon.com/appsync/latest/devguide/resolver-context-reference.html)  
- [Real-time Subscriptions](https://docs.aws.amazon.com/appsync/latest/devguide/real-time-data.html)

---
