# Ferramentas, Verificações, Modelos e Checklist de Proteção de Dados

> **Objetivo pedagógico:** verificar o estado de proteção de dados nos três principais provedores com comandos de **somente leitura** (acesso público, criptografia, versionamento, registros, bancos de dados e serviços de classificação), analisar um inventário exportado com DuckDB, usar modelos (inventário de dados, matriz de classificação e de controles, cronograma de retenção, registro de fluxos de dados e esqueleto de RIPD) e aplicar um checklist de proteção de dados em nuvem.

> **Segurança:** os comandos apenas **consultam** configurações. Use identidades de leitura, em contas autorizadas, e registre as saídas como evidência (data e hora em UTC, conta e comando). Identificadores são fictícios; as opções variam entre versões: confirme na documentação. **Não** exporte nem armazene conteúdo de dados pessoais em relatórios de verificação.

## 1. Ferramentas por finalidade

| Finalidade | Ferramentas |
|---|---|
| Descoberta e classificação | Amazon Macie; Microsoft Purview; Google Sensitive Data Protection; soluções de gestão de postura de segurança de dados (DSPM) |
| Verificação de configuração | AWS CLI, Azure CLI e Google Cloud CLI; AWS Config; Azure Policy; Cloud Asset Inventory |
| Infraestrutura como código e políticas | Terraform, Bicep e CloudFormation; Open Policy Agent; Checkov e Trivy |
| Prevenção de perda de dados | Serviços de DLP dos provedores e de plataformas de colaboração; proxies e CASB |
| Chaves e segredos | Serviços de gestão de chaves e de segredos (pasta 10) |
| Backup e retenção | AWS Backup, Azure Backup, backups de bancos e de armazenamento; recursos de ciclo de vida e de imutabilidade |
| Privacidade e conformidade | Plataformas de gestão de privacidade, registros de operações e RIPD |
| Análise de evidências | DuckDB, Athena e SIEM |

## 2. Verificações de somente leitura

### 2.1 AWS

```bash
# Bloqueio de acesso público por conta e por bucket, para todos os buckets
aws s3control get-public-access-block --account-id 111122223333

for b in $(aws s3api list-buckets --query "Buckets[].Name" --output text); do
  echo -n "$b: "
  aws s3api get-public-access-block --bucket "$b" \
    --query "PublicAccessBlockConfiguration.[BlockPublicAcls,IgnorePublicAcls,BlockPublicPolicy,RestrictPublicBuckets]" \
    --output text 2>/dev/null || echo "SEM bloqueio configurado no bucket"
done

# Criptografia padrão e versionamento por bucket
for b in $(aws s3api list-buckets --query "Buckets[].Name" --output text); do
  echo -n "$b | criptografia: "
  aws s3api get-bucket-encryption --bucket "$b" \
    --query "ServerSideEncryptionConfiguration.Rules[0].ApplyServerSideEncryptionByDefault.SSEAlgorithm" --output text 2>/dev/null || echo "nao configurada"
  echo -n "$b | versionamento: "
  aws s3api get-bucket-versioning --bucket "$b" --query "Status" --output text
done

# Bancos relacionais públicos ou sem criptografia; volumes sem criptografia
aws rds describe-db-instances --query "DBInstances[?PubliclyAccessible==\`true\` || StorageEncrypted==\`false\`].[DBInstanceIdentifier,PubliclyAccessible,StorageEncrypted]" --output table
aws ec2 describe-volumes --filters Name=encrypted,Values=false --query "Volumes[].[VolumeId,Size,State]" --output table

# Instantâneos compartilhados publicamente (próprios)
aws ec2 describe-snapshots --owner-ids self --restorable-by-user-ids all --query "Snapshots[].[SnapshotId,VolumeSize]" --output table

# Amazon Macie: sessão e trabalhos de classificação
aws macie2 get-macie-session
aws macie2 list-classification-jobs --query "items[].[jobId,name,jobStatus]" --output table
```

### 2.2 Azure

```bash
# Contas de armazenamento: acesso anônimo, chave compartilhada, TLS e tráfego apenas HTTPS
az storage account list \
  --query "[].{nome:name, anonimo:allowBlobPublicAccess, chaveCompartilhada:allowSharedKeyAccess, tlsMinimo:minimumTlsVersion, apenasHttps:enableHttpsTrafficOnly, acessoRede:networkRuleSet.defaultAction}" --output table

# Bancos SQL: acesso público à rede
az sql server list --query "[].{nome:name, rg:resourceGroup, acessoPublico:publicNetworkAccess}" --output table

# Discos sem criptografia por chave gerenciada pelo cliente (visão geral)
az disk list --query "[].{nome:name, rg:resourceGroup, criptografia:encryption.type}" --output table
```

### 2.3 Google Cloud

```bash
# Buckets e propriedades de acesso (confirme os nomes dos campos na sua versão da CLI)
for b in $(gcloud storage buckets list --format="value(name)"); do
  echo "== $b"
  gcloud storage buckets describe "gs://$b" --format="yaml(public_access_prevention,uniform_bucket_level_access,versioning_enabled,location)"
done

# Instâncias de banco com IP público
gcloud sql instances list --format="table(name,databaseVersion,settings.ipConfiguration.ipv4Enabled)"

# Restrição de localização de recursos na organização
gcloud resource-manager org-policies describe gcp.resourceLocations --organization=123456789012
```

## 3. Análise de inventário exportado com DuckDB

`inventario_dados.csv` (fictício):

```
recurso,tipo,classificacao,contem_dados_pessoais,regiao,acesso_publico,criptografia_chave_cliente,versionamento,dono
bucket-clientes-prod,armazenamento,restrito,sim,sa-east-1,nao,sim,sim,Area-Clientes
bucket-teste-importacao,armazenamento,restrito,sim,us-east-1,sim,nao,nao,
db-pagamentos,banco,restrito,sim,sa-east-1,nao,sim,n/a,Area-Financeira
bucket-relatorios,armazenamento,interno,nao,sa-east-1,nao,nao,sim,Area-BI
snapshot-legado-2023,instantaneo,confidencial,sim,eu-west-1,nao,nao,n/a,
```

```sql
-- Achados: dados restritos ou pessoais com exposição, região não aprovada, sem dono ou sem chave do cliente
SELECT recurso, classificacao, regiao,
       CASE WHEN acesso_publico = 'sim' THEN 'acesso publico' END AS achado_exposicao,
       CASE WHEN regiao NOT IN ('sa-east-1') THEN 'regiao nao aprovada' END AS achado_regiao,
       CASE WHEN dono IS NULL OR dono = '' THEN 'sem dono' END AS achado_dono,
       CASE WHEN classificacao = 'restrito' AND criptografia_chave_cliente = 'nao' THEN 'restrito sem chave do cliente' END AS achado_chave
FROM read_csv_auto('inventario_dados.csv')
WHERE acesso_publico = 'sim'
   OR regiao NOT IN ('sa-east-1')
   OR dono IS NULL OR dono = ''
   OR (classificacao = 'restrito' AND criptografia_chave_cliente = 'nao')
ORDER BY CASE classificacao WHEN 'restrito' THEN 1 WHEN 'confidencial' THEN 2 ELSE 3 END;
```

A lista de regiões aprovadas do exemplo (`sa-east-1`) é fictícia: use a da sua política.

## 4. Modelos

### 4.1 Inventário e catálogo de dados

| ID | Conjunto de dados | Descrição | Classificação | Categorias legais (pessoal, sensível ou de crianças) | Dono | Custodiante | Sistemas e contas | Regiões | Finalidade e base legal | Retenção | Controles aplicados |
|---|---|---|---|---|---|---|---|---|---|---|---|
| DAT-001 | | | | | | | | | | | |

### 4.2 Registro de fluxos de dados e de transferências

| ID | Origem | Destino | Dados e classificação | Finalidade | Operador ou terceiro | País ou região de destino | Hipótese legal e instrumento (arquivo 6) | Medidas complementares | Revisão |
|---|---|---|---|---|---|---|---|---|---|
| FLX-001 | | | | | | | | | |

### 4.3 Matriz de controles por classificação

| Controle | Público | Interno | Confidencial | Restrito |
|---|---|---|---|---|
| Acesso público permitido | Sim, se necessário | Não | Não | Não |
| Acesso privado (pontos privados) | Opcional | Recomendado | Obrigatório | Obrigatório |
| Criptografia em repouso | Padrão | Padrão | Padrão com chaves gerenciadas | Chaves gerenciadas pelo cliente, exclusivas |
| Registros de acesso a dados | Opcional | Recomendado | Obrigatório | Obrigatório, com alertas |
| Controle em nível de linha e de coluna e mascaramento | n/a | Opcional | Recomendado | Obrigatório |
| DLP | n/a | Monitoramento | Monitoramento e bloqueio seletivo | Bloqueio |
| Versionamento e imutabilidade | Opcional | Recomendado | Recomendado | Obrigatório para dados críticos |
| Backup isolado e testado | Opcional | Recomendado | Obrigatório | Obrigatório, imutável |
| Regiões | Livres | Aprovadas | Aprovadas | Restritas ao Brasil, salvo exceções formais |
| Dados em ambientes de teste | Permitido | Mascarados | Sintéticos ou mascarados | Proibido |

### 4.4 Cronograma de retenção

| Categoria de dados | Base e referência (lei, norma ou contrato) | Prazo | Local e mecanismo | Eliminação | Dono | Revisão |
|---|---|---|---|---|---|---|
| | | | | | | |

### 4.5 Esqueleto de RIPD (relatório de impacto à proteção de dados pessoais)

| Seção | Conteúdo |
|---|---|
| 1. Identificação | Controlador, operadores, encarregado e descrição da atividade de tratamento |
| 2. Descrição do tratamento | Dados tratados, categorias de titulares, finalidades, base legal, fluxos, operadores, transferências e retenção |
| 3. Necessidade e proporcionalidade | Minimização, alternativas menos intrusivas e justificativas |
| 4. Riscos aos titulares | Cenários de risco, probabilidade e severidade, com foco no impacto sobre os titulares |
| 5. Medidas e salvaguardas | Controles técnicos e organizacionais (esta pasta), com os riscos residuais |
| 6. Direitos dos titulares | Como os direitos serão atendidos |
| 7. Conclusão e aprovação | Decisão, ações pendentes, responsáveis e data de revisão |

## 5. Checklist de proteção de dados em nuvem

**Escopo:** ____________________
**Responsável:** ____________________
**Data:** ____________________

### 5.1 Descoberta e classificação

- [ ] Inventário de dados, de contas e de regiões atualizado; dados ocultos tratados
- [ ] Esquema de classificação aprovado e aplicado por etiquetas e por rótulos
- [ ] Donos de dados atribuídos; revisão periódica da classificação
- [ ] Registro das operações de tratamento e RIPD atualizados

### 5.2 Acesso e proteção

- [ ] Bloqueio de acesso público ativo na organização; nenhum armazenamento ou banco público não justificado
- [ ] Acesso privado para dados confidenciais e restritos; privilégio mínimo e acesso por atributos
- [ ] Criptografia em repouso e em trânsito; chaves gerenciadas pelo cliente para dados restritos (pasta 10)
- [ ] Registros de acesso a dados habilitados e centralizados, com alertas
- [ ] DLP por classificação; controles de compartilhamento externo
- [ ] Mascaramento e pseudonimização por finalidade; sem dados reais em testes

### 5.3 Retenção, backup e eliminação

- [ ] Cronograma de retenção aplicado por regras técnicas de ciclo de vida
- [ ] Backups isolados, criptografados, imutáveis e testados (3-2-1-1-0)
- [ ] Procedimento de eliminação com propagação e destruição criptográfica; evidências guardadas
- [ ] Encerramento de contratos com devolução e eliminação certificada

### 5.4 Residência e transferência

- [ ] Regiões aprovadas por classificação, impostas por política técnica
- [ ] Fluxos internacionais mapeados, com hipótese legal e instrumento (cláusulas-padrão)
- [ ] Lista de subprocessadores monitorada; medidas complementares avaliadas
- [ ] Verificação contínua de recursos fora das regiões aprovadas

## 6. Resultado da verificação

| Resultado | Condição |
|---|---|
| Conforme | Todos os itens aplicáveis marcados |
| Conforme com ressalvas | Pendências sem exposição de dados confidenciais ou restritos, com plano de ação e prazos |
| Não conforme | Dados confidenciais ou restritos expostos publicamente, sem criptografia, sem dono, em região não aprovada sem instrumento, ou sem backups testados; corrigir com prioridade |

**Assinatura do responsável:** ____________________
**Assinatura do revisor:** ____________________

## 7. Exemplo prático: auditoria de proteção de dados de um domínio de produto

| Verificação | Resultado | Ação |
|---|---|---|
| Buckets sem bloqueio de acesso público | 5 de 62 | Ativar o bloqueio na organização; 1 caso era uma exposição real, tratado como incidente |
| Dados restritos sem chave do cliente | 8 recursos | Migrar para chaves gerenciadas pelo cliente, com plano de 60 dias |
| Registros de acesso a dados | Ausentes em 14 buckets com dados pessoais | Habilitar e centralizar |
| Recursos fora da região aprovada | 3 instantâneos antigos e 1 réplica | Eliminar ou documentar a base legal e o instrumento |
| Dados reais em teste | 2 bancos | Substituir por dados mascarados; eliminar os dados reais |
| Retenção | Registros sem expiração configurada | Regras de ciclo de vida aplicadas |
| Resultado | Índice de conformidade de proteção de dados de 61% para 93% em 10 semanas | |

## 8. Casos de uso corporativos

- Auditorias periódicas de proteção de dados com evidências padronizadas.
- Painel de postura de dados por domínio, por conta e por classificação.
- Padronização de inventário, de cronograma de retenção, de registro de fluxos e de RIPD.

## 9. Normas e frameworks relevantes

- CIS Foundations Benchmarks (armazenamento e bancos de dados); CIS Controls v8.1 (controles 3 e 11)
- NIST SP 800-53 Rev. 5; NIST Privacy Framework; CSA CCM v4 (DSP e LOG); ISO/IEC 27002:2022 e 27701:2019
- LGPD, arts. 37, 38 e 46; ISO/IEC 29134:2023 (avaliação de impacto à privacidade)

## 10. Referências

- Documentação das ferramentas de linha de comando: AWS CLI, Azure CLI e Google Cloud CLI.
- Documentação do DuckDB.
- ISO/IEC 29134:2023. Guidelines for privacy impact assessment.
- ANPD. Orientações sobre o relatório de impacto à proteção de dados pessoais (documentação vigente).
- Pastas 06, 10, 11 e 17 deste repositório.
