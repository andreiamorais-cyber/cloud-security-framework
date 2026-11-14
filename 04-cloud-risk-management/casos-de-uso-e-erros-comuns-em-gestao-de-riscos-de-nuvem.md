# Casos de Uso e Erros Comuns em Gestão de Riscos de Nuvem

> **Objetivo pedagógico:** consolidar o aprendizado de gestão de riscos de nuvem com casos corporativos fictícios, os erros mais frequentes, suas consequências e a prevenção, exercícios práticos e perguntas de revisão.

Os casos são fictícios e destinados ao treinamento.

## 1. Caso 1: risco aceito sem dono e sem prazo

| Item | Descrição |
|---|---|
| Situação | Uma empresa registrou o risco "armazenamento com dados de clientes sem registro de acesso" e o marcou como "aceito" por um gerente de projeto, sem prazo e sem parecer da segurança |
| Incidente | Meses depois, um acesso indevido a dados de clientes foi descoberto, e não foi possível determinar quais registros foram acessados |
| Causas | Aceitação sem autoridade adequada (risco alto aceito por quem não tinha alçada); sem expiração; sem controles compensatórios |
| Resposta | Adoção do pior caso para a comunicação aos titulares; habilitação dos registros; revisão de todas as aceitações vigentes |
| Correções | Alçadas por nível, prazo máximo, parecer obrigatório da 2ª linha e relatório de aceitações vencendo |

**Lição:** aceitar um risco é uma decisão de gestão com alçada, prazo e controles compensatórios, e não um modo de encerrar pendências.

## 2. Caso 2: concentração em uma região

| Item | Descrição |
|---|---|
| Situação | Uma plataforma de pagamentos operava toda a produção e a recuperação em uma única região de um provedor |
| Incidente | Uma falha regional de serviços de rede interrompeu a operação por mais de 6 horas, acima do RTO de 4 horas |
| Causas | Plano de recuperação na mesma região; sem teste de recuperação regional; dependências ocultas de serviços globais não mapeadas |
| Resposta | Comunicação a clientes e ao regulador; retomada gradual; análise com o provedor |
| Correções | Segunda região para recuperação com teste semestral; mapa de dependências; acesso de emergência independente; indicador de concentração no painel de riscos |

**Lição:** a resiliência deve ser comprovada por testes; a concentração é um risco de primeira ordem.

## 3. Caso 3: quantificação que justificou o investimento

| Item | Descrição |
|---|---|
| Situação | A área de segurança de uma seguradora pediu orçamento para MFA resistente a phishing, e a diretoria questionava o custo |
| Análise | Cenário de comprometimento de conta administrativa: frequência estimada de 0,2 evento por ano e perda mediana de R$ 1,5 milhão; simulação indicou perda anual esperada de cerca de R$ 330 mil, reduzida para cerca de R$ 85 mil com o controle |
| Decisão | Benefício estimado de cerca de R$ 245 mil por ano contra custo de R$ 90 mil por ano: aprovado, com verificação de eficácia em 90 dias |
| Resultado | Contas administrativas com MFA forte em 8 semanas; o KRI correspondente passou a verde |

**Lição:** estimativas em intervalos, com suposições explícitas, tornam a conversa sobre investimento objetiva, sem falsa precisão.

## 4. Caso 4: aprisionamento tecnológico e saída forçada

| Item | Descrição |
|---|---|
| Situação | Uma empresa baseou sua análise de dados em um serviço proprietário de um provedor, sem plano de saída |
| Incidente | O provedor anunciou o fim do serviço com 12 meses de aviso; a migração estimada levaria 18 meses |
| Causas | Risco de aprisionamento aceito sem monitoramento; sem teste de exportação; formatos fechados |
| Resposta | Programa emergencial de migração; negociação de extensão; custos extras |
| Correções | Plano de saída obrigatório para serviços críticos; teste anual de exportação; preferência por formatos abertos; monitoramento de anúncios de descontinuação |

**Lição:** riscos aceitos precisam de monitoramento e de gatilhos de revisão (como anúncios do fornecedor).

## 5. Caso 5: registro de riscos desatualizado

| Item | Descrição |
|---|---|
| Situação | Um registro com 140 riscos de nuvem não era revisado havia 14 meses |
| Problema | Riscos já tratados continuavam "altos"; novos riscos (por exemplo, serviços de IA e novos fornecedores) não constavam; o comitê tomava decisões com base em informações antigas |
| Causas | Sem cadência; sem integração com incidentes e com postura; sem donos ativos |
| Resposta | Revisão completa; redução para 62 riscos consolidados; donos renovados |
| Correções | Gatilhos de atualização por eventos, KRIs automáticos, revisão trimestral dos riscos altos e relatório ao comitê |

**Lição:** o registro de riscos só tem valor se for vivo; sem atualização, vira um documento de conformidade que desinforma.

## 6. Erros comuns, consequências e prevenção

| Erro | Consequência provável | Prevenção |
|---|---|---|
| Avaliar riscos sem apetite e sem critérios aprovados | Estimativas inconsistentes e decisões arbitrárias | Aprovar apetite e escalas antes de avaliar |
| Registrar riscos vagos ("segurança em nuvem") | Impossibilidade de tratar e de medir | Enunciados em causa, evento e consequência |
| Não distinguir risco inerente de residual | Superestimar ou subestimar o efeito dos controles | Estimar os dois, com evidências de eficácia |
| Considerar controles como eficazes por existirem em política | Falsa redução de risco | Verificar a eficácia (testes e postura) |
| Usar números como medidas exatas | Falsa precisão | Declarar incertezas; usar intervalos |
| Ignorar riscos de terceiros e de concentração | Impacto sistêmico não previsto | Mapa de dependências e indicadores |
| Aceitar riscos sem alçada, prazo e controles compensatórios | Riscos permanentes e sem dono | Termo de aceitação e expiração |
| Tratar o seguro como substituto de controles | Perdas não cobertas e exigências não atendidas | Seguro como complemento; atender às exigências da apólice |
| Multicloud por reflexo, sem análise | Complexidade e riscos de segurança adicionais | Justificar por risco específico |
| Registro sem donos nem cadência | Documento desatualizado | Donos ativos, KRIs e revisão periódica |
| Não integrar incidentes e auditorias ao registro | Riscos reais fora do radar | Gatilhos de atualização |
| Reportar à direção apenas listas extensas | Falta de decisão | Resumo executivo com decisões solicitadas |
| Não testar planos de recuperação e de saída | Descoberta de falhas durante a crise | Testes periódicos |
| Deixar a segurança sozinha na gestão de riscos | Visão parcial | Participação de negócio, jurídico, finanças e DPO |

## 7. Exercícios práticos

1. Escreva três enunciados de risco (causa, evento e consequência) para uma aplicação com banco gerenciado, armazenamento de objetos e integração com um SaaS.
2. Defina, para uma organização fictícia, o apetite (cinco declarações) e as escalas de probabilidade e de impacto, com as alçadas de aceitação.
3. Use o catálogo (arquivo 3) para identificar 10 riscos de uma nova plataforma e preencha as fichas de risco (modelo 8.1).
4. Execute a função de pontuação do arquivo 2 e calcule os níveis inerente e residual de cinco riscos, com a justificativa das estimativas.
5. Rode a simulação de Monte Carlo do arquivo 4 com parâmetros de um cenário seu; compare "sem controle" e "com controle" e calcule o benefício anual.
6. Elabore a análise de concentração (arquivo 6) para um ambiente real ou fictício e proponha três ações.
7. Escreva um termo de aceitação de risco completo para um risco médio, com controles compensatórios e prazo.
8. Monte um painel com seis KRIs (arquivo 7) e simule uma situação em que dois KRIs ficam vermelhos: que ações e escalonamentos ocorrem?
9. Calcule, com os dados sintéticos do arquivo 7 e o DuckDB, a distribuição dos riscos por nível e prepare o resumo executivo.

Use somente dados fictícios ou anonimizados nos exercícios.

## 8. Perguntas de revisão

| Pergunta | Resposta esperada (resumo) |
|---|---|
| Qual a diferença entre apetite e tolerância ao risco? | Apetite é a quantidade e o tipo de risco que a organização busca ou aceita reter; tolerância é a variação aceitável em torno do apetite para objetivos específicos |
| Por que escrever riscos em causa, evento e consequência? | Permite entender o mecanismo, atribuir tratamento e medir |
| O que distingue risco inerente de residual? | Inerente: sem controles; residual: após controles existentes e eficazes |
| Quando a análise quantitativa se justifica? | Para riscos relevantes e decisões de investimento ou de seguro, com dados e intervalos |
| Quais são as opções de tratamento de riscos? | Mitigar, transferir, evitar e aceitar |
| O que o seguro cibernético não faz? | Não elimina o risco nem substitui controles, e pode ter exclusões e exigências |
| O que é concentração em nuvem? | Dependência de um mesmo ponto (provedor, região, serviço, identidade, DNS ou ferramenta) que pode causar impacto sistêmico |
| Qual a diferença entre saída planejada e saída sob estresse? | A planejada ocorre com tempo e controle; a sob estresse ocorre em uma crise, e exige alternativas já preparadas |
| O que são KRIs? | Indicadores antecedentes que sinalizam aumento de exposição antes de incidentes |
| Quem deve aceitar um risco crítico? | O comitê ou a alta direção, com prazo, controles compensatórios e parecer da 2ª linha |

## 9. Casos de uso corporativos

- Treinamento de donos de riscos, de arquitetos e de líderes sobre o ciclo de riscos de nuvem.
- Diagnóstico do programa de riscos antes de auditorias e de certificações.
- Base para o roteiro de segurança em nuvem (pasta 28) e para as métricas (pasta 27).

## 10. Normas e frameworks relevantes

- ISO 31000:2018; ISO/IEC 27005:2022; ISO/IEC 27001:2022; NIST SP 800-30 Rev. 1 e 800-37 Rev. 2; NIST IR 8286 (série)
- FAIR; CSA CCM v4 e Top Threats; ISO/IEC 27036-4:2016; ISO 22301:2019
- LGPD, arts. 38, 46, 48 e 50; Resolução CMN 4.893/2021; Resolução BCB 85/2021

## 11. Referências

- ISO 31000:2018; ISO/IEC 27005:2022.
- Hubbard, D.; Seiersen, R. How to Measure Anything in Cybersecurity Risk. 2. ed. Wiley, 2023.
- Jones, J.; Freund, J. Measuring and Managing Information Risk. 2014.
- Cloud Security Alliance. Top Threats to Cloud Computing: Pandemic Eleven. 2022.
- Arquivos 1 a 7 desta pasta.
