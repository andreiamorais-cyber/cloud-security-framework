# Casos de Uso e Erros Comuns em Arquitetura de Segurança

> **Objetivo pedagógico:** consolidar o aprendizado de arquitetura de segurança em nuvem com casos corporativos fictícios, os erros mais frequentes, suas consequências e a prevenção, exercícios práticos e perguntas de revisão.

Os casos são fictícios e destinados ao treinamento.

## 1. Caso 1: conta única e raio de impacto total

| Item | Descrição |
|---|---|
| Situação | Uma empresa de logística mantinha produção, desenvolvimento e backups em uma única conta, com uma rede plana |
| Incidente | A credencial de um desenvolvedor foi comprometida; o atacante acessou os servidores de produção e apagou os backups |
| Causas | Sem separação de ambientes; sem limites de permissão; backups na mesma conta e acessíveis com as mesmas credenciais |
| Resposta | Reconstrução a partir de uma cópia antiga em fita; perda de 9 dias de dados; comunicação a clientes |
| Correções | Estrutura multiconta (produção, não produção, registros, segurança e recuperação isolada), acesso temporário, políticas organizacionais e backups imutáveis |

**Lição:** o isolamento por conta e a imutabilidade das cópias reduzem o raio de impacto de uma única credencial.

## 2. Caso 2: chaves permanentes no pipeline

| Item | Descrição |
|---|---|
| Situação | O sistema de integração contínua de uma fintech usava chaves de acesso de longa duração, com permissões administrativas, para implantar na nuvem |
| Incidente | Um segredo do sistema de CI/CD vazou por uma dependência comprometida; o atacante criou recursos e acessou um armazenamento |
| Causas | Credenciais permanentes; permissões amplas; sem separação por ambiente; sem detecção de uso anômalo |
| Resposta | Revogação das chaves, análise dos registros e rotação de segredos |
| Correções | Federação de identidade do pipeline (credenciais temporárias, por repositório e por ramo), funções por ambiente com permissões mínimas e aprovação para produção |

**Lição:** eliminar credenciais de longa duração em pipelines é um dos ganhos de arquitetura de maior retorno.

## 3. Caso 3: recuperação que dependia do que estava indisponível

| Item | Descrição |
|---|---|
| Situação | Um plano de recuperação previa restaurar sistemas em uma segunda região |
| Incidente | Durante um exercício, a equipe não conseguiu autenticar-se: o provedor de identidade e o repositório de código estavam na região indisponível |
| Causas | Dependências circulares na recuperação; sem acesso de emergência independente; plano nunca testado de ponta a ponta |
| Resposta | Exercício interrompido; correção das dependências |
| Correções | Contas de emergência independentes, cópia do código e das credenciais de recuperação em local separado, mapa de dependências e exercício trimestral com RTO medido |

**Lição:** um plano de recuperação só é confiável se testado e livre de dependências do que se pretende recuperar.

## 4. Caso 4: revisão de arquitetura que evitou uma exposição

| Item | Descrição |
|---|---|
| Situação | Uma equipe de produto planejava lançar uma plataforma de análise com um banco de dados de clientes acessível por endereço público, protegido por lista de IP |
| Intervenção | A revisão de arquitetura (nível 3) apontou a exposição, as funções com permissões curinga e a ausência de registros de acesso a dados |
| Resultado | A equipe adotou ponto de extremidade privado (ADR), funções com permissões mínimas, eventos de dados habilitados e cópias imutáveis, em 3 semanas, antes do lançamento |
| Efeito | A verificação de postura pós-implantação não apresentou achados críticos |

**Lição:** a revisão é mais barata e mais eficaz quando ocorre antes da implantação, com critérios claros e tempo de resposta curto.

## 5. Caso 5: biblioteca de padrões que acelerou projetos

| Item | Descrição |
|---|---|
| Situação | Cada equipe de uma seguradora desenhava sua própria arquitetura, com resultados inconsistentes e revisões demoradas |
| Intervenção | Criação de uma biblioteca de arquiteturas de referência e de módulos de infraestrutura como código (aplicação web, API sem servidor e plataforma de dados) com controles incorporados |
| Resultado | O tempo médio de revisão caiu de 12 para 3 dias úteis; 78% das novas soluções passaram a usar os padrões; achados críticos de postura caíram 60% em um ano |

**Lição:** tornar o caminho seguro o caminho mais fácil melhora a segurança e a velocidade ao mesmo tempo.

## 6. Erros comuns, consequências e prevenção

| Erro | Consequência provável | Prevenção |
|---|---|---|
| Tratar a arquitetura de segurança como etapa final | Correções caras ou impossíveis | Segurança desde o desenho e revisão antes da implantação |
| Conta única para todos os ambientes | Raio de impacto total | Estrutura multiconta por ambiente, por produto e por função |
| Rede plana, sem segmentação | Movimento lateral livre | Segmentação, negação por padrão e pontos privados |
| Credenciais de longa duração em cargas e em pipelines | Abuso após vazamento | Identidades gerenciadas e federação |
| Backups na mesma conta e sem imutabilidade | Perda total em ransomware ou por erro | Cópias imutáveis em conta isolada |
| Diagramas desatualizados ou inexistentes | Revisões inúteis; exposição não percebida | Diagramas como código e conciliação com o ambiente |
| Revisão de arquitetura lenta e burocrática | *Shadow IT* e desvios silenciosos | Níveis por risco, autoatendimento e prazos curtos |
| Decisões sem registro | Perda de memória e repetição de erros | ADRs versionados |
| Padrões sem dono e sem versão | Obsolescência | Padrões como produto, com revisão periódica |
| Exceções permanentes | Políticas paralelas | Exceções com prazo e controles compensatórios |
| Resiliência não testada | Falhas descobertas durante a crise | Exercícios com RTO e RPO medidos |
| Dependências circulares na recuperação | Impossibilidade de recuperar | Mapa de dependências e acesso de emergência independente |
| Administradores com acesso padrão a dados | Abuso interno e movimento de ataque | Separação entre plano de controle e plano de dados |
| Não considerar o provedor de identidade como ponto crítico | Falha ou comprometimento sistêmico | Contas de emergência e monitoramento reforçado |
| Replicar a arquitetura local sem adaptação | Controles inadequados e custos altos | Arquitetura nativa de nuvem, com guardrails e automação |
| Multicloud sem necessidade comprovada | Complexidade e riscos adicionais | Justificar por risco específico |

## 7. Exercícios práticos

1. Redesenhe, em um diagrama Mermaid, a estrutura multiconta de uma organização fictícia com produção, não produção, registros, segurança, rede e recuperação isolada, indicando os guardrails de cada unidade.
2. Escolha um padrão do arquivo 2 (por exemplo, API sem servidor) e escreva a lista de controles obrigatórios, com a forma de verificação de cada um.
3. Desenhe, para uma plataforma multicliente, os modelos de silo, pool e ponte, e escolha um com justificativa de segurança e de custo.
4. Escreva a política de confiança de uma função assumida por um pipeline (por federação), restringindo o repositório e o ramo, e explique por que isso elimina chaves permanentes.
5. Defina a hierarquia de chaves de uma plataforma de saúde (chaves mestras por domínio, chaves de dados e separação de funções) e os registros necessários.
6. Classifique cinco serviços em camadas de criticidade (0 a 3), com RTO, RPO e estratégia de recuperação, e escreva o plano de teste de cada camada.
7. Preencha o questionário de revisão (arquivo 6) para uma solução fictícia e emita um parecer com três condições verificáveis.
8. Escreva um ADR sobre a decisão entre funções sem servidor e contêineres gerenciados para uma nova API.
9. Em uma conta de laboratório, execute as verificações de exposição do arquivo 7 e concilie o resultado com um diagrama que você desenhe; liste os desvios.

Use somente contas e dados de laboratório nos exercícios técnicos.

## 8. Perguntas de revisão

| Pergunta | Resposta esperada (resumo) |
|---|---|
| O que é raio de impacto e como reduzi-lo? | Conjunto de recursos e de dados afetados por uma falha ou comprometimento; reduz-se com isolamento por contas, redes e identidades, privilégio mínimo e células |
| Por que a conta é a principal fronteira de segurança em nuvem? | Por padrão, não há acesso entre contas, e ela agrupa identidade, recursos e cobrança, sendo um forte limite de isolamento |
| Qual a vantagem da federação de identidade em pipelines? | Credenciais temporárias, sem armazenar chaves permanentes, com condições por repositório e por ramo |
| O que é criptografia em envelope? | Chaves de dados criptografadas por chaves mestras, o que facilita rotação, revogação e controle de acesso |
| Quais as estratégias de recuperação de desastres, da mais simples à mais cara? | Backup e restauração, luz piloto, espera aquecida e multissite ativo |
| Por que backups imutáveis e isolados são importantes contra ransomware? | O atacante, com credenciais administrativas, tenta apagar os backups; a imutabilidade e a conta isolada impedem |
| O que é um ADR? | Registro de uma decisão de arquitetura, com contexto, alternativas, decisão e consequências |
| Por que a revisão deve ser proporcional ao risco? | Para dar velocidade aos casos padrão e profundidade aos de maior risco, evitando gargalos |
| O que são dependências circulares na recuperação? | Quando recuperar um sistema depende de outro que está indisponível pelo mesmo evento |

## 9. Casos de uso corporativos

- Treinamento de arquitetos, de engenheiros de plataforma e de equipes de produto.
- Diagnóstico da arquitetura de segurança de ambientes existentes.
- Base para o roteiro de evolução da segurança em nuvem (pasta 28).

## 10. Normas e frameworks relevantes

- NIST SP 800-160 Vol. 1 Rev. 1; NIST SP 800-204 (série); NIST SP 800-207; NIST SP 800-34 Rev. 1
- NCSC Cloud security principles; CISA Cloud Security Technical Reference Architecture; CSA CCM v4
- ISO 22301:2019; ISO/IEC 27001:2022 e 27002:2022; ISO/IEC/IEEE 42010:2022

## 11. Referências

- Sherwood, J. et al. Enterprise Security Architecture (SABSA). 2005.
- Ross, R. et al. NIST SP 800-160 Vol. 1 Rev. 1. 2022.
- Ford, N.; Parsons, R.; Kua, P. Building Evolutionary Architectures. O'Reilly, 2017.
- Binnie, C.; McCune, R. Cloud Native Security. Wiley, 2021.
- Arquivos 1 a 7 desta pasta.
