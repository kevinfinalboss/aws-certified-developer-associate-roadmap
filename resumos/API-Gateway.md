<p align="center">
	<img src="./img/aws-icons/aws-API-Gateway.png" alt="aws-api-gateway-icon" style="height:150px; width:150px;" /> 
  <br />
	<h1 align="center">
    Amazon API Gateway
  </h1>
</p>

---

## 📌 Índice

- [Introdução](#introdução)
- [Tipos de API](#tipos-de-api)
- [Estágios de Implantação](#estágios-de-implantação)
- [Caching de Respostas](#caching-de-respostas)
- [Throttling e Limites](#throttling-e-limites)
- [Planos de Uso e Chaves de API](#planos-de-uso-e-chaves-de-api)
- [Métricas e Monitoramento](#métricas-e-monitoramento)
- [Erros Comuns](#erros-comuns)
- [Segurança](#segurança)
- [Integrações](#integrações)
- [Exemplos de Questões de Prova](#exemplos-de-questões-de-prova)
- [Referências](#referências)

---

## 📌 Introdução

O **Amazon API Gateway** é um serviço totalmente gerenciado da AWS que permite criar, publicar, manter, monitorar e proteger APIs REST, HTTP e WebSocket em escala.

Ele serve como **frente de entrada única** (Gateway) para chamadas de API destinadas a backends, como funções Lambda, serviços AWS ou endpoints HTTP externos.

Principais recursos:

- Controle de tráfego
- Monitoramento
- Autenticação e Autorização
- Versionamento e implantação de APIs
- Cache de respostas

---

## 📌 Tipos de API

| Tipo de API | Características |
|---|---|
| **REST API** | Mais recursos, suporte a transformação de payload, validação, caching, authorizers avançados. Maior custo e latência. |
| **HTTP API** | Mais leve, rápida e barata. Ideal para integrações simples com Lambda e serviços HTTP externos. |
| **WebSocket API** | Comunicação bidirecional em tempo real. Exemplo: chats, jogos e notificações. |

> **⚠️ Atenção:** Na prova, caem questões comparando REST API com HTTP API (exemplo: quando usar cada uma, custo, cache, CORS, etc).

---

## 📌 Estágios de Implantação

- Um **Stage** representa uma versão da sua API (ex: `dev`, `test`, `prod`).
- Alterações só têm efeito após um **Deploy** para o Stage.
- Cada Stage pode ter:
  - **Variáveis de Stage** (ex: configurações de ambiente)
  - **Configurações de Logs**
  - **Throttling**
  - **Cache**
  - **Métricas customizadas**

---

## 📌 Caching de Respostas

| Recurso | Valor |
|---|---|
| **TTL Padrão** | 300 segundos (5 minutos) |
| **Intervalo Permitido** | 0 a 3600 segundos (1 hora) |
| **Tamanhos de Cache** | De 0.5GB até 237GB |
| **Criptografia** | Suportada |
| **Configuração por** | Stage e por Método |

**Vantagem:** Reduz chamadas ao backend.

### Invalidação de Cache

- Manual via Console AWS.
- Via Header HTTP:
*(Necessário permissão específica no IAM para invalidação programática.)*

---

## 📌 Throttling e Limites

| Tipo | Valor |
|---|---|
| **Limite por Conta (Padrão AWS)** | 10.000 requisições por segundo (pode ser aumentado via suporte AWS) |
| **Throttling por Stage** | Definido manualmente |
| **Throttling por Plano de Uso (Usage Plan)** | Controle por cliente/API Key |

> Se ultrapassar os limites → **Erro 429 (Too Many Requests)**

---

## 📌 Planos de Uso e Chaves de API

- **API Keys:** Identificam o cliente.
- **Usage Plans (Plano de Uso):**
  - Define **Rate Limit** (ex: X requisições por segundo)
  - **Quota** (ex: Y requisições por dia/mês)
  - **Associação de Stages e Métodos específicos**

> **Importante:** Apenas APIs REST suportam **Usage Plans + API Keys** (não HTTP API).

---

## 📌 Métricas e Monitoramento

### 📊 Principais métricas (via Amazon CloudWatch):

| Métrica | Descrição |
|---|---|
| **Count** | Número total de requisições |
| **4XXError** | Taxa de erros causados pelo cliente |
| **5XXError** | Taxa de erros causados pelo servidor |
| **Latency** | Tempo total de resposta |
| **IntegrationLatency** | Tempo de processamento apenas no backend (ex: Lambda) |
| **CacheHitCount / CacheMissCount** | Eficiência do cache |
| **ThrottledRequests** | Número de requisições bloqueadas por throttling |

### 📋 Log de Execução

- Envio opcional de **Access Logs** e **Execution Logs** para o CloudWatch Logs.

---

## 📌 Erros Comuns

| Código | Significado |
|---|---|
| 400 | Bad Request |
| 403 | Access Denied (pode ser por WAF ou IAM) |
| 429 | Too Many Requests (throttling/limite) |
| 500 | Internal Server Error |
| 502 | Bad Gateway (ex: problema de integração com Lambda) |
| 503 | Service Unavailable |
| 504 | Integration Timeout (Timeout de 29 segundos no backend) |

---

## 📌 Segurança

| Opção | Descrição |
|---|---|
| **IAM Roles/Policies** | Controle para usuários/funções dentro da AWS |
| **Resource Policies** | Controle de acesso entre contas AWS ou por IP |
| **Cognito User Pools** | Autenticação baseada em usuários finais (OpenID Connect) |
| **Lambda Authorizer (Custom Authorizer)** | Lógica de autenticação personalizada (ex: JWT, tokens externos) |
| **API Keys com Usage Plans** | Controle e identificação de clientes externos |

---

## 📌 Integrações

| Tipo | Exemplo de Uso |
|---|---|
| **Lambda Function** | Backend serverless |
| **AWS Service Proxy** | Acesso direto a serviços AWS (ex: DynamoDB, SNS, SQS) |
| **HTTP Backend** | Chamadas para endpoints externos |
| **Mock Integration** | Retorno de respostas fixas para testes |

---

## 📌 Exemplos de Questões de Prova

### Questão 1:

**Uma API REST no API Gateway está com alta latência de resposta. Onde analisar?**

- A) CloudTrail  
- B) IntegrationLatency no CloudWatch  
- C) Resource Policy  
- D) Cache Hit Ratio  

✅ **Resposta:** B

**Explicação:**  
`IntegrationLatency` mostra o tempo gasto no backend (exemplo: Lambda ou outro serviço).

---

### Questão 2:

**Sua API REST está retornando um erro 504. O que provavelmente está acontecendo?**

- A) Problema na autenticação IAM  
- B) Backend demorou mais de 29 segundos para responder  
- C) Problema com chave de API inválida  
- D) Cache expirado  

✅ **Resposta:** B

**Explicação:**  
504 indica timeout na integração (limite de 29 segundos).

---

### Questão 3:

**Qual cenário melhor justifica o uso de uma HTTP API ao invés de uma REST API?**

- A) Precisa de Cache de Resposta  
- B) Precisa de transformação de payload  
- C) Requer custo menor e baixa latência  
- D) Precisa de Usage Plans  

✅ **Resposta:** C

**Explicação:**  
HTTP APIs são mais simples e baratas, porém com menos recursos.

---

### Questão 4:

**Como limitar um cliente a 1000 requisições diárias?**

- A) IAM Policy  
- B) API Gateway Usage Plan + API Key  
- C) Lambda Authorizer  
- D) Configuração de CORS  

✅ **Resposta:** B

**Explicação:**  
Usage Plans são projetados exatamente para este tipo de controle.

---

## 📌 Referências

- [AWS API Gateway Developer Guide (pt-BR)](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/welcome.html)
- [API Gateway Metrics (CloudWatch)](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/api-gateway-metrics-and-dimensions.html)
- [API Gateway Limits](https://docs.aws.amazon.com/apigateway/latest/developerguide/limits.html)
- [API Gateway Caching](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/api-gateway-caching.html)
- [Throttling](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/api-gateway-request-throttling.html)

---


