# Casos de Uso e Erros Comuns em Governança de Nuvem

> **Objetivo pedagógico:** consolidar o aprendizado de governança de nuvem com casos corporativos fictícios, os erros mais frequentes, suas consequências e a prevenção, exercícios práticos e perguntas de revisão.

Os casos são fictícios e destinados ao treinamento.

## 1. Caso 1: expansão sem governança (muitas contas, nenhum controle)

| Item | Descrição |
|---|---|
| Situação | Uma empresa de varejo permitiu que cada área criasse contas de nuvem com cartões corporativos |
| Problema | Em 18 meses, surgiram 70 contas, sem estrutura, sem registros centralizados e com etiquetas inconsistentes |
| Incidente | Um ambiente de teste com dados reais de clientes ficou exposto por meses |
| Causas | Sem estratégia, sem CCoE, sem inventário, sem guardrails e sem proprietários |
| Resposta | Levantamento das contas, criação da organização, migração para a estrutura, guardrails de acesso público e de registros, classificação dos dados |
| Resultado | Em 6 meses, 100% das contas na organização; redução de 24% dos custos por encerramento de contas e de recursos ociosos |

**Lição:** governança é mais barata e mais segura quando vem antes da escala; recuperá-la depois exige esforço maior.

## 2. Caso 3: governança que virou gargalo

| Item | Descrição |
|---|---|
| Situação | Um banco criou um comitê que revisava manualmente todos os pedidos de conta e de serviço |
| Problema | O prazo médio de aprovação chegou a 9 semanas; as equipes passaram a contratar nuvem por fora (*shadow IT*) |
| Causas | Controles manuais para todos os casos; sem critérios automáticos; sem caminhos pavimentados |
| Resposta | Classificação por risco: pedidos conformes aprovados automaticamente; revisão humana apenas para dados restritos e serviços novos; catálogo de serviços e modelos de conta |
| Resultado | Prazo médio de 2 dias; redução do *shadow IT*; mais visibilidade |

**Lição:** o objetivo é habilitar o uso seguro com rapidez; guardrails automáticos substituem aprovações manuais em casos de baixo risco.

## 3. Caso 4: exceções eternas

| Item | Descrição |
|---|---|
| Situação | Uma empresa aprovou exceções à política de acesso público "temporariamente" para uma campanha de marketing |
| Problema | Dois anos depois, três exceções continuavam ativas, e um dos armazenamentos continha arquivos internos |
| Causas | Exceções sem prazo, sem dono do risco e sem revisão |
| Resposta | Registro central de exceções, expiração obrigatória, notificação 30 dias antes do vencimento e relatório trimestral ao comitê |
| Resultado | Redução de 80% das exceções em um ano |

**Lição:** toda exceção precisa de dono, de prazo e de revisão, ou vira política paralela.

## 4. Caso 5: alerta de custo que era um incidente de segurança

| Item | Descrição |
|---|---|
| Situação | A equipe de FinOps recebeu um alerta de anomalia de custo em uma conta de desenvolvimento e o tratou como problema de orçamento |
| Descoberta | Dois dias depois, a segurança identificou instâncias de mineração criadas com uma chave exposta |
| Causas | Os alertas de custo não chegavam à segurança; sem restrição de regiões e de tipos de instância |
| Resposta | Chave desativada, recursos removidos após preservação, restrição por política, alertas de custo também à segurança |
| Resultado | Detecção em horas, em vez de dias, no episódio seguinte |

**Lição:** custo é sinal de segurança; integre FinOps e SecOps.

## 5. Caso 6: auditoria sem evidências

| Item | Descrição |
|---|---|
| Situação | Uma auditoria pediu evidências de conformidade com a política de criptografia em nuvem |
| Problema | A organização tinha a política e os controles, mas as evidências estavam espalhadas e eram manuais |
| Causas | Sem conformidade contínua; sem mapeamento a controles; sem repositório de evidências |
| Resposta | Relatórios automáticos periódicos, mapeamento a normas e painel de conformidade |
| Resultado | A auditoria seguinte consumiu um terço do esforço e sem achados relevantes de evidência |

**Lição:** a governança precisa gerar evidências de forma contínua e automática.

## 6. Erros comuns, consequências e prevenção

| Erro | Consequência provável | Prevenção |
|---|---|---|
| Adotar a nuvem sem estratégia e sem governança | Contas dispersas, riscos e custos descontrolados | Estratégia, CCoE e landing zone antes da escala |
| Políticas apenas em documentos | Sem aplicação e sem verificação | Políticas como código e verificação contínua |
| Governança centralizada e manual para tudo | Gargalos e *shadow IT* | Guardrails automáticos e caminhos pavimentados |
| Sem proprietário para contas e recursos | Recursos órfãos e riscos sem dono | Etiquetas obrigatórias e revisões |
| Exceções sem prazo e sem dono do risco | Políticas paralelas e exposições duradouras | Registro, expiração e revisão |
| Não integrar custo e segurança | Abuso detectado tarde | Alertas de custo para a segurança |
| Aplicar guardrails direto em produção, sem teste | Interrupções de serviços | Introdução gradual: auditoria e depois negação |
| Muitos indicadores sem decisão associada | Painéis sem uso | Poucos indicadores, ligados a ações |
| Não medir a cobertura do inventário | Falsa sensação de conformidade | Medir primeiro o que existe |
| Responsabilidades pouco claras entre equipes | Controles não executados | RACI com um único responsável final |
| Ignorar a governança de dados e de privacidade | Descumprimento da LGPD | Classificação, residência, retenção e RIPD |
| Governança sem patrocínio executivo | Falta de autoridade e de recursos | Patrocinador e comitê com decisão |
| Não revisar políticas após incidentes ou mudanças | Políticas desatualizadas | Revisão anual e gatilhos de revisão |
| Tratar a governança como projeto com data de fim | Degradação com o tempo | Operação contínua, com calendário |

## 7. Exercícios práticos

1. Escreva a carta do CCoE para uma organização fictícia de 500 pessoas com dois provedores de nuvem, definindo missão, escopo, autoridade e indicadores.
2. Monte a matriz RACI para cinco atividades de governança (aprovar serviço novo, exceções, custos, conformidade e resposta a incidentes), com um único A por linha.
3. Redija uma política de uso de nuvem de uma página, com requisitos mensuráveis, e relacione cada requisito a um padrão técnico e a um guardrail.
4. Escreva as regras de um guardrail preventivo e de um detectivo para "armazenamentos sem acesso público" em um provedor à sua escolha, testando em uma conta de laboratório com o efeito de auditoria.
5. Defina o esquema de etiquetas obrigatórias e escreva a consulta que mede a cobertura (arquivo 4).
6. Simule o processo de exceções: crie três solicitações, com justificativa, controles compensatórios e prazo; avalie qual seria aprovada.
7. Calcule, com os dados sintéticos do arquivo 6, a conformidade por regra e proponha as três primeiras ações.
8. Monte um painel de uma página para o comitê, com seis indicadores, metas e tendência.
9. Faça a autoavaliação de maturidade (arquivo 6) para uma organização real ou fictícia e proponha um plano de 12 meses.

Use somente contas e dados de laboratório nos exercícios técnicos.

## 8. Perguntas de revisão

| Pergunta | Resposta esperada (resumo) |
|---|---|
| Qual a diferença entre governança, gestão e operação? | Governança decide e supervisiona; a gestão planeja e controla; a operação executa |
| Por que guardrails automáticos são essenciais em nuvem? | O autoatendimento e a velocidade exigem controles preventivos e detectivos que escalem |
| O que é um CCoE e qual seu papel? | Equipe multidisciplinar que define e apoia padrões, arquitetura e plataforma, habilitando o uso seguro |
| O que são caminhos pavimentados? | Modelos e serviços prontos e seguros que facilitam a conformidade e a velocidade |
| Por que toda exceção precisa de prazo e de dono do risco? | Para impedir exceções permanentes e garantir a aceitação consciente do risco |
| Como o FinOps se relaciona com a segurança? | Anomalias de custo podem indicar abuso; cotas e restrições reduzem impacto; custos orientam decisões de controles |
| Qual a diferença entre cobertura e conformidade? | Cobertura mede o que está sob governança; conformidade mede se o que está sob governança segue as políticas |
| O que a função Govern do NIST CSF 2.0 acrescenta? | Contexto, estratégia de riscos, papéis, políticas, supervisão e gestão de riscos da cadeia de suprimentos |

## 9. Casos de uso corporativos

- Treinamento de líderes de TI, segurança, finanças e negócio sobre governança de nuvem.
- Diagnóstico de governança antes de uma expansão ou de uma auditoria.
- Base para a construção do roteiro de segurança em nuvem (pasta 28).

## 10. Normas e frameworks relevantes

- NIST CSF 2.0 (Govern); ISO/IEC 27014:2020; ISO/IEC 38500:2015; COBIT 2019
- CSA CCM v4 (GRC); FinOps Framework; CIS Foundations Benchmarks
- LGPD, arts. 37, 38, 46 e 50

## 11. Referências

- NIST. Cybersecurity Framework 2.0. 2024.
- Storment, J. R.; Fuller, M. Cloud FinOps. 2. ed. O'Reilly, 2023.
- Cloud Security Alliance. Security Guidance v4.0 e CCM v4.
- Microsoft; Amazon Web Services; Google Cloud. Cloud Adoption Frameworks (documentação vigente).
- Arquivos 1 a 7 desta pasta.
