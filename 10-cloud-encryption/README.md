# 10 Criptografia em Nuvem

## Objetivo

Projetar, implementar e operar a criptografia em ambientes de nuvem de forma correta e verificável: compreender os fundamentos e as escolhas de algoritmos, gerir chaves com serviços de gestão de chaves e módulos de segurança de hardware, escolher entre chaves do provedor, do cliente e criptografia no lado do cliente, proteger dados em trânsito com TLS, TLS mútuo e VPN, gerir certificados e assinaturas digitais, decidir o grau de controle sobre as chaves (BYOK, HYOK e computação confidencial) e preparar a transição para a criptografia pós-quântica.

## Trilha de estudo recomendada

```mermaid
flowchart LR
    A["1. Fundamentos e algoritmos"] --> B["2. Gestão de chaves"]
    B --> C["3. Repouso e lado do cliente"]
    C --> D["4. Trânsito: TLS, mTLS e VPN"]
    D --> E["5. Certificados, PKI e assinatura"]
    E --> F["6. Controle de chaves e computação confidencial"]
    F --> G["7. Ferramentas, modelos e checklist"]
    G --> H["8. Casos e erros comuns"]
```

## Conteúdo da pasta

| Ordem | Arquivo | Objetivo |
|---|---|---|
| 1 | [fundamentos-de-criptografia-em-nuvem.md](fundamentos-de-criptografia-em-nuvem.md) | Primitivas, algoritmos recomendados, validação, agilidade criptográfica e pós-quântica |
| 2 | [gestao-de-chaves-kms-hsm-e-hierarquia-de-chaves.md](gestao-de-chaves-kms-hsm-e-hierarquia-de-chaves.md) | KMS, HSM, criptografia em envelope, ciclo de vida, políticas e proteção contra exclusão |
| 3 | [criptografia-em-repouso-e-no-lado-do-cliente.md](criptografia-em-repouso-e-no-lado-do-cliente.md) | Modos de chaves, discos, bancos, objetos e criptografia no lado do cliente |
| 4 | [criptografia-em-transito-tls-mtls-e-vpn.md](criptografia-em-transito-tls-mtls-e-vpn.md) | Configuração de TLS, autenticação mútua, VPN e MACsec |
| 5 | [certificados-pki-e-assinatura-digital.md](certificados-pki-e-assinatura-digital.md) | Ciclo de vida de certificados, autoridades privadas, ICP-Brasil e assinatura de artefatos |
| 6 | [modelos-de-controle-de-chaves-byok-hyok-e-computacao-confidencial.md](modelos-de-controle-de-chaves-byok-hyok-e-computacao-confidencial.md) | Níveis de controle de chaves, gestão externa e processamento protegido |
| 7 | [ferramentas-verificacoes-modelos-e-checklist-de-criptografia.md](ferramentas-verificacoes-modelos-e-checklist-de-criptografia.md) | Verificações de somente leitura, inventários, CBOM e checklist |
| 8 | [casos-de-uso-e-erros-comuns-em-criptografia-em-nuvem.md](casos-de-uso-e-erros-comuns-em-criptografia-em-nuvem.md) | Casos fictícios, erros comuns, exercícios e perguntas de revisão |

## Princípios da pasta

- A criptografia protege contra ameaças específicas: definir o modelo de ameaças antes de escolher a técnica
- Nunca criar algoritmos ou protocolos próprios: usar bibliotecas e serviços validados
- A segurança depende das chaves: gestão, separação de funções, rotação e proteção contra perda
- Criptografar não substitui o controle de acesso: credenciais válidas acessam dados decifrados
- Automatizar certificados e rotações para eliminar falhas operacionais
- Planejar a agilidade criptográfica e a transição para algoritmos pós-quânticos

## Relação com outras pastas

- 05: arquitetura (hierarquia de chaves e separação de funções)
- 06: identidade e segredos
- 08: segurança de rede (TLS na borda e VPN)
- 09: proteção de dados (classificação define o controle de chaves)
- 11 e 12: conformidade e controles
- 13: monitoramento (registros de uso de chaves)
- 21: DevSecOps (assinatura de artefatos e segredos)
- 22 e 23: resposta a incidentes e forense

## Referências

- Barker, E. NIST SP 800-57 Part 1 Rev. 5: Recommendation for Key Management. 2020
- Barker, E.; Roginsky, A. NIST SP 800-131A Rev. 2: Transitioning the Use of Cryptographic Algorithms and Key Lengths. 2019
- McKay, K.; Cooper, D. NIST SP 800-52 Rev. 2: Guidelines for TLS Implementations. 2019
- NIST FIPS 140-3; NIST FIPS 203, 204 e 205 (padrões pós-quânticos, 2024)
- CSA. Cloud Controls Matrix v4, domínio CEK; ISO/IEC 27002:2022, controle 8.24
- Brasil. LGPD, art. 46; MP 2.200-2/2001 (ICP-Brasil); Lei 14.063/2020
