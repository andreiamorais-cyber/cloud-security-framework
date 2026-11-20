# Casos de Uso e Erros Comuns em Conformidade em Nuvem

> **Objetivo pedagógico:** consolidar o aprendizado de conformidade em nuvem com casos corporativos fictícios, os erros mais frequentes, suas consequências e a prevenção, exercícios práticos e perguntas de revisão.

Os casos são fictícios e destinados ao treinamento. Não constituem aconselhamento jurídico.

## 1. Caso 1: "o provedor é certificado, logo estamos conformes"

| Item | Descrição |
|---|---|
| Situação | Uma empresa de saúde migrou para um provedor com certificações ISO/IEC 27001 e SOC 2 e declarou internamente a conformidade com a LGPD e com os requisitos de clientes |
| Problema | Em uma auditoria de cliente, constatou-se: buckets com acesso público, ausência de revisões de acesso, registros de auditoria desabilitados e ausência de controles para os CUEC do relatório SOC 2 |
| Causas | Confusão entre a conformidade do provedor e a do cliente; ausência de matriz de responsabilidade e de análise dos CUEC |
| Resposta | Plano de correção, matriz de conformidade com responsabilidades e CUEC mapeados, e verificação contínua de configuração |
| Resultado | Achados corrigidos em 90 dias; a auditoria seguinte, sem achados críticos |

**Lição:** a certificação do provedor cobre a infraestrutura dele; o uso, a configuração, os dados e os processos do cliente precisam de controles e de evidências próprios.

## 2. Caso 2: conformidade de ponto no tempo e desvio de configuração

| Item | Descrição |
|---|---|
| Situação | Uma fintech passou em uma avaliação anual com ótimos resultados, com evidências coletadas manualmente nas semanas anteriores |
| Problema | Dois meses depois, uma incidente mostrou que vários controles haviam se desviado: criptografia desativada em um novo banco, regras de rede abertas e acessos privilegiados sem revisão |
| Causas | Controles verificados apenas pré-auditoria; sem verificação contínua; criação de recursos fora de módulos aprovados |
| Resposta | Implantação de políticas como código, painel de conformidade e coleta contínua de evidências |
| Resultado | Desvios críticos detectados em minutos; tempo médio de correção de 11 para 2 dias |

**Lição:** o ambiente de nuvem muda continuamente; a conformidade precisa ser contínua, com detecção e correção automáticas.

## 3. Caso 3: contrato de SaaS sem cláusulas de tratamento de dados

| Item | Descrição |
|---|---|
| Situação | Uma área de marketing contratou, com cartão corporativo, um SaaS de automação que recebeu a base de contatos e histórico de compras dos clientes |
| Problema | Sem aditivo de tratamento de dados, sem avaliação de privacidade, sem lista de suboperadores e com transferência para o exterior sem instrumento; o contrato padrão permitia o uso de dados para melhoria de produtos |
| Causas | *Shadow IT*; processo de contratação sem triagem de privacidade e de segurança |
| Resposta | Suspensão do envio de novos dados, formalização de aditivo e de cláusulas-padrão, desabilitação do uso de dados pelo fornecedor, RIPD e avaliação de eventual comunicação |
| Correções | Triagem obrigatória de privacidade e de segurança para novos fornecedores; bloqueio de pagamentos sem aprovação; catálogo de fornecedores aprovados |

**Lição:** a contratação de nuvem e de SaaS exige avaliação prévia de privacidade e de segurança; contratos padronizados precisam ser revisados.

## 4. Caso 4: comunicação de incidente fora do prazo

| Item | Descrição |
|---|---|
| Situação | Um operador de pagamentos soube, por um alerta do provedor de nuvem, de uma exposição de armazenamento com dados de clientes |
| Problema | A equipe técnica tratou a contenção, mas a decisão sobre a comunicação à ANPD e aos titulares só foi tomada semanas depois, por falta de critérios e de responsável claro |
| Causas | Procedimento de comunicação inexistente; encarregado não acionado; ausência de registros de acesso a dados para estimar o alcance |
| Resposta | Avaliação tardia, comunicação à ANPD com explicação da demora, medidas de mitigação e revisão do processo |
| Correções | Procedimento com critérios de risco ou dano relevante, responsáveis, modelos e prazos; acionamento automático do encarregado em incidentes com dados pessoais; registros de acesso a dados habilitados; simulações semestrais |

**Lição:** o prazo de comunicação corre desde o conhecimento do incidente; preparar critérios, papéis e evidências antes é o que permite cumpri-lo.

## 5. Caso 5: evidências inconsistentes em uma auditoria

| Item | Descrição |
|---|---|
| Situação | Em uma auditoria de certificação, as evidências apresentadas eram capturas de tela, sem data, origem ou integridade, e algumas divergiam das configurações reais |
| Problema | O auditor considerou as evidências insuficientes e não confiáveis, e ampliou os testes, gerando não conformidades |
| Causas | Coleta manual e tardia; sem repositório; sem identificação dos responsáveis; configurações alteradas depois da coleta |
| Resposta | Retestes com consultas por API, com manifestos e hashes, e identidade de auditoria de somente leitura |
| Correções | Coleta automática e contínua; repositório imutável; mapeamento evidência–controle; treinamento dos donos |

**Lição:** a qualidade das evidências (suficiência, confiabilidade, atualidade e rastreabilidade) é tão importante quanto a existência dos controles.

## 6. Erros comuns, consequências e prevenção

| Erro | Consequência provável | Prevenção |
|---|---|---|
| Assumir que a certificação do provedor torna o cliente conforme | Lacunas e achados em auditorias; sanções | Matriz de responsabilidade; análise de escopo e de CUEC |
| Não manter um registro de obrigações | Requisitos esquecidos ou desatualizados | Registro com donos e revisão semestral |
| Conformidade de ponto no tempo | Desvios entre auditorias | Verificação e evidências contínuas |
| Evidências manuais, sem data, sem origem e sem integridade | Evidências rejeitadas | Coleta automática, metadados, hashes e repositório imutável |
| Usar relatórios SOC 2 Tipo I, antigos ou sem escopo adequado | Garantia ilusória | Tipo II, atuais, com carta de transição e escopo verificado |
| Ignorar os controles complementares do cliente (CUEC) | Controles do provedor ineficazes | Mapear CUEC para controles do cliente |
| Contratar SaaS e nuvem sem triagem de privacidade e de segurança | Transferência sem instrumento; operadores sem contrato | Processo de aquisição com triagem obrigatória |
| Aceitar contratos padrão sem revisão | Lacunas de notificação, de suboperadores e de uso de dados | Lista de cláusulas e negociação ou medidas compensatórias |
| Não definir procedimento de comunicação de incidentes | Descumprimento de prazos e sanções | Critérios, responsáveis, modelos e simulações |
| Não habilitar registros de auditoria e de acesso a dados | Impossibilidade de estimar o alcance de incidentes | Habilitar e reter os registros (pasta 13) |
| Matriz de conformidade duplicada por norma | Esforço excessivo e inconsistências | Conjunto único de controles mapeado a muitos requisitos |
| Dependência de uma única pessoa para coletar evidências | Risco operacional e atrasos | Automação e responsáveis por controle |
| Tratar a conformidade como tarefa apenas da segurança ou do Jurídico | Visão parcial | Governança conjunta (segurança, privacidade, Jurídico, produto e conformidade) |
| Não acompanhar mudanças normativas | Descumprimento por desatualização | Monitoramento regulatório com avaliação de impacto |
| Exceções permanentes e sem controles compensatórios | Lacunas crônicas | Exceções com prazo, aprovação e controles compensatórios |
| Cumprir o piso e ignorar o risco real | Incidentes mesmo em conformidade | Gestão de riscos acima do mínimo regulatório |

## 7. Exercícios práticos

1. Monte um registro de obrigações para uma organização fictícia (por exemplo, uma fintech) com 10 obrigações, indicando fonte, requisito acionável, controles e evidências.
2. Escolha cinco controles técnicos e escreva a matriz de conformidade (responsabilidade, requisitos mapeados, evidência e método de teste), usando os mapeamentos do CCM como base.
3. Leia um relatório SOC 2 Tipo II fictício (ou um modelo) e preencha o roteiro do arquivo 3: período, opinião, escopo, exceções, CUEC e subcontratados; liste três ações.
4. Execute, com os dados sintéticos do arquivo 4, a análise de lacunas em DuckDB e proponha o plano de ação.
5. Em uma conta de laboratório, execute as consultas de conformidade do arquivo 5 (padrões do Security Hub, resumo do Azure Policy ou políticas da organização) e use o roteiro de empacotamento de evidências para gerar o manifesto.
6. Avalie um contrato padrão de provedor com a lista de cláusulas do arquivo 6 e redija três propostas de negociação ou de medidas compensatórias.
7. Simule um incidente com dados pessoais em nuvem (por exemplo, um bucket exposto): preencha a linha do tempo, decida sobre a comunicação com os critérios do arquivo 6 e redija o rascunho da comunicação à ANPD.
8. Elabore o questionário de diligência de fornecedor (arquivo 7) para um SaaS crítico e o plano de monitoramento contínuo.
9. Monte a lista de solicitação de evidências (PBC) para uma auditoria de nuvem de uma plataforma e identifique as evidências que podem ser automatizadas.
10. Defina cinco indicadores de conformidade contínua e as metas, e desenhe o painel para a direção.

Use somente dados sintéticos e contas de laboratório nos exercícios técnicos.

## 8. Perguntas de revisão

| Pergunta | Resposta esperada (resumo) |
|---|---|
| A certificação ISO/IEC 27001 do provedor torna o cliente conforme? | Não: cobre a infraestrutura e os serviços do provedor, dentro do escopo; o cliente precisa de controles e de evidências próprios para seu uso |
| Qual a diferença entre SOC 2 Tipo I e Tipo II? | O Tipo I avalia o desenho dos controles em uma data; o Tipo II avalia desenho e eficácia operacional em um período |
| O que são os CUEC? | Controles complementares que o cliente deve operar para que os controles do provedor sejam eficazes |
| O que significa "cumprir uma vez, atender a muitos"? | Implementar um controle bem definido e mapeá-lo a vários requisitos de leis, normas e contratos |
| Por que a conformidade contínua é preferível à de ponto no tempo? | O ambiente de nuvem muda continuamente; a verificação e as evidências contínuas detectam desvios e reduzem o esforço |
| O que torna uma evidência confiável? | Origem confiável, metadados, integridade (hash), atualidade, rastreabilidade e armazenamento protegido |
| Quem é o controlador e quem é o operador em um serviço de nuvem? | O cliente decide finalidades e meios (controlador); o provedor trata os dados sob instruções (operador), salvo usos próprios |
| Qual o prazo de comunicação de incidentes pela Resolução CD/ANPD nº 15/2024? | Três dias úteis a partir do conhecimento de que o incidente afetou dados pessoais (confirmar o texto vigente) |
| Por que os contratos padrão de provedores exigem revisão? | Podem ter lacunas de notificação, de suboperadores, de uso de dados pelo provedor e de auditoria |
| O que é o RIPD e quando considerá-lo? | Relatório de impacto à proteção de dados pessoais, para tratamentos de maior risco, como dados sensíveis, larga escala e transferências internacionais |

## 9. Casos de uso corporativos

- Treinamento de equipes de segurança, de privacidade, de conformidade, de compras e de produto sobre conformidade em nuvem.
- Diagnóstico do programa de conformidade e plano de evolução para a conformidade contínua.
- Base para o roteiro de segurança em nuvem (pasta 28) e para as métricas (pasta 27).

## 10. Normas e frameworks relevantes

- LGPD e regulamentos da ANPD; Marco Civil da Internet; normas setoriais (CMN, Banco Central, Susep e outras)
- ISO/IEC 27001:2022, 27017:2015, 27018:2019 e 27701:2019; ISO 37301:2021; AICPA SOC 2; PCI DSS v4.0.1
- NIST SP 800-53 Rev. 5 e 800-53A; NIST SP 800-137; CSA CCM v4 e STAR

## 11. Referências

- Brasil. Lei nº 13.709/2018; Resoluções CD/ANPD nº 15/2024, 18/2024 e 19/2024.
- ISO 37301:2021; ISO/IEC 27001:2022.
- Cloud Security Alliance. CCM v4, CAIQ e STAR.
- ISACA. IT Audit Framework (ITAF), 4. ed. 2020.
- Arquivos 1 a 7 desta pasta.
