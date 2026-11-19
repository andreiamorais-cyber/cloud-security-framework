# Casos de Uso e Erros Comuns em Criptografia em Nuvem

> **Objetivo pedagógico:** consolidar o aprendizado de criptografia em nuvem com casos corporativos fictícios, os erros mais frequentes, suas consequências e a prevenção, exercícios práticos e perguntas de revisão.

Os casos são fictícios e destinados ao treinamento.

## 1. Caso 1: dados criptografados, mas lidos por credenciais válidas

| Item | Descrição |
|---|---|
| Situação | Uma empresa cumpria o requisito de "criptografia em repouso" com chaves gerenciadas pelo provedor em todos os armazenamentos |
| Incidente | Uma credencial de um engenheiro foi comprometida; o atacante, com permissão de leitura nos buckets, copiou milhares de arquivos de clientes, decifrados de forma transparente pelo serviço |
| Causas | A criptografia com chaves do provedor não impõe nenhuma barreira a quem já tem permissão de leitura; sem chaves do cliente com política própria; sem separação entre acesso ao dado e uso da chave; sem alertas de volumes anômalos |
| Resposta | Revogação das credenciais, avaliação do alcance e comunicação conforme a LGPD |
| Correções | Chaves do cliente por domínio, com políticas que exigem permissão específica de uso da chave; contexto de criptografia; alertas de picos de decifração; perímetro de dados e acesso por atributos |

**Lição:** a criptografia em repouso com chaves do provedor atende à conformidade básica, mas **não substitui** o controle de acesso; a política da chave cria uma segunda barreira.

## 2. Caso 2: exclusão acidental de uma chave

| Item | Descrição |
|---|---|
| Situação | Durante uma limpeza de recursos de teste, um administrador agendou a exclusão de uma chave cujo alias sugeria um ambiente de teste |
| Descoberta | A chave protegia o banco de dados de um sistema de produção; os alertas de exclusão agendada acionaram o SOC no mesmo dia |
| Efeito | Como havia um período de espera, a exclusão foi cancelada a tempo; sem esse período, os dados do banco e dos backups teriam sido perdidos |
| Causas | Nomes de chaves sem padrão; ausência de inventário de dependências; permissão de exclusão para administradores comuns |
| Correções | Padrão de nomes e etiquetas (ambiente, aplicação e classificação); inventário de dependências; política que restringe a exclusão de chaves a um papel específico, com aprovação por duas pessoas; período de espera máximo |

**Lição:** a exclusão de chaves é potencialmente destrutiva; proteções de espera, alertas e aprovação são essenciais, tanto contra erros quanto contra ataques.

## 3. Caso 3: interrupção por certificado expirado

| Item | Descrição |
|---|---|
| Situação | Uma operadora de pagamentos usava um certificado interno de 3 anos para a comunicação entre sistemas, renovado manualmente |
| Incidente | O certificado expirou em um sábado; as conexões entre os sistemas de autorização e de compensação falharam, causando indisponibilidade por 5 horas |
| Causas | Sem inventário nem alertas de expiração; renovação manual dependente de uma pessoa; nenhum teste de renovação |
| Resposta | Emissão emergencial e reinício dos serviços |
| Correções | Inventário de certificados; emissão automática com validade curta por CA privada; alertas a 60, 30, 14 e 7 dias; teste periódico de renovação; documentação do procedimento |

**Lição:** a maior parte das falhas relacionadas a certificados é operacional; a automação e o inventário eliminam essa classe de incidentes.

## 4. Caso 4: chave criptográfica no repositório de código

| Item | Descrição |
|---|---|
| Situação | Uma aplicação cifrava dados de clientes com uma chave AES embutida em um arquivo de configuração versionado |
| Incidente | O repositório foi clonado por um prestador, e a chave circulou em cópias locais; os dados cifrados podiam ser decifrados por quem tivesse acesso ao banco e ao repositório |
| Causas | Chave no código; ausência de serviço de gestão de chaves; sem rotação; sem varredura de segredos |
| Resposta | Geração de nova chave no serviço de chaves, recriptografia de todos os dados com criptografia em envelope, rotação de credenciais e limpeza do histórico do repositório |
| Correções | Chaves apenas em serviço de gestão de chaves; bibliotecas que usam criptografia em envelope; varredura de segredos nos repositórios e nos pipelines; revisão de código para criptografia |

**Lição:** a segurança da criptografia depende da proteção das chaves; chaves no código anulam o valor do algoritmo.

## 5. Caso 5: dependência de um gestor externo de chaves

| Item | Descrição |
|---|---|
| Situação | Uma instituição adotou gestão externa de chaves (HYOK) para dados de soberania crítica, com um único gestor local |
| Incidente | Uma falha de energia e de rede no centro de dados do gestor de chaves tornou os dados inacessíveis por 3 horas, interrompendo serviços críticos |
| Causas | Gestor sem alta disponibilidade; conectividade única; sem teste de falha; sem plano de degradação |
| Resposta | Restabelecimento do gestor e revisão do desenho |
| Correções | Gestores redundantes em dois locais, com conectividade redundante; acordos de nível de serviço internos; exercícios de falha; cache seguro de chaves de dados com validade curta para serviços críticos, conforme o risco aceito; plano de comunicação |

**Lição:** o controle total das chaves transfere ao cliente a responsabilidade pela disponibilidade; o desenho e os testes devem acompanhar essa responsabilidade.

## 6. Erros comuns, consequências e prevenção

| Erro | Consequência provável | Prevenção |
|---|---|---|
| Achar que criptografia em repouso substitui o controle de acesso | Vazamentos por credenciais válidas | Política de chaves, acesso mínimo e monitoramento |
| Usar uma única chave para tudo | Raio de impacto máximo e impossibilidade de separar funções | Chaves por domínio, por ambiente e por classificação |
| Administradores que também usam a chave | Sem separação de funções | Políticas de chave distintas para administração e para uso |
| Sem proteção contra exclusão de chaves | Perda irreversível de dados | Período de espera, alertas, aprovação e políticas de negação |
| Chaves e segredos no código ou em arquivos | Exposição e decifração por terceiros | Serviços de gestão de chaves e de segredos; varredura |
| Rotação inexistente ou sem teste | Chaves antigas com exposição prolongada; falhas na rotação de emergência | Rotação automática e procedimento de emergência testado |
| Criptografia própria ou uso incorreto de primitivas | Fraquezas exploráveis | Bibliotecas e serviços validados; revisão por especialistas |
| Reutilização de nonces no AES-GCM | Perda total de segurança | Chaves de dados por objeto; bibliotecas de alto nível |
| *Hash* simples para pseudonimização de identificadores | Reversão por força bruta | HMAC com chave protegida |
| TLS 1.0 e 1.1, ou suítes fracas, habilitados | Ataques de downgrade e de decifração | Política de TLS 1.2 ou superior; varreduras periódicas |
| Terminação de TLS na borda com tráfego interno em claro | Interceptação interna | Nova criptografia até a origem; mTLS |
| Clientes que desabilitam a validação de certificados | Ataques de intermediário | Validação obrigatória; testes automatizados |
| Certificados sem inventário, sem automação e sem alertas | Interrupções por expiração | Emissão automática e monitoramento |
| Chaves raiz de CA em servidores sempre ligados | Comprometimento de toda a PKI | Raiz offline, em HSM, com cerimônias controladas |
| Adotar HYOK ou BYOK sem avaliar a operação | Indisponibilidade e erros | Matriz de decisão; testes de falha; alta disponibilidade |
| Não planejar a transição pós-quântica | Exposição de dados de longa vida | Inventário criptográfico e agilidade |
| Backups cifrados sem guardar as chaves de recuperação | Dados irrecuperáveis | Custódia protegida e testes de restauração com as chaves |

## 7. Exercícios práticos

1. Para um sistema fictício, construa o modelo de ameaças da criptografia (repouso, trânsito e em uso) e escolha, para cada ameaça, o controle e o modelo de chaves adequados.
2. Desenhe a hierarquia de chaves (raiz, mestras por domínio e chaves de dados) de uma plataforma e as políticas de administração e de uso, indicando a separação de funções.
3. Em uma conta de laboratório, execute as verificações de somente leitura do arquivo 7 e produza um relatório de achados priorizados (volumes sem criptografia, chaves sem rotação, TLS fraco e certificados a vencer).
4. Implemente a criptografia em envelope do arquivo 3 em um ambiente de teste, com dados sintéticos, e adicione a verificação de que a decifração falha com um contexto de criptografia diferente.
5. Escreva uma política de chave com separação de funções e uma condição de uso por serviço e por conta (arquivos 2 e 6), e liste três testes de aceitação.
6. Teste (em sistemas próprios) a configuração de TLS de um servidor com as ferramentas do arquivo 4, aplique um perfil moderno e compare os resultados antes e depois.
7. Desenhe a hierarquia de uma CA privada (raiz offline e intermediárias), a política de emissão e o plano de comprometimento da CA.
8. Monte um pipeline de assinatura e verificação de imagens com chave em serviço de chaves e uma política de admissão que bloqueie imagens sem assinatura (arquivo 5).
9. Preencha o inventário criptográfico (CBOM) de uma aplicação e classifique o risco pós-quântico de cada componente, propondo o plano de migração.
10. Escreva o procedimento de revogação de uma chave comprometida, com o impacto na disponibilidade e o plano de comunicação, e simule-o em um exercício de mesa.

Use somente dados sintéticos e contas de laboratório nos exercícios técnicos. Realize testes ativos apenas em sistemas próprios ou autorizados.

## 8. Perguntas de revisão

| Pergunta | Resposta esperada (resumo) |
|---|---|
| Por que a criptografia em repouso com chaves do provedor não impede vazamentos por credenciais válidas? | A decifração é transparente para quem tem permissão de acesso; é preciso política de chave, acesso mínimo e monitoramento |
| O que é criptografia em envelope e quais são suas vantagens? | Chaves de dados cifram os dados e são cifradas por chaves mestras no serviço de chaves; facilita desempenho, rotação, revogação e isolamento |
| Qual a regra crítica do AES-GCM? | Nunca reutilizar o par (chave, nonce) |
| Qual a diferença entre CMK, BYOK e HYOK? | CMK: chave do cliente no serviço do provedor; BYOK: material gerado e importado pelo cliente; HYOK: a chave nunca sai de um gestor externo sob controle do cliente |
| Por que revogar a chave também é um risco de disponibilidade? | Interrompe o acesso aos dados, inclusive para o próprio cliente |
| O que é sigilo perfeito adiante? | Uso de chaves de sessão efêmeras (ECDHE) para que o comprometimento de uma chave de longo prazo não permita decifrar sessões passadas |
| Qual a diferença entre TLS e mTLS? | No TLS, apenas o servidor se autentica; no mTLS, cliente e servidor se autenticam com certificados |
| Por que usar certificados de curta duração? | Reduzem a janela de exposição, diminuem a dependência da revogação e forçam a automação |
| O que a computação confidencial adiciona? | Proteção de dados durante o processamento, com atestação e liberação de chaves condicionada ao ambiente verificado |
| O que significa "coletar agora e decifrar depois"? | Atacantes armazenam dados cifrados hoje para decifrá-los quando houver computadores quânticos; afeta dados de longa vida |
| O que são os padrões FIPS 203, 204 e 205? | Padrões do NIST (2024) para criptografia pós-quântica: ML-KEM (troca de chaves), ML-DSA e SLH-DSA (assinaturas) |

## 9. Casos de uso corporativos

- Treinamento de arquitetos, de desenvolvedores e de equipes de segurança em criptografia aplicada à nuvem.
- Diagnóstico do programa de criptografia e plano de evolução (chaves, certificados e pós-quântica).
- Base para o roteiro de segurança em nuvem (pasta 28) e para as métricas (pasta 27).

## 10. Normas e frameworks relevantes

- NIST SP 800-57, 800-52 Rev. 2, 800-131A e 800-77; NIST FIPS 140-3, 203, 204 e 205
- ISO/IEC 27002:2022 (8.24); CSA CCM v4 (CEK); PCI DSS v4.0 (requisitos 3 e 4); CIS Foundations Benchmarks
- OWASP Cryptographic Storage e Transport Layer Security Cheat Sheets; MITRE ATT&CK for Cloud

## 11. Referências

- Barker, E. NIST SP 800-57 Part 1 Rev. 5. 2020.
- McKay, K.; Cooper, D. NIST SP 800-52 Rev. 2. 2019.
- Ferguson, N.; Schneier, B.; Kohno, T. Cryptography Engineering. Wiley, 2010.
- Amazon Web Services. AWS KMS Cryptographic Details (documento técnico); Microsoft e Google Cloud: documentação de criptografia e de gestão de chaves (versões vigentes).
- Arquivos 1 a 7 desta pasta.
