# Casos de Uso e Erros Comuns em Responsabilidade Compartilhada

> **Objetivo pedagógico:** consolidar o aprendizado sobre responsabilidade compartilhada com casos corporativos fictícios, os erros mais frequentes, suas consequências e a prevenção, exercícios práticos e perguntas de revisão.

Os casos são fictícios e destinados ao treinamento.

## 1. Caso 1: "o provedor faz backup" (SaaS)

| Item | Descrição |
|---|---|
| Situação | Uma empresa de serviços adotou uma plataforma de colaboração em nuvem e desativou o backup local, supondo que o provedor guardava cópias |
| Incidente | Um ataque de ransomware criptografou arquivos sincronizados; a exclusão em massa foi propagada à nuvem |
| Descoberta | A restauração do provedor cobria apenas versões recentes dentro de uma janela de retenção e não as cópias já sobrescritas |
| Causas | Confusão entre retenção e backup; matriz de responsabilidades inexistente; backup independente não contratado |
| Resposta | Restauração parcial a partir de versões; reconstrução manual; comunicação a clientes |
| Correções | Backup independente com imutabilidade, teste de restauração trimestral, políticas de retenção e proteção contra exclusão em massa |

**Lição:** o provedor garante a disponibilidade do serviço; a recuperação dos dados do cliente exige controles do cliente.

## 2. Caso 2: Kubernetes gerenciado e os nós

| Item | Descrição |
|---|---|
| Situação | Uma equipe de produtos migrou para um serviço de Kubernetes gerenciado, supondo que o provedor atualizava tudo |
| Incidente | Uma vulnerabilidade no sistema operacional dos nós foi explorada 5 semanas depois da correção |
| Descoberta | O serviço gerenciava o plano de controle, mas os nós eram autogerenciados; a atualização dependia do cliente |
| Causas | Documentação não lida; modo de nós não considerado; sem dono para a atualização dos nós |
| Resposta | Atualização emergencial dos nós, rotação de segredos e análise de acesso |
| Correções | Passagem para nós com atualização automática gerenciada, dono definido, painel de versões e verificação contínua |

**Lição:** "gerenciado" tem significados diferentes; leia o modelo de responsabilidade do serviço e do modo escolhido.

## 3. Caso 3: banco de dados gerenciado exposto

| Item | Descrição |
|---|---|
| Situação | Um banco de dados gerenciado foi criado com acesso público para um teste e permaneceu assim em produção |
| Incidente | Credenciais fracas foram exploradas, e uma base de clientes foi acessada |
| Descoberta | Alerta de comportamento anômalo no banco 9 dias depois |
| Causas | Padrão de criação permissivo; ausência de políticas que impedissem acesso público; senhas fracas; sem auditoria habilitada |
| Resposta | Fechamento do acesso, rotação de credenciais, avaliação de acessos (limitada pela falta de auditoria) e comunicação conforme a LGPD |
| Correções | Política que exige acesso privado; autenticação integrada à identidade; auditoria habilitada; revisão de arquitetura obrigatória para bancos |

**Lição:** o provedor oferece o recurso de segurança; **cabe ao cliente decidir, habilitar e verificar**.

## 4. Caso 4: provedor gerenciado (MSP) e a resposta a incidentes

| Item | Descrição |
|---|---|
| Situação | A empresa terceirizou a operação de sua nuvem a um MSP, sem matriz de responsabilidades clara |
| Incidente | Credenciais comprometidas acionaram atividades de mineração em instâncias de computação |
| Descoberta | O alerta de custos chegou ao cliente, mas o MSP não tinha monitoramento de segurança contratado; ninguém agiu por 36 horas |
| Causas | Contrato sem responsabilidades de detecção e de resposta; falta de contatos; ausência de registros centralizados no cliente |
| Resposta | Contenção pelo cliente, com apoio do provedor; avaliação de custos e de acessos |
| Correções | Contrato com matriz de responsabilidades (operação, monitoramento e resposta), acesso do cliente aos registros, playbooks conjuntos e exercícios |

**Lição:** terceirizar a operação não terceiriza a responsabilização; o contrato deve dizer quem detecta e quem responde.

## 5. Caso 5: auditoria e a certificação do provedor

| Item | Descrição |
|---|---|
| Situação | Em uma auditoria, a organização apresentou apenas a certificação do provedor como evidência de seus controles em nuvem |
| Achado | O auditor exigiu evidências dos controles do cliente: revisão de acessos, criptografia habilitada, registros retidos e testes de restauração |
| Causas | Confusão entre garantias do provedor e conformidade do cliente; ausência da matriz de responsabilidades e do repositório de evidências |
| Resposta | Plano de ação para coletar evidências, formalizar a matriz e automatizar relatórios |
| Correções | Declaração de responsabilidades por serviço, evidências automáticas e revisão anual |

**Lição:** a certificação do provedor cobre a parte do provedor; o cliente precisa evidenciar a sua.

## 6. Erros comuns, consequências e prevenção

| Erro | Consequência provável | Prevenção |
|---|---|---|
| Assumir que o provedor protege dados e configuração | Exposição por configuração do cliente | Matriz P, C e S por serviço |
| Não ler a documentação de responsabilidade do serviço específico | Controles do cliente esquecidos | Leitura e arquivamento da documentação, com data |
| Confundir retenção e backup | Perda irrecuperável de dados | Backup independente e teste de restauração |
| Confiar em padrões de segurança sem verificar | Recursos expostos ou desprotegidos | Políticas como código e verificação contínua |
| Não habilitar recursos de segurança opcionais | Sem registros nem proteção quando necessário | Registro de recursos opcionais e decisões formais |
| Usar a certificação do provedor como prova de conformidade própria | Falhas em auditorias e em incidentes | Evidências próprias e CUECs |
| Não ter dono interno para os controles do cliente | Controles não executados | RACI com um único responsável final |
| Terceirizar a operação sem definir responsabilidades de detecção e de resposta | Atrasos e prejuízos | Contrato e playbooks conjuntos |
| Ignorar mudanças dos serviços e dos termos do provedor | Lacunas novas e riscos não percebidos | Monitoramento de comunicados e revisões periódicas |
| Não planejar a saída | Dependência e perda de dados | Plano de saída e testes de exportação |
| Ausência de matriz em ambientes multicloud | Políticas inconsistentes | Padrão comum de controles com implementações por provedor |
| Registrar apenas SLAs e ignorar segurança no contrato | Sem garantias de notificação e de cooperação | Cláusulas de incidentes, auditoria e acesso a registros |

## 7. Exercícios práticos

1. Escolha três serviços de nuvem (um IaaS, um PaaS e um SaaS) e preencha a matriz P, C e S de cada um, consultando a documentação oficial.
2. Execute os comandos do arquivo 7 em uma conta de laboratório, monte o inventário de serviços e gere a lista de controles do cliente com o roteiro em Python.
3. Escreva a declaração de responsabilidades de um serviço (modelo do arquivo 6) e a matriz RACI correspondente.
4. Faça a análise de lacunas (arquivo 4) de uma plataforma de colaboração: liste cinco lacunas, o risco e a ação.
5. Leia um relatório SOC 2 Tipo II público de um provedor (ou um modelo de exemplo) e identifique escopo, período, exceções e CUECs.
6. Redija cinco cláusulas desejáveis para um contrato de SaaS com dados pessoais (para revisão do Jurídico), com base no arquivo 5.
7. Prepare uma apresentação de 10 minutos à direção sobre "o que o provedor não faz por nós", com três exemplos reais de lacunas.

Use somente contas e dados de laboratório nos exercícios técnicos.

## 8. Perguntas de revisão

| Pergunta | Resposta esperada (resumo) |
|---|---|
| O que significa segurança "da" nuvem e "na" nuvem? | "Da" nuvem é a infraestrutura e os serviços operados pelo provedor; "na" nuvem é o que o cliente coloca e configura |
| Quais responsabilidades o cliente mantém em qualquer modelo de serviço? | Dados, identidades e contas, dispositivos e configuração de uso, além da responsabilização legal |
| Por que "gerenciado" não significa "sem responsabilidade do cliente"? | O provedor gerencia parte da pilha; o cliente configura, protege dados e identidades e usa o serviço |
| O que é um controle herdado? | Um controle operado pelo provedor, que o cliente herda e evidencia com documentos do provedor |
| O que são CUECs? | Controles complementares da entidade usuária que o cliente deve implementar para que o relatório SOC 2 do provedor se aplique |
| Por que o SLA não garante a segurança do cliente? | Mede disponibilidade do serviço e não cobre configurações do cliente nem perdas de negócio |
| Quem responde perante os titulares se o operador falha? | O controlador continua responsável, e o operador também pode responder conforme a LGPD |
| Qual o princípio prático quando a responsabilidade não está clara? | Tratar como responsabilidade do cliente até o provedor confirmar por escrito |

## 9. Casos de uso corporativos

- Material de treinamento para equipes de arquitetura, de segurança, de GRC e de contratos.
- Base para revisões de arquitetura e de contratos de nuvem.
- Preparação de auditorias com foco na divisão de controles.

## 10. Normas e frameworks relevantes

- NIST SP 800-144 e 800-53 Rev. 5; ISO/IEC 27017:2015; ISO/IEC 27036-4:2016
- CSA CCM v4 (STA); SOC 2; CIS Foundations Benchmarks
- LGPD, arts. 37 a 42, 46 e 48

## 11. Referências

- Jansen, W.; Grance, T. NIST SP 800-144. 2011.
- Cloud Security Alliance. Top Threats to Cloud Computing: Pandemic Eleven. 2022.
- Dotson, C. Practical Cloud Security. O'Reilly, 2019.
- Documentação de responsabilidade compartilhada da AWS, da Microsoft e do Google Cloud (versões vigentes).
- Arquivos 1 a 7 desta pasta.
