<p align="center">
  <img src="https://miro.medium.com/v2/resize:fit:1200/1*vr5UecdIEaM1Y8I-8lSl7A.jpeg" alt="aws-amplify-icon" style="height:150px; width:150px;" /> 
  <br />
  <h1 align="center">
    AWS Amplify
  </h1>
</p>

<br />

## Índice

- [O que é AWS Amplify?](#o-que-é-aws-amplify)
- [Componentes do Amplify](#componentes-do-amplify)
- [Amplify CLI](#amplify-cli)
- [Amplify Hosting](#amplify-hosting)
- [Amplify Libraries](#amplify-libraries)
- [Amplify Studio](#amplify-studio)
- [Benefícios do Amplify](#benefícios-do-amplify)
- [Integração com outros serviços AWS](#integração-com-outros-serviços-aws)
- [Autenticação e autorização](#autenticação-e-autorização)
- [Limitações e custos](#limitações-e-custos)
- [Questões para prova AWS Developer Associate](#questões-para-prova-aws-developer-associate)
- [Referências](#referências)

<br />

---

## O que é AWS Amplify?

AWS Amplify é uma **plataforma de desenvolvimento full-stack** que permite aos desenvolvedores criar, implantar e hospedar aplicações web e mobile modernas rapidamente. Oferece ferramentas para frontend, backend, CI/CD e hospedagem em uma solução integrada.

---

## Componentes do Amplify

**Amplify CLI:** Ferramenta de linha de comando para configurar recursos AWS  
**Amplify Hosting:** Serviço de hospedagem para aplicações web estáticas  
**Amplify Libraries:** SDKs para diferentes linguagens e frameworks  
**Amplify Studio:** Interface visual para configurar backend e frontend

---

## Amplify CLI

Ferramenta principal para:
- Configurar backend (APIs, bancos de dados, autenticação)
- Provisionar recursos AWS automaticamente
- Gerenciar ambientes (dev, staging, prod)
- Deploy de aplicações
- Configurar CI/CD pipelines

---

## Amplify Hosting

**Características:**
- Hospedagem global via CloudFront
- Deploy automático via Git
- Suporte a frameworks SPA (React, Vue, Angular)
- Preview de branches para testing
- Custom domains e SSL automático

---

## Amplify Libraries

**Suporte para:**
- JavaScript/TypeScript
- React/React Native
- Vue.js
- Angular
- iOS (Swift)
- Android (Kotlin/Java)
- Flutter

---

## Amplify Studio

Interface visual que permite:
- Design de data models
- Configuração de autenticação
- Criação de APIs GraphQL
- Geração de UI components
- Gerenciamento de usuários

---

## Benefícios do Amplify

- **Desenvolvimento rápido:** Templates e scaffolding automático
- **Full-stack:** Frontend e backend integrados
- **Escalabilidade:** Infraestrutura gerenciada pela AWS
- **CI/CD nativo:** Deploy automático via Git
- **Multi-ambiente:** Desenvolvimento, staging e produção

---

## Integração com outros serviços AWS

- **AWS AppSync:** Para APIs GraphQL
- **Amazon Cognito:** Para autenticação
- **AWS Lambda:** Para lógica serverless
- **Amazon S3:** Para armazenamento de arquivos
- **Amazon DynamoDB:** Para banco de dados NoSQL
- **CloudFront:** Para distribuição global

---

## Autenticação e autorização

**Amazon Cognito integrado:**
- Sign-up/Sign-in
- Social providers (Google, Facebook, Amazon)
- Multi-factor authentication (MFA)
- Forgot password
- User groups e roles

---

## Limitações e custos

**Limitações:**
- Focado em aplicações modernas (SPA)
- Menos flexibilidade que configuração manual
- Dependência do ecossistema AWS

**Custos:**
- Amplify Hosting: por GB transferido
- Recursos backend: baseado no uso (Lambda, DynamoDB, etc.)
- Build minutes para CI/CD

---

## Questões para prova AWS Developer Associate

### 1. O que é AWS Amplify e qual seu principal objetivo?

**Resposta:**  
AWS Amplify é uma plataforma de desenvolvimento full-stack para criar, implantar e hospedar aplicações web e mobile rapidamente. Seu objetivo é acelerar o desenvolvimento fornecendo ferramentas integradas para frontend, backend e deploy.

**Explicação:**  
Amplify abstrai a complexidade da infraestrutura AWS permitindo que desenvolvedores foquem no código da aplicação.

---

### 2. Quais são os principais componentes do AWS Amplify?

**Resposta:**  
Amplify CLI (ferramenta de linha de comando), Amplify Hosting (hospedagem), Amplify Libraries (SDKs) e Amplify Studio (interface visual).

**Explicação:**  
Cada componente atende uma necessidade específica no ciclo de desenvolvimento de aplicações modernas.

---

### 3. Como funciona o deploy automático no Amplify Hosting?

**Resposta:**  
Amplify conecta-se ao repositório Git e faz deploy automático quando há commits nas branches configuradas, incluindo build, test e deploy da aplicação.

**Explicação:**  
Isso implementa CI/CD nativo sem necessidade de configuração adicional de pipelines.

---

### 4. Qual serviço AWS o Amplify usa para distribuição global de aplicações?

**Resposta:**  
Amazon CloudFront para distribuir o conteúdo globalmente através de edge locations.

**Explicação:**  
Isso garante baixa latência e alta performance para usuários em qualquer localização.

---

### 5. Como o Amplify lida com diferentes ambientes (dev, staging, prod)?

**Resposta:**  
Amplify permite criar múltiplos ambientes onde cada um pode ter sua própria configuração de backend, permitindo isolamento entre desenvolvimento, teste e produção.

**Explicação:**  
Isso facilita o desenvolvimento em equipe e testing sem afetar produção.

---

### 6. Qual serviço o Amplify usa para autenticação de usuários?

**Resposta:**  
Amazon Cognito para gerenciar autenticação, incluindo sign-up, sign-in, social providers e MFA.

**Explicação:**  
Cognito oferece solução completa de gerenciamento de identidade integrada ao Amplify.

---

### 7. Como o Amplify cria APIs GraphQL?

**Resposta:**  
Usando AWS AppSync integrado, onde você define o schema GraphQL e o Amplify provisiona automaticamente resolvers, DynamoDB tables e configurações necessárias.

**Explicação:**  
Isso permite criar APIs completas sem configuração manual de infraestrutura.

---

### 8. Quais frameworks frontend são suportados pelo Amplify?

**Resposta:**  
React, Vue.js, Angular, React Native para mobile, além de vanilla JavaScript. Também suporta iOS, Android e Flutter.

**Explicação:**  
Amplify oferece libraries específicas para cada framework facilitando a integração.

---

### 9. Como funciona o feature de preview branches no Amplify?

**Resposta:**  
Para cada branch do Git, Amplify pode criar um preview environment com URL única, permitindo testar mudanças isoladamente antes do merge.

**Explicação:**  
Isso facilita code review e testing de features antes de afetar produção.

---

### 10. Qual é a vantagem de usar Amplify Studio?

**Resposta:**  
Amplify Studio oferece interface visual para configurar backend, criar data models, configurar autenticação e até gerar UI components, reduzindo a necessidade de código manual.

**Explicação:**  
Isso acelera o desenvolvimento especialmente para desenvolvedores menos familiarizados com AWS.

---

## Referências

- [O que é AWS Amplify?](https://aws.amazon.com/amplify/)  
- [Documentação oficial do AWS Amplify](https://docs.amplify.aws/)  
- [Amplify CLI](https://docs.amplify.aws/cli/)  
- [Amplify Hosting](https://docs.aws.amazon.com/amplify/latest/userguide/welcome.html)  
- [Amplify Libraries](https://docs.amplify.aws/lib/)

---

Feito com ♥ by Claude AI :wave:
