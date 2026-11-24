# Métricas, Modelos e Checklist de Monitoramento

> **Objetivo pedagógico:** medir a eficácia do monitoramento de segurança em nuvem com indicadores de cobertura, de qualidade, de detecção, de resposta e de custo; calcular indicadores a partir de dados exportados com DuckDB (cobertura de fontes por conta e tempo de detecção); definir um modelo de maturidade; e usar modelos (integração de fontes, ficha de detecção, procedimento de triagem, acordos de nível de serviço) e um checklist completo de monitoramento.

## 1. Famílias de indicadores

| Família | Pergunta | Exemplos |
|---|---|---|
| Cobertura | Enxergamos o que precisamos? | Percentual de contas com trilha organizacional; fontes críticas integradas |
| Qualidade dos dados | Os dados são confiáveis e utilizáveis? | Taxa de eventos analisados com sucesso; latência de entrega; fontes silenciosas |
| Detecção | As detecções funcionam? | Cobertura ATT&CK testada; precisão; tempo de detecção |
| Resposta | Respondemos em tempo útil? | Tempo de reconhecimento, de triagem, de contenção e de resolução |
| Eficiência | O esforço é sustentável? | Alertas por analista; taxa de automação; fila e envelhecimento |
| Custo | O custo é proporcional ao valor? | Volume por fonte; custo por GB; uso por detecção |
| Conformidade | Cumprimos requisitos de registro e de retenção? | Retenção conforme a política; integridade verificada |

## 2. Indicadores recomendados

| Indicador | Definição | Meta típica |
|---|---|---|
| Cobertura de registros de auditoria | Contas e assinaturas com trilha organizacional ativa ÷ total | 100% |
| Cobertura de fontes críticas | Fontes de prioridade 1 e 2 integradas ÷ fontes planejadas | Acima de 95% |
| Fontes silenciosas | Fontes críticas sem eventos além do intervalo esperado | Zero (alerta em minutos) |
| Latência de entrega (mediana e p95) | Tempo entre o evento e sua disponibilidade para detecção | Mediana de poucos minutos |
| Taxa de parsing | Eventos normalizados com sucesso ÷ eventos recebidos | Acima de 98% |
| Cobertura ATT&CK testada | Técnicas priorizadas com detecção validada ÷ técnicas priorizadas | Acima de 80% |
| Precisão das detecções | Alertas verdadeiros ÷ alertas gerados, por regra e no conjunto | Acima de 60 a 70%, com meta crescente |
| Tempo médio de detecção (MTTD) | Do início da atividade à geração do alerta | Minutos para casos críticos |
| Tempo de reconhecimento (MTTA) | Do alerta ao início do atendimento | Dentro do prazo por severidade |
| Tempo de contenção e de resolução (MTTR) | Do alerta à contenção e à resolução | Por severidade |
| Percentual de alertas fechados dentro do prazo | Alertas tratados no prazo ÷ total | Acima de 95% |
| Idade da fila | Alertas abertos além do prazo | Zero para críticos e altos |
| Taxa de automação | Alertas enriquecidos ou tratados automaticamente ÷ total | Crescente |
| Volume de ingestão por fonte | GB por dia, por fonte, com variação | Monitorado, com alertas de picos |
| Custo por GB e por fonte | Custos de ingestão, de armazenamento e de consulta | Em linha com o valor da fonte |
| Retenção conforme a política | Fontes com retenção igual ou superior à política ÷ total | 100% |
| Integridade verificada | Verificações de integridade bem-sucedidas no período | 100% |
| Incidentes detectados internamente | Incidentes descobertos pela organização ÷ incidentes totais (versus avisados por terceiros) | Crescente |
| Exercícios de detecção realizados | Emulações e equipes roxas concluídas no período | Conforme o plano |

## 3. Cálculos com DuckDB

### 3.1 Cobertura de fontes por conta

`cobertura_fontes.csv` (fictício; um registro por conta e fonte esperada):

```
conta,fonte,esperada,ativa,ultimo_evento_utc
conta-prod-01,auditoria_plano_controle,sim,sim,2026-11-20 10:42:00
conta-prod-01,fluxos_rede,sim,sim,2026-11-20 10:35:00
conta-prod-01,acesso_dados,sim,nao,
conta-prod-02,auditoria_plano_controle,sim,sim,2026-11-19 02:10:00
conta-dev-07,auditoria_plano_controle,sim,nao,
conta-dev-07,fluxos_rede,nao,nao,
```

```sql
-- Fontes esperadas inativas ou silenciosas (sem eventos há mais de 6 horas em relação a 2026-11-20 11:00 UTC)
SELECT conta, fonte,
       CASE
         WHEN ativa = 'nao' THEN 'nao habilitada'
         WHEN CAST(ultimo_evento_utc AS TIMESTAMP) < TIMESTAMP '2026-11-20 05:00:00' THEN 'silenciosa'
       END AS problema
FROM read_csv_auto('cobertura_fontes.csv')
WHERE esperada = 'sim'
  AND (ativa = 'nao' OR CAST(ultimo_evento_utc AS TIMESTAMP) < TIMESTAMP '2026-11-20 05:00:00')
ORDER BY conta, fonte;

-- Percentual de cobertura por fonte
SELECT fonte,
       count(*) AS esperadas,
       count(*) FILTER (WHERE ativa = 'sim') AS ativas,
       round(100.0 * count(*) FILTER (WHERE ativa = 'sim') / count(*), 1) AS cobertura_percentual
FROM read_csv_auto('cobertura_fontes.csv')
WHERE esperada = 'sim'
GROUP BY fonte;
```

### 3.2 Tempos de detecção e de resposta por severidade

`incidentes_tempos.csv` (fictício):

```
id,severidade,inicio_atividade_utc,alerta_utc,contencao_utc
INC-101,critica,2026-10-02 03:10:00,2026-10-02 03:16:00,2026-10-02 03:40:00
INC-102,alta,2026-10-09 14:00:00,2026-10-09 14:50:00,2026-10-09 17:30:00
INC-103,critica,2026-10-21 22:05:00,2026-10-21 22:09:00,2026-10-21 22:31:00
INC-104,alta,2026-11-03 08:20:00,2026-11-03 08:55:00,2026-11-03 10:05:00
```

```sql
-- Medianas de MTTD (início ao alerta) e de tempo até a contenção, em minutos, por severidade
SELECT severidade,
       count(*) AS incidentes,
       median(date_diff('minute', CAST(inicio_atividade_utc AS TIMESTAMP), CAST(alerta_utc AS TIMESTAMP))) AS mttd_mediana_min,
       median(date_diff('minute', CAST(alerta_utc AS TIMESTAMP), CAST(contencao_utc AS TIMESTAMP))) AS contencao_mediana_min
FROM read_csv_auto('incidentes_tempos.csv')
GROUP BY severidade
ORDER BY severidade;
```

## 4. Modelo de maturidade do monitoramento (avaliação)

| Dimensão | Nível 1 Básico | Nível 2 Centralizado | Nível 3 Gerenciado | Nível 4 Otimizado | Nível atual |
|---|---|---|---|---|---|
| Fontes e cobertura | Registros nativos ativos, sem centralização | Auditoria e identidade centralizadas em todas as contas | Cobertura ampla, com acesso a dados, rede e cargas | Cobertura contínua verificada, com alertas de silêncio | |
| Retenção e integridade | Retenção padrão do provedor | Conta de registros e retenção por política | Imutabilidade, validação e acesso controlado | Testes periódicos de recuperação e de integridade | |
| Detecção | Alertas dos serviços nativos | Detecções prioritárias manuais | Detecções como código, mapeadas ao ATT&CK, testadas | Detecção comportamental, inteligência e validação por emulação | |
| Operação do SOC | Atendimento ad hoc | Triagem com responsáveis e prazos | Playbooks, SOAR com salvaguardas e caça a ameaças | Automação ampla, melhoria contínua e exercícios regulares | |
| Indicadores | Sem indicadores | Indicadores de volume | Indicadores de cobertura, de qualidade, de detecção e de resposta | Indicadores integrados ao risco e à direção | |
| Governança e privacidade | Informal | Política de registros | Política, retenção, acesso e privacidade aprovados | Auditorias e melhorias regulares | |

## 5. Modelos

### 5.1 Ficha de integração de fonte de dados

| Campo | Preenchimento |
|---|---|
| ID e nome da fonte | |
| Provedor, serviço e escopo (contas e regiões) | |
| Prioridade (1 a 4) e justificativa | |
| Responsável pela fonte e pela integração | |
| Método de coleta e destino | |
| Esquema e mapeamento de campos | |
| Retenção (quente, morna e fria) e política de acesso | |
| Dados pessoais ou segredos possíveis e tratamento | |
| Detecções que dependem da fonte | |
| Monitoramento de silêncio (intervalo esperado e alerta) | |
| Custos estimados (volume por dia) | |
| Data de integração e última revisão | |

### 5.2 Procedimento de triagem (resumo por alerta)

| Campo | Preenchimento |
|---|---|
| ID da detecção e do alerta | |
| Descrição e gravidade | |
| Perguntas de validação (quem, o quê, de onde, quando e normalidade) | |
| Consultas e fontes para investigar | |
| Critérios para fechar como falso positivo | |
| Critérios para abrir incidente e escalar | |
| Ações de contenção (manuais e automáticas) | |
| Contatos e responsáveis | |
| Obrigações de comunicação (DPO, Jurídico e reguladores) | |
| Referências (playbook e ATT&CK) | |

### 5.3 Acordo de nível de serviço do monitoramento (modelo)

| Item | Meta |
|---|---|
| Latência de entrega de registros críticos | Mediana de até 5 minutos; p95 de até 15 minutos |
| Disponibilidade da plataforma de detecção | Conforme o acordo (por exemplo, 99,5%) |
| Reconhecimento de alertas críticos | Até 15 minutos (24x7) |
| Triagem inicial de alertas críticos | Até 30 minutos |
| Relatório mensal de indicadores | Até o 5º dia útil |
| Revisão trimestral de regras e de exceções | Trimestral |
| Testes de detecção (emulação) | Ao menos duas rodadas por ano |

### 5.4 Registro de exercícios de detecção

| Data | Técnica emulada | Ambiente (laboratório) | Prevenido? | Detectado? | Tempo de detecção | Resposta adequada? | Lacunas e ações | Responsável |
|---|---|---|---|---|---|---|---|---|
| | | | | | | | | |

## 6. Checklist de monitoramento de segurança em nuvem

**Escopo:** ____________________
**Responsável:** ____________________
**Data:** ____________________

### 6.1 Fontes e coleta

- [ ] Trilhas de auditoria organizacionais e registros de identidade ativos em 100% das contas
- [ ] Registros de acesso a dados, de uso de chaves, de rede (fluxo e DNS) e de Kubernetes habilitados conforme a prioridade
- [ ] Serviços de detecção e de postura habilitados em todas as contas e regiões
- [ ] Coleta centralizada na conta de registros, desacoplada do SIEM, com alertas de silêncio de fontes
- [ ] Esquema comum e enriquecimento com inventário, identidade e inteligência

### 6.2 Retenção, integridade e privacidade

- [ ] Retenção por fonte, aprovada com o Jurídico e a privacidade; camadas de armazenamento
- [ ] Armazenamento imutável, validação de integridade e políticas que impedem a exclusão e a desabilitação
- [ ] Acesso por papéis, com registro de acessos e aprovação para exportações
- [ ] Dados pessoais e segredos em registros tratados

### 6.3 Detecção e operação

- [ ] Catálogo de detecções priorizado por riscos, como código, testado e mapeado ao ATT&CK
- [ ] Severidades, prazos, playbooks e automação com salvaguardas definidos
- [ ] Caça a ameaças periódica e conversão de achados em detecções
- [ ] Monitoramento complementar: configuração, custos, superfície externa, comunicados dos provedores e inteligência
- [ ] Serviços gerenciados (se houver) com escopo, acessos mínimos, SLAs e privacidade definidos

### 6.4 Medição e melhoria

- [ ] Indicadores de cobertura, de qualidade, de detecção, de resposta e de custo reportados
- [ ] Exercícios de detecção por emulação em laboratório e equipe roxa, com lacunas tratadas
- [ ] Revisão trimestral das regras, das exceções e das fontes
- [ ] Revisão anual da estratégia e da maturidade

## 7. Resultado da verificação

| Resultado | Condição |
|---|---|
| Conforme | Todos os itens aplicáveis marcados |
| Conforme com ressalvas | Pendências sem lacunas de registros críticos, com plano de ação e prazos |
| Não conforme | Contas de produção sem auditoria, retenção insuficiente de registros críticos, ausência de proteção de integridade, ausência de detecções críticas ou de responsável por alertas; corrigir com prioridade |

**Assinatura do responsável:** ____________________
**Assinatura do revisor:** ____________________

## 8. Exemplo prático: relatório trimestral de monitoramento à direção

| Indicador | Trimestre anterior | Trimestre atual | Meta |
|---|---|---|---|
| Cobertura de registros de auditoria | 92% | 100% | 100% |
| Fontes críticas silenciosas | 4 | 0 | 0 |
| Cobertura ATT&CK testada (técnicas priorizadas) | 61% | 79% | 80% |
| Precisão das detecções | 58% | 72% | 70% |
| Tempo mediano de detecção (críticos) | 19 min | 7 min | 10 min |
| Tempo mediano de contenção (críticos) | 94 min | 28 min | 30 min |
| Alertas fechados no prazo | 83% | 96% | 95% |
| Custo por GB ingerido | referência | -18% | Redução com cobertura mantida |
| Decisões solicitadas | Aprovar a retenção de 12 meses para acesso a dados restritos; financiar o MDR noturno; priorizar as 4 técnicas sem detecção | | |

## 9. Casos de uso corporativos

- Painel e relatório trimestral de monitoramento para a direção e para o comitê de riscos.
- Avaliação de maturidade do monitoramento e priorização de investimentos.
- Evidências de monitoramento para auditorias, para reguladores e para clientes.

## 10. Normas e frameworks relevantes

- NIST SP 800-55 Rev. 1 (medição de desempenho de segurança da informação); NIST SP 800-137; NIST CSF 2.0 (DE e GV.OV)
- ISO/IEC 27004:2016 (medição); ISO/IEC 27001:2022 (9.1); CSA CCM v4 (LOG)
- MITRE ATT&CK (cobertura); CIS Controls v8.1 (controles 8 e 13)

## 11. Referências

- Chew, E. et al. NIST SP 800-55 Rev. 1: Performance Measurement Guide for Information Security. 2008.
- ISO/IEC 27004:2016. Information security management: Monitoring, measurement, analysis and evaluation.
- Documentação do DuckDB.
- MITRE. ATT&CK Navigator (documentação).
- Pastas 12, 27 e 28 deste repositório.
