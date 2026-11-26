# Casos de Uso e Erros Comuns em Modelagem de Ameaças

> **Objetivo pedagógico:** consolidar o aprendizado de modelagem de ameaças em nuvem com casos corporativos fictícios, os erros mais frequentes, suas consequências e a prevenção, exercícios práticos e perguntas de revisão.

Os casos são fictícios e destinados ao treinamento.

## 1. Caso 1: a modelagem que eliminou um risco antes de existir código

| Item | Descrição |
|---|---|
| Situação | Uma fintech planejava enviar extratos completos por e-mail, em PDF, aos clientes |
| Sessão | Em 90 minutos, o time identificou ameaças de interceptação, de reencaminhamento, de e-mail alterado por um atacante e de retenção dos PDFs em caixas de entrada |
| Decisão | Eliminar o envio do conteúdo: o e-mail passou a conter apenas um link assinado, de uso único e de 15 minutos, com autenticação adicional |
| Custo | Meio dia de trabalho adicional no desenho |
| Resultado | Risco de vazamento por e-mail eliminado, sem correções posteriores; um teste de intrusão posterior não encontrou achados nessa funcionalidade |

**Lição:** a modelagem no desenho permite **eliminar** riscos, a opção mais barata e mais eficaz.

## 2. Caso 2: o pipeline fora do modelo

| Item | Descrição |
|---|---|
| Situação | O modelo de ameaças de uma plataforma de pagamentos cobria bem a aplicação e o banco, mas não incluía o pipeline de CI/CD nem a identidade de implantação |
| Incidente | Um atacante comprometeu a conta de um desenvolvedor, alterou um fluxo de trabalho do pipeline e obteve as credenciais de implantação de produção |
| Causas | Pipeline fora do DFD; permissões amplas na identidade de implantação; ausência de revisão obrigatória de alterações de pipeline |
| Resposta | Contenção, rotação de credenciais, revisão das implantações e do código |
| Correções | Pipeline e plano de controle incluídos nos modelos; identidades federadas por ambiente e por ramo; revisão obrigatória; assinatura de artefatos; verificação em pipeline de que o modelo é atualizado quando a infraestrutura muda |

**Lição:** o que não está no diagrama não é analisado; inclua **plano de controle, pipelines, identidade e fornecedores**.

## 3. Caso 3: premissa nunca verificada

| Item | Descrição |
|---|---|
| Situação | O modelo assumia que "somente o pipeline implanta em produção" |
| Problema | Em um exercício de equipe roxa, uma equipe conseguiu alterar recursos de produção pelo console, usando um papel de operações com permissões amplas |
| Causas | Premissa registrada, mas sem verificação; permissões de console mantidas por conveniência |
| Resposta | Política que nega alterações fora do pipeline, acesso de emergência com aprovação e alertas de alterações manuais |
| Resultado | A premissa passou a ser um controle verificado por política e por alerta |

**Lição:** premissas críticas devem ser **transformadas em controles ou em testes**.

## 4. Caso 4: modelo detalhado, mas abandonado

| Item | Descrição |
|---|---|
| Situação | Uma empresa produziu, com consultoria, modelos de ameaças extensos para 12 sistemas, em documentos de 80 páginas |
| Problema | Um ano depois, as arquiteturas haviam mudado, e os modelos não foram atualizados; os times não os consultavam; ameaças identificadas não geraram ações no backlog |
| Causas | Esforço concentrado e externo; formato pesado e fora do repositório; ausência de dono, de gatilhos e de verificação |
| Resposta | Reinício com modelos leves, no repositório, mantidos pelos times, com gatilhos e verificação em pipeline |
| Resultado | Em seis meses, 80% dos sistemas críticos com modelos atuais; ameaças convertidas em requisitos com critérios de aceitação |

**Lição:** um modelo útil é **mantido pelo time**, próximo ao código e ligado às ações; um documento grande e estático perde valor rapidamente.

## 5. Caso 5: lista automática de ameaças sem conversa

| Item | Descrição |
|---|---|
| Situação | Uma equipe adotou uma ferramenta que gera ameaças automaticamente a partir do diagrama e entregou o relatório de 300 itens ao time de desenvolvimento |
| Problema | O time ignorou o relatório (excesso de itens genéricos), e as ameaças específicas do negócio (abuso de uma funcionalidade de estorno) não foram identificadas; um fraudador explorou a funcionalidade |
| Causas | Substituição da conversa pela ferramenta; sem priorização; sem conhecimento do domínio na análise |
| Resposta | Sessão com negócio, desenvolvimento e segurança; casos de abuso e requisitos de limites e de aprovação para estornos |
| Resultado | Redução das perdas por fraude; relatório automático passou a ser apenas insumo |

**Lição:** ferramentas ajudam, mas as ameaças mais relevantes vêm do **conhecimento do sistema e do negócio**; priorize e envolva o time.

## 6. Erros comuns, consequências e prevenção

| Erro | Consequência provável | Prevenção |
|---|---|---|
| Modelar tarde, depois da implantação | Correções caras ou impossíveis | Modelar no desenho e em mudanças relevantes |
| Modelagem como documento, e não como conversa | Ameaças superficiais; falta de apropriação | Sessões colaborativas com quem constrói e opera |
| Escopo vago ou diagrama que não reflete a realidade | Ameaças omitidas | Usar infraestrutura como código e validar com o time |
| Ignorar o plano de controle, o pipeline e a identidade | Ataques às partes mais críticas | Incluí-los no DFD |
| Não identificar fronteiras de confiança | Ameaças nos fluxos críticos não vistas | Marcar as fronteiras e analisar os fluxos que as cruzam |
| Premissas não registradas ou não verificadas | Falhas silenciosas | Registrar e testar as premissas críticas |
| Lista longa de ameaças sem priorização | Ruído e abandono | Avaliar riscos, priorizar e agrupar por causa |
| Considerar controles não testados como eficazes | Risco residual subestimado | Verificar os controles (pasta 12) |
| Pontuações subjetivas (como o DREAD) | Resultados inconsistentes | Critérios próprios de risco com definições claras |
| Substituir a conversa por ferramentas automáticas | Ameaças genéricas e perda de contexto | Ferramenta como apoio, com revisão humana |
| Sem ações, responsáveis e prazos | Modelo sem efeito | Requisitos e critérios de aceitação no backlog |
| Modelo desatualizado | Falsa sensação de segurança | Gatilhos, modelo junto ao código e verificação no pipeline |
| Não validar o modelo | Lacunas desconhecidas | Testes, emulação, equipe roxa e revisão pós-incidente |
| Esquecer a privacidade | Riscos de conformidade não tratados | LINDDUN e RIPD |
| Centralizar tudo na equipe de segurança | Gargalo e baixa adoção | Campeões e modelos de referência |
| Modelar apenas ameaças técnicas | Fraudes e abusos de funcionalidade ignorados | Casos de abuso e participação do negócio |

## 7. Exercícios práticos

1. Escolha um sistema que você conhece (ou um fictício, como uma API de pedidos) e desenhe o DFD de nível 1 com as fronteiras de confiança, usando Mermaid; liste ativos, atores e pontos de entrada.
2. Aplique o STRIDE por elemento ao seu DFD e registre pelo menos 15 ameaças no formato do registro de ameaças (arquivo 7).
3. Selecione 10 ameaças do catálogo (arquivo 4) aplicáveis ao seu sistema, reescreva-as no contexto e identifique os controles existentes (e se estão testados).
4. Construa uma árvore de ataque para o objetivo "exfiltrar a base de clientes" com pelo menos 6 folhas; indique em quais ramos há detecção.
5. Para um sistema com dados pessoais, aplique o LINDDUN e liste cinco ameaças de privacidade com mitigações.
6. Escreva três casos de abuso e os requisitos com critérios de aceitação verificáveis (arquivo 5).
7. Com os dados sintéticos do arquivo 5, execute a consulta em DuckDB e descreva as ações para as ameaças com "controle não testado".
8. Use o OWASP pytm (ou o Threagile) para descrever o seu sistema como código e gere o relatório; compare com o modelo feito à mão e identifique o que a ferramenta não captou.
9. Crie um fluxo de trabalho de verificação em pipeline (arquivo 6) para um repositório fictício e defina a regra de justificativa quando não houver impacto no modelo.
10. Planeje a validação do modelo: três testes de controle, uma emulação de técnica em laboratório e um exercício de mesa derivado de uma ameaça crítica.

Use somente sistemas fictícios, contas de laboratório e dados sintéticos nos exercícios técnicos.

## 8. Perguntas de revisão

| Pergunta | Resposta esperada (resumo) |
|---|---|
| Quais são as quatro perguntas da modelagem de ameaças? | O que estamos construindo? O que pode dar errado? O que vamos fazer a respeito? Fizemos um trabalho suficientemente bom? |
| O que significa STRIDE? | Falsificação de identidade, adulteração, repúdio, divulgação de informação, negação de serviço e elevação de privilégio |
| Por que as fronteiras de confiança são tão importantes? | É nelas que o nível de confiança muda, e os fluxos que as cruzam concentram a maioria das ameaças |
| Para que serve o LINDDUN? | Identificar ameaças à privacidade (vinculabilidade, identificabilidade, detectabilidade, divulgação, desconhecimento e não conformidade) |
| O que é uma árvore de ataque? | Decomposição do objetivo do atacante em subobjetivos e passos, com operadores E e OU, para analisar e priorizar caminhos |
| Por que não usar o DREAD? | É subjetivo e inconsistente; prefira critérios de risco com definições claras |
| O que é o problema do agente confuso? | Um serviço com confiança entre contas é induzido a agir, com suas permissões, em nome de outro cliente; mitiga-se com condições como o identificador externo |
| Por que o pipeline deve estar no modelo? | Quem controla o pipeline controla a produção; é um alvo de alto valor |
| O que fazer com as premissas? | Registrá-las e transformar as críticas em controles ou testes |
| Quando considerar um controle na avaliação do risco residual? | Somente se estiver verificado por teste ou por evidência |
| Como validar um modelo? | Testes de controles, emulação de detecções, testes de intrusão, equipe roxa, revisão pós-incidente e revisão por pares |

## 9. Casos de uso corporativos

- Treinamento de times e de campeões de segurança em modelagem de ameaças com exercícios práticos.
- Diagnóstico da prática de modelagem (cobertura, qualidade e ligação com ações) e plano de evolução.
- Base para as pastas de vulnerabilidades, avaliações e resposta a incidentes (15, 16 e 22).

## 10. Normas e frameworks relevantes

- Threat Modeling Manifesto; OWASP Threat Modeling Cheat Sheet; NIST SP 800-218 (SSDF)
- MITRE ATT&CK for Cloud; OWASP Top 10 CI/CD Security Risks; OWASP SAMM
- ISO/IEC 27005:2022; ISO/IEC 27002:2022 (8.25 a 8.27)

## 11. Referências

- Shostack, A. Threat Modeling: Designing for Security. Wiley, 2014.
- Threat Modeling Manifesto. 2020.
- Tarandach, I.; Coles, M. Threat Modeling: A Practical Guide for Development Teams. O'Reilly, 2020.
- MITRE. ATT&CK for Cloud (documentação).
- Arquivos 1 a 7 desta pasta.
