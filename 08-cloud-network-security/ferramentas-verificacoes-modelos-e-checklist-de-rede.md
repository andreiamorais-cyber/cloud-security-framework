# Ferramentas, Verificações, Modelos e Checklist de Rede

> **Objetivo pedagógico:** verificar o estado da segurança de rede nos três principais provedores com comandos de **somente leitura** (redes, rotas, peering, registros de fluxo, WAF e proteção DDoS), automatizar a análise de regras abertas com um roteiro em Python, relacionar as verificações às linhas de base do CIS, e usar modelos (plano de endereçamento, registro de regras e de exceções de rede, ficha de arquitetura de rede) e um checklist de segurança de rede.

> **Segurança:** os comandos apenas **consultam** configurações. Use identidades de leitura, em contas autorizadas, e registre as saídas como evidência (data e hora em UTC, conta e comando). Identificadores são fictícios; as opções variam entre versões: confirme na documentação.

## 1. Ferramentas por finalidade

| Finalidade | Ferramentas |
|---|---|
| Consulta e auditoria de configuração | AWS CLI, Azure CLI e Google Cloud CLI; Azure Resource Graph; Cloud Asset Inventory; AWS Config |
| Infraestrutura como código | Terraform, OpenTofu, CloudFormation, Bicep e Pulumi |
| Verificação de regras e de políticas em código | Checkov, tfsec, Trivy e Open Policy Agent |
| Postura e conformidade de rede | Serviços nativos de postura e ferramentas de gestão de postura (pasta 17) |
| Análise de tráfego e de registros | Athena, Kusto e DuckDB; Zeek e Suricata (com espelhamento) |
| Teste de exposição (somente em ambientes autorizados) | Verificações de portas e de serviços em suas próprias redes e com autorização formal |
| Documentação | Mermaid, modelo C4 e registros de decisão (pasta 05) |

Execute testes de exposição **apenas em sistemas próprios ou autorizados**, em conformidade com as políticas do provedor.

## 2. Verificações de somente leitura

### 2.1 AWS

```bash
# Redes, sub-redes e roteamento
aws ec2 describe-vpcs --query "Vpcs[].[VpcId,CidrBlock,Tags[?Key=='Name']|[0].Value]" --output table
aws ec2 describe-subnets --query "Subnets[].[SubnetId,VpcId,CidrBlock,MapPublicIpOnLaunch]" --output table
aws ec2 describe-route-tables --query "RouteTables[].[RouteTableId,VpcId,Routes[?GatewayId!=null].GatewayId|join(',',@)]" --output table

# Gateways de internet, peering e gateways de trânsito
aws ec2 describe-internet-gateways --query "InternetGateways[].[InternetGatewayId,Attachments[0].VpcId]" --output table
aws ec2 describe-vpc-peering-connections --query "VpcPeeringConnections[].[VpcPeeringConnectionId,RequesterVpcInfo.VpcId,AccepterVpcInfo.VpcId,Status.Code]" --output table
aws ec2 describe-transit-gateways --query "TransitGateways[].[TransitGatewayId,State]" --output table

# Registros de fluxo habilitados
aws ec2 describe-flow-logs --query "FlowLogs[].[FlowLogId,ResourceId,TrafficType,LogDestinationType]" --output table

# Interfaces com endereço IP público (exposição direta)
aws ec2 describe-network-interfaces \
  --query "NetworkInterfaces[?Association.PublicIp!=null].[NetworkInterfaceId,Association.PublicIp,Description]" --output table

# Grupos de segurança padrão com regras (devem estar sem regras de entrada nem de saída)
aws ec2 describe-security-groups --filters Name=group-name,Values=default \
  --query "SecurityGroups[].[GroupId,VpcId,length(IpPermissions),length(IpPermissionsEgress)]" --output table

# WAF e proteção DDoS
aws wafv2 list-web-acls --scope REGIONAL --query "WebACLs[].[Name,Id]" --output table
aws shield describe-subscription
```

### 2.2 Azure

```bash
# Redes virtuais, peerings, grupos de segurança de rede e IPs públicos
az network vnet list --query "[].{nome:name,rg:resourceGroup,espaco:addressSpace.addressPrefixes[0]}" --output table
az network vnet peering list --resource-group RG-EXEMPLO --vnet-name VNET-EXEMPLO --output table
az network nsg list --query "[].{nome:name,rg:resourceGroup}" --output table
az network public-ip list --query "[].{nome:name,ip:ipAddress,rg:resourceGroup}" --output table

# Regras de entrada que permitem qualquer origem
az network nsg rule list --nsg-name NSG-EXEMPLO --resource-group RG-EXEMPLO \
  --query "[?direction=='Inbound' && access=='Allow' && (sourceAddressPrefix=='*' || sourceAddressPrefix=='Internet')].{regra:name,porta:destinationPortRange,prioridade:priority}" --output table

# Registros de fluxo configurados na região (confirme o recurso vigente: NSG ou rede virtual)
az network watcher flow-log list --location brazilsouth --output table

# WAF e proteção DDoS
az network application-gateway waf-policy list --query "[].{nome:name,rg:resourceGroup}" --output table
az network ddos-protection list --query "[].{nome:name,rg:resourceGroup}" --output table
```

### 2.3 Google Cloud

```bash
# Redes, sub-redes (com registros de fluxo), rotas e roteadores
gcloud compute networks list
gcloud compute networks subnets list --format="table(name,region,network.basename(),ipCidrRange,logConfig.enable)"
gcloud compute routes list --format="table(name,network.basename(),destRange,nextHopGateway.basename())"
gcloud compute routers list

# Regras de firewall com origem aberta
gcloud compute firewall-rules list --filter="direction=INGRESS AND sourceRanges:0.0.0.0/0" \
  --format="table(name,network.basename(),priority,allowed[].map().firewall_rule().list())"

# Instâncias com endereço IP externo
gcloud compute instances list --format="table(name,zone.basename(),networkInterfaces[0].accessConfigs[0].natIP)"

# Políticas de segurança do Cloud Armor
gcloud compute security-policies list --format="table(name,description)"
```

## 3. Roteiro em Python para analisar regras abertas (AWS)

```python
# Lê o JSON de "aws ec2 describe-security-groups" e classifica regras de entrada abertas para a internet
# Uso: aws ec2 describe-security-groups --output json > sgs.json && python analisar_sgs.py sgs.json
import json
import sys

PORTAS_ADMIN = {22, 3389}
PORTAS_BANCOS = {1433, 1521, 3306, 5432, 6379, 9200, 27017}
PORTAS_WEB = {80, 443}

def aberta_ao_mundo(perm):
    v4 = any(r.get("CidrIp") == "0.0.0.0/0" for r in perm.get("IpRanges", []))
    v6 = any(r.get("CidrIpv6") == "::/0" for r in perm.get("Ipv6Ranges", []))
    return v4 or v6

def faixa_portas(perm):
    if perm.get("IpProtocol") == "-1":
        return None  # todos os protocolos e portas
    return range(perm.get("FromPort", 0), perm.get("ToPort", 0) + 1)

def classificar(grupo_id, nome, perm):
    portas = faixa_portas(perm)
    if portas is None:
        return ("CRITICO", "todas as portas abertas ao mundo")
    conjunto = set(portas) if len(portas) < 70000 else set()
    if conjunto & PORTAS_ADMIN:
        return ("CRITICO", "porta de administração aberta ao mundo")
    if conjunto & PORTAS_BANCOS:
        return ("CRITICO", "porta de banco de dados aberta ao mundo")
    if len(portas) > 100:
        return ("ALTO", f"faixa ampla de portas ({portas.start}-{portas.stop - 1}) aberta ao mundo")
    if conjunto and conjunto <= PORTAS_WEB:
        return ("INFO", "HTTP ou HTTPS público: verificar WAF e necessidade")
    return ("MEDIO", f"portas {sorted(conjunto)} abertas ao mundo")

with open(sys.argv[1], encoding="utf-8") as f:
    grupos = json.load(f)["SecurityGroups"]

for g in grupos:
    for perm in g.get("IpPermissions", []):
        if aberta_ao_mundo(perm):
            nivel, motivo = classificar(g["GroupId"], g.get("GroupName", ""), perm)
            print(f'{nivel:8} {g["GroupId"]:22} {g.get("GroupName", "")[:30]:30} {motivo}')
```

O roteiro é **didático**: ajuste as portas, os critérios e os níveis à sua política. Para ambientes grandes, use ferramentas de verificação contínua de postura.

## 4. Relação com linhas de base do CIS (exemplos de verificações de rede)

| Verificação | Provedor | Observação |
|---|---|---|
| Nenhum grupo de segurança permite entrada de `0.0.0.0/0` para portas de administração (22 e 3389) | AWS, Azure e Google Cloud | Verificação crítica |
| O grupo de segurança padrão restringe todo o tráfego | AWS | Uso exclusivo de grupos explícitos |
| Registros de fluxo habilitados em todas as redes virtuais | AWS, Azure e Google Cloud | Essencial para detecção e investigação |
| Listas de controle de rede não permitem entrada irrestrita para portas de administração | AWS | Camada adicional |
| Peering com tabelas de rotas de menor privilégio | AWS | Evitar rotas amplas |
| Rede padrão (default) removida ou restrita | AWS e Google Cloud | Redes padrão têm regras permissivas |
| Acesso público desabilitado em serviços gerenciados quando há ponto privado | Todos | Reduz a exposição |

Consulte as versões vigentes dos CIS Foundations Benchmarks de cada provedor para a lista completa e a numeração.

## 5. Modelos

### 5.1 Plano de endereçamento IP

| Bloco | Uso | Ambiente | Região | Responsável | Sobreposição verificada | Observações |
|---|---|---|---|---|---|---|
| 10.0.0.0/16 | Hub | Produção | Principal | Plataforma de rede | Sim | |

### 5.2 Registro de regras de rede de exceção

| ID | Rede ou grupo | Regra (origem, destino, porta e protocolo) | Justificativa de negócio | Risco e controles compensatórios | Dono | Aprovador | Expiração | Revisão |
|---|---|---|---|---|---|---|---|---|
| EXR-001 | | | | | | | | |

### 5.3 Ficha de arquitetura de rede de uma aplicação

| Seção | Conteúdo |
|---|---|
| Identificação | Aplicação, proprietário, criticidade e classificação de dados |
| Topologia | Redes, sub-redes, rotas e conexões (diagrama com fronteiras de confiança) |
| Entrada | Pontos de entrada, borda (CDN, WAF e DDoS) e TLS |
| Segmentação | Grupos de segurança, listas de controle e firewalls; fluxos permitidos |
| Serviços gerenciados | Pontos privados e políticas de ponto de extremidade |
| Saída | Rotas, proxy, filtros e DNS |
| Conectividade híbrida | Enlaces, criptografia e rotas |
| Registros e detecção | Fontes, retenção e alertas |
| Resiliência | Zonas, regiões e planos de recuperação |
| Riscos e decisões | Riscos residuais, ADRs e exceções |
| Verificações automáticas | Políticas e relatórios de postura |

### 5.3.1 Registro de revisão periódica de regras

| Data | Rede ou conta | Regras revisadas | Removidas | Ajustadas | Exceções renovadas | Responsável |
|---|---|---|---|---|---|---|
| | | | | | | |

## 6. Checklist de segurança de rede em nuvem

**Ambiente:** ____________________
**Responsável:** ____________________
**Data:** ____________________

### 6.1 Exposição e segmentação

- [ ] Nenhuma porta de administração (SSH e RDP) aberta à internet
- [ ] Nenhum banco de dados ou serviço interno com endereço público não justificado
- [ ] Grupos de segurança por função, com regras por referência a grupos e privilégio mínimo
- [ ] Redes padrão removidas ou restritas; grupo de segurança padrão sem regras
- [ ] Segmentação por aplicação e por classificação; dados restritos isolados

### 6.2 Arquitetura e conectividade

- [ ] Plano de endereçamento sem sobreposição; domínios de roteamento por criticidade
- [ ] Hub com inspeção (firewall de rede) quando exigido; tráfego entre domínios controlado
- [ ] Conectividade híbrida redundante, criptografada para dados sensíveis, com rotas mínimas
- [ ] Serviços gerenciados por pontos privados; acesso público desabilitado quando possível
- [ ] DNS privado, resolvedores controlados e filtragem; contas de DNS protegidas

### 6.3 Borda

- [ ] CDN, WAF e proteção DDoS nas aplicações públicas; origem protegida
- [ ] WAF em modo de bloqueio após período de contagem; limitação de taxa calibrada
- [ ] TLS 1.2 ou superior, HSTS e certificados gerenciados com alertas de expiração
- [ ] APIs com autenticação, autorização, validação e cotas

### 6.4 Saída e monitoramento

- [ ] Saída controlada por proxy ou firewall, com lista de destinos; DNS externo bloqueado
- [ ] Perímetro de dados para dados restritos (identidades, recursos e redes confiáveis)
- [ ] Registros de fluxo, de DNS, de firewall, de WAF e de balanceadores centralizados e retidos
- [ ] Detecções de rede implantadas e testadas; alertas de alteração de regras e de rotas
- [ ] Revisão trimestral de regras e remoção de regras não usadas

## 7. Resultado da verificação

| Resultado | Condição |
|---|---|
| Conforme | Todos os itens aplicáveis marcados |
| Conforme com ressalvas | Pendências sem exposição de serviços críticos ou de dados, com plano de ação e prazo |
| Não conforme | Administração exposta à internet, banco de dados público, ausência de registros de fluxo em produção ou ausência de proteção de borda em aplicações críticas; corrigir com prioridade |

**Assinatura do responsável:** ____________________
**Assinatura do revisor:** ____________________

## 8. Exemplo prático: auditoria de rede de uma conta de produção

| Verificação | Resultado | Ação |
|---|---|---|
| Interfaces com IP público | 9; 4 conhecidas (balanceadores); 5 sem justificativa | Investigar: 3 instâncias esquecidas encerradas; 2 com SSH exposto restringidas |
| Grupos de segurança abertos ao mundo | 14 regras; 3 críticas (SSH, RDP e porta de banco) | Remover e usar sessões gerenciadas e pontos privados |
| Registros de fluxo | Habilitados em 6 de 11 redes | Habilitar em todas, com envio à conta de registros |
| WAF | Associado a 2 de 5 balanceadores públicos | Associar aos 3 restantes, em modo de contagem por 2 semanas |
| Saída | Rota direta à internet em sub-redes de dados | Rota padrão ao hub, com filtro de destino |
| Resultado | Pontuação de postura de rede de 58% para 94% em 6 semanas | |

## 9. Casos de uso corporativos

- Auditorias periódicas de rede com evidências coletadas de forma consistente.
- Verificação contínua de exposição e de regras abertas, integrada ao pipeline.
- Padronização de modelos de plano de endereçamento, de exceções e de revisão de regras.

## 10. Normas e frameworks relevantes

- CIS Foundations Benchmarks (AWS, Azure e Google Cloud); CIS Controls v8.1 (controles 12 e 13)
- NIST SP 800-41 Rev. 1; NIST SP 800-53 Rev. 5 (SC-7 e CM-6); CSA CCM v4 (IVS e LOG)
- ISO/IEC 27002:2022 (8.9, 8.20 a 8.22); PCI DSS v4.0 (requisitos de controles de segurança de rede)

## 11. Referências

- Center for Internet Security. CIS Foundations Benchmarks (AWS, Azure e Google Cloud).
- Documentação das ferramentas de linha de comando: AWS CLI, Azure CLI e Google Cloud CLI.
- Documentação do Checkov, do tfsec, do Trivy e do Open Policy Agent.
- Pastas 05, 13, 17 e 22 deste repositório.
