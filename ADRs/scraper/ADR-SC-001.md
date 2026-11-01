# ADR-SC-001 - Stack do scraper: Python 3.12, uv, Requests + BeautifulSoup e Pydantic

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-05

## Contexto e Declaração do Problema

O SAUF-BR precisa coletar dados de cursos dos sites das universidades e ler planilhas oficiais (Censo, IGC, Sisu). O escopo inicial é pequeno (cerca de 5 IES públicas, [ADR-SC-004](ADR-SC-004.md)), as páginas são HTML estático na maioria e o time precisa entender e explicar o código na banca.

Qual linguagem, gerenciador de dependências e bibliotecas usar no scraper?

## Fatores da Decisão

- Poucas fontes, quase todas HTML estático;
- Código simples de ler e de testar;
- Validação forte da saída antes de enviar para a API;
- Ambiente reproduzível em qualquer máquina do time.

## Opções Consideradas

- Opção 1: Python 3.12 + `uv` + Requests + BeautifulSoup + Pydantic (Playwright só se uma fonte exigir JS)
- Opção 2: Scrapy
- Opção 3: Selenium/Playwright para todas as fontes

## Resultado da Decisão

Opção escolhida: "Opção 1".

- **Python 3.12** com **`uv`**: `pyproject.toml` e `uv.lock` no Git;
- **Requests** para baixar e **BeautifulSoup 4** para ler o HTML. Antes de escrever um parser, conferir no DevTools se o site tem uma API JSON;
- **Pydantic** valida toda saída de parser; **pydantic-settings** lê a configuração (`core/config.py`);
- **openpyxl** para as planilhas `.xlsx` (IGC e Sisu, [ADR-SC-007](ADR-SC-007.md));
- Desenvolvimento: **pytest**, **responses** (mock de HTTP) e **ruff**;
- **Sem Scrapy e sem Selenium** por enquanto. Playwright só entra quando a primeira fonte depender de JavaScript.

### Consequências Positivas

- Poucas dependências, cada uma com um papel claro;
- Os parsers são funções simples, testáveis com HTML salvo ([ADR-SC-003](ADR-SC-003.md));
- O contrato do lote é validado no scraper antes do POST.

### Consequências Negativas

- Agendamento, fila e retentativa ficam por nossa conta (o Scrapy já traria);
- Um site que dependa de JS exige adicionar o Playwright depois.

## Prós e Contras das Opções

### Opção 1 - Requests + BS4 + Pydantic

- Bom, porque é leve, explícito e fácil de explicar;
- Ruim, porque não traz controle de crawl pronto.

### Opção 2 - Scrapy

- Bom, porque já traz fila, retentativa e pipelines;
- Ruim, porque é um framework grande demais para 5 IES e esconde o fluxo.

### Opção 3 - Navegador automatizado para tudo

- Bom, porque funciona com sites que usam JS;
- Ruim, porque é lento, frágil e desnecessário para HTML estático.

## Links

- `sauf-scraper/pyproject.toml`
- Pasta do projeto: `scraper/01-analise-e-proposta.md`
- Relaciona-se com: [ADR-BE-001](../backend/ADR-BE-001.md) (stack do backend)
