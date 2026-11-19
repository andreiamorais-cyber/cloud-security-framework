# Ferramentas, Verificações, Modelos e Checklist de Criptografia

> **Objetivo pedagógico:** verificar o estado da criptografia nos três principais provedores com comandos de **somente leitura** (recursos sem criptografia, chaves sem rotação, políticas de TLS, certificados a vencer e cofres sem proteção contra exclusão), automatizar a verificação de expiração de certificados com um roteiro em Python, e usar modelos (padrão de criptografia, inventário de chaves, inventário de certificados e lista de materiais criptográficos, CBOM) e um checklist para sustentar o programa de criptografia.

> **Segurança:** os comandos apenas **consultam** configurações. Use identidades de leitura, em contas autorizadas, e registre as saídas como evidência (data e hora em UTC, conta e comando). Identificadores são fictícios; as opções variam entre versões: confirme na documentação. Os testes ativos de TLS devem ser feitos somente em sistemas próprios ou autorizados.

## 1. Ferramentas por finalidade

| Finalidade | Ferramentas |
|---|---|
| Consulta e auditoria de configuração | AWS CLI, Azure CLI e Google Cloud CLI; AWS Config; Azure Policy; Cloud Asset Inventory |
| Verificação de TLS | testssl.sh, sslyze, OpenSSL e nmap (somente em sistemas próprios ou autorizados) |
| Inventário e gestão de certificados | Serviços de certificados dos provedores; gestores de certificados; monitoramento de transparência de certificados |
| Análise estática de código | Ferramentas de análise de código que detectam usos criptográficos fracos e segredos expostos |
| Políticas e infraestrutura como código | Open Policy Agent; Checkov; tfsec; Trivy |
| Inventário criptográfico | Ferramentas de descoberta de criptografia e geração de CBOM (por exemplo, com o formato CycloneDX) |
| Análise de registros de chaves | SIEM; Athena; Kusto; DuckDB |

## 2. Verificações de somente leitura

### 2.1 AWS

```bash
# Chaves do KMS: estado, gestor e rotação
for k in $(aws kms list-keys --query "Keys[].KeyId" --output text); do
  info=$(aws kms describe-key --key-id "$k" --query "KeyMetadata.[KeyManager,KeyState,KeySpec]" --output text)
  # A rotação automática só se aplica a chaves simétricas gerenciadas pelo cliente
  if echo "$info" | grep -q "CUSTOMER.*SYMMETRIC_DEFAULT"; then
    rot=$(aws kms get-key-rotation-status --key-id "$k" --query KeyRotationEnabled --output text 2>/dev/null)
    echo "$k | $info | rotacao: $rot"
  fi
done

# Chaves com exclusão agendada (atenção imediata)
aws kms list-keys --query "Keys[].KeyId" --output text | tr '\t' '\n' | while read k; do
  aws kms describe-key --key-id "$k" --query "KeyMetadata[?KeyState=='PendingDeletion'].[KeyId,DeletionDate]" --output text
done

# Balanceadores: protocolos e políticas de TLS dos ouvintes
aws elbv2 describe-load-balancers --query "LoadBalancers[].LoadBalancerArn" --output text | tr '\t' '\n' | while read lb; do
  aws elbv2 describe-listeners --load-balancer-arn "$lb" --query "Listeners[].[Port,Protocol,SslPolicy]" --output table
done

# Certificados do ACM e datas de expiração
aws acm list-certificates --query "CertificateSummaryList[].CertificateArn" --output text | tr '\t' '\n' | while read c; do
  aws acm describe-certificate --certificate-arn "$c" --query "Certificate.[DomainName,Status,NotAfter]" --output text
done

# Criptografia por padrão de volumes; bancos sem criptografia
aws ec2 get-ebs-encryption-by-default
aws rds describe-db-instances --query "DBInstances[?StorageEncrypted==\`false\`].[DBInstanceIdentifier,Engine]" --output table
```

### 2.2 Azure

```bash
# Cofres de chaves: proteção contra exclusão, modelo de autorização e acesso de rede
for v in $(az keyvault list --query "[].name" --output tsv); do
  az keyvault show --name "$v" \
    --query "{cofre:name, exclusaoLogica:properties.enableSoftDelete, protecaoPurga:properties.enablePurgeProtection, rbac:properties.enableRbacAuthorization, redePublica:properties.publicNetworkAccess}" --output table
done

# Chaves e certificados com data de expiração
az keyvault key list --vault-name COFRE-EXEMPLO --query "[].{nome:name, ativa:attributes.enabled, expira:attributes.expires}" --output table
az keyvault certificate list --vault-name COFRE-EXEMPLO --query "[].{nome:name, expira:attributes.expires}" --output table

# Contas de armazenamento: TLS mínimo e somente HTTPS
az storage account list --query "[].{nome:name, tlsMinimo:minimumTlsVersion, apenasHttps:enableHttpsTrafficOnly}" --output table

# Política de TLS de um gateway de aplicação
az network application-gateway ssl-policy show --gateway-name GATEWAY-EXEMPLO --resource-group RG-EXEMPLO
```

### 2.3 Google Cloud

```bash
# Chaves, rotação e próxima rotação
gcloud kms keyrings list --location=southamerica-east1
gcloud kms keys list --keyring=ANEL-EXEMPLO --location=southamerica-east1 \
  --format="table(name,purpose,rotationPeriod,nextRotationTime,primary.state)"

# Políticas de TLS de balanceadores e certificados gerenciados
gcloud compute ssl-policies list --format="table(name,profile,minTlsVersion)"
gcloud certificate-manager certificates list --format="table(name,managed.state,expireTime)"

# Discos e recursos com chave do cliente (campo vazio indica chave padrão do Google)
gcloud compute disks list --format="table(name,zone.basename(),diskEncryptionKey.kmsKeyName)"
```

## 3. Verificação de expiração de certificados em Python (sistemas próprios)

```python
# Calcula os dias restantes para a expiração dos certificados de uma lista de endereços próprios
# Uso: python expiracao_certificados.py exemplo.org api.exemplo.org:8443
import datetime
import socket
import ssl
import sys

def dias_para_expirar(host: str, porta: int = 443) -> int:
    contexto = ssl.create_default_context()
    with socket.create_connection((host, porta), timeout=5) as sock:
        with contexto.wrap_socket(sock, server_hostname=host) as conexao:
            certificado = conexao.getpeercert()
    fim = datetime.datetime.strptime(certificado["notAfter"], "%b %d %H:%M:%S %Y %Z")
    fim = fim.replace(tzinfo=datetime.timezone.utc)
    return (fim - datetime.datetime.now(datetime.timezone.utc)).days

for alvo in sys.argv[1:]:
    host, _, porta = alvo.partition(":")
    try:
        dias = dias_para_expirar(host, int(porta) if porta else 443)
        situacao = "CRITICO" if dias < 14 else "ATENCAO" if dias < 30 else "OK"
        print(f"{situacao:8} {alvo:40} {dias} dias")
    except Exception as erro:
        print(f"ERRO     {alvo:40} {erro}")
```

O roteiro valida a cadeia de confiança do sistema operacional: certificados inválidos ou emitidos por autoridades privadas aparecerão como erros. Para CAs privadas, configure o arquivo de confiança correspondente. Use-o apenas em endereços que você administra, e prefira os monitores nativos de expiração dos provedores quando existirem.

## 4. Modelos

### 4.1 Padrão corporativo de criptografia (estrutura)

| Seção | Conteúdo |
|---|---|
| Objetivo e escopo | Sistemas, dados e ambientes cobertos |
| Algoritmos e protocolos aprovados | Simétricos, assimétricos, hash, MAC, assinaturas, TLS e VPN, com tamanhos mínimos |
| Algoritmos e protocolos proibidos | Lista de itens vedados e prazo de eliminação de usos existentes |
| Modelos de chaves por classificação | Matriz do arquivo 6 |
| Gestão de chaves | Hierarquia, separação de funções, rotação, exclusão e recuperação |
| Certificados | Autoridades, validade, automação e inventário |
| Bibliotecas e serviços aprovados | Opções validadas por linguagem e por plataforma |
| Validação de módulos | Requisitos de FIPS 140 ou equivalentes por classificação |
| Agilidade e transição pós-quântica | Diretrizes e prazos |
| Exceções | Processo, aprovação e prazo |
| Verificação e auditoria | Métricas e frequência |
| Responsáveis e revisão | Donos do padrão e calendário |

### 4.2 Inventário de chaves

| ID | Nome ou alias | Provedor e serviço | Tipo e algoritmo | Proteção (software, HSM ou externa) | Finalidade e dados protegidos | Proprietário | Administradores e usuários | Rotação | Região | Proteção contra exclusão | Última revisão |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CHV-001 | | | | | | | | | | | |

### 4.3 Inventário de certificados

| ID | Domínio ou identidade | Emissor (CA) | Tipo (público, privado ou ICP-Brasil) | Algoritmo e tamanho | Emissão | Expiração | Local de instalação | Renovação (automática ou manual) | Proprietário | Alertas configurados |
|---|---|---|---|---|---|---|---|---|---|---|
| CRT-001 | | | | | | | | | | |

### 4.4 Lista de materiais criptográficos (CBOM)

Uma **lista de materiais criptográficos (CBOM)** inventaria os ativos criptográficos de um sistema (algoritmos, bibliotecas, protocolos, chaves e certificados), apoiando a gestão de riscos e a transição pós-quântica. O formato CycloneDX inclui suporte a ativos criptográficos.

| Campo | Descrição |
|---|---|
| Sistema ou aplicação | Identificação |
| Componente | Biblioteca, serviço ou dispositivo que usa criptografia |
| Uso | Confidencialidade, integridade, assinatura, troca de chaves ou autenticação |
| Algoritmo e parâmetros | Por exemplo, AES-256-GCM, ECDHE P-256 e RSA-3072 |
| Protocolo e versão | TLS 1.3, IKEv2 e SSH |
| Material de chave e origem | Serviço de gestão de chaves, HSM ou arquivo |
| Vida útil dos dados protegidos | Curta, média ou longa (para priorizar a transição pós-quântica) |
| Risco pós-quântico | Alto (chave pública vulnerável com dados de longa vida), médio ou baixo |
| Plano de migração | Ação, responsável e prazo |

### 4.5 Registro de exceções criptográficas

| ID | Sistema | Desvio do padrão (algoritmo, protocolo ou modelo de chaves) | Justificativa | Risco e controles compensatórios | Aprovador | Expiração |
|---|---|---|---|---|---|---|
| EXC-001 | | | | | | |

## 5. Análise do inventário de chaves com DuckDB

`inventario_chaves.csv` (fictício):

```
id,alias,servico,tipo,protecao,classificacao,rotacao_dias,protecao_exclusao,regiao,proprietario
CHV-001,chave-extratos,kms,simetrica,hsm,restrito,365,sim,sa-east-1,Area-Financeira
CHV-002,chave-legado,kms,simetrica,software,confidencial,0,nao,us-east-1,
CHV-003,chave-assinatura-imagens,kms,assimetrica,hsm,confidencial,0,sim,sa-east-1,Plataforma
CHV-004,chave-backup,kms,simetrica,software,restrito,365,nao,sa-east-1,Operacoes
```

```sql
-- Achados: chaves sem proprietário, sem rotação (simétricas), sem proteção contra exclusão ou fora de HSM para dados restritos
SELECT id, alias, classificacao,
       CASE WHEN proprietario IS NULL OR proprietario = '' THEN 'sem proprietario' END AS achado_dono,
       CASE WHEN tipo = 'simetrica' AND rotacao_dias = 0 THEN 'sem rotacao' END AS achado_rotacao,
       CASE WHEN protecao_exclusao = 'nao' THEN 'sem protecao contra exclusao' END AS achado_exclusao,
       CASE WHEN classificacao = 'restrito' AND protecao <> 'hsm' THEN 'restrito fora de HSM' END AS achado_hsm
FROM read_csv_auto('inventario_chaves.csv')
WHERE proprietario IS NULL OR proprietario = ''
   OR (tipo = 'simetrica' AND rotacao_dias = 0)
   OR protecao_exclusao = 'nao'
   OR (classificacao = 'restrito' AND protecao <> 'hsm')
ORDER BY id;
```

## 6. Checklist de criptografia em nuvem

**Escopo:** ____________________
**Responsável:** ____________________
**Data:** ____________________

### 6.1 Padrão e inventário

- [ ] Padrão corporativo de criptografia aprovado e publicado
- [ ] Inventário de chaves, de certificados e CBOM atualizados, com proprietários
- [ ] Algoritmos e protocolos fracos identificados e com plano de eliminação
- [ ] Exceções registradas, com prazo e controles compensatórios

### 6.2 Dados em repouso e chaves

- [ ] Criptografia em repouso ativada por padrão; modo definido por classificação
- [ ] Chaves do cliente por domínio, com HSM para dados restritos
- [ ] Separação de funções, políticas de chaves e contexto de criptografia
- [ ] Rotação configurada; proteção contra exclusão; chaves disponíveis para a recuperação
- [ ] Registros de uso e de administração de chaves monitorados com detecções prioritárias

### 6.3 Dados em trânsito e certificados

- [ ] TLS 1.2 como mínimo, TLS 1.3 preferencial; políticas de TLS verificadas por varredura
- [ ] TLS imposto em serviços gerenciados e bancos de dados; mTLS para serviços críticos
- [ ] Certificados emitidos e renovados automaticamente; alertas de expiração
- [ ] VPNs e enlaces dedicados com parâmetros criptográficos atuais

### 6.4 Assinatura e preparação futura

- [ ] Artefatos e imagens assinados e verificados no pipeline
- [ ] Estratégia de assinatura eletrônica de documentos validada pelo Jurídico
- [ ] Plano de agilidade criptográfica e de transição pós-quântica, com priorização de dados de longa vida

## 7. Resultado da verificação

| Resultado | Condição |
|---|---|
| Conforme | Todos os itens aplicáveis marcados |
| Conforme com ressalvas | Pendências sem exposição de dados restritos, com plano de ação e prazos |
| Não conforme | Dados restritos sem criptografia ou sem controle de chaves adequado, certificados expirados em produção, chaves sem proteção contra exclusão ou TLS fraco em serviços expostos; corrigir com prioridade |

**Assinatura do responsável:** ____________________
**Assinatura do revisor:** ____________________

## 8. Exemplo prático: auditoria de criptografia de uma plataforma

| Verificação | Resultado | Ação |
|---|---|---|
| Volumes sem criptografia | 23 de 640 | Ativar a criptografia por padrão; recriptografar por cópia de instantâneos |
| Chaves simétricas sem rotação | 9 | Habilitar a rotação; 3 chaves com política a revisar |
| Chaves sem proteção contra exclusão | 5 (incluindo a de backups) | Política de negação e alertas; período de espera máximo |
| Balanceadores com TLS 1.0 | 2 | Aplicar política de TLS 1.2 ou superior |
| Certificados com expiração em menos de 30 dias | 7, sendo 3 sem renovação automática | Emissão automática; alertas |
| Bancos sem exigência de TLS | 4 | Parâmetros de TLS obrigatório e validação nos clientes |
| Inventário criptográfico | Inexistente | CBOM iniciado, priorizando dados de longa vida |
| Resultado | Pontuação do programa de criptografia de 54% para 91% em 12 semanas | |

## 9. Casos de uso corporativos

- Verificação contínua de criptografia, de chaves e de certificados, integrada ao pipeline.
- Inventário criptográfico e plano de transição pós-quântica.
- Evidências de criptografia para auditorias, para reguladores e para clientes.

## 10. Normas e frameworks relevantes

- NIST SP 800-57; NIST SP 800-52 Rev. 2; NIST IR 8547 (transição pós-quântica); CycloneDX (especificação de CBOM)
- CIS Foundations Benchmarks (criptografia, chaves e certificados); CSA CCM v4 (CEK); ISO/IEC 27002:2022 (8.24)
- PCI DSS v4.0 (requisitos 3 e 4); OWASP Cryptographic Storage e Transport Layer Security Cheat Sheets

## 11. Referências

- Documentação das ferramentas de linha de comando: AWS CLI, Azure CLI e Google Cloud CLI.
- Documentação do OpenSSL, do testssl.sh, do sslyze e do DuckDB.
- OWASP CycloneDX. Cryptography Bill of Materials (CBOM) (documentação).
- Mozilla. Server Side TLS (documentação).
- Pastas 05, 09, 13 e 17 deste repositório.
