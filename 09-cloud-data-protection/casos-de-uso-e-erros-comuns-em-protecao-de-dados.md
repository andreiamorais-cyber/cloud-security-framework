# Casos de Uso e Erros Comuns em Proteção de Dados

> **Objetivo pedagógico:** consolidar o aprendizado de proteção de dados em nuvem com casos corporativos fictícios, os erros mais frequentes, suas consequências e a prevenção, exercícios práticos e perguntas de revisão.

Os casos são fictícios e destinados ao treinamento.

## 1. Caso 1: armazenamento de objetos com dados de clientes exposto

| Item | Descrição |
|---|---|
| Situação | Uma varejista criou, fora do pipeline aprovado, um bucket para exportar dados de campanhas, com leitura pública, por conveniência |
| Incidente | Um pesquisador de segurança encontrou arquivos com nomes, e-mails e CPFs de clientes e comunicou a empresa |
| Causas | Bloqueio de acesso público ausente na organização; criação de recursos fora de módulos aprovados; ausência de classificação e de varredura de dados |
| Resposta | Bloqueio imediato do acesso; análise dos registros de acesso (que não estavam habilitados para dados, o que impediu determinar o alcance); avaliação pelo DPO e comunicação, aplicando o cenário mais conservador |
| Correções | Bloqueio de acesso público por política da organização; criação de armazenamento somente por módulos; descoberta e classificação contínuas; registros de acesso a dados habilitados |

**Lição:** o bloqueio global de acesso público e os registros de acesso a dados são controles baratos e decisivos tanto para prevenir quanto para avaliar o impacto.

## 2. Caso 2: dados reais em ambiente de teste

| Item | Descrição |
|---|---|
| Situação | Uma seguradora copiava periodicamente a base de produção para homologação, com controles reduzidos e acesso amplo a prestadores |
| Incidente | Uma credencial de um prestador foi comprometida, e dados de segurados foram exfiltrados do ambiente de homologação |
| Causas | Cópia direta de dados reais; ambiente de teste com proteção inferior; acesso excessivo de terceiros |
| Resposta | Contenção, avaliação, comunicação conforme a LGPD e revisão do processo |
| Correções | Dados sintéticos e mascarados em testes, com pipeline de desidentificação; proibição de cópia direta; varredura de dados sensíveis em ambientes não produtivos; acessos de terceiros mínimos e revisados |

**Lição:** o ambiente de teste herda o risco dos dados que recebe; sem dados reais, o impacto de um comprometimento cai drasticamente.

## 3. Caso 3: eliminação que não alcançou as réplicas e os backups

| Item | Descrição |
|---|---|
| Situação | Uma empresa de serviços atendeu a um pedido de eliminação de dados de um titular apagando o registro do banco de produção |
| Problema | Meses depois, uma restauração de backup devolveu o registro à produção; os dados também permaneciam em um data lake de análise e em exportações antigas |
| Causas | Fluxos de dados não mapeados; eliminação limitada à fonte; backups sem política de expiração clara; sem registro de eliminações para reaplicação após restaurações |
| Resposta | Eliminação nos locais remanescentes, comunicação ao titular e revisão do processo |
| Correções | Mapeamento de fluxos e do catálogo; eliminação propagada; backups com expiração programada; registro de "fora de uso" reaplicado a toda restauração; revisão periódica |

**Lição:** a eliminação precisa considerar o ciclo completo dos dados, inclusive derivados, cópias e backups.

## 4. Caso 4: transferência internacional sem instrumento

| Item | Descrição |
|---|---|
| Situação | Uma empresa contratou um SaaS de atendimento ao cliente hospedado no exterior e enviou para ele históricos de atendimento com dados pessoais |
| Problema | Em uma auditoria, constatou-se que não havia hipótese legal nem instrumento (cláusulas) para a transferência, que os subprocessadores não eram conhecidos e que o fornecedor retinha os dados por prazo indefinido |
| Causas | Contratação sem avaliação de privacidade; ausência de mapeamento de fluxos; contrato padrão do fornecedor sem as cláusulas necessárias |
| Resposta | Suspensão do envio de dados sensíveis, formalização das cláusulas-padrão, lista de subprocessadores, redução dos dados enviados (minimização) e RIPD |
| Correções | Avaliação de privacidade e de segurança obrigatória para novos fornecedores; catálogo de fluxos internacionais; cláusulas-padrão integradas ao processo de contratação |

**Lição:** a avaliação de privacidade deve ocorrer antes da contratação, e os fluxos internacionais precisam de instrumento e de monitoramento.

## 5. Caso 5: pseudonimização reversível

| Item | Descrição |
|---|---|
| Situação | Uma fintech pseudonimizou CPFs em uma base de análises aplicando SHA-256 sem chave e afirmou, internamente, que os dados estavam "anonimizados" |
| Problema | Uma avaliação de segurança mostrou que, com uma tabela de todos os CPFs possíveis, os *hashes* podiam ser revertidos em minutos; os dados continuavam a ser dados pessoais |
| Causas | Identificador de baixa entropia; ausência de segredo; confusão entre pseudonimização e anonimização |
| Resposta | Reclassificação dos dados como pessoais; suspensão de compartilhamentos com parceiros que dependiam da premissa de anonimização; avaliação de impacto |
| Correções | HMAC com chave em cofre e rotação; teste de reidentificação; documentação do método; controles de acesso e finalidade; compartilhamento apenas de agregados |

**Lição:** pseudonimizar não é anonimizar; identificadores de baixa entropia exigem chave secreta, e a avaliação de reidentificação deve ser documentada.

## 6. Erros comuns, consequências e prevenção

| Erro | Consequência provável | Prevenção |
|---|---|---|
| Não inventariar nem classificar os dados | Controles aplicados às cegas; dados ocultos | Descoberta contínua; etiquetas obrigatórias; donos |
| Armazenamento ou banco com acesso público | Vazamentos e sanções | Bloqueio de acesso público na organização; verificação contínua |
| Registros de acesso a dados desabilitados | Impossibilidade de avaliar o impacto de incidentes | Eventos de dados habilitados e centralizados |
| Dados reais em ambientes de teste | Vazamentos em ambientes frágeis | Dados sintéticos ou mascarados; varredura |
| Confundir pseudonimização com anonimização | Falsa sensação de conformidade | Distinção legal e técnica; avaliação de reidentificação |
| *Hash* simples de identificadores de baixa entropia | Reversão por força bruta | HMAC com chave protegida |
| Dados pessoais em registros e em telemetria | Exposição por logs | Filtros de mascaramento; retenção curta |
| Reter dados por tempo indefinido | Risco, custo e descumprimento | Cronograma de retenção e ciclo de vida automático |
| Eliminar só na fonte | Dados em réplicas, caches e backups | Mapeamento de fluxos; propagação; expiração de backups |
| Backups na mesma conta e sem imutabilidade | Perda total em ransomware | Backups isolados, imutáveis e testados |
| Imutabilidade configurada sem teste | Retenção irreversível indevida | Testes em ambiente controlado |
| Ignorar suporte e telemetria do provedor na residência | Transferência internacional não percebida | Mapeamento de fluxos e configuração de suporte |
| Contratar SaaS sem avaliação de privacidade | Transferência sem instrumento; subprocessadores desconhecidos | Avaliação prévia; cláusulas-padrão |
| Compartilhamentos e links sem validade | Acesso prolongado por terceiros | Validade curta e escopo mínimo |
| Não tratar os direitos dos titulares em nuvem | Prazos descumpridos | Catálogo, busca por titular e eliminação propagada |
| Tratar a proteção de dados como tarefa só da segurança | Lacunas de privacidade e de negócio | Governança conjunta (segurança, privacidade, jurídico e produto) |

## 7. Exercícios práticos

1. Desenhe o ciclo de vida dos dados de um processo (por exemplo, cadastro de clientes), indicando os riscos e os controles em cada fase.
2. Defina um esquema de classificação com quatro níveis e as regras de piso para dados pessoais, sensíveis e de crianças; liste cinco etiquetas obrigatórias.
3. Execute o roteiro de descoberta de CPF (arquivo 2) em um arquivo sintético que você criar (com CPFs de teste e números inválidos) e confira os falsos positivos.
4. Escreva as políticas de proteção de um bucket "restrito" (bloqueio de acesso público, negação de transporte sem TLS, criptografia e registros), em Terraform ou em JSON.
5. Em uma conta de laboratório, execute as verificações do arquivo 7 e use o DuckDB para produzir uma lista de achados priorizados.
6. Projete um pipeline de dados para analytics com desidentificação (arquivo 4), indicando a técnica de cada campo e o teste de reidentificação.
7. Escreva o cronograma de retenção para cinco categorias de dados e as regras de ciclo de vida correspondentes, indicando onde há dependência do Jurídico.
8. Elabore o procedimento de eliminação a pedido do titular, incluindo a propagação, os backups e o registro de "fora de uso".
9. Faça a avaliação de residência de um serviço de nuvem (arquivo 6): fluxos fora da região, hipótese legal e medidas complementares; use a árvore de decisão.
10. Preencha o esqueleto de RIPD (arquivo 7) para um tratamento de dados sensíveis em nuvem.

Use somente dados sintéticos nos exercícios técnicos e não armazene dados pessoais reais nos resultados.

## 8. Perguntas de revisão

| Pergunta | Resposta esperada (resumo) |
|---|---|
| Quem é responsável pelos dados em nuvem? | O cliente, em todos os modelos de serviço; o provedor protege a infraestrutura e oferece os mecanismos |
| Por que começar pela descoberta e pela classificação? | Os controles dependem de saber que dados existem, onde estão, a sensibilidade e o dono |
| Qual a diferença entre pseudonimização e anonimização na LGPD? | Pseudonimizados continuam sendo dados pessoais (a informação adicional é mantida separada); anonimizados não são dados pessoais, desde que não reversíveis por meios razoáveis |
| Por que um hash simples de CPF é inseguro? | O CPF tem poucas combinações; um atacante gera todos os hashes e reverte; usa-se HMAC com chave secreta |
| Qual o papel do bloqueio global de acesso público? | Impedir, por política, que buckets e contêineres sejam expostos por erro de configuração |
| O que é destruição criptográfica? | Tornar dados irrecuperáveis excluindo todas as cópias das chaves que os protegem, desde que a chave seja exclusiva do conjunto |
| Como tratar a eliminação a pedido do titular diante de backups imutáveis? | Expiração programada, registro de "fora de uso" reaplicado após restaurações e comunicação do prazo |
| Armazenar em uma região do Brasil elimina a transferência internacional? | Não necessariamente: suporte remoto, telemetria, serviços globais e replicação podem configurá-la |
| Quais instrumentos podem sustentar a transferência internacional? | Cláusulas-padrão contratuais, normas corporativas globais, adequação, entre outros previstos na LGPD e no regulamento da ANPD |
| Por que evitar dados reais em testes? | Ambientes de teste têm menos proteção; dados sintéticos ou mascarados reduzem o impacto de um comprometimento |

## 9. Casos de uso corporativos

- Treinamento de equipes de segurança, de privacidade, de dados e de produto sobre proteção de dados em nuvem.
- Diagnóstico da postura de dados e plano de evolução (classificação, acesso, retenção e residência).
- Base para o roteiro de segurança em nuvem (pasta 28) e para as métricas (pasta 27).

## 10. Normas e frameworks relevantes

- ISO/IEC 27002:2022, 27017:2015, 27018:2019 e 27701:2019; NIST SP 800-53 Rev. 5 e SP 800-88 Rev. 1; NIST Privacy Framework
- CSA CCM v4 (DSP, CEK e LOG); CIS Foundations Benchmarks; CIS Controls v8.1
- LGPD, arts. 5º, 6º, 11 a 16, 18, 33 a 38 e 46 a 49; Resolução CD/ANPD nº 19/2024; MITRE ATT&CK: T1530, T1537, T1567

## 11. Referências

- Brasil. Lei nº 13.709/2018 (LGPD).
- ANPD. Resolução CD/ANPD nº 15/2024 e nº 19/2024.
- ISO/IEC 27018:2019; ISO/IEC 27701:2019.
- Cloud Security Alliance. Security Guidance v4.0 e CCM v4.
- Arquivos 1 a 7 desta pasta.
