# Dockerfile SATD LLM Replication

Este repositório contém o pacote de replicação de um estudo empírico sobre **Self-Admitted Technical Debt (SATD)** em comentários de Dockerfiles. O trabalho replica parcialmente o estudo original de Azuma et al., utilizando modelos de linguagem de grande porte para classificar automaticamente comentários previamente classificados manualmente pelos autores.

## Descrição

A dívida técnica autoadmitida, ou **Self-Admitted Technical Debt (SATD)**, ocorre quando desenvolvedores registram explicitamente em comentários de código limitações, soluções temporárias, problemas conhecidos, melhorias pendentes ou decisões técnicas não ideais.

O estudo original, conduzido por **Hideaki Azuma, Shinsuke Matsumoto, Yasutaka Kamei e Shinji Kusumoto**, investigou SATD em Dockerfiles por meio da classificação manual de comentários extraídos de repositórios associados a imagens populares do Docker Hub.

Nesta replicação, os comentários classificados manualmente no estudo original foram submetidos a modelos de linguagem de grande porte. As classificações produzidas pelos modelos foram comparadas com a classificação manual de referência, com o objetivo de analisar concordância, diferenças entre modelos e variações entre classes e subclasses de SATD.

## Estudo original replicado

Este pacote de replicação é baseado no seguinte artigo:

> Azuma, H., Matsumoto, S., Kamei, Y., & Kusumoto, S. (2022). *An empirical study on self-admitted technical debt in Dockerfiles*. Empirical Software Engineering, 27, 49. https://doi.org/10.1007/s10664-021-10081-7

## Objetivo da replicação

O objetivo geral desta replicação é avaliar em que medida modelos de linguagem de grande porte se aproximam das classificações manuais realizadas no estudo original sobre SATD em Dockerfiles.

Os objetivos específicos são:

- utilizar o dataset disponibilizado pelos autores do estudo original;
- considerar apenas os comentários classificados manualmente;
- aplicar modelos das famílias GPT, Gemini e Claude à classificação dos comentários;
- comparar as classificações geradas com a classificação manual de referência;
- identificar qual modelo apresenta maior concordância com a classificação de referência;
- observar quais classes, subclasses ou comentários apresentam maior variação entre os modelos.

## Questões de pesquisa

Este estudo foi guiado pelas seguintes questões de pesquisa:

**RQ1:** Qual é o grau de concordância entre as classificações produzidas por modelos de linguagem e as classificações manuais do estudo original?

**RQ2:** Qual modelo de linguagem apresenta maior concordância com a classificação manual de referência do estudo original?

**RQ3:** Quais classes, subclasses ou comentários apresentam maior variação entre as classificações produzidas pelos modelos de linguagem?

## Dataset

Foi utilizada uma versão filtrada do dataset original disponibilizado por Azuma et al. O conjunto final utilizado na replicação contém **382 comentários classificados manualmente**.

Durante a preparação dos dados, foram removidos:

- comentários não amostrados;
- comentários duplicados;
- comentários de licença;
- comentários autogerados.

As principais colunas utilizadas foram:

| Coluna | Descrição |
|---|---|
| `id` | Identificador do comentário |
| `comment` | Comentário do Dockerfile a ser classificado |
| `below` | Código ou texto imediatamente abaixo do comentário, usado apenas como contexto auxiliar |
| `class` | Classificação manual de referência usada apenas na etapa de comparação |

Para evitar vazamento de rótulos, a coluna de classificação manual e qualquer outra coluna relacionada ao rótulo de referência não foram fornecidas aos modelos durante a classificação automática.

## Modelos avaliados

Foram avaliados modelos das famílias GPT, Gemini e Claude:

- GPT 4o-mini
- GPT 5.5
- Gemini 3.1 Pro
- Gemini 3.5 Flash (Descartado)
- Claude Sonnet 4.6 (Descartado)
- Claude Opus 4.8

As saídas que apresentaram inconsistências persistentes nos identificadores dos comentários foram descartadas da análise final, pois não foi possível alinhá-las de forma confiável ao dataset de referência.

## Prompt utilizado

O prompt utilizado para orientar os modelos deve ser inserido abaixo.

```text
You are a classifier of Dockerfile comments for an empirical study on Self-Admitted Technical Debt (SATD).

I will provide an Excel file named SATDs_Dockerfile_manual_filtrado.xlsx.

Your task is to read the dataset and classify all 382 Dockerfile comments contained in the file.

Use the following columns as input:

* id: the comment identifier.
* comment: the Dockerfile comment to be classified.
* below: the code or text immediately below the comment, provided only as contextual information.

Do not use any column that contains the original classification, label, class, subclass, or gold standard. These columns must be ignored during classification.

A comment must receive exactly one final classification:

1. SATD
2. Non-debt
3. Unidentifiable

Definition of SATD:
SATD is technical debt explicitly acknowledged by the developer in a source code comment. The comment indicates a temporary solution, incomplete implementation, poor or non-ideal implementation, postponed improvement, known limitation, risk, workaround, missing functionality, known bug, future bug, testing issue, deployment issue, or another technical problem intentionally left in the code.

Definition of Non-debt:
Use Non-debt when the comment only explains the code, documents a step, describes a configuration, describes expected behavior, gives usage information, or provides general information without indicating a problem, limitation, workaround, future improvement, technical debt, or pending technical task.

Definition of Unidentifiable:
Use Unidentifiable only when the comment is too ambiguous to determine whether it is SATD or Non-debt.

If the comment is SATD, assign exactly one class and exactly one subclass from the taxonomy below.

Allowed classes and subclasses:

1. Code/Workaround
   Use when the comment indicates a temporary, non-ideal, improvised, or suboptimal implementation that should be improved later, but is not directly related to a bug in an external system.

2. Code/MissingFunctionality
   Use when the comment indicates missing functionality inside the container or a desired functionality that has not yet been implemented.

3. Code/BaseImage
   Use when the comment refers to problems, limitations, bugs, changes, or desired changes related to the Docker base image.

4. Code/Version
   Use when the comment refers to pinning, changing, updating, locking, upgrading, downgrading, or controlling versions of packages, tools, frameworks, libraries, or dependencies.

5. Test/IntegrityCheck
   Use when the comment indicates a missing or needed integrity check, signature verification, hash verification, checksum, SHA, PGP, GPG, or validation of downloaded files.

6. Test/ImprovementForTest
   Use when the comment indicates a needed improvement in tests, testing environment, test organization, test efficiency, or test maintainability.

7. Defect/Workaround
   Use when the comment indicates a workaround for an existing bug, error, failure, or broken behavior in an external dependency, tool, system, package, or component.

8. Defect/LatentBug
   Use when the comment indicates a future bug, future breakage, deprecation, incompatibility, or risk caused by future changes.

9. Design/SizeReduction
   Use when the comment indicates a need to reduce Docker image size, remove unnecessary files, remove redundant binaries, reduce layers, clean artifacts, or optimize disk space.

10. Process/Deployment
    Use when the comment indicates a problem, limitation, warning, or pending task related to deployment, production usage, production configuration, or runtime environment.

11. Process/Review
    Use when the comment asks for review, investigation, confirmation, clarification, or later analysis of the Dockerfile or Docker image itself.

12. Unclassifiable
    Use when the comment is clearly SATD, but does not fit safely into any of the subclasses above.

Important rules:

* Classify primarily based on the comment column.
* Use the below column only as auxiliary context when needed.
* Classify the comment, not the below field.
* Do not classify something as SATD only because the below field contains problematic code.
* The final classification must be justified by the comment itself, with the below field used only to resolve ambiguity.
* Do not use external context.
* Do not infer information that is not present in the comment or auxiliary below context.
* Do not use the original article, dataset labels, original classes, or any external source.
* Each comment must receive only one final class and one final subclass.
* If a comment appears to fit two or more subclasses, choose only one according to the priority order below.
* Keywords such as TODO, FIXME, hack, workaround, temporary, should, need to, remove later, deprecated, bug, broken, not ideal, or similar expressions are indicators of possible SATD, but they are not sufficient by themselves. Analyze the meaning of the comment.

Priority order for overlapping subclasses:

1. Code/Workaround
2. Code/MissingFunctionality
3. Code/BaseImage
4. Code/Version
5. Test/IntegrityCheck
6. Test/ImprovementForTest
7. Defect/Workaround
8. Defect/LatentBug
9. Design/SizeReduction
10. Process/Deployment
11. Process/Review
12. Unclassifiable

Examples of priority rule:

* If a comment could be classified as both Code/Version and Test/IntegrityCheck, choose Code/Version.
* If a comment could be classified as both Code/Workaround and Defect/Workaround, choose Code/Workaround unless the comment clearly refers to a bug or failure in an external system.
* If a comment could be classified as both Test/ImprovementForTest and Design/SizeReduction, choose Test/ImprovementForTest.

Output format:
Return only CSV. Do not use Markdown. Do not add explanations outside the CSV.

The CSV must have exactly these columns:

id,classification,class,subclass,confidence,justification

Column rules:

* id: the comment identifier provided in the dataset.
* classification: SATD, Non-debt, or Unidentifiable.
* class: Code, Test, Defect, Design, Process, Unclassifiable, or N/A.
* subclass: one of the allowed subclasses or N/A.
* confidence: High, Medium, or Low.
* justification: one short sentence explaining the decision.

If classification is Non-debt:

* class must be N/A.
* subclass must be N/A.

If classification is Unidentifiable:

* class must be N/A.
* subclass must be N/A.

Execution instructions:

* Classify all 382 comments from the dataset.
* Do not stop after a small sample.
* Do not ask me to paste the rows manually.
* Send the results in sequential parts of 30 rows per message until all 382 comments have been classified.
* Each part must continue from the previous one without repeating already classified comments.
* Each output message must contain only CSV rows.
* Include the CSV header only in the first output message.
* Preserve the original id values exactly as they appear in the dataset.
* Return exactly one output row for each dataset row.
* Do not skip any comment.
* Do not merge comments.
* Continue sending parts of 30 rows until the classification is complete.
````

## Metodologia

A metodologia foi organizada em três etapas principais.

### 1. Preparação do dataset

A base original foi filtrada para manter apenas os comentários classificados manualmente pelos autores do estudo original. Comentários não amostrados, duplicados, de licença ou autogerados foram removidos.

### 2. Classificação automática

Os comentários foram enviados aos modelos por meio de prompts padronizados. Todos os modelos receberam as mesmas instruções, contendo:

* definição de SATD;
* definição de `Non-debt`;
* definição de `Unidentifiable`;
* taxonomia de classes e subclasses;
* regras para lidar com ambiguidades;
* orientação para usar `comment` como entrada principal;
* orientação para usar `below` apenas como contexto auxiliar.

Cada comentário deveria receber:

* uma classificação geral: `SATD`, `Non-debt` ou `Unidentifiable`;
* uma classe principal, quando classificado como SATD;
* uma subclasse, quando classificado como SATD.

### 3. Comparação das classificações

As classificações produzidas pelos modelos foram comparadas com a classificação manual de referência do estudo original. A análise considerou:

* concordância geral;
* comparação entre modelos;
* divergências entre classes e subclasses;
* comentários com maior variação classificatória.

## Autores da replicação

* Artur Pigari Prata
* Letícia Cristina da Silva
* Marlus Zabla Lipsuk

## Referência

Azuma, H., Matsumoto, S., Kamei, Y., & Kusumoto, S. (2022). *An empirical study on self-admitted technical debt in Dockerfiles*. Empirical Software Engineering, 27, 49. https://doi.org/10.1007/s10664-021-10081-7
