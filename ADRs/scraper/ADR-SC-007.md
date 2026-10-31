# ADR-SC-007 - Importadores gerais leem um arquivo oficial baixado à mão

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-16
- História Técnica: D-48, D-51 do `docs/DECISOES.md` do `sauf-api`; `scraper/06-plano-scraper-geral.md`, `scraper/07-fase-a-igc.md`, `scraper/08-fase-b-sisu.md`; commit TR-263

## Contexto e Declaração do Problema

Parte dos dados do protótipo não está no site da universidade: a nota MEC (IGC do INEP) e as notas de corte (planilhas do Sisu, do MEC). Essas fontes são planilhas oficiais publicadas uma ou duas vezes por ano no gov.br, que bloqueia robôs com CAPTCHA, e os endereços mudam (o antigo `inep.gov.br/...` nem resolve mais no DNS).

Como trazer esses dados para o sistema?

## Fatores da Decisão

- Não contornar bloqueios ([ADR-SC-004](ADR-SC-004.md));
- Estabilidade e auditoria: saber qual arquivo gerou o dado;
- Ligação direta com o que já temos (código e-MEC);
- Frequência baixa de atualização (anual).

## Opções Consideradas

- Opção 1: Importador de arquivo local (`sauf-scraper <fonte> --arquivo ...`), filtrado pelo escopo e casado pelo código e-MEC
- Opção 2: Spider que baixa a planilha do gov.br automaticamente
- Opção 3: Digitar os valores à mão no banco

## Resultado da Decisão

Opção escolhida: "Opção 1".

- Uma pessoa baixa a planilha oficial e roda, por exemplo, `sauf-scraper igc --arquivo IGC_2023.xlsx` ou `sauf-scraper sisu --arquivo ...`;
- Base comum em `geral/base.py`: lê o arquivo (openpyxl, com `strip` nos cabeçalhos), **filtra as IES do escopo** (`codigos_emec()`, D-43), **casa pelo código e-MEC** e registra no log as IES do escopo que não casaram;
- **IGC** (`geral/igc.py`) → `POST /api/v1/ingestao/igc`; **Sisu** (`geral/sisu.py`) → `POST /api/v1/ingestao/sisu`, ligado por `CO_IES` + `CO_IES_CURSO` + turno;
- `--dry-run` grava `output/<fonte>-<ano>.json`; sem ele, envia para a API;
- Regras de valor (faixa do IGC, só ampla concorrência, ano mais recente) ficam no backend ([ADR-BE-011](../backend/ADR-BE-011.md));
- **Fora do plano** (D-51): G4 (PROUni/FIES, porque as 5 IES são públicas) e G5 (e-MEC, porque o endereço vem do arquivo de IES do Censo, D-58).

### Consequências Positivas

- Nenhum acesso automático ao gov.br;
- O arquivo usado fica guardado e a importação pode ser repetida;
- O mesmo padrão serve para fontes futuras.

### Consequências Negativas

- Depende de alguém lembrar de baixar o arquivo novo a cada edição;
- Uma mudança de layout da planilha quebra o importador até ajustarmos.

## Prós e Contras das Opções

### Opção 1 - Arquivo local

- Bom, porque é estável, ético e auditável;
- Ruim, porque tem um passo manual.

### Opção 2 - Download automático

- Bom, porque não exige ninguém;
- Ruim, porque o gov.br bloqueia robôs e os links mudam.

### Opção 3 - Digitação manual

- Bom, porque não exige código;
- Ruim, porque é lenta, sujeita a erro e não tem histórico.

## Links

- `sauf-scraper/src/sauf_scraper/geral/` (`base.py`, `igc.py`, `sisu.py`)
- `sauf-api/docs/DECISOES.md`: D-48, D-51, D-58
- Pasta do projeto: `scraper/06-plano-scraper-geral.md`, `scraper/07-fase-a-igc.md`, `scraper/08-fase-b-sisu.md`
- Relaciona-se com: [ADR-BE-011](../backend/ADR-BE-011.md), [ADR-SC-004](ADR-SC-004.md)
