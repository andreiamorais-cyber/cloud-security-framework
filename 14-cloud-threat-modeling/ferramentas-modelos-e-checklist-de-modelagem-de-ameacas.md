# Ferramentas, Modelos e Checklist de Modelagem de Ameaças

> **Objetivo pedagógico:** conhecer as ferramentas de modelagem de ameaças (de quadro e de código aberto, de modelos como código e comerciais), padronizar os modelos de trabalho (oficina, registro de ameaças, abuso e revisão), ver um **exemplo completo e resumido** de modelo de ameaças para uma API de extratos em nuvem, com ameaças mapeadas a controles, detecções e testes, e usar um checklist completo da prática.

## 1. Ferramentas por finalidade

| Finalidade | Ferramentas | Observações |
|---|---|---|
| Quadro e diagramas manuais | Quadros brancos, draw.io (diagrams.net), Excalidraw e Mermaid | Simples, rápidas e acessíveis; boas para sessões |
| Modelagem com diagramas e sugestão de ameaças | OWASP Threat Dragon; Microsoft Threat Modeling Tool (Windows) | Geram ameaças a partir do diagrama e regras (STRIDE); revisar sempre |
| Modelos como código | OWASP pytm (Python); Threagile (YAML) | Versionamento e integração em pipelines; regras genéricas |
| Plataformas comerciais | IriusRisk, ThreatModeler e similares | Biblioteca de componentes, integração com ferramentas de desenvolvimento e relatórios |
| Jogos e cartas | Elevation of Privilege (EoP), OWASP Cornucopia | Facilitam a participação |
| Cobertura de técnicas | MITRE ATT&CK Navigator | Mapa de calor de técnicas e de detecções |
| Registro e acompanhamento | Planilhas, repositório Git e gerenciador de tarefas | Registro de ameaças, requisitos e rastreabilidade |
| Documentação de arquitetura | Modelo C4 e registros de decisão (pasta 05) | Base da decomposição |

**Critérios de escolha:** facilidade de adoção pelos times, integração com o fluxo de trabalho (repositório, backlog e pipeline), possibilidade de versionar, custo e **capacidade de ajustar o conteúdo ao contexto de nuvem**. A ferramenta importa menos do que a **conversa** e o **acompanhamento das ações**.

## 2. Modelos de trabalho

### 2.1 Pauta de oficina de modelagem (90 minutos)

| Tempo | Atividade | Responsável |
|---|---|---|
| 0 a 10 min | Objetivo, escopo, regras da sessão e papéis | Facilitador |
| 10 a 30 min | Apresentação do sistema, DFD, ativos, atores e fronteiras de confiança; premissas | Arquiteto ou líder técnico |
| 30 a 60 min | Identificação de ameaças (STRIDE nas fronteiras, catálogo de nuvem e jogos, se útil) | Todos |
| 60 a 75 min | Priorização rápida e decisão de tratamento (eliminar, mitigar, aceitar) | Todos |
| 75 a 85 min | Requisitos, critérios de aceitação, responsáveis e prazos; validação planejada | Dono do produto e facilitador |
| 85 a 90 min | Resumo, próximos passos e data de revisão | Facilitador |

### 2.2 Material de preparação (envio prévio)

| Item | Conteúdo |
|---|---|
| Descrição do sistema ou da mudança | Objetivo, usuários, dados e integrações |
| Diagramas | Arquitetura, DFD e implantação (se existirem) |
| Classificação dos dados | Tipos, dados pessoais e sensíveis |
| Riscos e incidentes relacionados | Do registro de riscos e de incidentes anteriores |
| Dúvidas e premissas conhecidas | Lista inicial |

### 2.3 Registro de ameaças

| ID | Ameaça (ator, ativo e caminho) | Componente ou fluxo | STRIDE | ATT&CK | Probabilidade | Impacto | Controles existentes (testados?) | Risco residual | Tratamento | Requisito ou ação | Responsável | Prazo | Detecção | Teste | Situação |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| T-001 | | | | | | | | | | | | | | | |

### 2.4 Registro de premissas

| ID | Premissa | Fonte ou justificativa | Como será verificada | Responsável | Estado |
|---|---|---|---|---|---|
| P-001 | O pipeline é a única via de implantação em produção | Política da organização | Política que nega implantações manuais; relatório de alterações | Plataforma | Em verificação |

### 2.5 Ficha de caso de abuso

| Campo | Preenchimento |
|---|---|
| Caso de uso legítimo relacionado | |
| Caso de abuso (ator, objetivo e passos) | |
| Ativos afetados | |
| Requisitos de segurança derivados | |
| Critérios de aceitação (testes) | |
| Detecção associada | |

### 2.6 Revisão do modelo (lista de perguntas)

| Pergunta | Verificação |
|---|---|
| O escopo está claro, e o que ficou de fora está registrado e justificado? | |
| O DFD reflete a implantação real (incluindo plano de controle, pipelines e fornecedores)? | |
| Todas as fronteiras de confiança e os fluxos que as cruzam foram analisados? | |
| Foram considerados os atores certos e as capacidades assumidas? | |
| As premissas críticas têm verificação? | |
| As ameaças têm tratamento, responsável e prazo? | |
| Os controles considerados foram testados? | |
| Há detecção ou justificativa para as ameaças críticas? | |
| Dados pessoais foram considerados (LINDDUN e RIPD)? | |
| O modelo está versionado e com data de revisão? | |

## 3. Exemplo completo e resumido: modelo de ameaças da API de extratos

### 3.1 Escopo e premissas

| Item | Descrição |
|---|---|
| Sistema | API pública de consulta de extratos, banco gerenciado, função de geração de PDF e armazenamento de PDFs, em nuvem pública |
| Ativos principais | Dados financeiros e pessoais de clientes (restrito), credenciais do pipeline, chaves de criptografia, disponibilidade da API |
| Atores | Cliente, atacante oportunista, atacante com credencial roubada, agente interno malicioso, parceiro, fornecedor comprometido |
| Fora do escopo | Sistemas de origem e aplicativo móvel (modelo próprio) |
| Premissas | P-001: o pipeline é a única via de implantação; P-002: o provedor isola os inquilinos; P-003: administradores usam MFA resistente a phishing |

### 3.2 Ameaças principais (12 de 33)

| ID | Ameaça | STRIDE | Nível residual | Mitigações (controles da linha de base) | Detecção | Teste ou validação |
|---|---|---|---|---|---|---|
| T-001 | Roubo de token de cliente e acesso aos extratos | S | Médio | Tokens curtos e vinculados ao dispositivo; MFA para clientes sensíveis (BSL-08 e acesso condicional) | Uso de token por origens distintas | Teste de reutilização de token em outro dispositivo |
| T-002 | Leitura de extratos de outro cliente por manipulação do identificador | E, I | Alto (controle não testado) | Autorização por objeto e testes automatizados (requisito REQ-021) | Padrão de acessos cruzados | Teste automatizado de acesso cruzado; teste de intrusão |
| T-003 | Cópia da base por credencial com permissão de leitura | I | Alto | Chaves do cliente com política própria; perímetro de dados; acesso por atributos (BSL-18 e BSL-20) | Volume anômalo de leitura | Emulação de leitura em massa |
| T-004 | Armazenamento de PDFs exposto à internet | I | Baixo | Bloqueio de acesso público; módulo seguro (BSL-17) | Alteração de política pública | Teste de criação pública negada |
| T-005 | Link do PDF previsível ou de longa duração | I | Baixo | Links assinados, de uso único e de 15 minutos | Downloads em volume | Teste de expiração e de uso único |
| T-006 | Pipeline alterado para implantar código malicioso | T, E | Médio | Revisão obrigatória, assinatura de artefatos, identidade federada, aprovação para produção (BSL-09, BSL-26 e BSL-28) | Alterações em definições de pipeline | Teste de intrusão do pipeline |
| T-007 | Função de PDF com permissões amplas explorada por entrada maliciosa | E | Médio | Papel mínimo, validação de entrada, limites de recursos (BSL-27) | Invocações anômalas | Teste de permissões efetivas |
| T-008 | Ataque de negação de serviço de camada 7 e aumento de custos | D | Baixo | WAF, limitação de taxa, cotas (BSL-16) | Picos de requisições e de custos | Teste de carga controlado |
| T-009 | Exclusão de registros de auditoria para ocultar ações | R | Baixo | Conta de registros imutável; política que impede a desabilitação (BSL-03 e BSL-04) | Alteração de trilhas | Teste de tentativa em laboratório |
| T-010 | Exclusão de chave de criptografia (destruição de dados) | T, D | Baixo | Proteção contra exclusão e aprovação por duas pessoas (BSL-22) | Agendamento de exclusão de chave | Teste de política de negação |
| T-011 | Reidentificação em exportações para análises | I | Médio | Pseudonimização com HMAC e chave protegida; exportações aprovadas (pasta 09) | Acessos a bases de análise | Teste de reidentificação |
| T-012 | Premissa P-002 falha (falha de isolamento do provedor) | I | Médio (aceitar com monitoramento) | Chaves do cliente; relatórios do provedor; plano de contingência | Comunicados do provedor | Revisão anual de relatórios |

### 3.3 Resultado da sessão (resumo)

| Item | Resultado |
|---|---|
| Ameaças identificadas | 33 (12 altas ou críticas antes dos controles testados) |
| Requisitos criados | 19, com critérios de aceitação e 14 testes automatizados |
| Detecções novas ou validadas | 7 |
| Eliminações | 1 (envio de dados por e-mail) |
| Aceitações | 3 ameaças baixas e a premissa P-002, com monitoramento |
| Próxima revisão | Após a próxima mudança relevante e, no máximo, em 12 meses |

## 4. Checklist de modelagem de ameaças

**Sistema:** ____________________
**Facilitador:** ____________________
**Data:** ____________________

### 4.1 Preparação e decomposição

- [ ] Escopo, objetivos, premissas e participantes definidos
- [ ] DFD (nível 1 e implantação) com fluxos rotulados e fronteiras de confiança, validado com quem constrói e opera
- [ ] Ativos, atores (com capacidades) e pontos de entrada listados
- [ ] Dados classificados; dados pessoais identificados
- [ ] Plano de controle, pipelines, identidade e fornecedores incluídos

### 4.2 Identificação e avaliação

- [ ] Ameaças identificadas com STRIDE por elemento e foco nas fronteiras de confiança
- [ ] Catálogo de ameaças de nuvem, LINDDUN (se houver dados pessoais) e ATT&CK usados para enriquecer
- [ ] Riscos avaliados com os critérios da organização; controles considerados apenas se testados
- [ ] Premissas críticas com verificação

### 4.3 Tratamento e validação

- [ ] Tratamento definido por ameaça (eliminar, mitigar, transferir ou aceitar), com justificativa
- [ ] Requisitos com critérios de aceitação no backlog, responsáveis e prazos
- [ ] Rastreabilidade com controles, detecções e testes
- [ ] Ameaças altas e críticas integradas ao registro de riscos; aceitações com alçada e prazo
- [ ] Validação planejada: testes, emulação, equipe roxa e revisão pelo menos anual

### 4.4 Manutenção

- [ ] Modelo versionado junto ao código; revisão em mudanças relevantes
- [ ] Indicadores de cobertura, de atualidade e de lacunas acompanhados

## 5. Resultado da verificação

| Resultado | Condição |
|---|---|
| Conforme | Todos os itens aplicáveis marcados |
| Conforme com ressalvas | Pendências sem ameaças críticas sem tratamento, com plano e prazos |
| Não conforme | Sistema crítico sem modelo atual, ameaças críticas sem tratamento ou sem responsável, ou mitigações críticas não verificadas; corrigir com prioridade |

**Assinatura do facilitador:** ____________________
**Assinatura do dono do sistema:** ____________________

## 6. Casos de uso corporativos

- Biblioteca de modelos de referência com registros de ameaças pré-preenchidos.
- Padronização de oficinas, registros e revisões entre os times.
- Evidências da prática de modelagem de ameaças para auditorias e para clientes.

## 7. Normas e frameworks relevantes

- OWASP Threat Modeling Cheat Sheet e Threat Dragon; Microsoft SDL; Threagile e pytm
- NIST SP 800-218 (SSDF); OWASP SAMM; BSIMM; ISO/IEC 27002:2022 (8.25 a 8.27)
- MITRE ATT&CK Navigator; OWASP Cornucopia; Elevation of Privilege

## 8. Referências

- OWASP. Threat Dragon, pytm e Cornucopia (documentação).
- Schneider, C. Threagile (documentação).
- Shostack, A. Threat Modeling: Designing for Security. Wiley, 2014.
- MITRE. ATT&CK Navigator (documentação).
- Pastas 05, 12 e 13 deste repositório.
