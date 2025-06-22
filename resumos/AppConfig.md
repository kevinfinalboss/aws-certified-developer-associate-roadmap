<p align="center">
  <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Management-Governance/64/Arch_AWS-AppConfig_64@5x.png" alt="aws-appconfig-icon" style="height:150px; width:150px;" /> 
  <br />
  <h1 align="center">
    AWS AppConfig
  </h1>
</p>

<br />

## Índice

- [O que é AWS AppConfig?](#o-que-é-aws-appconfig)
- [Componentes principais](#componentes-principais)
- [Configuração e deploy](#configuração-e-deploy)
- [Estratégias de deployment](#estratégias-de-deployment)
- [Validação de configurações](#validação-de-configurações)
- [Rollback automático](#rollback-automático)
- [Feature flags](#feature-flags)
- [Integração com aplicações](#integração-com-aplicações)
- [Monitoramento e alarmes](#monitoramento-e-alarmes)
- [Limitações e custos](#limitações-e-custos)
- [Questões para prova AWS Developer Associate](#questões-para-prova-aws-developer-associate)
- [Referências](#referências)

<br />

---

## O que é AWS AppConfig?

AWS AppConfig é um **serviço de gerenciamento de configurações** que permite implantar configurações de aplicação de forma segura e monitorada. Faz parte do AWS Systems Manager e oferece validação, deployment gradual e rollback automático para mudanças de configuração.

---

## Componentes principais

**Application:** Container lógico para configurações relacionadas  
**Environment:** Representação de onde a aplicação roda (dev, staging, prod)  
**Configuration Profile:** Define fonte e tipo da configuração  
**Deployment Strategy:** Define como implantar configurações  
**Validator:** Valida configurações antes do deployment

---

## Configuração e deploy

**Processo de deployment:**

**1. Criação da Application:**
- Nome e descrição
- Tags para organização
- Agrupa configurações relacionadas

**2. Definição de Environments:**
- Separação por estágio (dev/staging/prod)
- Monitors específicos por ambiente
- IAM permissions granulares

**3. Configuration Profile:**
- Source location (S3, Parameter Store, Document Store)
- Content type (JSON, YAML, text, feature flags)
- Validators opcionais

**4. Deployment Strategy:**
- Timing e percentage de rollout
- Bake time para observação
- Grow factor para expansão gradual

---

## Estratégias de deployment

**Predefined Strategies:**

**All at once:**
- Deploy imediato para 100% dos targets
- Sem bake time
- Risco alto mas deploy rápido

**Linear:**
- Crescimento linear (ex: 10% a cada 2 minutos)
- Bake time entre incrementos
- Balanceio entre velocidade e segurança

**Exponential:**
- Crescimento exponencial (ex: 1%, 2%, 4%, 8%)
- Validação progressiva
- Detecção rápida de problemas

**Custom Strategies:**
- Controle total sobre timing
- Grow intervals personalizados
- Final bake time configurável

---

## Validação de configurações

**Tipos de validators:**

**JSON Schema Validator:**
- Valida estrutura JSON
- Type checking automático
- Required fields validation

**Lambda Validator:**
- Lógica customizada de validação
- Business rules específicas
- Integração com sistemas externos

**Validation process:**
- Pre-deployment validation
- Blocking vs non-blocking
- Error reporting detalhado

---

## Rollback automático

**CloudWatch Alarms Integration:**
- Monitora métricas específicas
- Rollback automático em caso de alarme
- Configurable alarm thresholds

**Rollback triggers:**
- Error rates aumentadas
- Latency spikes
- Custom business metrics
- Health check failures

**Rollback behavior:**
- Immediate stop de novo deployment
- Revert para versão anterior
- Notification automática
- Detailed rollback logs

---

## Feature flags

**Características:**
- Boolean flags para features
- Percentage-based rollouts
- User segment targeting
- A/B testing capabilities

**Configuração:**
- JSON-based flag definitions
- Conditional logic support
- Default values por environment
- Real-time flag updates

**Casos de uso:**
- Gradual feature rollouts
- Kill switches para problemas
- A/B testing de features
- Environment-specific behaviors

---

## Integração com aplicações

**AppConfig Agent:**
- Local agent para polling
- Caching automático
- Reduced API calls
- Improved performance

**SDKs disponíveis:**
- Java, .NET, Python, Node.js
- Direct API integration
- Polling e callback mechanisms
- Error handling built-in

**Polling strategies:**
- Fixed intervals
- Exponential backoff
- Event-driven updates
- Cached responses

---

## Monitoramento e alarmes

**CloudWatch Integration:**
- Deployment metrics
- Configuration retrieve metrics
- Error rates tracking
- Custom metrics support

**Key metrics:**
- ConfigurationRetrievals
- DeploymentDuration
- InvalidConfigurations
- RollbackTriggers

**Alerting:**
- Failed deployments
- High error rates
- Rollback events
- Validation failures

---

## Limitações e custos

**Limites de serviço:**
- Maximum de 100 applications por region
- Configuration size até 1MB
- Maximum de 50 deployment strategies
- Rate limits para API calls

**Custos:**
- Por configuration request
- Hosted configuration pricing
- CloudWatch metrics charges
- No upfront costs

**Best practices:**
- Use caching para reduzir requests
- Batch configuration updates
- Monitor usage patterns
- Optimize polling intervals

---

## Questões para prova AWS Developer Associate

### 1. O que é AWS AppConfig e qual seu principal objetivo?

**Resposta:**  
AWS AppConfig é um serviço de gerenciamento de configurações que permite implantar mudanças de configuração de forma segura com validação, deployment gradual e rollback automático.

**Explicação:**  
AppConfig reduz o risco de mudanças de configuração oferecendo controles de segurança e monitoramento automático.

---

### 2. Quais são os componentes principais do AppConfig?

**Resposta:**  
Application (container), Environment (onde roda), Configuration Profile (fonte dos dados), Deployment Strategy (como implantar) e Validators (validação).

**Explicação:**  
Cada componente tem uma função específica no processo de gerenciamento de configurações.

---

### 3. Qual a diferença entre deployment strategies no AppConfig?

**Resposta:**  
All at once (deploy imediato), Linear (crescimento linear), Exponential (crescimento exponencial) e Custom (totalmente configurável). Cada uma oferece diferentes balanceios entre velocidade e segurança.

**Explicação:**  
A escolha da strategy depende do nível de risco aceitável e velocidade de deployment desejada.

---

### 4. Como funciona a validação de configurações no AppConfig?

**Resposta:**  
AppConfig oferece JSON Schema Validator para validação estrutural e Lambda Validator para lógica customizada. Validações podem ser blocking ou non-blocking.

**Explicação:**  
A validação previne deployment de configurações inválidas que poderiam causar problemas na aplicação.

---

### 5. Como o AppConfig implementa rollback automático?

**Resposta:**  
Integração com CloudWatch Alarms que monitora métricas específicas. Quando alarmes são triggered, AppConfig automaticamente reverte para a configuração anterior.

**Explicação:**  
Isso permite detecção e correção rápida de problemas causados por mudanças de configuração.

---

### 6. O que são feature flags no AppConfig?

**Resposta:**  
Feature flags são configurações boolean que permitem habilitar/desabilitar features, fazer rollouts graduais e A/B testing sem deploy de código.

**Explicação:**  
Feature flags permitem controle granular sobre funcionalidades da aplicação em runtime.

---

### 7. Como as aplicações consomem configurações do AppConfig?

**Resposta:**  
Através de SDKs específicos, AppConfig Agent local, ou direct API calls. Suporte a polling, caching e callback mechanisms.

**Explicação:**  
Diferentes métodos de integração permitem escolher a abordagem mais adequada para cada aplicação.

---

### 8. Quais fontes de configuração o AppConfig suporta?

**Resposta:**  
S3 buckets, Systems Manager Parameter Store, Systems Manager Documents e Hosted configurations dentro do próprio AppConfig.

**Explicação:**  
Multiple sources oferecem flexibilidade para diferentes padrões de armazenamento de configuração.

---

### 9. Como monitorar deployments no AppConfig?

**Resposta:**  
Através de CloudWatch metrics como ConfigurationRetrievals, DeploymentDuration e error rates. Também suporta custom metrics e alerting.

**Explicação:**  
Monitoramento é essencial para identificar problemas e otimizar o processo de deployment.

---

### 10. Quando usar AppConfig vs Parameter Store vs Secrets Manager?

**Resposta:**  
Use AppConfig para configurações que mudam frequentemente e precisam de deployment controlado. Parameter Store para configurações simples. Secrets Manager para dados sensíveis como passwords.

**Explicação:**  
Cada serviço tem casos de uso específicos baseados no tipo de dado e requirements de segurança.

---

## Referências

- [O que é AWS AppConfig?](https://aws.amazon.com/systems-manager/features/appconfig/)  
- [Documentação oficial do AWS AppConfig](https://docs.aws.amazon.com/appconfig/)  
- [AppConfig User Guide](https://docs.aws.amazon.com/appconfig/latest/userguide/)  
- [Deployment Strategies](https://docs.aws.amazon.com/appconfig/latest/userguide/appconfig-creating-deployment-strategy.html)  
- [Configuration Validation](https://docs.aws.amazon.com/appconfig/latest/userguide/appconfig-creating-configuration-and-profile.html)

---
