# Ferramentas, Políticas, Modelos e Checklist de Zero Trust

> **Objetivo pedagógico:** verificar o estado atual de elementos de Zero Trust nos três provedores com comandos de **somente leitura** (políticas de acesso condicional, níveis de acesso, acesso verificado e conectividade privada), organizar um catálogo de políticas de acesso por sensibilidade, usar modelos de registro (superfícies de proteção, matriz de maturidade e fichas de política) e aplicar um checklist de implantação.

> **Segurança:** os comandos apenas **consultam** configurações. Use identidades de leitura, em contas autorizadas, e registre as saídas como evidência (data e hora em UTC, conta e comando). Identificadores são fictícios; as opções variam entre versões: confirme na documentação.

## 1. Ferramentas por categoria

| Categoria | Exemplos de ferramentas e de serviços |
|---|---|
| Identidade e acesso condicional | Provedor de identidade corporativo; Microsoft Entra ID (acesso condicional); Google Cloud (Access Context Manager); políticas com condições da AWS |
| Gestão de dispositivos e postura | UEM e MDM (por exemplo, Microsoft Intune e soluções de terceiros); EDR; atestação de dispositivos |
| Acesso por aplicação (ZTNA e SSE) | Google IAP e BeyondCorp Enterprise; AWS Verified Access; Microsoft Entra Private Access; plataformas SSE e SASE de terceiros |
| Microssegmentação | Grupos de segurança e firewalls nativos; políticas de rede do Kubernetes; plataformas de microssegmentação |
| Autenticação entre serviços | Malhas de serviços; SPIFFE e SPIRE; certificados gerenciados |
| Motor de políticas | Open Policy Agent; mecanismos de autorização nativos |
| Proteção de dados | Classificação, DLP, gestão de chaves e de segredos |
| Telemetria e análise | SIEM e plataformas de dados de segurança; UEBA; EDR e NDR |
| Automação e orquestração | SOAR; automação por código; pipelines |
| Documentação | Mermaid, modelo C4 e registros de decisão (pasta 05) |

## 2. Verificações de somente leitura

### 2.1 Microsoft Entra ID e Azure

```bash
# Políticas de acesso condicional existentes (requer permissão de leitura no Microsoft Graph, por exemplo Policy.Read.All)
az rest --method get --url "https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies" \
  --query "value[].{nome:displayName, estado:state}" --output table

# Pontos de extremidade privados e endereços IP públicos (superfície de exposição)
az network private-endpoint list --query "[].{nome:name,rg:resourceGroup}" --output table
az network public-ip list --query "[].{nome:name,ip:ipAddress,rg:resourceGroup}" --output table
```

Revise: políticas em modo "somente relatório" por tempo prolongado, políticas desabilitadas, exclusões amplas de usuários e de aplicativos, e a existência de política para administradores, para autenticação legada e para risco de entrada.

### 2.2 Google Cloud

```bash
# Políticas de acesso e níveis de acesso (acesso sensível ao contexto)
gcloud access-context-manager policies list --organization=123456789012
gcloud access-context-manager levels list --policy=ID-DA-POLITICA --format="table(name,title)"

# Perímetros de serviço (controles de serviço da VPC) e regras de firewall com origem aberta
gcloud access-context-manager perimeters list --policy=ID-DA-POLITICA --format="table(name,title,status.resources.len())"
gcloud compute firewall-rules list --filter="sourceRanges:0.0.0.0/0 AND direction=INGRESS" \
  --format="table(name,network.basename(),allowed[].map().firewall_rule().list())"
```

### 2.3 AWS

```bash
# Instâncias do AWS Verified Access (acesso por aplicação sem VPN)
aws ec2 describe-verified-access-instances --query "VerifiedAccessInstances[].[VerifiedAccessInstanceId,Description]" --output table

# Pontos de extremidade de VPC (conectividade privada com serviços gerenciados)
aws ec2 describe-vpc-endpoints --query "VpcEndpoints[].[VpcEndpointId,ServiceName,VpcEndpointType,State]" --output table

# Grupos de segurança com acesso de administração aberto para a internet
aws ec2 describe-security-groups \
  --filters Name=ip-permission.cidr,Values=0.0.0.0/0 Name=ip-permission.from-port,Values=22,3389 \
  --query "SecurityGroups[].[GroupId,GroupName]" --output table
```

## 3. Catálogo de políticas de acesso por sensibilidade

| ID | Recurso ou superfície | Sensibilidade | Sujeitos | Condições exigidas | Controle de sessão | Ação em violação | Dono | Última revisão |
|---|---|---|---|---|---|---|---|---|
| ZT-001 | Console de nuvem (plano de controle) | Privilegiado | Administradores com elevação aprovada | MFA resistente a phishing; estação dedicada; risco baixo; local esperado | Sessão de até 4 horas; reautenticação para alterar identidades | Negar e alertar | Segurança | |
| ZT-002 | Sistema de pagamentos | Restrito | Equipe financeira | MFA resistente a phishing; dispositivo gerenciado conforme (nível 1) | Sessão curta; reautenticação em ações sensíveis | Negar | Dono da aplicação | |
| ZT-003 | Portal interno de RH | Confidencial | Colaboradores e RH | MFA; dispositivo gerenciado (nível 1 ou 2); BYOD limitado sem download | Reavaliação por evento | Limitar ou negar | RH e segurança | |
| ZT-004 | Intranet | Interno | Colaboradores | MFA padrão | Normal | Exigir MFA | Comunicação interna | |
| ZT-005 | APIs entre serviços | Confidencial a restrito | Identidades de carga | mTLS; token de curta duração; escopo mínimo | Tokens de minutos | Negar e registrar | Equipe da plataforma | |

## 4. Modelos

### 4.1 Registro de superfícies de proteção

| ID | Superfície (dados, ativos, aplicações e serviços) | Classificação | Criticidade | Proprietário | Fluxos principais (usuários, serviços e dados) | Controles atuais | Lacunas | Prioridade |
|---|---|---|---|---|---|---|---|---|
| SP-001 | | | | | | | | |
| SP-002 | | | | | | | | |

### 4.2 Ficha de política de acesso

| Campo | Preenchimento |
|---|---|
| ID, nome e versão | |
| Superfície ou recurso protegido | |
| Quem (sujeitos) | |
| O quê (ações permitidas) | |
| Quando (horários e duração) | |
| Onde (locais, redes e dispositivos) | |
| Como (autenticação e condições) | |
| Por quê (justificativa e base de negócio) | |
| Ação em caso de violação | |
| Exceções e controles compensatórios | |
| Modo de implantação (relatório ou imposição) e data | |
| Dono, revisor e próxima revisão | |

### 4.3 Matriz de maturidade por pilar

(Utilize o modelo do arquivo 6, seção 3.2.)

### 4.4 Registro de exceções de Zero Trust

| ID | Política | Recurso, usuário ou dispositivo | Justificativa | Controles compensatórios | Aprovador | Expiração |
|---|---|---|---|---|---|---|
| EZT-001 | | | | | | |

## 5. Checklist de implantação de Zero Trust (visão geral)

**Organização ou escopo:** ____________________
**Responsável:** ____________________
**Data:** ____________________

### 5.1 Estratégia e governança

- [ ] Estratégia, patrocinador, responsáveis por pilar e orçamento definidos
- [ ] Superfícies de proteção priorizadas e fluxos mapeados
- [ ] Catálogo de políticas de acesso com donos e revisão
- [ ] Gestão de exceções com prazo e controles compensatórios

### 5.2 Identidade e dispositivos

- [ ] MFA resistente a phishing para privilegiados e para recursos restritos
- [ ] Acesso condicional implantado (linha de base), com contas de emergência protegidas
- [ ] Avaliação contínua de acesso e tokens curtos
- [ ] Inventário, gestão e conformidade de dispositivos como condição de acesso
- [ ] Política de BYOD e de dispositivos não gerenciados

### 5.3 Redes

- [ ] Acesso por aplicação para as superfícies críticas; sem portas de administração expostas
- [ ] Microssegmentação com negação por padrão; pontos privados para serviços gerenciados
- [ ] mTLS e autorização entre serviços críticos
- [ ] Saída controlada e DNS filtrado

### 5.4 Aplicações, cargas e dados

- [ ] Autorização por requisição com motor de políticas; escopos mínimos
- [ ] Identidades de carga e pipelines sem credenciais de longa duração
- [ ] Artefatos assinados e admissão por política
- [ ] Dados classificados, acesso por atributos, criptografia e registros de acesso

### 5.5 Visibilidade, análise e automação

- [ ] Telemetria dos pilares centralizada e retida
- [ ] Detecções correlacionadas e pontuação de risco integradas às políticas
- [ ] Respostas automatizadas com salvaguardas e preservação de evidências
- [ ] Indicadores e avaliação de maturidade reportados

## 6. Resultado da verificação

| Resultado | Condição |
|---|---|
| Conforme | Todos os itens aplicáveis marcados |
| Conforme com ressalvas | Pendências sem exposição de recursos críticos, com plano de ação e prazos |
| Não conforme | Contas privilegiadas sem MFA forte, portas de administração expostas, ausência de telemetria de identidade, políticas apenas em relatório há mais de 90 dias sem decisão; corrigir com prioridade |

**Assinatura do responsável:** ____________________
**Assinatura do revisor:** ____________________

## 7. Exemplo prático: auditoria das políticas de acesso condicional

| Verificação | Resultado | Ação |
|---|---|---|
| Políticas de acesso condicional | 14 políticas; 4 em modo somente relatório há mais de 6 meses | Decidir: impor, ajustar ou remover; prazo de 30 dias |
| Política para administradores | Existe, mas exige apenas MFA (não resistente a phishing) | Atualizar para exigir força de autenticação resistente a phishing |
| Autenticação legada | Bloqueada, com exclusão para duas aplicações | Migrar as aplicações e remover as exclusões |
| Exclusões de usuários | 9 contas excluídas (3 sem justificativa) | Revisar; manter apenas contas de emergência monitoradas |
| Risco de entrada | Sem política de risco | Criar política de reautenticação e de bloqueio por risco |
| Dispositivos | Conformidade exigida apenas para o e-mail | Estender aos sistemas restritos e ao console de nuvem |

## 8. Casos de uso corporativos

- Auditoria periódica das políticas de acesso e das exclusões.
- Catálogo único de políticas de acesso, com donos e revisão.
- Evidências de Zero Trust para auditorias, reguladores e clientes.

## 9. Normas e frameworks relevantes

- NIST SP 800-207; CISA ZTMM 2.0; NSA Zero Trust Implementation Guidelines; DoD Zero Trust Strategy
- CSA CCM v4 (IAM, IVS e LOG); CIS Foundations Benchmarks (identidade e rede); ISO/IEC 27002:2022
- NIST CSF 2.0 (PR.AA, PR.IR e DE.CM)

## 10. Referências

- Documentação do acesso condicional do Microsoft Entra ID e do Microsoft Graph; do Access Context Manager e do VPC Service Controls (Google Cloud); do AWS Verified Access e do Amazon VPC (versões vigentes).
- Rose, S. et al. NIST SP 800-207. 2020.
- CISA. Zero Trust Maturity Model 2.0. 2023.
- Pastas 05, 06 e 08 deste repositório.
