# ADR-SC-003 - Separação spider × parser × pipeline, parsers puros testados com fixtures e falha isolada por instituição

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-05
- História Técnica: `scraper/01-analise-e-proposta.md` (seção 3); `scraper/02-guia-spider-uem.md`; `sauf-scraper/docs/ESTRUTURA.md`

## Contexto e Declaração do Problema

Cada universidade tem um site diferente e cada site muda sem aviso. O time precisa adicionar uma IES nova sem mexer no resto, testar a leitura do HTML sem acessar a internet e não perder a carga da UEL só porque a UEM falhou. O regulamento também exige testes unitários e de integração.

Como organizar o código do scraper e como ele se comporta quando uma fonte falha?

## Fatores da Decisão

- Adicionar uma IES nova com um arquivo de spider e um de parser;
- Testes rápidos e determinísticos, sem rede;
- Uma falha não derruba as outras instituições;
- O agendador precisa saber se a execução falhou.

## Opções Consideradas

- Opção 1: Spider (baixa) × parser (HTML → dados, função pura) × pipeline (valida e envia), com fixtures de HTML e falha isolada por IES
- Opção 2: Um script por universidade, que baixa, lê e envia
- Opção 3: Testes contra o site real

## Resultado da Decisão

Opção escolhida: "Opção 1".

- **`spiders/`**: um spider por IES (ex.: `UemSpider`, `chave = "uem"`, `codigo_emec = 57`). Baixa as páginas com o `HttpEducado` ([ADR-SC-004](ADR-SC-004.md)), chama o parser e monta os `Curso`s;
- **`parsers/`**: funções puras, HTML → dados, sem rede e sem estado;
- **`core/`**: modelos Pydantic, configuração, logging e o **pipeline**, que valida o lote e entrega ao destino ([ADR-SC-002](ADR-SC-002.md));
- **Fixtures**: HTML real salvo e anonimizado ([ADR-SC-009](ADR-SC-009.md)) em `tests/`, versionado no Git. Os testes do parser usam só as fixtures; os de HTTP usam `responses`;
- **Falha isolada**: um lote por IES; se uma falha, as outras seguem e o erro vai para o log;
- **Código de saída**: o comando termina com **1** quando alguma IES falhou e **0** quando todas deram certo, para o agendador ou CI perceber.

### Consequências Positivas

- Adicionar uma IES é seguir o guia do spider (`02-guia-spider-uem.md`) e o `CHECKLIST_SCRAPERS.md`;
- Testes do parser rodam em milissegundos, sem internet;
- Quando o site muda, basta salvar a página nova como fixture e ajustar o parser.

### Consequências Negativas

- As fixtures envelhecem; um teste verde não garante que o site ainda é igual;
- Mais arquivos por IES do que num script único.

## Prós e Contras das Opções

### Opção 1 - Camadas separadas + fixtures

- Bom, porque é testável, previsível e fácil de dividir no time;
- Ruim, porque exige manter fixtures atualizadas.

### Opção 2 - Um script por IES

- Bom, porque é rápido de escrever;
- Ruim, porque mistura rede, leitura e envio e fica difícil de testar.

### Opção 3 - Testes contra o site real

- Bom, porque detecta mudança no site;
- Ruim, porque é lento, instável e sobrecarrega o site da universidade.

## Links

- `sauf-scraper/docs/ESTRUTURA.md`, `sauf-scraper/docs/CHECKLIST_SCRAPERS.md`, `sauf-scraper/CONTRIBUTING.md`
- Pasta do projeto: `scraper/02-guia-spider-uem.md`
- Relaciona-se com: [ADR-SC-002](ADR-SC-002.md), [ADR-SC-004](ADR-SC-004.md), [ADR-SC-009](ADR-SC-009.md)
