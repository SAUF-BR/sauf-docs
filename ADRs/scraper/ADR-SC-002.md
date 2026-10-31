# ADR-SC-002 - Scraper como container separado que envia lotes para a API (com dry-run em JSON)

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-05
- História Técnica: `scraper/01-analise-e-proposta.md` (seções 1, 2.4 e 3); `scraper/03-scraper-x-api.md`

## Contexto e Declaração do Problema

Os dados coletados precisam chegar ao banco do SAUF-BR. O scraper poderia gravar direto no PostgreSQL ou passar pela API Java. O backend já é o único ponto de acesso ao banco ([ADR-BE-002](../backend/ADR-BE-002.md)) e o regulamento cobra controle transacional e integração de serviços. Além disso, o scraper começou antes do backend existir.

Como o scraper entrega os dados, e como desenvolver sem a API no ar?

## Fatores da Decisão

- Regra de negócio e transação num lugar só (o backend);
- Scraper "burro": só produz um JSON válido;
- Desenvolvimento do scraper independente do backend;
- Isolamento de falhas entre instituições.

## Opções Consideradas

- Opção 1: Scraper faz POST na API (`/api/v1/ingestao/...`) com `X-Api-Key`; `--dry-run` grava o JSON em arquivo
- Opção 2: Scraper grava direto no banco
- Opção 3: Scraper gera arquivos e a API importa por agendamento

## Resultado da Decisão

Opção escolhida: "Opção 1".

- O scraper é um **container do próprio SAUF** (Scraper Python → API Spring Boot → PostgreSQL), não um sistema externo. O diagrama C4 precisa mostrar isso;
- **Um lote por instituição por execução** em `POST /api/v1/ingestao/lotes`; o Censo vai para `POST /api/v1/ingestao/censo`; os importadores gerais têm rotas próprias ([ADR-SC-007](ADR-SC-007.md));
- Autenticação de máquina com o header **`X-Api-Key`**, nunca JWT de usuário ([ADR-BE-008](../backend/ADR-BE-008.md));
- O **backend é dono do upsert e da transação**; o scraper não conhece IDs do banco, só chaves naturais ([ADR-SC-005](ADR-SC-005.md));
- **`--dry-run`** grava o lote em `output/` em vez de enviar. **`--mostrar`** imprime o resultado. O JSON do dry-run também serve de mock para o front;
- O envio fica em `client/destinos.py` (POST ou arquivo).

### Consequências Positivas

- A regra de negócio fica no Java, testada lá;
- O scraper pode ser desenvolvido e testado sem a API;
- Trocar o banco ou o schema não quebra o scraper, só o contrato importa.

### Consequências Negativas

- Mudança no contrato exige mudar os dois lados juntos;
- A API precisa estar no ar para a carga real;
- A chave da API precisa ser guardada como segredo nos dois lados.

## Prós e Contras das Opções

### Opção 1 - POST na API

- Bom, porque centraliza regra e transação no backend;
- Ruim, porque acopla o scraper ao contrato da API.

### Opção 2 - Direto no banco

- Bom, porque é mais rápido de escrever no começo;
- Ruim, porque duplica regra de negócio em Python e contradiz a [ADR-BE-002](../backend/ADR-BE-002.md).

### Opção 3 - Arquivos importados pela API

- Bom, porque desacopla os horários;
- Ruim, porque exige armazenamento compartilhado e um agendador a mais.

## Links

- `sauf-scraper/src/sauf_scraper/client/destinos.py`
- Pasta do projeto: `scraper/01-analise-e-proposta.md`, `scraper/03-scraper-x-api.md`
- Relaciona-se com: [ADR-BE-002](../backend/ADR-BE-002.md), [ADR-BE-008](../backend/ADR-BE-008.md), [ADR-SC-003](ADR-SC-003.md)
