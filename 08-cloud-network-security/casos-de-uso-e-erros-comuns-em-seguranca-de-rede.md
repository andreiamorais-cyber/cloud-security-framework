# Casos de Uso e Erros Comuns em Segurança de Rede

> **Objetivo pedagógico:** consolidar o aprendizado de segurança de rede em nuvem com casos corporativos fictícios, os erros mais frequentes, suas consequências e a prevenção, exercícios práticos e perguntas de revisão.

Os casos são fictícios e destinados ao treinamento.

## 1. Caso 1: SSH aberto ao mundo e força bruta

| Item | Descrição |
|---|---|
| Situação | Uma empresa de tecnologia manteve, por conveniência, a porta 22 de servidores de aplicação aberta para `0.0.0.0/0` |
| Incidente | Um ataque de força bruta obteve acesso a um servidor com credenciais fracas e instalou um minerador de criptomoedas |
| Causas | Administração exposta à internet; autenticação por senha; sem detecção de tentativas; sem revisão de regras |
| Resposta | Isolamento do servidor, remoção do minerador, rotação de credenciais e análise do alcance |
| Correções | Fechamento das portas de administração; acesso por sessões gerenciadas e autenticadas; política que impede regras abertas para portas de administração; alerta para tentativas e para criação de regras abertas |

**Lição:** a administração nunca deve estar exposta à internet; o acesso por sessões autenticadas elimina a superfície.

## 2. Caso 2: rede plana e propagação de ransomware

| Item | Descrição |
|---|---|
| Situação | Uma indústria operava uma rede virtual única, com regras amplas entre servidores |
| Incidente | Um ransomware executado em um servidor de aplicação alcançou, em horas, os servidores de arquivos e de banco de dados |
| Causas | Sem segmentação; regras de saída e de leste a oeste irrestritas; sem inspeção entre segmentos |
| Resposta | Isolamento, recuperação a partir de backups e comunicação |
| Correções | Segmentação por função e por classificação; microssegmentação por referência a grupos; domínios de roteamento separados; firewall de rede no hub; saída filtrada; backups imutáveis |

**Lição:** a segmentação limita a propagação; sem ela, um único comprometimento alcança tudo.

## 3. Caso 3: ataque de negação de serviço de aplicação mitigado na borda

| Item | Descrição |
|---|---|
| Situação | Um portal de serviços recebeu, durante uma campanha promocional, um ataque de requisições em massa (camada 7) |
| Controles existentes | CDN, WAF com regras gerenciadas e limitação de taxa calibrada em contagem por duas semanas e depois em bloqueio; origem aceitando tráfego apenas da borda |
| Efeito | O tráfego malicioso foi absorvido e bloqueado na borda; a origem manteve a operação; a taxa de erros para clientes legítimos ficou abaixo de 0,5% |
| Resposta | O SOC ajustou regras com os padrões observados, comunicou-se com o provedor e registrou lições |
| Lições | Preparação, calibração prévia e proteção da origem foram decisivas |

**Lição:** a proteção em camadas e a calibração antes do incidente definem o resultado; um WAF novo e não ajustado teria bloqueado clientes legítimos ou falhado.

## 4. Caso 4: exfiltração por DNS detectada pelos registros

| Item | Descrição |
|---|---|
| Situação | Uma seguradora registrava as consultas de DNS de todas as redes e tinha casos de uso de detecção de túneis |
| Incidente | Uma aplicação comprometida enviava dados codificados em consultas a subdomínios de um domínio controlado pelo atacante |
| Detecção | Alerta por volume e por tamanho médio de nomes; correlação com a ausência de outros tráfegos de saída |
| Resposta | Isolamento da carga, bloqueio do domínio no filtro de DNS, preservação dos registros e investigação |
| Correções | Saída de DNS exclusivamente pelo resolvedor filtrado; bloqueio de DNS externo; correção da vulnerabilidade |

**Lição:** o DNS é um canal de saída que precisa ser registrado, filtrado e monitorado.

## 5. Caso 5: perímetro de dados impediu a cópia para uma conta externa

| Item | Descrição |
|---|---|
| Situação | Uma fintech implementou um perímetro de dados com políticas de pontos de extremidade, saída por proxy e alertas |
| Incidente | Credenciais de um engenheiro foram comprometidas; o atacante tentou copiar dados para um bucket em outra organização, de dentro da rede |
| Efeito | A política do ponto de extremidade negou o acesso a recursos de fora da organização; a saída pela internet foi bloqueada pelo proxy |
| Resposta | Alerta em minutos, revogação das credenciais e investigação |
| Resultado | Nenhum dado exfiltrado |

**Lição:** o perímetro de dados protege mesmo quando as credenciais são válidas, porque restringe para onde os dados podem ir.

## 6. Erros comuns, consequências e prevenção

| Erro | Consequência provável | Prevenção |
|---|---|---|
| Portas de administração abertas à internet | Invasões por força bruta e por exploração | Sessões gerenciadas; políticas que impedem regras abertas |
| Bancos de dados e serviços internos com endereço público | Exposição de dados | Pontos privados; negar acesso público |
| Rede plana e regras amplas entre sistemas | Movimentação lateral e ransomware | Segmentação e microssegmentação |
| Regras por endereço IP em ambientes elásticos | Regras obsoletas e brechas | Referência a grupos, etiquetas e funções |
| Ignorar as regras de IPv6 | Exposição por um canal esquecido | Regras equivalentes e verificação |
| Redes e grupos de segurança padrão em uso | Regras permissivas herdadas | Remover ou restringir os padrões |
| Saída irrestrita para a internet | Comando e controle e exfiltração | Saída por proxy ou firewall com lista de destinos |
| Não controlar o DNS | Túneis e exfiltração | Resolvedor filtrado; bloqueio de DNS externo; monitoramento |
| WAF em modo de bloqueio sem calibração | Bloqueio de clientes legítimos | Modo de contagem, ajuste e testes |
| Origem acessível diretamente, contornando a borda | Ataques ignoram o WAF e o DDoS | Origem privada e aceitando apenas a borda |
| Confiar no WAF em vez de corrigir vulnerabilidades | Exploração com contornos | WAF como camada, e não como substituto |
| Registros de fluxo e de DNS desabilitados ou com retenção curta | Impossibilidade de detectar e de investigar | Registros centralizados e retidos |
| Conectividade híbrida plana, sem filtros | Ataques locais alcançam a nuvem e vice-versa | Segmentação e firewalls na borda híbrida |
| Enlaces dedicados sem criptografia para dados sensíveis | Exposição em trânsito | MACsec ou IPsec conforme os requisitos |
| Alterações de rede sem controle e sem alertas | Aberturas silenciosas por erro ou por ataque | Políticas, revisão e alertas de alteração |
| Regras sem dono e sem revisão | Acúmulo de regras e de risco | Descrição, dono e revisão periódica |
| Subdomínios órfãos e contas de DNS sem proteção | Tomada de subdomínios e de domínios | Inventário, verificação e MFA nas contas |

## 7. Exercícios práticos

1. Desenhe, em um diagrama Mermaid, uma arquitetura hub e spoke para uma empresa com produção, dados restritos e não produção, indicando os domínios de roteamento, o firewall e a saída controlada.
2. Escreva um plano de endereçamento IP para três regiões, sem sobreposição com uma rede local 172.16.0.0/12, e explique as regras para IPv6.
3. Escreva, em Terraform, os grupos de segurança de uma aplicação de três camadas por referência a grupos e remova a saída padrão do banco.
4. Em uma conta de laboratório, execute as verificações do arquivo 7 e o roteiro em Python para classificar regras abertas; liste os achados e as ações.
5. Elabore um plano de implantação gradual do controle de saída (arquivo 6) para uma aplicação, com as fases, os destinos permitidos e os critérios de passagem.
6. Escreva uma política de ponto de extremidade para o armazenamento de objetos que restrinja os recursos à organização, e liste três exceções que podem ser necessárias.
7. Defina cinco detecções de rede (arquivo 5), com as fontes, a lógica, a gravidade e os falsos positivos esperados; escreva a consulta de uma delas.
8. Projete a proteção de borda de uma API pública (CDN, WAF, DDoS, TLS e gateway), indicando os limites de taxa e o plano de ajuste.
9. Escreva um procedimento de resposta a uma suspeita de exfiltração por DNS, incluindo preservação de registros, contenção e comunicação.

Use somente contas e dados de laboratório nos exercícios técnicos e realize testes de exposição apenas em ambientes próprios e autorizados.

## 8. Perguntas de revisão

| Pergunta | Resposta esperada (resumo) |
|---|---|
| Qual a diferença entre grupos de segurança e listas de controle de acesso de rede? | Os grupos de segurança são filtros com estado, por recurso; as listas de controle são filtros sem estado, por sub-rede, com regras de permissão e de negação |
| Por que referenciar grupos em vez de endereços IP? | Em ambientes elásticos, os endereços mudam; as referências mantêm a regra válida e por função |
| O que é microssegmentação e por que é importante? | Divisão da rede em segmentos pequenos com regras explícitas, limitando a movimentação lateral e a propagação |
| Por que a saída precisa de controle? | Comando e controle, download de ferramentas e exfiltração dependem de saída; o padrão costuma ser permissivo |
| O que é um perímetro de dados? | Controles que garantem que somente identidades, recursos e redes confiáveis acessem os dados, impedindo a cópia para fora da organização |
| Quais as limitações dos registros de fluxo? | Não têm conteúdo de pacotes, são agregados por janelas, não cobrem todo o tráfego e têm custo e latência |
| Por que o WAF deve começar em modo de contagem? | Para observar e ajustar falsos positivos antes de bloquear clientes legítimos |
| Qual a função de proteger a origem? | Impedir que ataques contornem a CDN, o WAF e a proteção DDoS acessando a aplicação diretamente |
| Como o DNS pode ser usado para exfiltração? | Dados codificados em nomes de consultas a um domínio controlado pelo atacante |
| Por que conexões dedicadas exigem atenção à criptografia? | São privadas, mas geralmente não criptografadas por padrão; dados sensíveis exigem MACsec ou IPsec |

## 9. Casos de uso corporativos

- Treinamento de arquitetos de rede, de equipes de segurança e de operações.
- Diagnóstico da segurança de rede e plano de evolução (segmentação, saída e borda).
- Base para o roteiro de segurança em nuvem (pasta 28) e para as métricas (pasta 27).

## 10. Normas e frameworks relevantes

- NIST SP 800-41 Rev. 1; NIST SP 800-77 Rev. 1; NIST SP 800-52 Rev. 2; NIST SP 800-53 Rev. 5 (SC-7)
- CIS Foundations Benchmarks; CIS Controls v8.1 (12 e 13); CSA CCM v4 (IVS)
- OWASP Top 10 e API Security Top 10; MITRE ATT&CK for Cloud

## 11. Referências

- Scarfone, K.; Hoffman, P. NIST SP 800-41 Rev. 1. 2009.
- Amazon Web Services. Building a Data Perimeter on AWS; Google Cloud. VPC Service Controls (documentação vigente).
- OWASP. API Security Top 10 (2023).
- Dotson, C. Practical Cloud Security. O'Reilly, 2019.
- Arquivos 1 a 7 desta pasta.
