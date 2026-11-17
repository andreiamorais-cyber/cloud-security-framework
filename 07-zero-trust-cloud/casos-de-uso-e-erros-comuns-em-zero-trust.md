# Casos de Uso e Erros Comuns em Zero Trust

> **Objetivo pedagógico:** consolidar o aprendizado de Zero Trust em nuvem com casos corporativos fictícios, os erros mais frequentes, suas consequências e a prevenção, exercícios práticos e perguntas de revisão.

Os casos são fictícios e destinados ao treinamento.

## 1. Caso 1: VPN comprometida e migração para acesso por aplicação

| Item | Descrição |
|---|---|
| Situação | Uma empresa industrial usava VPN de acesso amplo, com senha e código de aplicativo, para 900 usuários |
| Incidente | Uma credencial de um colaborador de terceiros foi roubada; o atacante entrou na VPN, varreu a rede e alcançou servidores de produção |
| Causas | Acesso amplo à rede; fator fraco; sem verificação do dispositivo; sem segmentação |
| Resposta | Revogação do acesso, reconstrução de sistemas e análise do alcance |
| Correções | Acesso por aplicação (ZTNA) com MFA resistente a phishing e dispositivo conforme; microssegmentação do ambiente de produção; terceiros com acesso apenas às aplicações do contrato; telemetria correlacionada |

**Lição:** trocar a conectividade por rede pelo acesso por aplicação reduz o alcance de qualquer credencial roubada.

## 2. Caso 2: dispositivo não conforme com acesso a dados restritos

| Item | Descrição |
|---|---|
| Situação | Uma seguradora exigia MFA, mas aceitava acessos de qualquer dispositivo ao sistema de sinistros |
| Incidente | Um notebook pessoal infectado por *infostealer* roubou o cookie de sessão de uma analista, e o atacante acessou dados de clientes |
| Causas | Sem conformidade de dispositivo como condição; sessões longas; tokens não vinculados ao dispositivo |
| Resposta | Revogação de sessões, avaliação de dados acessados e comunicação conforme a LGPD |
| Correções | Dispositivo gerenciado e conforme exigido para recursos restritos; acesso limitado (sem download) para dispositivos pessoais; sessões mais curtas; vinculação de token ao dispositivo; EDR integrado às decisões de acesso |

**Lição:** o MFA protege a autenticação; a confiança do dispositivo e a proteção da sessão protegem o uso.

## 3. Caso 3: microssegmentação que conteve um ransomware

| Item | Descrição |
|---|---|
| Situação | Um hospital segmentou, ao longo de 18 meses, os ambientes de prontuários, de imagens e de administração, com negação por padrão e pontos privados |
| Incidente | Um ransomware foi executado em uma estação do administrativo |
| Efeito | A propagação ficou restrita ao segmento administrativo; os servidores de prontuários e de imagens não foram alcançados; backups imutáveis estavam intactos |
| Resposta | Isolamento do segmento, recuperação das estações e comunicação; operação clínica mantida |
| Lições | A segmentação e a recuperação planejada transformaram um incidente potencialmente crítico em um evento contido |

**Lição:** a microssegmentação não impede a invasão inicial, mas limita fortemente o raio de impacto.

## 4. Caso 4: "Zero Trust" como compra de produto

| Item | Descrição |
|---|---|
| Situação | Uma organização adquiriu uma plataforma de ZTNA cara e declarou o Zero Trust concluído |
| Problema | As políticas continuavam permissivas ("qualquer usuário, qualquer aplicação"); privilégios permanentes; sem conformidade de dispositivos; sem telemetria correlacionada |
| Descoberta | Uma avaliação de maturidade mostrou todos os pilares nos estágios tradicional e inicial, apesar do investimento |
| Correções | Estratégia por pilar e por superfície de proteção; políticas por aplicação; elevação sob demanda; conformidade de dispositivos; indicadores de progresso |

**Lição:** ferramentas habilitam o Zero Trust; as políticas, os processos e a governança o concretizam.

## 5. Caso 5: avaliação contínua que bloqueou o uso de um token roubado

| Item | Descrição |
|---|---|
| Situação | Uma fintech habilitou avaliação contínua de acesso, tokens vinculados ao dispositivo e pontuação de risco integrada ao acesso condicional |
| Incidente | Um atacante capturou o cookie de sessão de um usuário por um proxy reverso (phishing) e tentou usá-lo de outro país e de outro dispositivo |
| Efeito | O token não pôde ser reutilizado por não estar vinculado ao dispositivo; o evento de risco elevou a pontuação do usuário, revogou as sessões e exigiu reautenticação resistente a phishing; o SOC foi alertado em minutos |
| Resultado | Nenhum dado acessado; indicadores do phishing bloqueados; treinamento direcionado ao usuário |

**Lição:** verificação contínua e vinculação de sessões reduzem o valor de credenciais e de sessões roubadas.

## 6. Erros comuns, consequências e prevenção

| Erro | Consequência provável | Prevenção |
|---|---|---|
| Tratar Zero Trust como produto ou como projeto com data final | Gasto sem redução de risco | Estratégia, pilares, roteiro e governança contínuos |
| Substituir a VPN por ZTNA, mas manter políticas permissivas | Acesso amplo com outro nome | Políticas por aplicação, com privilégio mínimo |
| Impor políticas sem fase de observação | Interrupções e rejeição das áreas | Modo de relatório, pilotos e comunicação |
| Ignorar contas de emergência | Bloqueio total em caso de falha | Contas de emergência independentes e monitoradas |
| MFA forte, mas sem conformidade de dispositivo | Roubo de sessão por dispositivo infectado | Dispositivo conforme como condição de acesso |
| Sessões longas e tokens não vinculados | Reutilização de sessões roubadas | Sessões curtas, vinculação ao dispositivo e avaliação contínua |
| Rede plana e confiança entre cargas | Movimentação lateral e ransomware | Microssegmentação e autenticação entre serviços |
| Segmentação baseada em endereços IP em ambientes elásticos | Regras obsoletas e brechas | Regras por identidade, etiquetas e funções |
| Telemetria dispersa ou insuficiente | Decisões e detecções cegas | Centralização e correlação entre pilares |
| Automação sem salvaguardas | Bloqueios indevidos e danos | Aprovação para ações amplas, limites e reversibilidade |
| Foco em apenas um pilar | Lacunas exploradas em outros | Avaliação e plano por pilar |
| Dados sem classificação e sem controle de acesso por atributos | Zero Trust sem proteger o que importa | Classificação e controles centrados em dados |
| Não medir maturidade e progresso | Sem prova de eficácia | Indicadores e avaliação anual |
| Ignorar a experiência do usuário | Contornos e *shadow IT* | Autenticação simples e acesso transparente |

## 7. Exercícios práticos

1. Desenhe, em Mermaid, a arquitetura lógica de Zero Trust (mecanismo, administrador e pontos de aplicação) para uma aplicação web em nuvem, indicando as fontes de dados do mecanismo de políticas.
2. Escolha três superfícies de proteção de uma organização fictícia e aplique os cinco passos (arquivo 6), com os fluxos de transações e a política dos cinco Ws.
3. Defina quatro níveis de confiança de dispositivos e as camadas de acesso por sensibilidade (arquivo 2) para o seu cenário.
4. Execute o código do motor de decisão (arquivo 2) e adicione duas regras novas (por exemplo, horário e geografia); teste cinco combinações de entrada.
5. Escreva as políticas de rede do Kubernetes (arquivo 3) com negação por padrão e a liberação mínima para uma aplicação de três camadas.
6. Escreva uma política em Rego para autorização por requisição de uma API de pedidos, com autorização por objeto e escopos, e liste os casos de teste.
7. Em uma conta de laboratório, execute as verificações de somente leitura (arquivo 7) e prepare um relatório de achados (políticas em modo de relatório, exclusões e exposições).
8. Elabore um fluxo de resposta automatizada para "entrada de risco alto seguida de acesso a dados restritos", com as salvaguardas e as aprovações.
9. Faça a avaliação de maturidade por pilar (arquivo 6) de uma organização real ou fictícia e escreva o roteiro dos primeiros 12 meses.

Use somente contas e dados de laboratório nos exercícios técnicos.

## 8. Perguntas de revisão

| Pergunta | Resposta esperada (resumo) |
|---|---|
| Qual a ideia central do Zero Trust? | Nenhuma confiança implícita por localização: verificar explicitamente cada acesso, com privilégio mínimo e suposição de violação |
| Quais são os componentes lógicos do NIST SP 800-207? | Mecanismo de políticas (PE), administrador de políticas (PA) e ponto de aplicação (PEP), alimentados por fontes de dados |
| O que é ZTNA e qual a diferença para uma VPN ampla? | Acesso por aplicação, após verificação de identidade, dispositivo e contexto, sem conceder acesso à rede |
| O que é microssegmentação? | Divisão da rede em segmentos pequenos com políticas explícitas de negar por padrão, limitando a movimentação lateral |
| Por que o MFA não basta no Zero Trust? | O roubo de sessão e os dispositivos comprometidos exigem conformidade de dispositivo, sessões curtas e avaliação contínua |
| O que é avaliação contínua de acesso? | Troca de eventos críticos entre o provedor de identidade e os recursos para revogar sessões quase em tempo real |
| Por que a autorização deve ocorrer por requisição? | Para que cada chamada seja avaliada no contexto, evitando que o acesso à rede ou a sessões longas conceda confiança herdada |
| Quais os estágios do modelo de maturidade da CISA? | Tradicional, inicial, avançado e ideal |
| Por que começar por superfícies de proteção? | Para priorizar o que é mais crítico, obtendo resultados incrementais e reduzindo a complexidade |
| Zero Trust é um produto? | Não: é uma estratégia e uma arquitetura, apoiada por ferramentas |

## 9. Casos de uso corporativos

- Treinamento de arquitetos, de equipes de segurança e de líderes sobre Zero Trust em nuvem.
- Diagnóstico de maturidade e planejamento do programa de Zero Trust.
- Base para o roteiro de segurança em nuvem (pasta 28) e para as métricas (pasta 27).

## 10. Normas e frameworks relevantes

- NIST SP 800-207 e 800-207A; CISA ZTMM 2.0; NSA Zero Trust Implementation Guidelines; DoD Zero Trust Strategy
- NIST SP 800-63, 800-204 e 800-53 Rev. 5; CSA CCM v4; CIS Controls v8.1
- MITRE ATT&CK for Cloud; OWASP ASVS e API Security Top 10

## 11. Referências

- Rose, S. et al. NIST SP 800-207. 2020.
- CISA. Zero Trust Maturity Model 2.0. 2023.
- Ward, R.; Beyer, B. BeyondCorp. 2014.
- Gilman, E.; Barth, D. Zero Trust Networks. O'Reilly, 2017.
- Arquivos 1 a 7 desta pasta.
