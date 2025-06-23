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
- [Request e Response Transformation](#request-e-response-transformation)
- [Binary Media Types](#binary-media-types)
- [WebSocket APIs](#websocket-apis)
- [Versioning](#versioning)
- [SDK Generation](#sdk-generation)
- [Client-Side SSL](#client-side-ssl)
- [WAF Integration](#waf-integration)
- [Erros Comuns](#erros-comuns)
- [Exemplos de Questões de Prova](#exemplos-de-questões-de-prova)
- [📚 Referências](#📚-referências)

---

## Introdução

O **Amazon API Gateway** é um serviço da AWS que permite criar, publicar, proteger, monitorar e manter APIs REST, HTTP e WebSocket em escala. Ele atua como um ponto único de entrada para suas APIs, podendo conectar diferentes tipos de backend (como funções Lambda, serviços AWS ou endpoints HTTP externos).

### Principais benefícios:
- Gerenciamento completo de APIs sem servidor
- Escalabilidade automática
- Cobrança por uso (pay-per-use)
- Integração nativa com outros serviços AWS
- Monitoramento e logging integrados

---

## Tipos de API: REST x HTTP x WebSocket

| Tipo | Características | Quando Usar | Limitações |
|---|---|---|---|
| **REST API** | API tradicional, com suporte a recursos avançados como caching, usage plans, transformação de payload, Lambda Authorizers e deploys complexos. | Sistemas legados, APIs com alta customização e controle granular | Mais cara, maior latência |
| **HTTP API** | Mais simples, mais barata (até 70% menos) e com baixa latência, suporta autenticação JWT e OAuth 2.0 nativa, mas não suporta caching nem usage plans. | Microserviços, apps modernos e cenários que priorizam performance e custo | Não suporta caching, usage plans, request validation |
| **WebSocket API** | Comunicação bidirecional em tempo real para enviar/receber dados instantaneamente | Aplicações como chat, jogos, notificações em tempo real | Mais complexa de implementar |

### REST API vs HTTP API - Comparação Detalhada

| Recurso | REST API | HTTP API |
|---|---|---|
| **Preço** | Mais cara | Até 70% mais barata |
| **Latência** | Maior | Menor |
| **Caching** | ✅ Suportado | ❌ Não suportado |
| **Usage Plans** | ✅ Suportado | ❌ Não suportado |
| **Request Validation** | ✅ Suportado | ❌ Não suportado |
| **JWT Authorizer** | ❌ Precisa Lambda | ✅ Nativo |
| **CORS** | Manual | ✅ Configuração simplificada |
| **Private Integration** | ✅ VPC Link | ✅ VPC Link |

---

## Estágios de Implantação (Stages)

### O que é um Stage?

Um **Stage** representa um ambiente para sua API (exemplo: `dev`, `test`, `prod`). Cada stage é uma versão implantada da sua API e pode ter configurações independentes.

### Por que usar Stages?

- Permitem **isolar ambientes** para desenvolvimento, testes e produção
- Cada stage pode ter **variáveis de ambiente** próprias
- Você pode configurar **logs, cache e throttling diferentes** para cada stage
- Permite fazer deploys independentes sem impactar outros ambientes

### Como funciona o deploy?

Após fazer alterações na API, é necessário realizar o **deploy** para um stage. Só após o deploy as mudanças ficam visíveis nesse ambiente.

### Stage Variables

As **Stage Variables** funcionam como variáveis de ambiente que podem ser usadas para:
- Configurar diferentes endpoints de backend por ambiente
- Personalizar configurações de integração
- Referenciar diferentes versões de Lambda Functions

**Exemplo de uso:**
```
Integration Endpoint: https://${stageVariables.lambdaAlias}.execute-api.region.amazonaws.com
```

---

## Caching de Respostas

### O que é cache no API Gateway?

Cache é um armazenamento temporário das respostas de uma API. Quando habilitado, o API Gateway armazena respostas para evitar chamar o backend repetidamente para a mesma requisição, reduzindo latência e custo.

### Benefícios

- Reduz a carga no backend
- Diminui latência das respostas
- Pode melhorar a experiência do usuário
- Reduz custos de invocação do backend

### Configurações principais

- **TTL (Time to Live)**: tempo que o cache permanece válido (0 a 3600 segundos, default 300s)
- **Tamanho**: varia de 0,5 GB até 237 GB
- Pode ser configurado por stage e por método
- **Cache Key**: baseado em parâmetros de query string, headers e path parameters

### Invalidação de Cache

- Pode ser feita manualmente no console
- Pode ser invalidada programaticamente via header HTTP `Cache-Control: max-age=0`
- Requer permissões IAM específicas (`apigateway:FlushStageCache`)
- Pode ser invalidada por padrão de cache key

### Importante sobre Cache

- **Apenas disponível para REST APIs**
- HTTP APIs não suportam caching
- Cache é cobrado por hora, mesmo se não usado
- Pode ser habilitado/desabilitado por método HTTP

---

## Plano de Uso e Chaves de API

### O que são?

- **Usage Plans** definem limites de uso e quotas para clientes
- **API Keys** identificam clientes que usam a API

### Para que servem?

- Controlar o acesso à API
- Limitar número de requisições para evitar abuso
- Associar limites a clientes diferentes
- Implementar diferentes níveis de serviço (tiers)

### Configurações de Usage Plans

- **Throttle**: limite de requisições por segundo
- **Quota**: limite de requisições por período (dia, semana, mês)
- **Burst**: rajadas temporárias permitidas

### Importante

- Só estão disponíveis para **REST APIs**
- HTTP APIs não suportam usage plans nem API Keys
- API Keys devem ser enviadas no header `x-api-key`
- Não devem ser usadas como método principal de autenticação

---

## Registro e Rastreamento

### CloudWatch Logs

- Permite capturar logs detalhados de requisições e respostas
- Pode ser configurado por stage e método
- Tipos de log: `ERROR`, `INFO`
- Pode incluir request/response body (cuidado com dados sensíveis)

### Métricas CloudWatch

Métricas que ajudam a monitorar performance e uso:

| Métrica | Descrição |
|---|---|
| **Count** | Número total de chamadas da API |
| **Latency** | Tempo total da requisição (Gateway + Backend) |
| **IntegrationLatency** | Tempo gasto na comunicação com backend |
| **CacheHitCount** | Número de hits no cache |
| **CacheMissCount** | Número de misses no cache |
| **4XXError** | Erros do lado do cliente |
| **5XXError** | Erros do lado do servidor |

### AWS X-Ray

- Ferramenta para rastreamento distribuído
- Permite identificar gargalos e problemas em toda a cadeia da requisição
- Mostra mapa de serviços e tempos de resposta
- Pode ser habilitado por stage

---

## Throttling e Quotas

### O que é throttling?

Mecanismo que limita o número de requisições para proteger o backend de sobrecarga.

### Como funciona?

- **Limites padrão da conta AWS**: 10.000 RPS steady-state, 5.000 burst
- Pode definir limites por stage, método e cliente
- Quando ultrapassa o limite, retorna erro **429 - Too Many Requests**
- Usa algoritmo **Token Bucket**

### Tipos de Throttling

1. **Account-level throttling**: limite da conta AWS
2. **API-level throttling**: limite por API
3. **Stage-level throttling**: limite por stage
4. **Method-level throttling**: limite por método HTTP
5. **Usage Plan throttling**: limite por cliente/API Key

### Ordem de Prioridade

1. Usage Plan (mais específico)
2. Method-level
3. Stage-level
4. API-level
5. Account-level (menos específico)

---

## Segurança

### Mecanismos de Autenticação e Autorização

| Mecanismo | Descrição | Quando Usar |
|---|---|---|
| **IAM Policies** | Controle de acesso para usuários/funções na AWS | Acesso interno AWS, aplicações server-to-server |
| **Resource Policies** | Restrição de acesso por conta, IP ou VPC | Controle de acesso baseado em rede/conta |
| **Lambda Authorizer** | Código customizado para autenticação | Lógica de autorização complexa, tokens externos |
| **Cognito User Pools** | Autenticação gerenciada e escalável da AWS | Aplicações web/mobile com usuários |
| **JWT Authorizer (HTTP APIs)** | Autenticação nativa via tokens JWT | APIs modernas com tokens JWT padrão |

### Lambda Authorizer

- **TOKEN**: valida token no header Authorization
- **REQUEST**: valida toda a requisição (headers, query params, etc.)
- Pode cachear resultado da autorização (TTL configurável)
- Deve retornar política IAM permitindo/negando acesso

### Resource-Based Policies

Permite controlar:
- Quais contas AWS podem acessar
- Quais IPs podem acessar
- Quais VPCs podem acessar
- Horários de acesso permitidos

---

## Integrações Suportadas

### Tipos de Integração

| Tipo | Descrição | Casos de Uso |
|---|---|---|
| **Lambda Function** | Invoca função Lambda | Lógica de negócio, processamento |
| **HTTP/HTTPS** | Proxy para endpoint HTTP externo | APIs de terceiros, microserviços |
| **AWS Service** | Integração direta com serviços AWS | SQS, SNS, DynamoDB, S3 |
| **Mock** | Resposta fixa sem backend | Testes, protótipos |

### Integração com Lambda

- **Lambda Proxy Integration**: passa toda requisição para Lambda
- **Lambda Custom Integration**: transforma request/response
- Suporta versioning e aliasing
- Pode usar Stage Variables para diferentes versões

### Integração com Serviços AWS

Permite integração direta com:
- **SQS**: enviar mensagens
- **SNS**: publicar notificações
- **DynamoDB**: operações CRUD
- **S3**: upload/download de arquivos
- **Step Functions**: iniciar workflows

---

## Transformação de Payload (Mapping Templates)

### O que é?

É a capacidade de transformar a estrutura da requisição ou resposta usando templates escritos em **Velocity Template Language (VTL)**.

### Para que serve?

- Adaptar o payload para formatos que o backend espera
- Remover ou modificar dados sensíveis
- Enriquecer ou simplificar a resposta para o cliente
- Transformar entre diferentes formatos (JSON, XML, Form-data)

### Tipos de Mapping

1. **Request Mapping**: transforma requisição antes de enviar ao backend
2. **Response Mapping**: transforma resposta antes de enviar ao cliente

### Disponível apenas para REST APIs

HTTP APIs não suportam mapping templates (usam apenas proxy integration).

---

## VPC Link

### O que é?

Permite conectar o API Gateway a serviços dentro de uma VPC privada usando Network Load Balancer.

### Tipos de VPC Link

- **VPC Link for REST APIs**: conecta via NLB
- **VPC Link for HTTP APIs**: conecta diretamente a ALB, NLB ou Cloud Map

### Casos de Uso

- Integrar APIs com backends não públicos
- Garantir segurança e isolamento
- Conectar a microserviços em ECS/EKS
- Acessar databases privados via API

### Configuração

1. Criar Network Load Balancer na VPC
2. Criar VPC Link apontando para o NLB
3. Configurar integração da API para usar VPC Link

---

## CORS (Cross-Origin Resource Sharing)

### O que é?

Mecanismo de segurança dos navegadores que controla o acesso de páginas web a recursos de domínios diferentes do domínio da página.

### Por que é importante?

Sem CORS configurado corretamente, seu front-end pode ser impedido de acessar a API, gerando erros do tipo **CORS policy: No 'Access-Control-Allow-Origin' header**.

### Headers CORS importantes

- `Access-Control-Allow-Origin`: domínios permitidos
- `Access-Control-Allow-Methods`: métodos HTTP permitidos
- `Access-Control-Allow-Headers`: headers permitidos
- `Access-Control-Max-Age`: tempo de cache do preflight

### Configuração

- **REST API**: Configuração manual via headers ou templates
- **HTTP API**: Configuração nativa simplificada
- **Preflight Request**: requisição OPTIONS automática do browser

### Importante

- CORS é uma política do navegador, não do servidor
- APIs chamadas server-to-server não precisam de CORS
- Sempre configurar para requisições de SPAs (Single Page Applications)

---

## Custom Domain Names

Permite expor sua API usando um domínio próprio (exemplo: `api.meusite.com`).

### Benefícios

- Facilita o uso da API por clientes
- Permite usar certificados SSL personalizados via AWS Certificate Manager
- Branding personalizado
- Independência da URL padrão da AWS

### Configuração

1. Registrar domínio no Route 53 ou DNS externo
2. Criar certificado SSL no ACM
3. Configurar Custom Domain no API Gateway
4. Criar registros DNS apontando para o domínio do API Gateway

### Tipos de Endpoint

- **Edge-optimized**: usa CloudFront para reduzir latência global
- **Regional**: endpoint regional para menor latência local
- **Private**: apenas acessível dentro da VPC

---

## Deployment Canary

### O que é?

É uma técnica de deploy gradual em que uma nova versão da API é liberada para uma pequena porcentagem de usuários antes do rollout completo.

### Por que usar?

- Minimiza riscos em deploys
- Permite monitorar comportamento da nova versão em produção com pouco impacto
- Facilita rollback em caso de problemas

### Como funciona?

1. Deploy da nova versão como Canary
2. Configurar porcentagem de tráfego (ex: 10%)
3. Monitorar métricas e logs
4. Promover para produção ou fazer rollback

### Configurações

- **Canary Weight**: porcentagem de tráfego (0-100%)
- **Stage Variables**: podem ser diferentes entre produção e canary
- **Métricas separadas**: CloudWatch mostra métricas separadas

---

## Request e Response Transformation

### Request Transformation

- Modificar query parameters
- Adicionar/remover headers
- Transformar body da requisição
- Validar schema da requisição

### Response Transformation

- Filtrar dados sensíveis
- Reformatar estrutura de dados
- Adicionar headers de resposta
- Tratar diferentes códigos de status

### Request Validation

- Validar parâmetros obrigatórios
- Validar formato de dados (JSON Schema)
- Validar headers e query parameters
- Retornar erro 400 para dados inválidos

**Disponível apenas para REST APIs**

---

## Binary Media Types

### O que é?

Permite o API Gateway processar conteúdo binário (imagens, PDFs, vídeos, etc.) ao invés de apenas texto.

### Configuração

1. Definir Binary Media Types na API (ex: `image/jpeg`, `application/pdf`)
2. Configurar integração para processar conteúdo binário
3. Usar Base64 encoding para transport

### Casos de Uso

- Upload de imagens
- Download de arquivos
- Streaming de conteúdo
- Integração com S3 para arquivos

---

## WebSocket APIs

### Características

- Comunicação bidirecional em tempo real
- Conexão persistente
- Baixa latência
- Suporte a routing baseado em mensagens

### Conceitos Importantes

- **Connection**: sessão WebSocket
- **Route**: define como processar mensagens
- **Integration**: backend que processa a mensagem

### Rotas Especiais

- `$connect`: quando cliente conecta
- `$disconnect`: quando cliente desconecta
- `$default`: rota padrão para mensagens

### Casos de Uso

- Chat em tempo real
- Notificações push
- Jogos multiplayer
- Dashboards em tempo real
- Streaming de dados

---

## Versioning

### Estratégias de Versionamento

1. **Stage-based**: usar stages para versões (v1, v2)
2. **Path-based**: incluir versão no path (`/v1/users`, `/v2/users`)
3. **Header-based**: usar header para especificar versão
4. **Query parameter**: usar query param (`?version=v1`)

### Melhores Práticas

- Manter compatibilidade com versões anteriores
- Comunicar mudanças breaking changes
- Usar semantic versioning
- Documentar lifecycle de versões

---

## SDK Generation

### O que é?

API Gateway pode gerar automaticamente SDKs para diferentes linguagens de programação.

### Linguagens Suportadas

- JavaScript
- Java
- Python
- PHP
- .NET
- Ruby
- Go

### Benefícios

- Acelera desenvolvimento de clientes
- Padroniza integração com a API
- Inclui autenticação e retry logic
- Reduz erros de implementação

---

## Client-Side SSL

### Autenticação Mutual TLS (mTLS)

- Cliente e servidor autenticam mutuamente
- Requer certificado no cliente
- Maior segurança para APIs críticas
- Configurado via Custom Domain

### Configuração

1. Configurar Custom Domain com certificado
2. Configurar Truststore com CAs aceitos
3. Cliente deve apresentar certificado válido
4. API Gateway valida certificado do cliente

---

## WAF Integration

### O que é?

AWS WAF (Web Application Firewall) pode ser integrado ao API Gateway para proteção adicional.

### Proteções Oferecidas

- Proteção contra SQL injection
- Proteção contra XSS
- Rate limiting avançado
- Bloqueio por geolocalização
- Proteção contra bot attacks

### Configuração

1. Criar Web ACL no AWS WAF
2. Configurar rules de proteção
3. Associar Web ACL ao API Gateway stage
4. Monitorar via CloudWatch

---

## Erros Comuns

| Código | Descrição | Causa Comum | Solução |
|---|---|---|---|
| **400** | Bad Request | Requisição mal formada, parâmetros inválidos | Validar payload e parâmetros |
| **403** | Forbidden | Políticas IAM, Resource Policies, API Key inválida | Verificar permissões e autenticação |
| **429** | Too Many Requests | Limite de throttling excedido | Implementar retry com backoff |
| **502** | Bad Gateway | Erro no backend (Lambda, HTTP endpoint) | Verificar logs do backend |
| **503** | Service Unavailable | Backend indisponível | Verificar saúde do backend |
| **504** | Gateway Timeout | Timeout na integração (limite 29s) | Otimizar backend ou usar async |

### Timeouts Importantes

- **API Gateway timeout**: 29 segundos (não configurável)
- **Lambda timeout**: até 15 minutos
- **HTTP integration timeout**: 29 segundos

---

## Exemplos de Questões de Prova

### Questão 1: Latência Alta
**Sua API REST está com latência alta. Qual métrica investigar primeiro?**

✅ **Resposta:** IntegrationLatency - mostra o tempo gasto na comunicação com o backend, ajudando a identificar se o problema está no backend ou no API Gateway.

---

### Questão 2: Autenticação JWT
**Quer autenticação JWT simples e barata. Que tipo de API usar?**

✅ **Resposta:** HTTP API com JWT Authorizer - HTTP APIs são mais baratas e têm suporte nativo a JWT.

---

### Questão 3: Backend Privado
**Quer expor backend privado na VPC via API Gateway. Como fazer?**

✅ **Resposta:** VPC Link - permite conectar API Gateway a recursos privados na VPC via Network Load Balancer.

---

### Questão 4: Limites de Uso
**Quer limitar um cliente a 1000 requisições/dia. Qual recurso usar?**

✅ **Resposta:** Usage Plan + API Key (REST API) - Usage Plans definem quotas e limites por cliente.

---

### Questão 5: Transformação de Dados
**Precisa transformar payload JSON para XML antes do backend.**

✅ **Resposta:** Mapping Template (REST API) - permite transformar request/response usando VTL.

---

### Questão 6: Cache Performance
**API com muitas consultas repetidas. Como melhorar performance?**

✅ **Resposta:** Habilitar caching (REST API) - reduz latência e carga no backend para requisições repetidas.

---

### Questão 7: Erro CORS
**SPA retorna erro CORS ao chamar API. Qual a causa mais provável?**

✅ **Resposta:** CORS não configurado no API Gateway - browsers bloqueiam requisições cross-origin sem headers CORS apropriados.

---

### Questão 8: Timeout 504
**API retorna erro 504 frequentemente. Qual o problema?**

✅ **Resposta:** Backend demora mais que 29 segundos para responder - API Gateway tem timeout fixo de 29 segundos.

---

### Questão 9: Deploy Seguro
**Quer fazer deploy gradual de nova versão. Qual estratégia usar?**

✅ **Resposta:** Canary Deployment - permite liberar nova versão para porcentagem pequena de usuários primeiro.

---

### Questão 10: Autenticação AWS
**API interna precisa autenticar serviços AWS. Qual método usar?**

✅ **Resposta:** IAM Authentication - ideal para autenticação server-to-server dentro da AWS.

---

### Questão 11: Custo vs Funcionalidade
**Microserviço simples precisa de API barata sem caching. Qual tipo escolher?**

✅ **Resposta:** HTTP API - até 70% mais barata que REST API, ideal para casos simples.

---

### Questão 12: Monitoramento
**Quer rastrear requests através de múltiplos serviços. Qual ferramenta usar?**

✅ **Resposta:** AWS X-Ray - fornece rastreamento distribuído completo.

---

### Questão 13: Domínio Personalizado
**Quer usar api.empresa.com ao invés da URL padrão da AWS. Como fazer?**

✅ **Resposta:** Custom Domain Name - permite usar domínio próprio com certificado SSL.

---

### Questão 14: Validação de Dados
**Quer validar JSON schema antes de processar. Qual tipo de API suporta?**

✅ **Resposta:** REST API - HTTP API não suporta request validation nativa.

---

### Questão 15: WebSocket Real-time
**Chat em tempo real precisa de comunicação bidirecional. Qual tipo usar?**

✅ **Resposta:** WebSocket API - suporta comunicação bidirecional em tempo real.

---

### Questão 16: Throttling Hierarchy
**Cliente tem Usage Plan (100 RPS) e método tem limit (50 RPS). Qual prevalece?**

✅ **Resposta:** Usage Plan (100 RPS) - Usage Plan tem prioridade mais alta que method-level throttling.

---

### Questão 17: Binary Content
**API precisa processar upload de imagens. Como configurar?**

✅ **Resposta:** Configurar Binary Media Types (ex: image/jpeg) - permite processamento de conteúdo binário.

---

### Questão 18: Cache Invalidation
**Dados mudaram e cache precisa ser limpo. Como fazer via código?**

✅ **Resposta:** Header Cache-Control: max-age=0 - invalida cache programaticamente.

---

### Questão 19: Private API
**API só deve ser acessível dentro da VPC. Como configurar?**

✅ **Resposta:** Private API + VPC Endpoint - restringe acesso apenas à VPC específica.

---

### Questão 20: Error Handling
**Lambda retorna erro 500. Que erro API Gateway mostra?**

✅ **Resposta:** 502 Bad Gateway - erro de integração com backend resulta em 502.

---

## 📚 Referências

- [AWS API Gateway - Developer Guide](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/welcome.html)
- [Diferenças REST API e HTTP API](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/http-api-vs-rest.html)
- [Mapping Templates](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html)
- [CORS no API Gateway](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/apigateway-enable-cors.html)
- [API Gateway Metrics](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/api-gateway-metrics-and-dimensions.html)
- [VPC Link](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/getting-started-with-private-integration.html)
- [Deployment Canary](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/canary-release.html)
- [WebSocket APIs](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/apigateway-websocket-api.html)
- [Custom Domain Names](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/how-to-custom-domains.html)
- [Security in API Gateway](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/security.html)
