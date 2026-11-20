# Teste de Eficácia e Garantia de Controles

> **Objetivo pedagógico:** verificar se os controles funcionam: distinguir a eficácia de desenho da eficácia de operação, aplicar os métodos de avaliação (examinar, entrevistar e testar), escolher entre testes manuais, automatizados e por amostragem, automatizar a verificação de controles técnicos com scripts de somente leitura, validar controles preventivos e detectivos com simulações de ataque em ambientes de laboratório (equipe roxa e emulação de técnicas), avaliar a maturidade dos controles e usar os resultados para atualizar o risco residual.

## 1. Eficácia de desenho e eficácia de operação

| Conceito | Pergunta | Exemplo |
|---|---|---|
| Eficácia de desenho | O controle, se executado como projetado, atinge o objetivo? | A política de MFA exige método resistente a phishing para todos os privilegiados? |
| Eficácia de operação | O controle foi executado de forma consistente e correta no período? | Todos os privilegiados realmente usaram o método exigido, sem exceções, durante o ano? |
| Implementação | O controle existe e está em funcionamento neste momento? | A política está ativa hoje? |

Um controle pode ter bom desenho e má operação (exclusões, desvios e falhas), ou má concepção apesar da execução perfeita. **Os dois** precisam ser avaliados.

## 2. Métodos de avaliação

O NIST SP 800-53A define três métodos principais de avaliação:

| Método | Descrição | Exemplos em nuvem |
|---|---|---|
| Examinar | Revisar documentos, configurações e registros | Exportar uma política, revisar a configuração de registros, ler um relatório de revisão |
| Entrevistar | Conversar com responsáveis e usuários | Entrevistar o dono de aplicação sobre o processo de revisão de acessos |
| Testar | Executar o controle ou simular uma condição e observar o resultado | Tentar criar um bucket público em conta de teste; reexecutar uma consulta de verificação |

| Técnica de auditoria | Descrição |
|---|---|
| Indagação | Perguntas aos responsáveis (evidência de menor força, deve ser corroborada) |
| Observação | Acompanhar a execução do controle |
| Inspeção | Examinar documentos, registros e configurações |
| Reexecução | O avaliador executa novamente o controle de forma independente |
| Procedimentos analíticos | Análise de dados e de tendências para identificar anomalias |
| Teste automatizado | Verificação por scripts, consultas e ferramentas, sobre toda a população |

**Força relativa das evidências** (do menor para o maior): indagação, observação, inspeção, reexecução e teste automatizado independente sobre a população completa.

## 3. Testes automatizados de controles técnicos

Em nuvem, a maior parte dos controles técnicos pode ser verificada **sobre toda a população de recursos**, por consultas de somente leitura, com resultado "passou ou falhou" por recurso.

### Exemplo em Python: verificar o bloqueio de acesso público em todos os buckets (AWS, somente leitura)

```python
# Teste do controle BSL-17: todos os buckets devem ter o bloqueio de acesso público totalmente ativo
# Requer: pip install boto3 e credenciais de leitura (por exemplo, o papel de auditoria)
import csv
import datetime

import boto3
from botocore.exceptions import ClientError

s3 = boto3.client("s3")
CAMPOS = ("BlockPublicAcls", "IgnorePublicAcls", "BlockPublicPolicy", "RestrictPublicBuckets")

def verificar_bucket(nome: str) -> tuple[str, str]:
    try:
        cfg = s3.get_public_access_block(Bucket=nome)["PublicAccessBlockConfiguration"]
    except ClientError as erro:
        if erro.response["Error"]["Code"] == "NoSuchPublicAccessBlockConfiguration":
            return "FALHOU", "sem configuracao de bloqueio no bucket"
        return "ERRO", erro.response["Error"]["Code"]
    faltando = [c for c in CAMPOS if not cfg.get(c)]
    return ("PASSOU", "") if not faltando else ("FALHOU", "desativados: " + ",".join(faltando))

def executar() -> None:
    buckets = [b["Name"] for b in s3.list_buckets()["Buckets"]]
    agora = datetime.datetime.now(datetime.timezone.utc).isoformat()
    with open("resultado_bsl17.csv", "w", newline="", encoding="utf-8") as f:
        escritor = csv.writer(f)
        escritor.writerow(["controle", "recurso", "resultado", "detalhe", "verificado_em_utc"])
        for nome in buckets:
            resultado, detalhe = verificar_bucket(nome)
            escritor.writerow(["BSL-17", nome, resultado, detalhe, agora])
    print(f"{len(buckets)} buckets verificados; resultados em resultado_bsl17.csv")

if __name__ == "__main__":
    executar()
```

Observações: o teste verifica a configuração **no nível do bucket**; combine-o com a verificação **no nível da conta e da organização** e com o status de política (por exemplo, `get-bucket-policy-status`). O resultado em CSV é uma **evidência** (arquivo com hash e metadados, pasta 11, arquivo 5).

### Resultado por controle: taxa de conformidade

```sql
-- DuckDB: taxa de conformidade por controle, a partir dos resultados dos testes automatizados
SELECT controle,
       count(*) AS recursos,
       count(*) FILTER (WHERE resultado = 'PASSOU') AS aprovados,
       round(100.0 * count(*) FILTER (WHERE resultado = 'PASSOU') / count(*), 1) AS percentual_conforme
FROM read_csv_auto('resultado_bsl17.csv')
GROUP BY controle;
```

## 4. Teste por amostragem

Quando o controle é manual ou depende de julgamento, use **amostragem** com critério:

| Elemento | Orientação |
|---|---|
| População | Definir claramente o conjunto (por exemplo, todas as revisões de acesso do ano) |
| Tamanho da amostra | Proporcional à frequência do controle e ao risco (por exemplo, controles trimestrais: 2 de 4; mensais: 2 a 5 de 12; diários: 25 a 40) |
| Seleção | Aleatória ou orientada por risco (por exemplo, priorizar sistemas críticos) |
| Atributos testados | O que deve ser verdadeiro em cada item (aprovação, prazo e evidência) |
| Desvios | Tolerância e investigação de causas; extrapolação, quando aplicável |
| Registro | Papéis de trabalho com itens, evidências e conclusões |

Os tamanhos acima são **referências práticas** comuns em auditoria; use a metodologia da sua auditoria interna.

## 5. Validação baseada em ataques

Testes de configuração dizem se o controle **está configurado**; a validação por ataques mostra se ele **detecta ou impede** a técnica de um atacante.

| Abordagem | Descrição | Frequência |
|---|---|---|
| Teste de intrusão em nuvem | Avaliação ofensiva autorizada, com escopo e regras definidos (pasta 16) | Anual e após mudanças relevantes |
| Equipe roxa (*purple team*) | Equipes ofensiva e defensiva trabalham juntas: executam técnicas e ajustam controles e detecções | Trimestral ou semestral |
| Emulação de técnicas de ataque | Execução controlada de técnicas do ATT&CK em ambientes de laboratório para verificar a detecção e a resposta | Contínua em laboratório |
| Simulação de violação e de ataque (BAS) | Ferramentas que executam cenários de ataque de forma automatizada | Contínua |
| Exercícios de mesa | Simulações de decisão e de comunicação | Semestral |
| Exercícios de recuperação | Testes de restauração e de recuperação regional (pasta 05, arquivo 5) | Conforme a criticidade |

### 5.1 Ferramentas de emulação para laboratório

| Ferramenta | Uso |
|---|---|
| Stratus Red Team (DataDog) | Emula técnicas de ataque específicas para nuvem (AWS, Azure e Google Cloud, entre outras) para validar detecções |
| Atomic Red Team (Red Canary) | Testes atômicos mapeados ao ATT&CK, inclusive alguns de nuvem e de identidade |
| CloudGoat e outros ambientes vulneráveis intencionais | Ambientes de laboratório para treinamento e para testes de detecção |
| Ferramentas ofensivas especializadas em nuvem | Usadas por equipes autorizadas de teste de intrusão |

> **Atenção (segurança e legalidade):** execute emulações **somente em contas de laboratório dedicadas**, com autorização formal, isoladas da produção, com orçamentos e limites, e com a equipe de monitoramento informada. Algumas técnicas criam, alteram ou apagam recursos. **Nunca** execute em contas de produção ou de terceiros.

### 5.2 Exemplo de ciclo de validação de uma detecção (laboratório)

```bash
# Stratus Red Team: listar técnicas disponíveis e emular uma técnica de evasão de defesa em conta de laboratório
stratus list
stratus show aws.defense-evasion.cloudtrail-stop
stratus warmup aws.defense-evasion.cloudtrail-stop     # prepara a infraestrutura necessária
stratus detonate aws.defense-evasion.cloudtrail-stop   # executa a técnica (para a trilha de auditoria da conta de laboratório)
stratus cleanup aws.defense-evasion.cloudtrail-stop    # remove os recursos criados
```

Depois de cada emulação, verifique:

| Pergunta | Resultado esperado |
|---|---|
| O controle **preventivo** impediu a ação? | A política organizacional negou a parada da trilha (preferível) |
| Se não impediu, o controle **detectivo** gerou o alerta? | Alerta em minutos, com a identidade e o contexto |
| O alerta foi **triado** e levou à **resposta**? | Playbook acionado e ação corretiva registrada |
| O tempo de detecção e de resposta foi aceitável? | Dentro das metas (pasta 13) |
| As lacunas foram registradas? | Regras ajustadas ou controles novos, com prazos |

## 6. Avaliação da maturidade dos controles

| Nível | Descrição |
|---|---|
| 0 Inexistente | Sem controle |
| 1 Inicial | Controle informal, dependente de pessoas, sem documentação |
| 2 Repetível | Controle documentado e executado, com inconsistências e evidências manuais |
| 3 Definido | Controle padronizado, com dono, evidências e testes periódicos |
| 4 Gerenciado | Controle medido, com indicadores, automação e verificação contínua |
| 5 Otimizado | Controle automatizado, validado por ataques e melhorado continuamente |

Defina o **nível-alvo por controle**, de acordo com o risco (por exemplo, nível 4 para controles críticos de identidade e de dados e nível 3 para os demais).

## 7. Classificação da eficácia e efeito no risco

| Resultado | Critério | Efeito no risco residual (pasta 04) |
|---|---|---|
| Eficaz | Desenho adequado e operação consistente, com evidências suficientes | A redução de risco estimada é mantida |
| Parcialmente eficaz | Desvios ou cobertura incompleta, sem falha crítica | Redução parcial; o risco residual é elevado proporcionalmente |
| Ineficaz | Falha de desenho ou de operação que impede o objetivo | Sem redução de risco atribuível ao controle; risco residual igual ou próximo ao inerente |
| Não testado | Sem evidência de eficácia | Tratado como ineficaz até que seja testado, para fins de risco |

Os resultados dos testes **alimentam o registro de riscos**: um controle "documentado, mas não testado" não deve ser tratado como redução de risco.

## 8. Frequência e priorização de testes

| Tipo de controle | Frequência de teste (referência) |
|---|---|
| Críticos e automatizáveis | Contínua (verificação automática) |
| Críticos e manuais | Trimestral |
| Importantes | Semestral |
| Demais | Anual |
| Após mudanças relevantes | Imediata (teste dos controles afetados) |
| Controles detectivos (alertas) | Teste de ativação periódico ou por emulação |

## 9. Exemplo prático: programa de teste de controles de uma plataforma de pagamentos

| Atividade | Resultado |
|---|---|
| Inventário de controles | 138 controles; 61 técnicos |
| Testes automatizados contínuos | 52 controles técnicos verificados diariamente sobre a população de recursos |
| Testes manuais por amostragem | 38 controles de processo testados trimestralmente ou semestralmente |
| Emulação de técnicas em laboratório | 24 técnicas do ATT&CK relevantes emuladas ao longo do ano; 18 detectadas em até 10 minutos; 6 lacunas de detecção corrigidas |
| Equipe roxa | Duas rodadas por ano; 11 melhorias de detecção e de resposta |
| Resultados | 121 controles eficazes, 14 parcialmente eficazes, 3 ineficazes (planos de ação com prazo) |
| Efeito no risco | Riscos residuais de dois riscos altos foram revistos para cima por causa de controles parcialmente eficazes, justificando investimentos |

## 10. Casos de uso corporativos

- Programa de testes de controles com automação para os técnicos e amostragem para os manuais.
- Validação periódica das detecções com emulação de técnicas em laboratório.
- Relatórios de eficácia para a direção, para auditorias e para o registro de riscos.

## 11. Normas e frameworks relevantes

- NIST SP 800-53A Rev. 5 (avaliação de controles); NIST SP 800-115 (testes técnicos de segurança); NIST SP 800-137 (monitoramento contínuo)
- ISO/IEC 27001:2022 (9.1 monitoramento, medição, análise e avaliação; 9.2 auditoria interna); ISO/IEC 27007; ISO 19011
- MITRE ATT&CK for Cloud; MITRE Center for Threat-Informed Defense; ISACA ITAF; COBIT 2019 (modelo de capacidade)

## 12. Checklist operacional: teste de eficácia

- [ ] Eficácia de desenho e de operação avaliadas para os controles críticos
- [ ] Controles técnicos verificados por testes automatizados sobre a população completa, com resultados guardados como evidência
- [ ] Controles manuais testados por amostragem, com papéis de trabalho
- [ ] Validação por ataques em laboratório (emulação, equipe roxa e testes de intrusão), com autorização e isolamento
- [ ] Lacunas de prevenção, de detecção e de resposta registradas e corrigidas
- [ ] Maturidade e nível-alvo definidos por controle
- [ ] Classificação da eficácia integrada ao registro de riscos
- [ ] Frequência de testes definida por criticidade e após mudanças
- [ ] Relatório periódico de eficácia à direção

## 13. Referências

- Joint Task Force. NIST SP 800-53A Rev. 5: Assessing Security and Privacy Controls. 2022.
- Scarfone, K. et al. NIST SP 800-115: Technical Guide to Information Security Testing and Assessment. 2008.
- DataDog. Stratus Red Team (documentação); Red Canary. Atomic Red Team (documentação).
- MITRE. ATT&CK for Cloud; MITRE Center for Threat-Informed Defense (documentação).
- Pastas 04, 13, 16 e 29 deste repositório.
