# Casos de Uso e Erros Comuns em Identidade e Acesso

> **Objetivo pedagógico:** consolidar o aprendizado de gestão de identidade e de acesso em nuvem com casos corporativos fictícios, os erros mais frequentes, suas consequências e a prevenção, exercícios práticos e perguntas de revisão.

Os casos são fictícios e destinados ao treinamento.

## 1. Caso 1: fadiga de MFA contra um administrador

| Item | Descrição |
|---|---|
| Situação | Uma empresa de serviços usava MFA por notificações simples para todos os usuários, inclusive administradores de nuvem |
| Incidente | Um atacante, com a senha de um administrador obtida em um vazamento, enviou dezenas de pedidos de MFA durante a madrugada; o administrador aprovou um deles; o atacante criou um usuário e chaves de acesso |
| Detecção | Alerta de criação de chaves de acesso fora do horário e detecção de múltiplas falhas de MFA seguidas de sucesso |
| Resposta | Revogação de sessões, desativação das chaves, remoção do usuário criado, redefinição das credenciais e revisão dos registros |
| Correções | Correspondência de número e contexto; chaves FIDO2 para administradores; bloqueio de senhas vazadas; alerta para sequências de pedidos de MFA; privilégios por elevação sob demanda |

**Lição:** MFA por notificação simples não protege administradores; autenticadores resistentes a phishing e a fadiga são necessários.

## 2. Caso 2: chave de conta de serviço vazada em um repositório

| Item | Descrição |
|---|---|
| Situação | Uma chave de conta de serviço com papel Editor do projeto foi incluída por engano em um repositório público |
| Incidente | Em minutos, a chave foi usada para criar instâncias de computação e ler armazenamentos |
| Causas | Chave de longa duração; papel básico amplo; sem varredura de segredos antes do commit; sem restrição de criação de chaves |
| Resposta | Desativação da chave, remoção de recursos, avaliação dos acessos e rotação de segredos relacionados |
| Correções | Política da organização que impede chaves de contas de serviço; identidade de carga e federação do pipeline; papéis específicos em vez de papéis básicos; varredura de segredos e alertas de uso de chave por origem desconhecida |

**Lição:** o controle mais eficaz é impedir a existência de chaves de longa duração, e não apenas tentar protegê-las.

## 3. Caso 3: papel administrativo amplo e escalonamento por `PassRole`

| Item | Descrição |
|---|---|
| Situação | Desenvolvedores tinham permissão para criar funções e passá-las a serviços, sem limite de permissão |
| Incidente | Um desenvolvedor, com credenciais comprometidas, criou uma função com privilégios de administrador, a associou a uma instância e obteve acesso total à conta |
| Causas | Permissão de criar e passar funções sem limite; sem alerta para criação de funções privilegiadas |
| Resposta | Contenção, revisão de todas as funções criadas, remoção de funções indevidas e análise de acessos |
| Correções | Limites de permissão para quem cria funções; políticas organizacionais que negam a criação de funções com privilégios altos; alertas de criação e de alteração de papéis; revisão trimestral de caminhos de escalonamento |

**Lição:** permissões de gestão de identidades são tão perigosas quanto permissões administrativas; trate-as como privilegiadas.

## 4. Caso 4: contas órfãs e convidados esquecidos

| Item | Descrição |
|---|---|
| Situação | Uma empresa de saúde tinha contas de ex-colaboradores e de prestadores ativas, sem revisões |
| Incidente | Uma conta de um prestador cujo contrato havia terminado há 8 meses foi usada para acessar dados de pacientes |
| Causas | Sem integração com o RH; convidados sem patrocinador e sem expiração; sem revisões de acesso |
| Resposta | Desativação imediata, avaliação do acesso a dados pessoais sensíveis, comunicação à ANPD e aos titulares |
| Correções | Provisionamento por SCIM com o RH; expiração automática de convidados; revisões trimestrais de acesso a dados restritos; alerta de entradas de contas inativas |

**Lição:** o ciclo de vida de identidades é um controle de segurança e de privacidade; contas esquecidas são risco de incidente de dados.

## 5. Caso 5: conta de emergência que salvou a operação

| Item | Descrição |
|---|---|
| Situação | Uma política de acesso condicional mal configurada bloqueou todos os administradores do diretório, em plena operação |
| Mitigação preparada | Havia duas contas de emergência nativas, com chaves FIDO2 físicas guardadas em cofres distintos, com alerta em qualquer uso e teste semestral |
| Resposta | A conta de emergência foi usada para corrigir a política em 22 minutos; todos os eventos foram revisados; as credenciais foram rotacionadas após o uso |
| Correções | Modo somente relatório para novas políticas; revisão por pares de mudanças de acesso condicional; exercícios adicionais |

**Lição:** contas de emergência bem preparadas, testadas e monitoradas transformam uma crise em um incidente de 22 minutos.

## 6. Erros comuns, consequências e prevenção

| Erro | Consequência provável | Prevenção |
|---|---|---|
| MFA por SMS ou por notificação simples em contas privilegiadas | Fadiga de MFA, troca de chip e phishing | MFA resistente a phishing (FIDO2 e certificados) |
| Conta raiz ou proprietário sem proteção forte e sem alertas | Controle total pelo atacante | MFA físico, sem chaves, alerta de uso |
| Privilégios administrativos permanentes para muitos usuários | Grande raio de impacto | Elevação sob demanda e poucos administradores |
| Chaves de longa duração em pipelines e em aplicações | Vazamento e abuso | Federação e identidades de carga; bloqueio de chaves |
| Papéis amplos (Owner, Editor e Administrator) para o dia a dia | Escalonamento e abuso | Papéis por função e privilégio mínimo |
| Permissões de gestão de identidades sem limites | Escalonamento de privilégio | Limites de permissão e políticas organizacionais |
| Sem desativação imediata no desligamento | Acessos de ex-colaboradores | Integração com RH e SCIM |
| Acúmulo de direitos nas mudanças de função | Privilégios excessivos | Remoção de direitos antigos e revisões |
| Convidados e terceiros sem expiração e sem patrocinador | Acessos esquecidos | Expiração e revisão periódica |
| Políticas de acesso condicional aplicadas sem teste e sem contas de emergência | Bloqueio total | Modo somente relatório, revisão e contas de emergência |
| Registros de identidade desabilitados ou com retenção curta | Impossibilidade de investigar | Centralização e retenção (pasta 13) |
| Não monitorar alterações de papéis, de confiança e de métodos de autenticação | Persistência não detectada | Detecções prioritárias |
| Recuperação de conta sem verificação robusta | Engenharia social no suporte | Verificação forte e aprovação do gestor |
| Ignorar identidades não humanas no inventário | Contas órfãs e permissões amplas | Inventário, proprietários e revisões |
| Revisões de acesso sem padrão de revogação | Acessos mantidos por inércia | Revogação por padrão, evidências e verificação |
| Tratar IAM como projeto de ferramenta | Falta de governança | Processos, donos e indicadores |

## 7. Exercícios práticos

1. Desenhe a arquitetura de IAM de uma organização fictícia (RH, provedor de identidade, federação com duas nuvens, SCIM e PAM) em um diagrama Mermaid.
2. Defina uma política de autenticação com três níveis de usuários (comum, privilegiado e de alto risco) e os métodos de MFA permitidos em cada um.
3. Escreva dez políticas de acesso condicional da linha de base (arquivo 2), com o modo de implantação (relatório e depois imposição) e as exclusões justificadas.
4. Projete cinco papéis por função para uma aplicação (leitor de registros, operador de implantação, operador de banco, auditor e administrador de elevação) e a matriz de segregação de funções.
5. Em uma conta de laboratório, execute as consultas de permissões amplas (arquivo 3) e os comandos de acesso privilegiado (arquivo 4) e liste os achados.
6. Escreva a política de confiança de uma função assumida por um pipeline (federação) e o procedimento para eliminar uma chave de longa duração.
7. Monte o processo de entrada, mudança e saída para uma empresa de 500 pessoas e o indicador de tempo de desativação.
8. Execute, com os dados fictícios do arquivo 6, a análise em DuckDB e proponha as ações para cada achado.
9. Escreva três detecções de identidade (arquivo 7) com a lógica, a gravidade e o teste, e simule a resposta inicial a uma conta comprometida.
10. Redija o procedimento de uso de uma conta de emergência, com os critérios de acionamento, a revisão pós-uso e a rotação.

Use somente contas e dados de laboratório nos exercícios técnicos.

## 8. Perguntas de revisão

| Pergunta | Resposta esperada (resumo) |
|---|---|
| Por que a identidade é o novo perímetro em nuvem? | As APIs do plano de controle são acessadas por credenciais; quem as controla controla a infraestrutura, de qualquer lugar |
| Qual a diferença entre MFA e MFA resistente a phishing? | MFA exige mais de um fator; o resistente a phishing (FIDO2 e certificados) liga a autenticação à origem e não pode ser repassado a páginas falsas |
| O que é a fadiga de MFA e como mitigá-la? | Bombardeio de notificações até a aprovação; mitiga-se com correspondência de número, limites e autenticadores resistentes a phishing |
| O que significa zero privilégios permanentes? | Ninguém mantém privilégios administrativos ativos o tempo todo; a elevação é sob demanda, temporária e aprovada |
| Para que servem as contas de emergência? | Garantir acesso quando o provedor de identidade, o MFA ou uma política bloqueiam os administradores |
| Como eliminar chaves de longa duração em pipelines? | Federação de identidade (OIDC) com credenciais temporárias e condições por repositório e por ramo |
| O que é escalonamento de privilégio por papéis? | Uso de permissões de gestão de identidades para conceder mais privilégios a si mesmo ou a terceiros |
| Por que remover direitos antigos na mudança de função? | Para evitar o acúmulo de privilégios (privilege creep) |
| O que uma revisão de acesso eficaz exige? | Dados de uso, padrão de revogação, prazos, evidências e verificação das remoções |
| Cite três detecções de identidade prioritárias | Uso da conta raiz, criação de chaves ou usuários por contas incomuns e alteração de políticas de confiança |

## 9. Casos de uso corporativos

- Treinamento de arquitetos, de administradores e de equipes de segurança em IAM de nuvem.
- Diagnóstico do estado de identidade e plano de evolução (resistência a phishing, privilégios, identidades não humanas).
- Preparação de auditorias de controle de acesso para certificações e para reguladores.

## 10. Normas e frameworks relevantes

- NIST SP 800-63 (partes A, B e C); NIST SP 800-53 Rev. 5 (AC e IA); NIST SP 800-210; NIST SP 800-207
- ISO/IEC 27002:2022 (5.15 a 5.18 e 8.2 a 8.5); CSA CCM v4 (IAM); CIS Controls v8.1 (controles 5 e 6)
- MITRE ATT&CK for Cloud; CISA Implementing Phishing-Resistant MFA
- LGPD, arts. 11, 46 e 48

## 11. Referências

- Grassi, P. et al. NIST SP 800-63. Digital Identity Guidelines.
- CISA. Implementing Phishing-Resistant MFA. 2022.
- Rhino Security Labs. AWS IAM Privilege Escalation. 2018 e atualizações.
- OWASP. Non-Human Identities Top 10.
- Arquivos 1 a 7 desta pasta.
