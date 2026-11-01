# ADR-SC-004 - Coleta educada, dados oficiais do Censo e escopo restrito às IES com spider

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-08

## Contexto e Declaração do Problema

O scraper acessa sites públicos de universidades e portais do governo. O e-MEC respondeu **403** e o gov.br devolveu **CAPTCHA** para acessos automáticos. O Censo da Educação Superior do INEP é um dado aberto, oficial e com código e-MEC para cada instituição e curso, mas cobre o país inteiro. A meta da apresentação é ter poucas IES com dados completos, e não muitas IES com poucos dados.

Como o scraper se comporta com os sites, de onde vem a lista de instituições e qual é o escopo?

## Fatores da Decisão

- Respeito aos sites e aos termos de uso (o projeto não pode parecer um ataque);
- Uso de dados oficiais e auditáveis;
- Qualidade da demonstração acima de quantidade;
- Nada de contornar bloqueios.

## Opções Consideradas

- Opção 1: Coleta educada (robots.txt, User-Agent próprio, intervalo entre requisições), sem contornar bloqueios; instituições vindas do Censo, só as que têm spider
- Opção 2: Importar todas as IES do Censo, com ou sem spider
- Opção 3: Coletar e-MEC e gov.br direto, contornando bloqueios quando preciso

## Resultado da Decisão

Opção escolhida: "Opção 1".

- **`HttpEducado`** (`client/http.py`) é o único cliente HTTP dos spiders:
  - lê e respeita o **robots.txt** de cada domínio;
  - identifica o bot com o **User-Agent `SAUF-BR-Bot/0.1`**;
  - espera **`request_delay_seconds` (padrão 2 s)** entre requisições ao mesmo domínio;
- **Nunca contornar bloqueio**: 403, CAPTCHA ou login encerram a coleta daquela fonte. É por isso que o e-MEC e o gov.br viram importação de arquivo baixado à mão ([ADR-SC-007](ADR-SC-007.md));
- **Instituições vêm do Censo (INEP)** (D-22), pelo comando `sauf-scraper censo`;
- **Escopo = IES com spider** (D-43): o Censo e os importadores são filtrados por `codigos_emec()` (`spiders/__init__.py`), que lista o código e-MEC de cada spider. A meta é cerca de **5 IES completas**;
- Antes de criar um spider, avaliar o site (robots.txt, estrutura, campos disponíveis, LGPD), como em `11-avaliacao-ies.md`.

### Consequências Positivas

- Baixo risco de bloqueio e de reclamação das universidades;
- A lista de IES é oficial e tem chave estável (código e-MEC);
- Cada IES do catálogo tem dados de verdade, não só nome.

### Consequências Negativas

- A coleta é lenta (2 s por página);
- Uma IES só aparece no catálogo depois de ter spider;
- Os critérios para escolher as próximas IES ainda não estão escritos (ver "Questões em aberto" no [README](../README.md)).

## Prós e Contras das Opções

### Opção 1 - Coleta educada, escopo pelas IES com spider

- Bom, porque é ética, estável e focada na qualidade;
- Ruim, porque o catálogo cresce devagar.

### Opção 2 - Todas as IES do Censo

- Bom, porque o catálogo fica grande logo;
- Ruim, porque a maioria das IES ficaria sem cursos, textos e links.

### Opção 3 - Contornar bloqueios

- Bom, porque daria acesso a mais dados;
- Ruim, porque viola os termos dos sites, é frágil e expõe o projeto.

## Links

- `sauf-scraper/src/sauf_scraper/client/http.py`, `sauf-scraper/src/sauf_scraper/core/config.py`
- `sauf-api/docs/DECISOES.md`: D-22, D-43
- Pasta do projeto: `scraper/06-plano-scraper-geral.md`, `scraper/11-avaliacao-ies.md`
- Relaciona-se com: [ADR-BE-009](../backend/ADR-BE-009.md), [ADR-SC-007](ADR-SC-007.md), [ADR-SC-009](ADR-SC-009.md)
