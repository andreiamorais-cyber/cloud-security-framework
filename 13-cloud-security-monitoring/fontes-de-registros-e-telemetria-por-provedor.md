# Fontes de Registros e Telemetria por Provedor

> **Objetivo pedagógico:** conhecer o catálogo de fontes de registros e de telemetria de segurança em AWS, Azure e Google Cloud (plano de controle, identidade, dados, rede, computação, contêineres, funções sem servidor, SaaS, serviços de segurança e configuração), saber o que **já vem habilitado** e o que **precisa ser habilitado**, os limites de retenção padrão, os requisitos de qualidade dos registros (horário, campos e atribuição) e verificar o estado da habilitação com comandos de somente leitura.

> **Nota:** retenções padrão, recursos e custos mudam. Confirme na documentação vigente de cada provedor.

## 1. Catálogo de fontes por camada

| Camada | Fonte | AWS | Azure | Google Cloud |
|---|---|---|---|---|
| Plano de controle | Registros de auditoria de chamadas de API | CloudTrail (eventos de gerenciamento, de dados e Insights) | Log de atividade (nível de assinatura) | Cloud Audit Logs: atividade de administrador, acesso a dados, evento de sistema e acesso negado por política |
| Identidade | Autenticações e alterações de identidade | Eventos de IAM e do IAM Identity Center no CloudTrail | Registros de entrada e de auditoria do Microsoft Entra ID | Registros de auditoria do Cloud Identity e do Google Workspace |
| Acesso a dados | Leitura e gravação em dados | Eventos de dados do CloudTrail (S3, Lambda e outros) e registros de acesso do S3 | Registros de recursos do armazenamento e auditoria de bancos | Registros de acesso a dados do Cloud Audit Logs |
| Rede | Fluxos, DNS, firewall, balanceadores e WAF | VPC Flow Logs; registros de consultas do Route 53 Resolver; Network Firewall; registros de ALB e de WAF | Registros de fluxo de rede virtual; Azure Firewall; Application Gateway e WAF | VPC Flow Logs; Cloud DNS; Cloud NGFW; balanceadores e Cloud Armor |
| Computação | Eventos de sistema e de endpoint | Agentes (CloudWatch Agent), EDR e Systems Manager | Azure Monitor Agent e EDR | Ops Agent e EDR |
| Contêineres e Kubernetes | Auditoria do cluster e registros de cargas | Registros do plano de controle do EKS (inclui auditoria) | Configurações de diagnóstico do AKS (inclui auditoria do servidor de API) | Registros de auditoria do GKE |
| Sem servidor | Execução e invocações | CloudWatch Logs e métricas | Application Insights e Azure Monitor | Cloud Logging |
| Bancos de dados | Auditoria de acesso e de administração | Registros de auditoria do RDS e fluxos de atividade | Auditoria do Azure SQL e de outros serviços | Registros de auditoria do Cloud SQL |
| Segredos e chaves | Uso de chaves e de segredos | Eventos do KMS e do Secrets Manager no CloudTrail | Registros do Key Vault | Cloud Audit Logs de KMS e de Secret Manager |
| SaaS | Atividade de usuários e de administradores | Registros de auditoria de SaaS integrados (por exemplo, por APIs) | Registro de auditoria unificado do Microsoft 365 | Registros de auditoria do Google Workspace |
| Serviços de segurança | Achados e alertas | GuardDuty, Security Hub e Inspector | Defender for Cloud e Microsoft Sentinel | Security Command Center |
| Configuração | Estado e histórico de recursos | AWS Config | Azure Resource Graph e histórico de alterações | Cloud Asset Inventory |
| Custos | Uso e anomalias | Cost Explorer e detecção de anomalias de custo | Cost Management | Faturamento e alertas de orçamento |

## 2. O que já vem habilitado e o que precisa ser habilitado

| Provedor | Habilitado por padrão (em geral) | Precisa de habilitação ou de configuração |
|---|---|---|
| AWS | Histórico de eventos de gerenciamento do CloudTrail (visualização dos últimos 90 dias, sem trilha) | **Trilha** para entrega a armazenamento (retenção longa e centralização); eventos de dados; Insights; registros de fluxo; registros de DNS; Config; GuardDuty e Security Hub; registros de acesso do S3 |
| Azure | Log de atividade (retido por 90 dias por padrão); alguns registros de Entra ID por retenção limitada, conforme a licença | **Configurações de diagnóstico** para exportar o log de atividade e os registros de recursos para um espaço de trabalho ou para armazenamento; exportação dos registros do Entra ID; registros de fluxo; Defender for Cloud; auditoria de bancos |
| Google Cloud | Atividade de administrador e eventos de sistema (sempre ativos, em *buckets* obrigatórios de retenção longa) | **Registros de acesso a dados** (desabilitados por padrão, exceto em alguns serviços como o BigQuery); coletores de organização; registros de fluxo por sub-rede; Security Command Center |

**Princípio:** nunca dependa dos registros padrão e de curta retenção como evidência. Habilite as trilhas organizacionais, exporte para a conta de registros e defina a retenção **antes** que um incidente aconteça.

## 3. Retenções padrão (referências)

| Fonte | Retenção padrão típica | Observação |
|---|---|---|
| CloudTrail, histórico de eventos | 90 dias, apenas eventos de gerenciamento | Sem trilha, não há entrega a armazenamento nem eventos de dados |
| Log de atividade do Azure | 90 dias | Exportar para retenção maior |
| Registros do Microsoft Entra ID | De alguns dias a cerca de 30 dias, conforme o licenciamento | Exportar imediatamente para armazenamento e para o SIEM |
| Cloud Audit Logs no Google Cloud | *Bucket* obrigatório de 400 dias (atividade de administrador e eventos de sistema); *bucket* padrão com retenção mais curta (por exemplo, 30 dias), configurável | Configurar coletores e retenção |
| Registros de fluxo e de DNS | Dependem do destino configurado | Definir retenção na entrega |
| Alertas de serviços de segurança | Variável por serviço | Exportar para o SIEM |

## 4. Qualidade dos registros

| Atributo | Requisito |
|---|---|
| Horário | UTC, com relógios sincronizados (ISO/IEC 27002:2022, 8.17) |
| Identificação da identidade | Quem fez (usuário, função, sessão ou serviço) e como se autenticou |
| Origem | Endereço IP, agente e localização |
| Alvo e ação | Recurso, operação e parâmetros relevantes |
| Resultado | Sucesso, falha e motivo |
| Correlação | Identificadores de requisição e de sessão |
| Completude | Sem lacunas por falhas de entrega ou por filtragem excessiva |
| Integridade | Proteção contra alteração (arquivo 3) |
| Privacidade | Sem dados pessoais ou segredos desnecessários; mascaramento quando preciso |

### Pontos de atenção

- Parâmetros de requisição e de resposta nos registros podem conter dados sensíveis; revise o que é registrado.
- Alguns serviços não registram todos os campos por padrão (por exemplo, parâmetros de determinadas chamadas); consulte a documentação para entender as limitações.
- Registros do **provedor** sobre sua própria infraestrutura não estão acessíveis ao cliente: use os relatórios e os canais de segurança.

## 5. Verificações de somente leitura

```bash
# AWS: trilhas, estado de entrega e seletores de eventos (gerenciamento e dados)
aws cloudtrail describe-trails --query "trailList[].[Name,IsOrganizationTrail,IsMultiRegionTrail,S3BucketName,LogFileValidationEnabled]" --output table
aws cloudtrail get-trail-status --name NOME-DA-TRILHA --query "[IsLogging,LatestDeliveryTime,LatestDeliveryError]"
aws cloudtrail get-event-selectors --trail-name NOME-DA-TRILHA

# AWS: registros de fluxo, de consultas de DNS e detectores de ameaças
aws ec2 describe-flow-logs --query "FlowLogs[].[FlowLogId,ResourceId,TrafficType,LogDestinationType]" --output table
aws route53resolver list-resolver-query-log-configs --query "ResolverQueryLogConfigs[].[Name,Status]" --output table
aws guardduty list-detectors

# Azure: configurações de diagnóstico da assinatura (log de atividade) e espaços de trabalho de análise
az monitor diagnostic-settings subscription list --query "value[].{nome:name, espacoTrabalho:workspaceId, armazenamento:storageAccountId}" --output table
az monitor log-analytics workspace list --query "[].{nome:name, retencaoDias:retentionInDays, rg:resourceGroup}" --output table

# Google Cloud: coletores de registros e configuração de auditoria de acesso a dados de um projeto
gcloud logging sinks list --project=ID-DO-PROJETO
gcloud projects get-iam-policy ID-DO-PROJETO --format="yaml(auditConfigs)"
```

Se `auditConfigs` estiver vazio no Google Cloud, os registros de **acesso a dados** estão desabilitados nos serviços (com as exceções de padrão do provedor).

## 6. Priorização de fontes

| Prioridade | Fontes | Justificativa |
|---|---|---|
| 1 (crítica) | Auditoria do plano de controle; identidade (entrada e auditoria); alertas de serviços de segurança | Base de detecção de abuso de identidade e de alterações |
| 2 (alta) | Registros de acesso a dados para dados restritos; uso de chaves e de segredos; registros de rede (fluxo e DNS) | Impacto e alcance de incidentes com dados |
| 3 (média) | Auditoria do Kubernetes; registros de WAF e de balanceadores; auditoria de bancos; SaaS críticos | Cobertura de cargas e de aplicações |
| 4 (complementar) | Registros de sistema e de aplicações; EDR em servidores; configuração; custos | Profundidade e correlação |

Comece pelas **fontes de prioridade 1 e 2** em 100% das contas, e expanda conforme o risco e o orçamento.

## 7. Exemplo prático: auditoria da habilitação de registros em uma organização

| Verificação | Achado | Ação |
|---|---|---|
| CloudTrail organizacional | Trilha em 31 de 40 contas; 9 contas sem trilha própria | Trilha organizacional única e multirregional |
| Eventos de dados | Habilitados em 4 de 62 buckets com dados pessoais | Habilitar para todos os buckets com dados restritos e confidenciais |
| Registros de fluxo | Em 6 de 11 redes de produção | Habilitar nas demais, com entrega à conta de registros |
| Azure: configurações de diagnóstico | 12 de 20 assinaturas sem exportação do log de atividade | Política que impõe a exportação |
| Entra ID | Registros de entrada não exportados (retenção de poucos dias) | Exportar ao SIEM e ao armazenamento imutável |
| Google Cloud: acesso a dados | Desabilitado nos serviços de armazenamento e de banco de dados com dados restritos | Habilitar auditoria de acesso a dados nos serviços relevantes |
| Resultado | Cobertura de registros críticos de 58% para 100% em 4 semanas | |

## 8. Casos de uso corporativos

- Linha de base de habilitação de registros por provedor, verificada continuamente.
- Catálogo de fontes de dados com responsáveis, retenção e prioridade.
- Evidências de registro e de monitoramento para auditorias e reguladores.

## 9. Normas e frameworks relevantes

- ISO/IEC 27002:2022 (8.15, registro; 8.17, sincronização de relógios); NIST SP 800-92; NIST SP 800-53 Rev. 5 (AU-2, AU-3, AU-8 e AU-12)
- CSA CCM v4 (LOG-01 a LOG-13); CIS Foundations Benchmarks (seções de registro e monitoramento); CIS Controls v8.1 (controle 8)
- Decreto nº 8.771/2016 (padrões de guarda de registros); Marco Civil da Internet, arts. 13 e 15

## 10. Checklist operacional: fontes de registros

- [ ] Catálogo de fontes por camada e por provedor, com responsável, prioridade e retenção
- [ ] Trilhas de auditoria organizacionais habilitadas em todas as contas e regiões
- [ ] Exportação do log de atividade e de registros de identidade para armazenamento e para o SIEM
- [ ] Registros de acesso a dados habilitados para dados confidenciais e restritos
- [ ] Registros de fluxo, de DNS, de firewall e de WAF habilitados em redes de produção
- [ ] Auditoria de Kubernetes, de bancos de dados e de uso de chaves habilitada
- [ ] Serviços de detecção e de postura habilitados em todas as contas
- [ ] Horário em UTC e relógios sincronizados; campos e atribuição verificados
- [ ] Verificação contínua da habilitação (alerta se um registro for desabilitado)
- [ ] Revisão semestral do catálogo

## 11. Referências

- Amazon Web Services. AWS CloudTrail, VPC Flow Logs e Route 53 Resolver Query Logging; Microsoft. Azure Monitor, log de atividade e registros do Microsoft Entra ID; Google Cloud. Cloud Audit Logs e Cloud Logging (documentação vigente).
- Chuvakin, A.; Schmidt, K.; Phillips, C. Logging and Log Management. Syngress, 2012.
- MITRE. ATT&CK Data Sources (documentação).
- Pastas 02, 06, 08 e 09 deste repositório.
