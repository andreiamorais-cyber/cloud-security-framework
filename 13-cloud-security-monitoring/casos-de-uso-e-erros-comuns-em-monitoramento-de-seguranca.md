# Casos de Uso e Erros Comuns em Monitoramento de Segurança

> **Objetivo pedagógico:** consolidar o aprendizado de monitoramento de segurança em nuvem com casos corporativos fictícios, os erros mais frequentes, suas consequências e a prevenção, exercícios práticos e perguntas de revisão.

Os casos são fictícios e destinados ao treinamento.

## 1. Caso 1: incidente sem registros para investigar

| Item | Descrição |
|---|---|
| Situação | Uma empresa dependia do histórico de eventos padrão do provedor (90 dias, apenas eventos de gerenciamento), sem trilhas e sem eventos de dados |
| Incidente | Um acesso indevido a um armazenamento com dados de clientes foi descoberto 5 meses depois, por um aviso de terceiros |
| Problema | Não foi possível determinar quais objetos foram acessados, por quem e quando; a organização teve de assumir o pior caso na comunicação aos titulares |
| Causas | Registros de dados desabilitados; retenção curta; ausência de centralização |
| Correções | Trilhas organizacionais, eventos de dados para dados restritos, conta de registros imutável e retenção de 12 meses ou mais, com verificação contínua da habilitação |

**Lição:** os registros precisam ser habilitados e retidos **antes** do incidente; sem eles, a investigação e a comunicação ficam comprometidas.

## 2. Caso 2: atacante desabilitou os registros

| Item | Descrição |
|---|---|
| Situação | Uma credencial administrativa foi comprometida em uma conta de produção |
| Incidente | O atacante parou a trilha de auditoria da conta e removeu a configuração de registros de fluxo antes de agir |
| Efeito | Como havia uma política organizacional que impedia a parada da trilha organizacional, a tentativa falhou, e o alerta crítico DET-NUV-002 acionou o SOC em 4 minutos; os registros de fluxo removidos foram detectados pela verificação de configuração |
| Resposta | Revogação das credenciais, investigação com registros íntegros e correção |
| Resultado | Nenhum dado exfiltrado; atacante contido em 27 minutos |

**Lição:** proteger a própria infraestrutura de registros (políticas preventivas, imutabilidade e alertas de alteração) é essencial; os atacantes tentam cegar o monitoramento.

## 3. Caso 3: milhares de alertas e nenhuma resposta

| Item | Descrição |
|---|---|
| Situação | A organização ativou todos os serviços de detecção e integrou todas as fontes ao SIEM, sem ajuste |
| Problema | 6 mil alertas por semana; analistas sobrecarregados fechavam alertas em massa; um alerta de credencial usada de um país incomum ficou 9 dias sem triagem e levou a um incidente |
| Causas | Sem priorização por contexto; sem ajuste de regras; sem responsáveis e prazos; sem automação |
| Resposta | Análise do volume por regra, ajuste ou retirada de regras ruidosas, enriquecimento com criticidade do ativo, playbooks e SLAs |
| Resultado | 85% de redução de volume; tempo mediano de resposta a alertas críticos de 3 dias para 22 minutos |

**Lição:** mais dados e mais alertas não significam mais segurança; qualidade, priorização e responsabilidade importam mais.

## 4. Caso 4: abuso de recursos descoberto pelo custo

| Item | Descrição |
|---|---|
| Situação | Credenciais de um usuário de integração vazaram em um repositório público |
| Incidente | Um atacante criou dezenas de instâncias de GPU em uma região nunca usada pela empresa para minerar criptomoedas |
| Detecção | O alerta de anomalia de custo (e não uma detecção de segurança) apontou o aumento, 2 dias depois |
| Resposta | Correlação com a auditoria, desativação da chave, encerramento dos recursos e investigação |
| Correções | Política que nega regiões não aprovadas, cotas de GPU, varredura de segredos, detecção de criação de instâncias em regiões incomuns e alertas de custo roteados ao SOC |

**Lição:** custos e uso são sinais de segurança; correlacioná-los com os registros de auditoria acelera a detecção.

## 5. Caso 5: automação que causou o próprio incidente

| Item | Descrição |
|---|---|
| Situação | Um fluxo de SOAR desativava automaticamente qualquer credencial associada a um alerta de "uso de origem incomum" |
| Incidente | Uma falha de enriquecimento classificou como "incomum" o endereço de saída do sistema de implantação, e o fluxo desativou a credencial de produção do pipeline, interrompendo implantações e a recuperação de um serviço |
| Causas | Sem listas de exclusão para identidades críticas; sem limites de escopo; sem validação do enriquecimento; ações de alto impacto sem aprovação |
| Resposta | Reativação, revisão do fluxo e do enriquecimento |
| Correções | Listas de exclusão, limites de ações por hora, aprovação para identidades críticas, reversibilidade documentada e testes em laboratório |

**Lição:** a automação de resposta precisa de salvaguardas; ações de alto impacto exigem aprovação humana.

## 6. Erros comuns, consequências e prevenção

| Erro | Consequência provável | Prevenção |
|---|---|---|
| Depender dos registros padrão e de curta retenção | Sem evidências em investigações | Trilhas organizacionais e retenção por política |
| Não habilitar registros de acesso a dados | Impossibilidade de estimar o alcance de incidentes | Habilitar para dados confidenciais e restritos |
| Registros nas mesmas contas das cargas | Atacantes apagam os registros | Conta de registros separada, imutável e protegida |
| Não monitorar a própria coleta | Fontes silenciosas e cegueira | Alertas de silêncio e de alteração de configuração |
| Coletar tudo sem priorizar | Custos altos e ruído | Priorização por valor e por risco |
| Filtrar ou amostrar registros críticos para reduzir custos | Lacunas de investigação | Reduzir custos com camadas e filtros seletivos |
| Detecções sem dono, sem teste e sem procedimento | Alertas ignorados ou imprecisos | Detecções como código, com ficha e responsável |
| Regras importadas sem ajuste ao ambiente | Falsos positivos e fadiga | Ajuste, medição de precisão e revisão |
| Cobertura ATT&CK declarada, mas não testada | Falsa segurança | Validação por emulação em laboratório |
| Não correlacionar identidade, rede, dados e custos | Cadeias de ataque não percebidas | Normalização, enriquecimento e correlação |
| Alertas sem prazos e sem escalonamento | Respostas tardias | Severidades, SLAs e escalonamento |
| Automação de resposta sem salvaguardas | Incidentes causados pela automação | Listas de exclusão, limites e aprovações |
| Terceirizar o SOC sem RACI, sem acessos mínimos e sem contexto | Respostas inadequadas e riscos de acesso | Contrato, RACI, privilégios mínimos e contexto atualizado |
| Dados pessoais e segredos em registros sem tratamento | Riscos de privacidade e de comprometimento | Filtros, mascaramento, acesso restrito e retenção por finalidade |
| Ignorar a superfície externa e o *shadow IT* | Exposições não inventariadas | Monitoramento da superfície externa |
| Não medir | Sem prova de melhoria | Indicadores e relatórios periódicos |

## 7. Exercícios práticos

1. Para uma organização fictícia com três contas de nuvem, liste as fontes de registros por camada (arquivo 2), atribua prioridades e defina a retenção de cada uma.
2. Em uma conta de laboratório, execute as verificações de habilitação do arquivo 2 e produza um relatório de achados (trilhas, fluxos, DNS e acesso a dados).
3. Desenhe, em Mermaid, a arquitetura de coleta (arquivo 3) de uma organização com AWS e Azure, indicando a conta de registros, a normalização e as camadas.
4. Escreva uma política de retenção com três camadas para cinco fontes, justificando cada prazo e as obrigações aplicáveis.
5. Escreva uma regra Sigma para a detecção "política de armazenamento com acesso público" e o teste unitário com quatro eventos (dois positivos e dois negativos), usando o modelo do arquivo 4.
6. Preencha a ficha de detecção de três casos de uso do catálogo, com procedimento de triagem e falsos positivos esperados.
7. Monte um playbook para "consentimento de aplicativo com permissões amplas", indicando as ações manuais e as automáticas, as salvaguardas e as comunicações.
8. Com os dados sintéticos do arquivo 7, execute as consultas em DuckDB (cobertura e tempos) e prepare o relatório trimestral.
9. Elabore uma hipótese de caça a ameaças (arquivo 5), as consultas necessárias e os critérios para converter os achados em detecção.
10. Em um laboratório isolado e autorizado, emule uma técnica (por exemplo, parada de trilha) e preencha o registro de exercício de detecção do arquivo 7.

Use somente contas de laboratório e dados sintéticos nos exercícios técnicos, e execute emulações apenas com autorização e isolamento.

## 8. Perguntas de revisão

| Pergunta | Resposta esperada (resumo) |
|---|---|
| Por que os registros padrão de curta retenção não bastam? | Têm retenção curta e escopo limitado (por exemplo, apenas eventos de gerenciamento); investigações e conformidade exigem trilhas, eventos de dados e retenção longa |
| Por que centralizar os registros em uma conta separada e imutável? | Para impedir que atacantes ou administradores das cargas apaguem ou alterem as evidências |
| Qual a diferença entre observabilidade e monitoramento de segurança? | A observabilidade avalia saúde e desempenho; o monitoramento de segurança busca atividade maliciosa e falhas de controles, com requisitos de integridade e de retenção |
| O que é detecção como código? | Regras versionadas, revisadas, testadas e implantadas por pipeline, com metadados e dono |
| O que é o formato Sigma? | Formato aberto e genérico de regras de detecção, convertido para linguagens de diferentes SIEM |
| Por que mapear detecções ao ATT&CK? | Para medir a cobertura de técnicas relevantes, priorizar lacunas e validar a eficácia |
| Para que serve a validação por emulação? | Verificar se as técnicas de ataque são impedidas ou detectadas, e em quanto tempo |
| Por que as anomalias de custo são úteis à segurança? | Abusos de credenciais e de recursos (como mineração) geram custos e podem ser o primeiro sinal |
| O que é EASM? | Monitoramento da superfície de ataque externa: ativos, DNS, certificados e *shadow IT* expostos à internet |
| Quais salvaguardas a automação de resposta exige? | Limites de escopo, listas de exclusão, reversibilidade, registro, permissões mínimas, testes e aprovação para ações de alto impacto |
| A organização continua responsável ao contratar um MDR? | Sim: pelos dados, pelas decisões de resposta e pelas comunicações obrigatórias |

## 9. Casos de uso corporativos

- Treinamento de analistas de SOC, de engenheiros de detecção e de equipes de plataforma.
- Diagnóstico do monitoramento de nuvem (cobertura, retenção, qualidade e detecção) e plano de evolução.
- Base para as pastas de ameaças, de postura, de resposta a incidentes e de métricas (14, 17, 22 e 27).

## 10. Normas e frameworks relevantes

- NIST SP 800-92; NIST SP 800-137; NIST SP 800-61 Rev. 3; NIST CSF 2.0 (função Detectar)
- ISO/IEC 27002:2022 (8.15 a 8.17); CSA CCM v4 (LOG); CIS Controls v8.1 (controles 8 e 13)
- MITRE ATT&CK for Cloud; OCSF; Sigma; PCI DSS v4.0.1 (requisito 10)

## 11. Referências

- Kent, K.; Souppaya, M. NIST SP 800-92. 2006.
- NIST. Cybersecurity Framework 2.0. 2024.
- MITRE. ATT&CK for Cloud (documentação).
- OCSF e SigmaHQ (documentação).
- Arquivos 1 a 7 desta pasta.
