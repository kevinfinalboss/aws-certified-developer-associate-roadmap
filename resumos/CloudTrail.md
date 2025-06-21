<p align="center">
	<img src="./img/aws-icons/aws-CloudTrail.png" alt="aws-cloudtrail-icon" style="height:150px; width:150px;" /> 
  <br />
	<h1 align="center">
    CloudTrail
  </h1>
</p>	

<br />

## :pushpin: Índice

- [Introdução](#introdução)
- [Eventos do CloudTrail](#eventos-do-cloudtrail)
- [Configuração e Trilha (Trail)](#configuração-e-trilha-trail)
- [Tipos de eventos](#tipos-de-eventos)
- [Integração com outros serviços AWS](#integração-com-outros-serviços-aws)
- [Segurança e compliance](#segurança-e-compliance)
- [Monitoramento e análise](#monitoramento-e-análise)
- [Custos](#custos)
- [Questões para prova AWS Developer Associate](#questões-para-prova-aws-developer-associate)
- [Referências](#referências)

<br />

## Introdução

Amazon **CloudTrail** é um serviço que provê governança, compliance, auditoria e monitoramento da sua conta AWS. 

Ele registra e armazena o histórico de chamadas de API feitas na sua conta, tanto via Console, SDK, CLI ou outros serviços AWS. Isso permite rastrear atividades, mudanças de configuração e operações realizadas.

Por padrão, o CloudTrail está habilitado e grava eventos de gerenciamento. Os logs podem ser enviados para buckets S3 e para o CloudWatch Logs, facilitando análises, alertas e auditoria.

---

## Eventos do CloudTrail

### Tipos de eventos

1. **Eventos de Gerenciamento (Management Events):**
   - São operações relacionadas ao gerenciamento dos recursos da AWS, como criar, modificar ou deletar recursos.
   - Por padrão, são gravados em uma trilha padrão.
   - Úteis para auditoria e governança.

2. **Eventos de Dados (Data Events):**
   - Registros detalhados sobre chamadas de API em recursos específicos, como leitura/escrita em buckets S3 ou invocações de funções Lambda.
   - Não são registrados por padrão devido ao alto volume.
   - Precisam ser habilitados explicitamente em uma trilha para serem monitorados.

3. **Eventos do CloudTrail Insights:**
   - Analisam o histórico de eventos para detectar atividades incomuns, anômalas ou suspeitas.
   - Desabilitados por padrão e geram custo adicional quando ativados.

---

## Configuração e Trilha (Trail)

- Uma **trilha** é uma configuração do CloudTrail que define onde os logs serão entregues (ex: bucket S3, CloudWatch Logs).
- Pode ser criada para cobrir todas as regiões AWS da conta.
- É possível ter múltiplas trilhas para diferentes objetivos (ex: uma trilha para auditoria geral, outra para dados específicos).
- Logs podem ser criptografados com KMS.
- Você pode configurar notificações para eventos específicos via SNS.

---

## Integração com outros serviços AWS

- **Amazon S3:** Armazena os arquivos de log do CloudTrail.
- **AWS CloudWatch Logs:** Recebe eventos em tempo real para monitoramento, alertas e análise.
- **AWS Lambda:** Pode processar eventos para automações de resposta a incidentes.
- **Amazon Athena:** Usado para consultas SQL diretas nos logs armazenados no S3.
- **AWS Config:** Complementa o CloudTrail para monitorar configurações e compliance.

---

## Segurança e compliance

- CloudTrail ajuda a cumprir requisitos regulatórios ao fornecer histórico confiável e imutável de ações na conta.
- Logs podem ser protegidos com criptografia e políticas IAM restritivas.
- Integrar com CloudWatch Events permite responder a ações suspeitas rapidamente.

---

## Monitoramento e análise

- Usar o CloudWatch para criar métricas customizadas e alertas baseados nos logs do CloudTrail.
- Athena permite consultas analíticas para auditoria e detecção de padrões.
- Insights do CloudTrail identificam padrões anormais automaticamente.

---

## Custos

- CloudTrail é gratuito para a gravação de eventos de gerenciamento.
- Há custo adicional para:
  - Eventos de dados gravados.
  - Uso do CloudTrail Insights.
  - Armazenamento e consultas nos serviços associados (S3, CloudWatch, Athena).

---

## Questões para prova AWS Developer Associate

**1. O que o Amazon CloudTrail registra em uma conta AWS?**  
**Resposta:** Ele registra o histórico de chamadas de API realizadas via console, SDK, CLI ou serviços AWS, incluindo operações de gerenciamento e dados quando configurado.  
**Explicação:** Isso é fundamental para auditoria, compliance e monitoramento.

---

**2. Quais tipos de eventos o CloudTrail pode registrar?**  
**Resposta:** Eventos de gerenciamento, eventos de dados e eventos do CloudTrail Insights.  
**Explicação:** Cada tipo atende a necessidades diferentes de monitoramento, sendo os eventos de dados opcionais e os Insights com custo adicional.

---

**3. Como o CloudTrail pode ser integrado para análise em tempo real?**  
**Resposta:** Enviando logs para o CloudWatch Logs e configurando regras de eventos para alertas e acionamento de Lambda.  
**Explicação:** Isso permite monitoramento proativo e resposta automática a incidentes.

---

**4. Qual a forma recomendada para manter logs do CloudTrail por mais de 90 dias?**  
**Resposta:** Armazenar os logs em um bucket S3 configurado para manter o histórico.  
**Explicação:** O CloudTrail retém logs localmente apenas 90 dias; o S3 permite retenção prolongada.

---

**5. O que é uma trilha (trail) no contexto do CloudTrail?**  
**Resposta:** É a configuração que determina quais eventos serão registrados e para onde os logs serão enviados.  
**Explicação:** Pode ser regional ou global e suporta múltiplas trilhas por conta.

---

## :books: Referências

- [O que é o Amazon CloudTrail?](https://docs.aws.amazon.com/pt_br/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)    
- [Conceitos do CloudTrail](https://docs.aws.amazon.com/pt_br/awscloudtrail/latest/userguide/cloudtrail-concepts.html#cloudtrail-concepts-events)  
- [CloudTrail Insights](https://docs.aws.amazon.com/pt_br/awscloudtrail/latest/userguide/cloudtrail-insights.html)  
- [Integração CloudTrail com CloudWatch Logs](https://docs.aws.amazon.com/pt_br/awscloudtrail/latest/userguide/send-cloudtrail-events-to-cloudwatch-logs.html)

---
