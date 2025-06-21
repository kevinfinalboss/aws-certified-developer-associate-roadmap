<p align="center">
  <img src="https://cdn.worldvectorlogo.com/logos/aws-cloudfront.svg" alt="aws-cloudfront-icon" style="height:150px; width:150px;" /> 
  <br />
  <h1 align="center">
    Amazon CloudFront
  </h1>
</p>

<br />

## Índice

- [O que é Amazon CloudFront?](#o-que-é-amazon-cloudfront)
- [Como funciona o CloudFront](#como-funciona-o-cloudfront)
- [Tipos de distribuição](#tipos-de-distribuição)
- [Protocolos suportados](#protocolos-suportados)
- [Edge Locations](#edge-locations)
- [Benefícios do CloudFront](#benefícios-do-cloudfront)
- [Integração com outros serviços AWS](#integração-com-outros-serviços-aws)
- [Segurança no CloudFront](#segurança-no-cloudfront)
- [Limitações e custos](#limitações-e-custos)
- [Questões para prova AWS Developer Associate](#questões-para-prova-aws-developer-associate)
- [Referências](#referências)


<br />

---

## O que é Amazon CloudFront?

Amazon CloudFront é uma **Content Delivery Network (CDN)** da AWS que distribui conteúdo estático e dinâmico globalmente, garantindo baixa latência e alta velocidade ao entregar arquivos e aplicações para usuários finais usando uma rede de servidores distribuídos chamados **Edge Locations**.

---

## Como funciona o CloudFront

O CloudFront armazena cópias do conteúdo em cache em várias Edge Locations pelo mundo. Quando um usuário acessa o conteúdo, ele é servido a partir da Edge Location mais próxima, melhorando a performance e reduzindo a latência.

---

## Tipos de distribuição

- **Distribuição Web:** Para conteúdo HTTP/HTTPS como websites, APIs, vídeos e arquivos estáticos.
- **Distribuição RTMP:** Para streaming de mídia on-demand usando o protocolo RTMP (menos comum atualmente).

---

## Protocolos suportados

- HTTP e HTTPS para entrega segura.
- TLS para criptografia ponta-a-ponta.
- Suporte para WebSocket para comunicação bidirecional em tempo real.

---

## Edge Locations

Locais físicos onde o CloudFront armazena conteúdo em cache, distribuídos globalmente para aproximar o conteúdo dos usuários, reduzindo o tempo de resposta.

---

## Benefícios do CloudFront

- Redução da latência
- Alta disponibilidade e escalabilidade automática
- Segurança integrada (HTTPS, WAF, Shield)
- Customização via Lambda@Edge
- Cache eficiente

---

## Integração com outros serviços AWS

- Amazon S3 para armazenamento de conteúdo estático.
- Elastic Load Balancer para aplicações dinâmicas.
- AWS Lambda@Edge para lógica personalizada na borda.
- AWS WAF e Shield para segurança.

---

## Segurança no CloudFront

- Suporte a HTTPS e TLS
- Integração com AWS WAF (firewall de aplicações web)
- AWS Shield para proteção DDoS
- Controle de acesso via assinaturas de URLs/cookies
- Restrição geográfica

---

## Limitações e custos

- Custo baseado em transferência de dados e requisições
- Configurações de cache impactam custos e performance
- Distribuição RTMP está em desuso

---

## Questões para prova AWS Developer Associate

### 1. O que é Amazon CloudFront e qual o seu principal objetivo?

**Resposta:**  
Amazon CloudFront é uma CDN (Content Delivery Network) da AWS que distribui conteúdo globalmente para usuários finais com baixa latência e alta velocidade. Seu principal objetivo é acelerar a entrega de conteúdo estático e dinâmico usando uma rede de servidores (Edge Locations) próximos ao usuário final.

**Explicação:**  
A CDN reduz o tempo que leva para entregar conteúdo ao usuário, replicando e armazenando o conteúdo em servidores próximos a ele.

---

### 2. Quais são os dois tipos de distribuições que o CloudFront suporta?

**Resposta:**  
- Distribuição Web (HTTP/HTTPS)  
- Distribuição RTMP (para streaming on-demand via Flash)

**Explicação:**  
A distribuição Web é a mais utilizada hoje em dia para qualquer tipo de conteúdo HTTP/HTTPS. A RTMP é antiga e pouco usada atualmente.

---

### 3. Como o CloudFront reduz a latência no acesso ao conteúdo?

**Resposta:**  
Servindo o conteúdo a partir da Edge Location mais próxima do usuário, evitando que o dado precise viajar até o servidor de origem.

**Explicação:**  
Isso reduz a distância física e o número de hops na rede, diminuindo o tempo de carregamento.

---

### 4. Qual a diferença entre distribuição Web e RTMP no CloudFront?

**Resposta:**  
Distribuição Web entrega conteúdo via HTTP/HTTPS para sites, APIs e streaming moderno. RTMP entrega streaming de mídia usando o protocolo Real-Time Messaging Protocol, focado em mídia sob demanda via Flash.

**Explicação:**  
RTMP está sendo substituído por métodos mais modernos como HLS e DASH via HTTP.

---

### 5. Como o CloudFront pode ser integrado com AWS Lambda@Edge e para que serve?

**Resposta:**  
Lambda@Edge permite executar funções Lambda próximas dos usuários nas Edge Locations para personalizar requisições/respostas, como reescrever URLs, validar autenticação ou modificar cabeçalhos.

**Explicação:**  
Isso melhora a experiência do usuário ao aplicar lógica personalizada sem necessidade de backend centralizado.

---

### 6. Quais medidas de segurança o CloudFront oferece para proteger conteúdo e aplicações?

**Resposta:**  
- Suporte HTTPS/TLS  
- Integração com AWS WAF para firewall de aplicação  
- Proteção contra DDoS com AWS Shield  
- URLs e cookies assinados para controle de acesso  
- Restrição geográfica para bloquear países

**Explicação:**  
Essas funcionalidades garantem que o conteúdo seja entregue de forma segura e protegida contra ameaças.

---

### 7. Explique como o CloudFront trabalha junto com Amazon S3.

**Resposta:**  
Amazon S3 pode ser usado como origem para o CloudFront, que fará cache do conteúdo armazenado no S3 nas Edge Locations para acelerar o acesso.

**Explicação:**  
Isso é comum para servir sites estáticos, imagens e vídeos armazenados no S3.

---

### 8. O que são Edge Locations no contexto do CloudFront?

**Resposta:**  
São data centers localizados globalmente onde o CloudFront armazena o conteúdo em cache para entrega rápida ao usuário.

**Explicação:**  
Esses pontos de presença aumentam a performance e reduzem latência.

---

### 9. Como funciona a política de cache no CloudFront?

**Resposta:**  
Você pode configurar TTL (tempo de vida) para o conteúdo em cache, invalidar cache manualmente, e definir quais cabeçalhos, cookies e query strings influenciam o cache.

**Explicação:**  
Uma boa configuração evita atrasos e mantém o conteúdo atualizado sem carregar demais o backend.

---

### 10. Como o CloudFront pode ser usado para entregar APIs de forma eficiente?

**Resposta:**  
Distribuindo endpoints de API via CloudFront, aproveitando cache para respostas estáticas e usando Lambda@Edge para manipulação dinâmica de requisições, melhorando latência e escalabilidade.

**Explicação:**  
Reduz a carga direta no backend e acelera a entrega global.

---

## Referências

- [O que é o Amazon CloudFront?](https://docs.aws.amazon.com/pt_br/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)  
- [Documentação oficial do AWS CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Welcome.html)  
- [AWS Lambda@Edge](https://docs.aws.amazon.com/lambda/latest/dg/lambda-edge.html)  
- [AWS WAF e Shield](https://aws.amazon.com/waf/)

---

Feito com ♥ by :man_astronaut: Guilherme Bezerra :wave: [Entrar em contato!](https://www.linkedin.com/in/gbdsantos/)
