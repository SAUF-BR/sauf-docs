# ADR-SC-005 - Chave natural do curso, urlOrigem obrigatório e codigosEmec como lista

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-08
- História Técnica: D-23, D-34, D-35 do `docs/DECISOES.md` do `sauf-api`; `scraper/01-analise-e-proposta.md` (seção 2.2); `scraper/03-scraper-x-api.md`

## Contexto e Declaração do Problema

A carga precisa ser idempotente: rodar o scraper duas vezes não pode duplicar cursos. O scraper não conhece os IDs do banco. Nem todo curso do site tem código e-MEC (Engenharia de Software da UEM é novo e não está no Censo 2024), e uma página pode corresponder a vários códigos (bacharelado e licenciatura, turnos diferentes).

Como identificar um curso entre execuções, e o que é obrigatório no lote?

## Fatores da Decisão

- Upsert idempotente no backend;
- A mesma página gera sempre a mesma chave;
- Aceitar cursos novos sem código e-MEC;
- Rastreabilidade: de onde veio cada dado.

## Opções Consideradas

- Opção 1: Chave natural gerada por `montar_chave_curso` a partir de textos da página; `urlOrigem` obrigatório; `codigosEmec` como lista, podendo ser vazia
- Opção 2: Código e-MEC como chave do curso
- Opção 3: ID gerado pelo banco, consultado pelo scraper antes de enviar

## Resultado da Decisão

Opção escolhida: "Opção 1".

- **Chave**: `montar_chave_curso("uem", nome, cidade, turno)` → `uem:engenharia-de-software:maringa:noturno`. Tira acentos, deixa minúsculo, troca o que não é letra ou número por `-` e junta as partes com `:`. Todo spider usa essa função;
- **Uma página do site = um curso no lote** (D-34), com a lista de **`opcoes`** (`grau` + `turno`) que a página mostra;
- **`urlOrigem` é obrigatório**: é o crédito do texto, o link "acessar o site" e a fonte da grade curricular ("ver grade no site"). Se a página redireciona, vale a URL final;
- **`codigosEmec` é uma lista** (D-35), preenchida pela tabela de casamento ([ADR-SC-006](ADR-SC-006.md)). Lista vazia é aceita (D-23); a oferta entra sem área, vagas e nota de corte;
- Enums em minúsculas, como no contrato público da API ([ADR-BE-010](../backend/ADR-BE-010.md));
- `fonte.coletadoEm` vai em todo lote.

### Consequências Positivas

- O backend faz upsert pela `chave` sem consultar nada antes;
- Cursos novos aparecem no catálogo;
- Toda oferta tem link para a página oficial.

### Consequências Negativas

- **Mudar o nome, a cidade ou o turno na página muda a chave**: a oferta antiga fica inativa e nasce uma nova;
- Ofertas sem código ficam sem dados do Censo até alguém revisar a tabela.

## Prós e Contras das Opções

### Opção 1 - Chave natural do site

- Bom, porque é determinística e não depende do Censo;
- Ruim, porque é sensível a mudanças de texto na página.

### Opção 2 - Código e-MEC como chave

- Bom, porque é oficial e estável;
- Ruim, porque cursos novos não têm código e uma página pode ter vários.

### Opção 3 - ID do banco

- Bom, porque não exige regra de chave;
- Ruim, porque acopla o scraper ao banco e contradiz a [ADR-SC-002](ADR-SC-002.md).

## Links

- `sauf-scraper/src/sauf_scraper/core/models.py` (`montar_chave_curso`)
- `sauf-api/docs/DECISOES.md`: D-23, D-34, D-35
- Relaciona-se com: [ADR-BE-005](../backend/ADR-BE-005.md), [ADR-BE-007](../backend/ADR-BE-007.md), [ADR-BE-008](../backend/ADR-BE-008.md), [ADR-SC-006](ADR-SC-006.md)
