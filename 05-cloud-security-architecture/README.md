# 05 Arquitetura de Segurança em Nuvem

## Objetivo

Projetar a segurança de ambientes e de aplicações em nuvem desde o desenho: compreender os fundamentos e os referenciais de arquitetura de segurança, adotar arquiteturas de referência e padrões reutilizáveis, projetar isolamento, segmentação e limitação do raio de impacto, desenhar identidade, dados e chaves, construir resiliência e recuperação segura, conduzir revisões de arquitetura com registro de decisões, e usar diagramas, modelos e checklists que tornem a arquitetura verificável.

## Trilha de estudo recomendada

```mermaid
flowchart LR
    A["1. Fundamentos e referenciais"] --> B["2. Arquiteturas de referência e padrões"]
    B --> C["3. Isolamento e raio de impacto"]
    C --> D["4. Identidade, dados e chaves"]
    D --> E["5. Resiliência e recuperação"]
    E --> F["6. Revisão de arquitetura e decisões"]
    F --> G["7. Ferramentas, modelos e checklist"]
    G --> H["8. Casos e erros comuns"]
```

## Conteúdo da pasta

| Ordem | Arquivo | Objetivo |
|---|---|---|
| 1 | [fundamentos-de-arquitetura-de-seguranca-em-nuvem.md](fundamentos-de-arquitetura-de-seguranca-em-nuvem.md) | Conceitos, camadas SABSA, princípios dos provedores, NCSC e CISA |
| 2 | [arquiteturas-de-referencia-e-padroes-de-seguranca.md](arquiteturas-de-referencia-e-padroes-de-seguranca.md) | Landing zone, hub e spoke e padrões de aplicação com controles |
| 3 | [isolamento-segmentacao-e-raio-de-impacto.md](isolamento-segmentacao-e-raio-de-impacto.md) | Fronteiras de contas, ambientes, locação, células e planos de controle |
| 4 | [desenho-de-identidade-dados-e-chaves.md](desenho-de-identidade-dados-e-chaves.md) | Identidade de pessoas e cargas, fluxos de dados, hierarquia de chaves e segredos |
| 5 | [arquitetura-de-resiliencia-e-recuperacao-segura.md](arquitetura-de-resiliencia-e-recuperacao-segura.md) | Estratégias de recuperação, ambientes isolados e testes |
| 6 | [revisao-de-arquitetura-e-registro-de-decisoes.md](revisao-de-arquitetura-e-registro-de-decisoes.md) | Processo de revisão, questionário e registros de decisão (ADR) |
| 7 | [ferramentas-diagramas-modelos-e-checklist-de-arquitetura.md](ferramentas-diagramas-modelos-e-checklist-de-arquitetura.md) | Diagramas como código, verificações de exposição, modelos e checklist |
| 8 | [casos-de-uso-e-erros-comuns-em-arquitetura-de-seguranca.md](casos-de-uso-e-erros-comuns-em-arquitetura-de-seguranca.md) | Casos fictícios, erros comuns, exercícios e perguntas de revisão |

## Princípios da pasta

- A segurança é uma propriedade do desenho: o que não está na arquitetura raramente é corrigido depois
- Fronteiras de isolamento (contas, redes e identidades) limitam o dano de qualquer falha
- Identidade é o plano de controle da arquitetura: projetar sem credenciais de longa duração
- Dados dirigem a arquitetura: classificação, fluxos e chaves definem as zonas
- Resiliência e recuperação seguras fazem parte da arquitetura de segurança
- Decisões de arquitetura são registradas, revisadas e verificadas por código

## Relação com outras pastas

- 01 a 04: fundamentos, responsabilidade, governança e riscos
- 06 a 10: identidade, zero trust, rede, dados e criptografia em detalhe
- 12 e 17: controles e verificação de postura
- 14: modelagem de ameaças
- 18 a 21: cargas de trabalho, contêineres, Kubernetes e DevSecOps
- 22 e 23: resposta a incidentes e forense
- 24 a 26: implementação em Azure, AWS e Google Cloud

## Referências

- Sherwood, J.; Clark, A.; Lynas, D. Enterprise Security Architecture: A Business-Driven Approach (SABSA). CMP Books, 2005
- Ross, R. et al. NIST SP 800-160 Vol. 1 Rev. 1: Engineering Trustworthy Secure Systems. 2022
- AWS, Microsoft e Google Cloud: frameworks de boas práticas de arquitetura (pilar de segurança)
- NCSC (Reino Unido). Cloud security principles; CISA. Cloud Security Technical Reference Architecture
- NIST SP 800-204 (série): segurança de aplicações baseadas em microsserviços
