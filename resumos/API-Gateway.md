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
- [Estágios de Implantação (Stages)](#estágios-de-implantação-stages)
- [Caching de Respostas](#caching-de-respostas)
  - [Invalidação de Cache](#invalidação-de-cache)
- [Plano de Uso e Chaves de API](#plano-de-uso-e-chaves-de-api)
- [Registro e Rastreamento](#registro-e-rastreamento)
  - [Métricas CloudWatch](#métricas-cloudwatch)
- [Throttling e Quotas](#throttling-e-quotas)
- [Segurança](#segurança)
- [Integrações Suportadas](#integrações-suportadas)
- [Transformação de Payload (Mapping Templates)](#transformação-de-payload-mapping-templates)
- [VPC Link](#vpc-link)
- [CORS (Cross-Origin Resource Sharing)](#cors-cross-origin-resource-sharing)
- [Custom Domain Names](#custom-domain-names)
- [Deployment Canary](#deployment-canary)
- [Erros Comuns](#erros-comuns)
- [Exemplos de Questões de Prova](#exemplos-de-questões-de-prova)
- [📚 Referências](#📚-referências)

---

## Introdução

O **Amazon API Gateway** é um serviço da AWS que permite criar, publicar, proteger, monitorar e manter APIs REST, HTTP e WebSocket em escala. Ele atua como um ponto único de entrada para suas APIs, podendo conectar diferentes tipos de backend (como funções Lambda, serviços AWS ou endpoints HTTP externos).

---

## Tipos de API: REST x HTTP x WebSocket

| Tipo | Características | Quando Usar |
|---|---|---|
| **REST API** | API tradicional, com suporte a recursos avançados como caching, usage plans, transformação de payload, Lambda Authorizers e deploys complexos. | Sistemas legados, APIs com alta customização e controle granular |
| **HTTP API** | Mais simples, mais barata e com baixa latência, suporta autenticação JWT e OAuth 2.0 nativa, mas não suporta caching nem usage plans. | Microserviços, apps modernos e cenários que priorizam performance e custo |
| **WebSocket API** | Comunicação bidirecional em tempo real para enviar/receber dados instantaneamente | Aplicações como chat, jogos, notificações em tempo real |

---

## Estágios de Implantação (Stages)

### O que é um Stage?

Um **Stage** representa um ambiente para sua API (exemplo: `dev`, `test`, `prod`). Cada stage é uma versão implantada da sua API e pode ter configurações independentes.

### Por que usar Stages?

- Permitem **isolar ambientes** para desenvolvimento, testes e produção.
- Cada stage pode ter **variáveis de ambiente** próprias.
- Você pode configurar **logs, cache e throttling diferentes** para cada stage.
- Permite fazer deploys independentes sem impactar outros ambientes.

### Como funciona o deploy?

Após fazer alterações na API, é necessário realizar o **deploy** para um stage. Só após o deploy as mudanças ficam visíveis nesse ambiente.

---

## Caching de Respostas

### O que é cache no API Gateway?

Cache é um armazenamento temporário das respostas de uma API. Quando habilitado, o API Gateway armazena respostas para evitar chamar o backend repetidamente para a mesma requisição, reduzindo latência e custo.

### Benefícios

- Reduz a carga no backend.
- Diminui latência das respostas.
- Pode melhorar a experiência do usuário.

### Configurações principais

- TTL (Time to Live): tempo que o cache permanece válido (default 300s).
- Tamanho: varia de 0,5 GB até 237 GB.
- Pode ser configurado por stage e por método.

### Invalidação de Cache

- Pode ser feita manualmente no console.
- Pode ser invalidada programaticamente via header HTTP.
- Requer permissões IAM específicas para isso.

---

## Plano de Uso e Chaves de API

### O que são?

- **Usage Plans** definem limites de uso e quotas para clientes.
- **API Keys** identificam clientes que usam a API.

### Para que servem?

- Controlar o acesso à API.
- Limitar número de requisições para evitar abuso.
- Associar limites a clientes diferentes.

### Importante

- Só estão disponíveis para **REST APIs**.
- HTTP APIs não suportam usage plans nem API Keys.

---

## Registro e Rastreamento

### CloudWatch Logs

- Permite capturar logs detalhados de requisições e respostas.
- Pode ser configurado por stage e método.

### CloudWatch Metrics

- Métricas que ajudam a monitorar performance e uso, como:
  - `Latency`: tempo total da requisição.
  - `IntegrationLatency`: tempo gasto na comunicação com backend.
  - `CacheHitCount` e `CacheMissCount`.
  - Contagem de erros (4XX, 5XX).

### AWS X-Ray

- Ferramenta para rastreamento distribuído.
- Permite identificar gargalos e problemas em toda a cadeia da requisição.

---

## Throttling e Quotas

### O que é throttling?

Mecanismo que limita o número de requisições para proteger o backend de sobrecarga.

### Como funciona?

- Limites padrão da conta AWS (exemplo: 10.000 RPS).
- Pode definir limites por stage, método e cliente.
- Quando ultrapassa o limite, retorna erro **429 - Too Many Requests**.

---

## Segurança

| Mecanismo | Descrição |
|---|---|
| **IAM Policies** | Controle de acesso para usuários/funções na AWS |
| **Resource Policies** | Restrição de acesso por conta, IP ou VPC |
| **Lambda Authorizer** | Código customizado para autenticação (ex: validar token JWT externo) |
| **Cognito User Pools** | Autenticação gerenciada e escalável da AWS |
| **JWT Authorizer (HTTP APIs)** | Autenticação nativa via tokens JWT, sem necessidade de Lambda |

---

## Integrações Suportadas

- **Lambda Functions**
- **Serviços AWS (SNS, SQS, Step Functions)**
- **Endpoints HTTP/HTTPS externos**
- **Mock Integration** (respostas fixas para testes)

---

## Transformação de Payload (Mapping Templates)

### O que é?

É a capacidade de transformar a estrutura da requisição ou resposta (ex: JSON para XML) usando templates escritos em Velocity Template Language (VTL).

### Para que serve?

- Adaptar o payload para formatos que o backend espera.
- Remover ou modificar dados sensíveis.
- Enriquecer ou simplificar a resposta para o cliente.

### Disponível apenas para REST APIs.

---

## VPC Link

Permite conectar o API Gateway a serviços dentro de uma VPC privada usando Network Load Balancer.

Útil para integrar APIs com backends não públicos, garantindo segurança e isolamento.

---

## CORS (Cross-Origin Resource Sharing)

### O que é?

Mecanismo de segurança dos navegadores que controla o acesso de páginas web a recursos de domínios diferentes do domínio da página.

### Por que é importante?

Sem CORS configurado corretamente, seu front-end pode ser impedido de acessar a API, gerando erros do tipo **CORS policy: No 'Access-Control-Allow-Origin' header**.

### Como funciona no API Gateway?

- Você precisa habilitar CORS para permitir requisições de origens externas.
- Configurações envolvem adicionar headers HTTP específicos na resposta da API:
  - `Access-Control-Allow-Origin`
  - `Access-Control-Allow-Methods`
  - `Access-Control-Allow-Headers`

### Configuração

- REST API: Pode ser configurado manualmente via headers ou usando templates.
- HTTP API: Possui configuração nativa simplificada para CORS.

---

## Custom Domain Names

Permite expor sua API usando um domínio próprio (exemplo: `api.meusite.com`).

### Benefícios

- Facilita o uso da API por clientes.
- Permite usar certificados SSL personalizados via AWS Certificate Manager.

---

## Deployment Canary

### O que é?

É uma técnica de deploy gradual em que uma nova versão da API é liberada para uma pequena porcentagem de usuários antes do rollout completo.

### Por que usar?

- Minimiza riscos em deploys.
- Permite monitorar comportamento da nova versão em produção com pouco impacto.

---

## Erros Comuns

| Código | Descrição |
|---|---|
| 400 | Requisição mal formada |
| 403 | Acesso negado (políticas IAM ou Resource Policies) |
| 429 | Limite de requisições excedido (throttling) |
| 502 | Erro no backend (ex: Lambda com erro) |
| 503 | Backend indisponível |
| 504 | Timeout na integração (limite 29s) |

---

## Exemplos de Questões de Prova

### Questão 1:

**Qual métrica investigar para alta latência em API REST?**

✅ *Resposta:* IntegrationLatency.

---

### Questão 2:

**Quer autenticação JWT simples e barata. Que tipo de API usar?**

✅ *Resposta:* HTTP API com JWT Authorizer.

---

### Questão 3:

**Quer expor backend privado na VPC via API Gateway. Como?**

✅ *Resposta:* VPC Link.

---

### Questão 4:

**Quer limitar um cliente a 1000 requisições/dia. Qual recurso usar?**

✅ *Resposta:* Usage Plan + API Key (REST API).

---

### Questão 5:

**Precisa transformar payload JSON para XML antes do backend.**

✅ *Resposta:* Mapping Template (REST API).

---

## 📚 Referências

- [AWS API Gateway - Developer Guide](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/welcome.html)
- [Diferenças REST API e HTTP API](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/http-api-vs-rest.html)
- [Mapping Templates](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html)
- [CORS no API Gateway](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/apigateway-enable-cors.html)
- [API Gateway Metrics](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/api-gateway-metrics-and-dimensions.html)
- [VPC Link](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/getting-started-with-private-integration.html)
- [Deployment Canary](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/canary-release.html)

