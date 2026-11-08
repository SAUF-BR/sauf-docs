# ADR-SC-010 - Questões do ENEM no MVP vêm do dataset maritaca-ai/enem, cruzado com os microdados do INEP

- Status: aceito
- Decisores: time do SAUF-BR
- Data: 2026-11-08

## Contexto e Declaração do Problema

As telas de simulados precisam de um banco de questões. Hoje elas usam só mocks, e as questões não podem ir no bundle do front ([ADR-006](../front/ADR-006.md)). Na fonte oficial, o ENEM é distribuído em PDF por caderno, e extrair desses PDFs é difícil: as páginas têm duas colunas, as fórmulas e frações saem quebradas, as tabelas se desmancham e algumas alternativas são só figura (teste de 07/11/2026 com o caderno 5 do 2º dia de 2024).

De onde vêm as questões do ENEM no MVP?

## Fatores da Decisão

- Gabarito confiável, conferido com o oficial;
- Pouco trabalho manual para ter um volume suficiente para o MVP;
- Licença que permita usar os dados no projeto;
- Metadados para filtros e acompanhamento (área, habilidade, dificuldade);
- Não depender de um serviço de terceiros no ar.

## Opções Consideradas

- Opção 1: dataset [maritaca-ai/enem](https://huggingface.co/datasets/maritaca-ai/enem) cruzado com o `ITENS_PROVA` dos microdados do INEP
- Opção 2: extrair dos PDFs do INEP
- Opção 3: texto DOSVOX (leitor de tela) que vem nos microdados de 2023
- Opção 4: API comunitária [enem.dev](https://enem.dev)

## Resultado da Decisão

Opção escolhida: "Opção 1". Foi a única que deu volume e gabarito conferido sem extrair PDF.

- **Fonte:** `maritaca-ai/enem` (Apache-2.0), provas de 2022, 2023 e 2024: 540 questões, as 180 de cada ano.
- **Cruzamento:** a numeração do dataset é a do caderno laranja (acessível). Pela posição nesse caderno, cada questão se liga ao `CO_ITEM` do `ITENS_PROVA_<ano>.csv` dos microdados do ENEM (INEP). Dali vêm a área, a habilidade da matriz de referência, a dificuldade (parâmetros TRI) e o gabarito oficial, que confirma o `label` do dataset.
- **Imagens:** as figuras (PNG no GitHub do autor do dataset) são copiadas para o nosso Storage. A questão não depende do repositório deles.
- **Fórmulas por extenso:** o texto vem do caderno ledor, então fórmulas aparecem escritas como lidas em voz alta ("vinte e quatro vinte e cinco avos…"). Essas questões (pelo menos 35) e as anuladas entram como pendentes de revisão e só aparecem nos simulados depois de revisadas.
- **O importador fica no `sauf-scraper`**, no mesmo padrão dos importadores gerais ([ADR-SC-007](ADR-SC-007.md)): lê os arquivos baixados e envia para a API por uma rota de ingestão ([ADR-BE-008](../backend/ADR-BE-008.md)).

Decididas junto:

- **O front mostra imagem na questão desde a v1.** 145 das 540 questões dependem de figura, a maioria em Natureza e Matemática.
- **No ENEM, o filtro de categorias é por área:** Linguagens, Humanas, Natureza e Matemática. O ENEM não separa Física, Química e Biologia, nem História e Geografia. Separar por matéria fica para depois.

Adiado:

- O que cada plano de assinatura libera (P-14 do `DECISOES.md`).

### Consequências Positivas

- Gabarito conferido 100% com o `ITENS_PROVA` nos três anos;
- 540 questões sem extrair PDF, o suficiente para o MVP;
- Habilidade e dificuldade (TRI) sem trabalho extra, úteis para os filtros e para o "onde focar" do acompanhamento;
- Nenhuma dependência de serviço de terceiros em tempo de uso.

### Consequências Negativas

- Não há as questões de espanhol (1 a 5): o dataset só traz as de inglês;
- Texto-base e comando vêm juntos no campo `question`. A separação é por heurística, com revisão;
- Algumas tabelas vêm com sobras da conversão ("Unnamed: 2", "nan") e precisam de revisão;
- O dataset não deve ganhar 2025: as provas de 2025 em diante entram pelo PDF do INEP, com outro importador;
- Licenças e citações. A Apache-2.0 do dataset cobre o trabalho dos autores; o conteúdo das questões é do INEP e dos autores dos textos citados. Por isso:
  - cada questão guarda a fonte (ENEM <ano>, INEP);
  - a documentação cita os papers pedidos no README do dataset: arXiv [2311.14169](https://arxiv.org/abs/2311.14169) e [2303.17003](https://arxiv.org/abs/2303.17003).

## Prós e Contras das Opções

### Opção 1 - maritaca-ai/enem + ITENS_PROVA

- Bom, porque o gabarito bate com o oficial, a licença é aberta e as figuras vêm com descrição em texto;
- Ruim, porque cobre só 3 anos, sem espanhol, e as fórmulas vêm por extenso.

### Opção 2 - PDFs do INEP

- Bom, porque é a fonte oficial e cobre todos os anos;
- Ruim, porque exige revisão pesada: duas colunas, fórmulas quebradas, figuras e tabelas.

### Opção 3 - DOSVOX de 2023

- Bom, porque é texto limpo e oficial, com as figuras descritas;
- Ruim, porque só existe para um ano (2023).

### Opção 4 - enem.dev

- Bom, porque cobre 2009 a 2023 e já separa texto-base e comando;
- Ruim, porque não é oficial, não traz habilidade nem TRI, tem imagens quebradas e a licença dos dados é ambígua (repositório GPL-2.0).

## Links

- Dataset: https://huggingface.co/datasets/maritaca-ai/enem
- Microdados do ENEM (INEP): `https://download.inep.gov.br/microdados/microdados_enem_<ano>.zip` (`DADOS/ITENS_PROVA_<ano>.csv`)
- Fontes avaliadas: `sauf-scraper/docs/FONTES.md`, seção "Banco de questões (simulados): fontes avaliadas"
- Relaciona-se com: [ADR-SC-007](ADR-SC-007.md), [ADR-BE-008](../backend/ADR-BE-008.md), [ADR-006](../front/ADR-006.md)
