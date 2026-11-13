# Casos de Uso e Erros Comuns em Fundamentos

> **Objetivo pedagógico:** consolidar o aprendizado dos fundamentos de segurança em nuvem com casos corporativos fictícios, os erros mais frequentes na adoção, suas consequências e a prevenção, exercícios práticos e perguntas de revisão.

Os casos são fictícios e destinados ao treinamento.

## 1. Caso 1: migração *lift and shift* com armazenamento público

| Item | Descrição |
|---|---|
| Situação | Uma indústria migrou 120 servidores em 6 semanas por *rehost*, sem landing zone, usando uma única conta |
| Problema | Um bucket de backups de um sistema de RH foi criado com acesso público para "facilitar o envio por um fornecedor" |
| Descoberta | Alerta de um pesquisador externo 19 dias depois |
| Causas | Sem guardrails de acesso público; sem classificação de dados; sem revisão de mudanças; sem alerta de exposição |
| Resposta | Fechamento do acesso, preservação de registros, avaliação de acessos (que, por falta de registros de dados, exigiu o pior caso) e comunicação conforme a LGPD |
| Correções | Landing zone com contas separadas; bloqueio de acesso público na organização; classificação e etiquetas obrigatórias; verificação contínua de postura |

**Lição:** o *lift and shift* sem base de governança leva as más práticas para uma superfície de ataque maior.

## 2. Caso 2: *shadow IT* de SaaS com dados de clientes

| Item | Descrição |
|---|---|
| Situação | A área comercial contratou um SaaS de análise de dados e enviou uma base de clientes com dados pessoais |
| Problema | Sem avaliação de segurança, sem contrato de proteção de dados e sem integração com a identidade corporativa |
| Descoberta | A análise do faturamento do cartão corporativo e do tráfego de rede revelou o serviço |
| Causas | Processo de aprovação lento; ausência de catálogo de serviços aprovados; falta de visibilidade |
| Resposta | Avaliação do fornecedor (CAIQ e certificações), contrato e regularização, integração com SSO e MFA, e migração para uma alternativa aprovada |
| Correções | Processo de aprovação ágil, catálogo de serviços, descoberta de uso de SaaS e treinamento das áreas |

**Lição:** processos de aprovação lentos geram *shadow IT*; a solução é facilitar o caminho seguro.

## 3. Caso 3: conta raiz sem MFA e sem alertas

| Item | Descrição |
|---|---|
| Situação | Uma startup operava a nuvem com a conta raiz usada por três pessoas, com a senha compartilhada em um documento |
| Problema | Credenciais vazadas em um incidente de e-mail; criação de recursos de computação por terceiros e custos elevados |
| Descoberta | Alerta de cobrança do provedor |
| Causas | Sem MFA na conta raiz; credenciais compartilhadas; sem alertas de custo e de uso da conta raiz |
| Resposta | Recuperação do controle da conta com suporte do provedor, rotação de credenciais, remoção de recursos, preservação de registros |
| Correções | MFA com chaves físicas, contas individuais por SSO, cofre para a conta raiz, alertas de uso e de custos, landing zone mínima |

**Lição:** a conta raiz é o ativo mais crítico da nuvem: proteja-a antes de qualquer outra coisa.

## 4. Erros comuns, consequências e prevenção

| Erro | Consequência provável | Prevenção |
|---|---|---|
| Assumir que o provedor cuida de toda a segurança | Dados expostos por configuração do cliente | Entender o modelo de responsabilidade por serviço (pasta 02) |
| Migrar sem landing zone e sem guardrails | Configurações inseguras replicadas em escala | Construir a base antes de migrar |
| Não classificar dados antes de decidir onde hospedá-los | Dados sensíveis em locais ou configurações inadequados | Classificação, etiquetas e matriz de decisão |
| Tratar a identidade como detalhe | Contas comprometidas e abuso do plano de controle | MFA resistente a phishing, privilégio mínimo e monitoramento |
| Contas e credenciais compartilhadas | Sem rastreabilidade e com risco elevado | Identidades individuais e acesso temporário |
| Registros de auditoria desabilitados, locais ou sem retenção | Impossibilidade de investigar | Trilhas centralizadas, protegidas e retidas (pasta 13) |
| Ignorar a residência e a transferência internacional | Descumprimento legal | Regiões aprovadas, contratos e avaliação do DPO |
| Confiar apenas na certificação do provedor | Lacunas na parte do cliente | Verificar o escopo e implementar os controles do cliente |
| Falta de proprietário nos recursos | Recursos órfãos e sem responsável | Etiquetas obrigatórias e revisões periódicas |
| Ausência de alertas de custo | Abuso e surpresas financeiras | Orçamentos e alertas de anomalia |
| Configurar manualmente sem infraestrutura como código | Divergência e falta de rastreabilidade | IaC e revisão de mudanças |
| Ignorar a cadeia de fornecedores e as integrações SaaS | Acessos de terceiros não controlados | Gestão de terceiros e revisão de aplicativos autorizados |
| Pensar na resposta a incidentes só depois do incidente | Respostas improvisadas | Preparação, playbooks e preservação de registros (pastas 22 e 23) |

## 5. Exercícios práticos

1. Classifique cinco conjuntos de dados da sua organização (ou fictícios) com a matriz do arquivo 6 e registre as decisões de implantação.
2. Escolha uma carga de trabalho e identifique, para cada camada do arquivo 2, um controle preventivo, um detectivo e um corretivo.
3. Monte o mapeamento do tema "gestão de identidade e de acesso" entre o CCM (domínio IAM), a ISO/IEC 27002 e os CIS Controls (arquivo 4).
4. Desenhe a estrutura de contas, assinaturas ou projetos de uma landing zone para uma empresa com produção, desenvolvimento e uma equipe de segurança, e liste cinco guardrails.
5. Em uma conta de laboratório, execute as verificações do arquivo 7 e preencha o registro de desvios.
6. Escreva um registro de decisão de segurança (ADR) para a escolha entre PaaS e IaaS para uma aplicação nova.
7. Elabore o roteiro de uma apresentação de 10 minutos à direção sobre "por que a nuvem muda a segurança", com três mensagens principais.

Use apenas contas e dados de laboratório nos exercícios técnicos.

## 6. Perguntas de revisão

| Pergunta | Resposta esperada (resumo) |
|---|---|
| Quais são as cinco características essenciais da nuvem segundo o NIST? | Autoatendimento sob demanda, amplo acesso pela rede, agrupamento de recursos, elasticidade rápida e serviço mensurado |
| Qual a diferença entre plano de controle e plano de dados? | O plano de controle cria e gerencia recursos; o plano de dados processa e armazena os dados dos clientes |
| Por que a identidade é considerada o novo perímetro? | Os acessos vêm de qualquer lugar e controlam as APIs que gerenciam tudo |
| Cite três princípios de projeto seguro aplicáveis à nuvem | Por exemplo, privilégio mínimo, padrões seguros e defesa em profundidade |
| Qual a função de uma landing zone? | Fornecer a base governada (contas, identidade, rede, registros e guardrails) antes da implantação das cargas |
| Por que a classificação de dados precede a decisão de implantação? | Os requisitos de proteção, de residência e de resiliência dependem da sensibilidade e da criticidade |
| O que a certificação ISO/IEC 27001 de um provedor garante ao cliente? | Garante a gestão de segurança do provedor, no escopo certificado; não elimina as responsabilidades do cliente |
| Cite duas normas brasileiras relevantes para nuvem no setor financeiro | Resolução CMN nº 4.893/2021 e Resolução BCB nº 85/2021 |

## 7. Casos de uso corporativos

- Material de treinamento para equipes de arquitetura, de segurança, de TI e de negócio.
- Base de discussão em comitês de adoção de nuvem.
- Revisão de maturidade de fundamentos antes de avançar para as pastas de arquitetura e de controles.

## 8. Normas e frameworks relevantes

- NIST SP 800-144 e 800-145; ISO/IEC 27017:2015; CSA CCM v4 e Top Threats
- CIS Foundations Benchmarks; NIST CSF 2.0; LGPD; Resolução CMN 4.893/2021

## 9. Referências

- Cloud Security Alliance. Top Threats to Cloud Computing: Pandemic Eleven. 2022.
- Jansen, W.; Grance, T. NIST SP 800-144. 2011.
- Dotson, C. Practical Cloud Security. O'Reilly, 2019.
- Binnie, C.; McCune, R. Cloud Native Security. Wiley, 2021.
- Arquivos 1 a 7 desta pasta.
