<p align="center">
	<img src="./img/aws-icons/aws-Cognito.png" alt="aws-cognito-icon" style="height:150px; width:150px;" /> 
  <br />
	<h1 align="center">
    Cognito
  </h1>
</p>	
<br />

## :pushpin: Índice
- [Introdução](#introdução)
- [Cognito User Pools](#cognito-user-pools)
  - [Características Principais](#características-principais-user-pools)
  - [Autenticação e Autorização](#autenticação-e-autorização)
  - [Tokens JWT](#tokens-jwt)
  - [Triggers Lambda](#triggers-lambda)
  - [Hosted UI](#hosted-ui)
  - [Federação de Identidades](#federação-de-identidades)
- [Cognito Identity Pools](#cognito-identity-pools)
  - [Características Principais](#características-principais-identity-pools)
  - [Roles IAM](#roles-iam)
  - [Usuários Autenticados vs Não Autenticados](#usuários-autenticados-vs-não-autenticados)
  - [Identity Providers](#identity-providers)
- [Integração com Serviços AWS](#integração-com-serviços-aws)
  - [API Gateway](#api-gateway)
  - [Application Load Balancer](#application-load-balancer)
  - [AppSync](#appsync)
- [Fluxos de Autenticação](#fluxos-de-autenticação)
- [Segurança e MFA](#segurança-e-mfa)
- [Conceitos de Prova](#conceitos-de-prova)
- [Quiz](#quiz)
- [Referências](#books-referências)

<br />

## Introdução

Amazon Cognito é um serviço de **gerenciamento de identidade e acesso** que fornece autenticação, autorização e gerenciamento de usuários para aplicações web e móveis. Cognito elimina a necessidade de construir e manter infraestrutura de autenticação customizada.

**Componentes principais:**
- **User Pools**: Diretório de usuários com autenticação
- **Identity Pools**: Credenciais AWS temporárias para usuários
- **Cognito Sync**: (Depreciado - usar AppSync)

**Casos de uso:**
- Autenticação de usuários em aplicações
- Federação com provedores de identidade externos
- Acesso seguro a recursos AWS
- Single Sign-On (SSO)

<br />

## Cognito User Pools

### Características Principais User Pools

User Pools são **diretórios de usuários** que fornecem funcionalidades completas de registro e login.

**Funcionalidades principais:**
- Registro e login de usuários
- Verificação de email/telefone
- Recuperação de senha
- Multi-Factor Authentication (MFA)
- Políticas de senha customizáveis
- Atributos de usuário personalizados

```json
{
  "UserPoolId": "us-east-1_abc123def",
  "ClientId": "1example23456789",
  "Username": "user@example.com",
  "UserAttributes": [
    {
      "Name": "email",
      "Value": "user@example.com"
    },
    {
      "Name": "email_verified",
      "Value": "true"
    }
  ]
}
```

### Autenticação e Autorização

**Fluxos de autenticação suportados:**
- **USER_SRP_AUTH**: Secure Remote Password (padrão)
- **USER_PASSWORD_AUTH**: Username e password direto
- **ADMIN_NO_SRP_AUTH**: Admin iniciado, sem SRP
- **CUSTOM_AUTH**: Fluxo customizado com Lambda

**App Clients:**
- Cada aplicação precisa de um App Client
- Configurações de segurança específicas por client
- Pode ter diferentes fluxos de auth habilitados

```json
{
  "UserPoolId": "us-east-1_abc123def",
  "ClientName": "MyMobileApp",
  "ExplicitAuthFlows": [
    "ALLOW_USER_SRP_AUTH",
    "ALLOW_REFRESH_TOKEN_AUTH"
  ],
  "GenerateSecret": false,
  "RefreshTokenValidity": 30
}
```

### Tokens JWT

User Pools emitem **três tipos de tokens JWT**:

1. **ID Token**: Contém informações sobre o usuário
2. **Access Token**: Usado para autorização de API
3. **Refresh Token**: Renova ID e Access tokens

**Estrutura do ID Token:**
```json
{
  "sub": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
  "aud": "xxxxxxxxxxxxexample",
  "cognito:groups": ["admin"],
  "email_verified": true,
  "iss": "https://cognito-idp.us-west-2.amazonaws.com/us-west-2_example",
  "cognito:username": "janedoe",
  "aud": "xxxxxxxxxxxxexample",
  "event_id": "01234567-0123-0123-0123-012345678901",
  "token_use": "id",
  "auth_time": 1545420314,
  "exp": 1545423914,
  "iat": 1545420314,
  "email": "janedoe@example.com"
}
```

### Triggers Lambda

User Pools suportam **triggers Lambda** em diferentes eventos:

**Pre-authentication triggers:**
- **Pre-sign-up**: Validação customizada antes do registro
- **Pre-authentication**: Validação antes do login
- **Custom message**: Personalizar mensagens de verificação

**Post-authentication triggers:**
- **Post-confirmation**: Ações após confirmação de conta
- **Post-authentication**: Ações após login bem-sucedido

```javascript
exports.handler = async (event) => {
    if (event.triggerSource === 'PreSignUp_SignUp') {
        const email = event.request.userAttributes.email;
        if (!email.endsWith('@company.com')) {
            throw new Error('Only company emails allowed');
        }
    }
    return event;
};
```

### Hosted UI

Cognito fornece uma **interface web hospedada** para login/registro:

**Características:**
- UI responsiva pronta para uso
- Suporte a CSS customizado
- Integração com provedores sociais
- OAuth 2.0 e OIDC compliance

**URL padrão:**
```
https://your-domain.auth.region.amazoncognito.com/login?client_id=your-client-id&response_type=code&scope=openid&redirect_uri=your-callback-url
```

### Federação de Identidades

User Pools suportam federação com **provedores externos**:

**Provedores suportados:**
- **Social**: Facebook, Google, Amazon, Apple
- **SAML**: Active Directory, Okta, etc.
- **OIDC**: Qualquer provedor OpenID Connect

**Configuração SAML:**
```json
{
  "ProviderName": "ExampleSAML",
  "ProviderType": "SAML",
  "MetadataURL": "https://example.com/saml/metadata",
  "AttributeMapping": {
    "email": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
    "given_name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname"
  }
}
```

<br />

## Cognito Identity Pools

### Características Principais Identity Pools

Identity Pools fornecem **credenciais AWS temporárias** para usuários autenticados e não autenticados.

**Funcionalidades:**
- Credenciais temporárias AWS (STS)
- Suporte a usuários convidados (não autenticados)
- Mapeamento de roles IAM
- Integração com múltiplos provedores de identidade

### Roles IAM

Identity Pools usam **duas roles IAM**:

**Authenticated Role**: Para usuários logados
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/${cognito-identity.amazonaws.com:sub}/*"
    }
  ]
}
```

**Unauthenticated Role**: Para usuários convidados
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::my-public-bucket/*"
    }
  ]
}
```

### Usuários Autenticados vs Não Autenticados

**Usuários Autenticados:**
- Fazem login via User Pool ou provedor externo
- Recebem credenciais com mais permissões
- Podem acessar recursos privados

**Usuários Não Autenticados (Guest):**
- Não precisam fazer login
- Recebem credenciais limitadas
- Acesso apenas a recursos públicos

### Identity Providers

Identity Pools aceitam tokens de múltiplos provedores:

```json
{
  "IdentityPoolId": "us-east-1:12345678-1234-1234-1234-123456789012",
  "AllowUnauthenticatedIdentities": true,
  "SupportedLoginProviders": {
    "accounts.google.com": "your-google-client-id",
    "graph.facebook.com": "your-facebook-app-id",
    "cognito-idp.us-east-1.amazonaws.com/us-east-1_abc123def": null
  }
}
```

<br />

## Integração com Serviços AWS

### API Gateway

**Cognito User Pool Authorizer:**
```yaml
Resources:
  MyApi:
    Type: AWS::ApiGateway::RestApi
    Properties:
      Name: MyAPI
  
  MyAuthorizer:
    Type: AWS::ApiGateway::Authorizer
    Properties:
      Name: CognitoAuthorizer
      Type: COGNITO_USER_POOLS
      IdentitySource: method.request.header.Authorization
      RestApiId: !Ref MyApi
      ProviderARNs:
        - !GetAtt MyCognitoUserPool.Arn
```

**Headers necessários na requisição:**
```javascript
const headers = {
    'Authorization': `Bearer ${idToken}`,
    'Content-Type': 'application/json'
};
```

### Application Load Balancer

ALB pode **autenticar usuários** usando Cognito:

```yaml
ListenerRule:
  Type: AWS::ElasticLoadBalancingV2::ListenerRule
  Properties:
    Actions:
      - Type: authenticate-cognito
        AuthenticateCognitoConfig:
          UserPoolArn: !GetAtt UserPool.Arn
          UserPoolClientId: !Ref UserPoolClient
          UserPoolDomain: my-app-domain
        Order: 1
      - Type: forward
        TargetGroupArn: !Ref TargetGroup
        Order: 2
```

### AppSync

**GraphQL API com Cognito:**
```yaml
GraphQLApi:
  Type: AWS::AppSync::GraphQLApi
  Properties:
    Name: MyAPI
    AuthenticationType: AMAZON_COGNITO_USER_POOLS
    UserPoolConfig:
      UserPoolId: !Ref UserPool
      AwsRegion: !Ref AWS::Region
      DefaultAction: ALLOW
```

<br />

## Fluxos de Autenticação

### Fluxo User Pool + Identity Pool

1. **Usuário faz login** no User Pool
2. **Recebe tokens JWT** (ID, Access, Refresh)
3. **Troca ID token** por credenciais AWS no Identity Pool
4. **Usa credenciais temporárias** para acessar serviços AWS

```javascript
const authUser = await Auth.signIn(username, password);
const session = await Auth.currentSession();
const idToken = session.getIdToken().getJwtToken();

AWS.config.credentials = new AWS.CognitoIdentityCredentials({
    IdentityPoolId: 'us-east-1:12345678-abcd-1234-efgh-123456789012',
    Logins: {
        'cognito-idp.us-east-1.amazonaws.com/us-east-1_abc123def': idToken
    }
});
```

### Fluxo Social Login

1. **Usuário faz login** no provedor social (Google, Facebook)
2. **Provedor retorna token** de acesso
3. **Token é enviado** para Identity Pool
4. **Identity Pool retorna** credenciais AWS

```javascript
Auth.federatedSignIn({provider: 'Google'})
    .then(credentials => {
        console.log('AWS Credentials:', credentials);
    });
```

<br />

## Segurança e MFA

### Multi-Factor Authentication

**Tipos de MFA suportados:**
- **SMS**: Código via mensagem de texto
- **TOTP**: Time-based One-Time Password (Google Authenticator)
- **Software tokens**: Aplicativos autenticadores

**Configurações de MFA:**
- **OFF**: MFA desabilitado
- **OPTIONAL**: Usuário escolhe se quer MFA
- **REQUIRED**: MFA obrigatório para todos

```json
{
  "MfaConfiguration": "REQUIRED",
  "EnabledMfas": ["SMS_MFA", "SOFTWARE_TOKEN_MFA"],
  "SmsConfiguration": {
    "SnsCallerArn": "arn:aws:iam::123456789012:role/service-role/CognitoSNSRole",
    "ExternalId": "12345678-1234-1234-1234-123456789012"
  }
}
```

### Políticas de Senha

```json
{
  "PasswordPolicy": {
    "MinimumLength": 8,
    "RequireUppercase": true,
    "RequireLowercase": true,
    "RequireNumbers": true,
    "RequireSymbols": true,
    "TemporaryPasswordValidityDays": 7
  }
}
```

### Account Takeover Protection

```json
{
  "AccountTakeoverRiskConfiguration": {
    "NotifyConfiguration": {
      "From": "noreply@example.com",
      "SourceArn": "arn:aws:ses:us-east-1:123456789012:identity/example.com"
    },
    "Actions": {
      "LowAction": {
        "Notify": true,
        "EventAction": "NO_ACTION"
      },
      "MediumAction": {
        "Notify": true,
        "EventAction": "MFA_IF_CONFIGURED"
      },
      "HighAction": {
        "Notify": true,
        "EventAction": "MFA_REQUIRED"
      }
    }
  }
}
```

<br />

## Conceitos de Prova

### 1. User Pool vs Identity Pool
- **User Pool**: Autenticação (quem você é)
- **Identity Pool**: Autorização (o que você pode fazer)

### 2. Tokens JWT
- **ID Token**: Informações do usuário
- **Access Token**: Autorização de APIs
- **Refresh Token**: Renovar outros tokens

### 3. Federação
- User Pools: Federam com provedores para autenticação
- Identity Pools: Aceitam tokens de qualquer provedor

### 4. Roles IAM
- Identity Pools sempre precisam de roles IAM
- User Pools não precisam de roles IAM

### 5. Usuários Não Autenticados
- Apenas Identity Pools suportam usuários guest
- User Pools sempre requerem autenticação

### 6. Integração com API Gateway
- User Pool Authorizer: Valida JWT tokens
- IAM Authorizer: Usa credenciais do Identity Pool

<br />

## Quiz

### 1. Qual a principal diferença entre User Pools e Identity Pools?
**a)** User Pools são para mobile, Identity Pools para web  
**b)** User Pools fazem autenticação, Identity Pools fornecem credenciais AWS  
**c)** User Pools são gratuitos, Identity Pools são pagos  
**d)** Não há diferença significativa

**Resposta:** b) User Pools fazem autenticação, Identity Pools fornecem credenciais AWS

**Explicação:** User Pools são diretórios de usuários que fazem autenticação (quem você é), enquanto Identity Pools fornecem credenciais AWS temporárias para autorização (o que você pode fazer nos serviços AWS).

### 2. Quantos tipos de tokens JWT um User Pool emite?
**a)** 1 (apenas access token)  
**b)** 2 (access e refresh)  
**c)** 3 (ID, access e refresh)  
**d)** 4 (ID, access, refresh e session)

**Resposta:** c) 3 (ID, access e refresh)

**Explicação:** User Pools emitem três tipos de tokens: ID token (informações do usuário), Access token (autorização), e Refresh token (renovar os outros tokens).

### 3. Qual token deve ser usado para autorização em API Gateway?
**a)** ID Token  
**b)** Access Token  
**c)** Refresh Token  
**d)** Session Token

**Resposta:** a) ID Token

**Explicação:** Para User Pool Authorizer no API Gateway, deve-se usar o ID Token no header Authorization. O Access Token é usado para APIs do próprio Cognito.

### 4. Identity Pools podem fornecer acesso a usuários não autenticados?
**a)** Não, apenas usuários autenticados  
**b)** Sim, através de role de usuário convidado  
**c)** Apenas com permissão especial da AWS  
**d)** Apenas para aplicações móveis

**Resposta:** b) Sim, através de role de usuário convidado

**Explicação:** Identity Pools suportam usuários não autenticados (guest) através de uma role IAM específica para unauthenticated users, permitindo acesso limitado a recursos AWS.

### 5. Qual serviço substituiu o Cognito Sync?
**a)** Cognito Identity  
**b)** AppSync  
**c)** DynamoDB  
**d)** S3 Sync

**Resposta:** b) AppSync

**Explicação:** Cognito Sync foi depreciado e substituído pelo AWS AppSync, que oferece sincronização de dados em tempo real com capacidades GraphQL.

### 6. Qual fluxo de autenticação é o padrão (mais seguro) em User Pools?
**a)** USER_PASSWORD_AUTH  
**b)** ADMIN_NO_SRP_AUTH  
**c)** USER_SRP_AUTH  
**d)** CUSTOM_AUTH

**Resposta:** c) USER_SRP_AUTH

**Explicação:** USER_SRP_AUTH (Secure Remote Password) é o fluxo padrão e mais seguro, pois nunca envia a senha em texto claro pela rede.

### 7. Como o Application Load Balancer pode usar Cognito?
**a)** Apenas como Identity Pool  
**b)** Apenas como User Pool  
**c)** Como autenticador antes de rotear tráfego  
**d)** Não é possível integrar

**Resposta:** c) Como autenticador antes de rotear tráfego

**Explicação:** ALB pode usar User Pools para autenticar usuários antes de rotear tráfego para targets, redirecionando usuários não autenticados para a página de login do Cognito.

### 8. Qual é o propósito dos triggers Lambda em User Pools?
**a)** Substituir a autenticação padrão  
**b)** Personalizar fluxos de autenticação e registro  
**c)** Fazer backup dos dados  
**d)** Integrar com outros serviços AWS

**Resposta:** b) Personalizar fluxos de autenticação e registro

**Explicação:** Triggers Lambda permitem executar código customizado em diferentes pontos do fluxo de autenticação, como validação antes do registro, personalização de mensagens, etc.

### 9. Qual é a vantagem do Hosted UI do Cognito?
**a)** É gratuito  
**b)** Interface pronta para uso com suporte a OAuth  
**c)** Funciona apenas em mobile  
**d)** Não requer configuração

**Resposta:** b) Interface pronta para uso com suporte a OAuth

**Explicação:** Hosted UI fornece uma interface web responsiva pronta para uso, com suporte completo a OAuth 2.0, OIDC e integração com provedores sociais, sem necessidade de desenvolver interface customizada.

### 10. Como Identity Pools determinam qual role IAM usar?
**a)** Baseado no username  
**b)** Baseado se o usuário está autenticado ou não  
**c)** Sempre usa a mesma role  
**d)** Baseado no provedor de identidade

**Resposta:** b) Baseado se o usuário está autenticado ou não

**Explicação:** Identity Pools usam duas roles: uma para usuários autenticados (authenticated role) e outra para usuários não autenticados (unauthenticated role), determinando automaticamente qual usar.

### 11. Qual variável pode ser usada em políticas IAM para isolar recursos por usuário?
**a)** ${cognito:username}  
**b)** ${cognito-identity.amazonaws.com:sub}  
**c)** ${aws:userid}  
**d)** ${cognito:sub}

**Resposta:** b) ${cognito-identity.amazonaws.com:sub}

**Explicação:** A variável ${cognito-identity.amazonaws.com:sub} contém o identificador único do usuário no Identity Pool e é comumente usada para criar paths únicos por usuário em recursos como S3.

### 12. MFA em User Pools pode ser configurado como:
**a)** Apenas obrigatório ou desabilitado  
**b)** OFF, OPTIONAL, ou REQUIRED  
**c)** Apenas para administradores  
**d)** Apenas via SMS

**Resposta:** b) OFF, OPTIONAL, ou REQUIRED

**Explicação:** MFA em User Pools tem três configurações: OFF (desabilitado), OPTIONAL (usuário escolhe), e REQUIRED (obrigatório para todos os usuários).

### 13. Qual token é usado para renovar outros tokens expirados?
**a)** ID Token  
**b)** Access Token  
**c)** Refresh Token  
**d)** Session Token

**Resposta:** c) Refresh Token

**Explicação:** Refresh Token é usado para obter novos ID e Access tokens quando eles expiram, sem necessidade do usuário fazer login novamente.

### 14. Provedores SAML podem ser integrados com:
**a)** Apenas Identity Pools  
**b)** Apenas User Pools  
**c)** Ambos User Pools e Identity Pools  
**d)** Nenhum dos dois

**Resposta:** c) Ambos User Pools e Identity Pools

**Explicação:** Provedores SAML podem ser integrados tanto com User Pools (para autenticação federada) quanto com Identity Pools (para obter credenciais AWS usando tokens SAML).

### 15. Qual é o limite de tempo padrão para tokens JWT em User Pools?
**a)** ID Token: 1 hora, Access Token: 1 hora  
**b)** ID Token: 24 horas, Access Token: 1 hora  
**c)** ID Token: 1 hora, Access Token: 24 horas  
**d)** Ambos: 30 minutos

**Resposta:** a) ID Token: 1 hora, Access Token: 1 hora

**Explicação:** Por padrão, tanto ID Token quanto Access Token têm validade de 1 hora, enquanto Refresh Token tem validade de 30 dias (configurável).

<br />

## :books: Referências

- [O que é o Amazon Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html)
- [User Pools](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html)
- [Identity Pools](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-identity.html)
- [Cognito Authentication Flow](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-authentication-flow.html)
- [JWT Tokens](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-with-identity-providers.html)

<br />

---
