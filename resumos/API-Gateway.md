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
- [Tipos de API: REST x HTTP x WebSocket](#tipos-de-api-rest-x-http-x-websocket)
- [Estágios de implantação](#estágios-de-implantação)
- [Caching de respostas](#caching-de-respostas)
  - [Invalidação de Cache](#invalidação-de-cache)
- [Plano de uso e chaves de API](#plano-de-uso-e-chaves-de-api)
- [Registro e Rastreamento](#registro-e-rastreamento)
  - [Métricas CloudWatch](#métricas-cloudwatch)
- [Throttling e Quotas](#throttling-e-quotas)
- [Segurança](#segurança)
- [Integrações suportadas](#integrações-suportadas)
- [Transformação de Payload (Mapping Templates)](#transformação-de-payload-mapping-templates)
- [VPC Link](#vpc-link)
- [CORS](#cors)
- [Custom Domain Names](#custom-domain-names)
- [Deployment Canary](#deployment-canary)
- [Erros Comuns](#erros-comuns)
- [Exemplos de Questões de Prova](#exemplos-de-questões-de-prova)
- [📚 Referências](#📚-referências)

---

## Introdução

O Amazon API Gateway é um serviço totalmente gerenciado da AWS para criação, publicação, manutenção, monitoramento e segurança de APIs em escala.

Ele suporta **3 tipos principais de APIs**:

---

## Tipos de API: REST x HTTP x WebSocket

| Tipo | Características | Quando Usar |
|---|---|---|
| **REST API** | Mais recursos, suporta **caching**, **usage plans**, **transformação de payload**, **Lambda Authorizers** | Sistemas legados ou com necessidade de recursos avançados |
| **HTTP API** | Mais simples, **baixa latência**, **baixo custo**, **OAuth2 + JWT**, **Cognito Authorizer**, **Lambda Proxy**, não suporta cache nem usage plans | Microserviços, apps modernos |
| **WebSocket API** | Comunicação **bidirecional em tempo real** | Chat apps, games online, notificações push |

---

## Estágios de Implantação

Cada **Stage** representa um ambiente (ex: dev, qa, prod).

Recursos por Stage:

- Variáveis de Stage
- Logs
- Cache
- Throttling
- Deployment Canary (ver mais abaixo)

**Dica:** Alterações feitas na API só entram em vigor após um **Deploy para um Stage**.

---

## Caching de Respostas

Habilitável apenas para **REST APIs**.

- TTL padrão: 300 segundos
- Intervalo: 0-3600 segundos
- Tamanhos: 0.5GB até 237GB
- Criptografia: Suportada

**Invalidação:**

- Manual
- Via Header (`Cache-Control: max-age=0`)  
- Requer IAM Policy

---

## Plano de Uso e Chaves de API

- Apenas para **REST APIs**
- Controlam: **Rate Limits**, **Quota**, **Associação com Stages**
- Chaves de API identificam clientes

**Dica:** Não confunda: **HTTP APIs não têm Usage Plan nem API Keys**.

---

## Registro e Rastreamento

| Recurso | Suporte |
|---|---|
| **CloudWatch Logs** | Sim |
| **CloudWatch Metrics** | Sim |
| **X-Ray** | Sim |

Exemplos de métricas:

- Latency
- IntegrationLatency
- 4XXError, 5XXError
- CacheHitCount / CacheMissCount

---

## Throttling e Quotas

**Limites Globais:**  
10.000 RPS (por conta por região)

**Limites Configuráveis:**

- Por Stage
- Por Método
- Por Cliente (com Usage Plans)

**Erros comuns:**

- **429 - Too Many Requests**
- **403 - Access Denied** (se throttle policy do IAM negar)

---

## Segurança

| Recurso | Tipo |
|---|---|
| **IAM Roles/Policies** | Para controle de acesso interno |
| **Resource Policies** | Controle entre contas ou por IP |
| **Lambda Authorizer** | Lógica custom (ex: JWT de terceiros) |
| **Cognito User Pools** | Autenticação integrada |
| **JWT Authorizer (HTTP APIs)** | Verificação de JWT sem usar Lambda |

**Dicas de prova:**

- Se for API REST + JWT → Lambda Authorizer  
- Se for HTTP API + JWT → Native JWT Authorizer  

---

## Integrações Suportadas

| Tipo | Exemplo |
|---|---|
| **AWS Service** | Invocar SNS, SQS, etc |
| **Lambda Function** | Serverless backend |
| **HTTP/HTTPS Endpoint** | Chamadas externas |
| **Mock Integration** | Resposta fixa (útil para testes) |

---

## Transformação de Payload (Mapping Templates)

Apenas para **REST APIs**.

- Permite modificar Request ou Response
- Exemplo: Converter JSON → XML, ou alterar headers
- Usado quando precisa adaptar o formato entre cliente e backend.

**Dica de prova:** HTTP APIs **não têm Mapping Templates**.

---

## VPC Link

Permite expor serviços que estão dentro de uma **VPC privada**.

- Exemplo: Integração com um **NLB** (Network Load Balancer)
- Usado quando o backend não é público.

**Tipo de integração:** **Private Integration**

---

## CORS

Cross-Origin Resource Sharing.

- Permite que navegadores acessem APIs de diferentes domínios.

| Configuração | Onde Fazer |
|---|---|
| REST APIs | Via Mapping Templates e Headers |
| HTTP APIs | Configuração nativa e mais simples |

**Erro comum se não configurar:**  
**403 Forbidden (CORS Error no browser)**

---

## Custom Domain Names

Permite expor a API por um domínio próprio (ex: `api.meusite.com`)

- Suporta **Base Path Mapping** (ex: `/v1`, `/v2`)
- Requer configuração de **Route53** e **Certificados SSL via ACM**

---

## Deployment Canary

Permite implantar uma nova versão da API para apenas uma **porcentagem de tráfego** antes de fazer rollout completo.

- Exemplo: Liberar para 10% dos usuários e testar antes de atingir 100%.
- Suportado apenas em **REST APIs**.

---

## Erros Comuns

| Código | Causa |
|---|---|
| 400 | Bad Request |
| 403 | Access Denied |
| 429 | Rate limit excedido |
| 502 | Lambda mal configurada ou erro de integração |
| 503 | Backend indisponível |
| 504 | Timeout de 29s na integração |

---

## Exemplos de Questões de Prova

### Questão 1:

**Você precisa reduzir a latência geral de uma API REST. O que investigar primeiro?**

✅ **Resposta:** IntegrationLatency e uso de Cache.

---

### Questão 2:

**Seu cliente quer JWT Auth com o menor esforço de setup. Que tipo de API?**

✅ **Resposta:** HTTP API com JWT Authorizer.

---

### Questão 3:

**Você precisa integrar API Gateway com um backend privado na VPC. Solução?**

✅ **Resposta:** VPC Link.

---

### Questão 4:

**Quer limitar um cliente a 1000 requisições por dia. Qual solução?**

✅ **Resposta:** Usage Plan + API Key (só REST APIs).

---

### Questão 5:

**Precisa fazer transformação de payload antes de enviar ao Lambda.**

✅ **Resposta:** Mapping Template (REST API).

---

## 📚 Referências

- [AWS API Gateway - Developer Guide](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/welcome.html)
- [Diferenças entre REST API e HTTP API](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/http-api-vs-rest.html)
- [Mapping Templates](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html)
- [CORS no API Gateway](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/apigateway-enable-cors.html)
- [API Gateway Metrics](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/api-gateway-metrics-and-dimensions.html)
- [VPC Link](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/getting-started-with-private-integration.html)
- [Deployment Canary](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/canary-release.html)
