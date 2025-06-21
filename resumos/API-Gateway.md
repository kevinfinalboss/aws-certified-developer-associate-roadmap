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
- [Estágios de implantação](#estágios-de-implantação)
- [Caching respostas da API](#caching-respostas-da-api)
  - [Invalidação de Cache](#invalidação-de-cache)
- [Plano de uso e chaves de API](#plano-de-uso-e-chaves-de-api)
- [Registro e Rastreamento](#registro-e-rastreamento)
  - [Métricas CloudWatch](#métricas-cloudwatch)
- [Throttling](#throttling)
- [Erros](#erros)
- [Segurança](#segurança)  
- [Exemplos de Questões de Prova](#exemplos-de-questões-de-prova)
- [📚 Referências](#📚-referências)

---

## Introdução

Amazon API Gateway é um serviço totalmente gerenciado da AWS que permite **criar, publicar, monitorar, proteger e manter APIs REST, HTTP e WebSocket** em escala.

Serve como **ponto único de entrada** para chamadas de APIs que se integram com backends como **Lambda**, **serviços AWS**, ou **endpoints HTTP externos**.

---

## Estágios de implantação

Para que as alterações feitas no API Gateway entrem em vigor, é necessário fazer a **implantação (Deploy)** para um **Stage** (ex: `dev`, `homolog`, `prod`).

Cada **Stage** pode ter:

- **Variáveis de ambiente**
- **Logs customizados**
- **Cache**
- **Throttling**
- **Planos de Uso**

---

## Caching respostas da API

O cache reduz o número de chamadas feitas ao backend. Ele é configurado por **Stage**, mas pode ser sobrescrito por método.

| Recurso | Valor |
|---|---|
| **TTL padrão** | 300 segundos (5 minutos) |
| **Intervalo permitido** | 0 a 3600 segundos (1 hora) |
| **Tamanhos de cache** | 0.5GB a 237GB |
| **Criptografia** | Suportada |

**💡 Dica de prova:** Cache é caro → Geralmente recomendado só em produção.

---

### Invalidação de Cache

- Manualmente via Console AWS.
- Ou programaticamente pelos clientes via Header HTTP:


> ⚠️ Importante: Para invalidar o cache via API, o cliente precisa ter permissão adequada no IAM.

---

## Plano de uso e chaves de API

Os **Planos de Uso (Usage Plans)** controlam:

- **Quem** pode acessar
- **Velocidade (Rate Limit)**: Exemplo → X requisições por segundo
- **Quota**: Exemplo → Y requisições por dia/mês
- **Associação com Stages e Métodos**

As **API Keys** identificam os clientes e medem o consumo.

> **Importante:** Apenas APIs do tipo **REST API** suportam **Usage Plans + API Keys**.

---

## Registro e Rastreamento

O API Gateway permite:

- **Logs de execução e acesso** → enviados para o **CloudWatch Logs**
- **Nível de stage** → Configurações podem ser personalizadas por stage ou por método
- **Integração com AWS X-Ray** → Para rastreamento detalhado das requisições e latências.

---

### Métricas CloudWatch

| Métrica | Descrição |
|---|---|
| **CacheHitCount** | Quantidade de requisições atendidas pelo cache |
| **CacheMissCount** | Quantidade de requisições que não encontraram resposta em cache |
| **Count** | Número total de requisições |
| **IntegrationLatency** | Tempo gasto na integração com o backend (exemplo: Lambda) |
| **Latency** | Tempo total entre a requisição do cliente e a resposta (incluindo o tempo de integração) |

**⚠️ Interpretação de CacheMissCount:**  
Se o valor for alto → indica baixa eficiência do cache (muitas requisições não estão sendo atendidas via cache).

---

## Throttling

- **Limite por conta AWS:** Por padrão o API Gateway permite até **10.000 requisições por segundo por conta**.
- Se uma API atingir muito uso, **outras APIs da mesma conta também podem ser impactadas** (compartilham o mesmo limite).

> **Erro mais comum:**  
**HTTP 429 - Too Many Requests**

**Opções para limitar requisições:**

- Definir **limite por Stage**
- Definir **limite por Método**
- Usar **Planos de Uso** com **API Keys**, limitando por cliente.

---

## Erros

| Código | Descrição |
|---|---|
| **400** | Bad Request - Requisição malformada |
| **403** | Access Denied - Acesso negado (ex: WAF ou Resource Policy) |
| **429** | Too Many Requests - Limite ou quota excedida |
| **502** | Bad Gateway - Exemplo: Lambda não respondeu corretamente |
| **503** | Service Unavailable - Backend indisponível |
| **504** | Integration Failure - Timeout na integração (limite de 29 segundos no backend) |

---

## Segurança

| Opção | Descrição |
|---|---|
| **Permissões IAM** | Controle para usuários/funções dentro da AWS |
| **Resource Policies** | Controle de acesso entre contas AWS ou por IP |
| **Cognito User Pools** | Autenticação gerenciada de usuários finais |
| **Lambda Authorizer (Custom Authorizer)** | Lógica de autenticação customizada (ex: tokens JWT externos) |

**Exemplo de caso de uso de Lambda Authorizer:**  
Quando você tem um banco de usuários externo, fora do Cognito.

---

## Exemplos de Questões de Prova

### Questão 1:

**Sua API REST apresenta alta latência. Qual métrica investigar?**

- A) CloudTrail  
- B) IntegrationLatency  
- C) Resource Policy  
- D) Cache Hit Ratio  

✅ **Resposta:** B → IntegrationLatency mostra o tempo gasto no backend.

---

### Questão 2:

**Sua API está retornando erro 504. O que pode ser?**

- A) Problema de autenticação IAM  
- B) Backend demorou mais de 29 segundos para responder  
- C) Chave de API inválida  
- D) Cache expirado  

✅ **Resposta:** B → Timeout na integração (limite de 29 segundos).

---

### Questão 3:

**Qual cenário justifica uso de HTTP API ao invés de REST API?**

- A) Precisa de cache  
- B) Transformação de payload  
- C) Baixo custo e baixa latência  
- D) Precisa de Usage Plans  

✅ **Resposta:** C → HTTP API é mais barata e rápida.

---

### Questão 4:

**Como limitar um cliente a 1000 requisições diárias?**

- A) IAM Policy  
- B) Usage Plan + API Key  
- C) Lambda Authorizer  
- D) Configuração de CORS  

✅ **Resposta:** B → Usage Plans são feitos para esse controle.

---

## 📚 Referências

- [O que é o Amazon API Gateway?](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/welcome.html)
- [Criar e usar planos de uso com chaves de API](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/api-gateway-api-usage-plans.html)
- [CloudWatch Metrics for API Gateway](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/api-gateway-metrics-and-dimensions.html)
- [API Gateway Throttling](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/api-gateway-request-throttling.html)
- [Caching no API Gateway](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/api-gateway-caching.html)

---
