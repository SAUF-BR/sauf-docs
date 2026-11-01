# ADR-BE-007 - UUID como chave primária e chaves naturais à parte

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-06

## Contexto e Declaração do Problema

O modelo antigo usava `id int` gerado pelo banco. Os dados do SAUF, porém, chegam de fora (Censo, sites, Sisu, IGC) e são reenviados periodicamente: a ingestão precisa reconhecer "a mesma instituição" e "a mesma oferta" para atualizar em vez de duplicar. Ao mesmo tempo, a API expõe ids nas URLs (`/universidades/{id}`).

Que chave usar como identificador interno, e como reconhecer um registro que chega de novo pela ingestão?

## Fatores da Decisão

- Upsert idempotente: reenviar o mesmo lote não duplica nada;
- Não expor a quantidade de registros nem permitir "adivinhar" ids;
- Ids estáveis mesmo quando a fonte muda de código;
- Compatibilidade com o front, que já trata ids como `string`.

## Opções Consideradas

- Opção 1: Id sequencial (`bigint`) como única chave
- Opção 2: Chave natural como chave primária (ex.: código e-MEC)
- Opção 3: UUID como chave primária, com chaves naturais em colunas únicas à parte

## Resultado da Decisão

Opção escolhida: "Opção 3 - UUID + chaves naturais".

- Todas as tabelas usam **UUID** como PK (D-19);
- **Instituição**: `codigo_emec` único;
- **Oferta**: `chave` única por instituição, gerada pelo scraper com `montar_chave_curso()` (ex.: `uem:engenharia-de-software:maringa:noturno`), sem acento e em minúsculas ([ADR-SC-005](../scraper/ADR-SC-005.md)). O hash da URL do site não serve, porque muda se o site for refeito;
- **Notas e indicadores**: chaves compostas pelos campos de negócio (ex.: código e-MEC do curso + turno + ano na `nota_corte_sisu`).

### Consequências Positivas

- Reenviar Censo, lote, IGC ou Sisu só atualiza o que mudou;
- Ids nas URLs não revelam volume nem ordem;
- Uma oferta sem código e-MEC (curso novo) ainda tem identidade, pela chave.

### Consequências Negativas

- UUID ocupa mais espaço e é menos legível nos logs e no Table Editor;
- Se a universidade renomear um curso, a chave muda: a oferta antiga fica inativa ([ADR-BE-008](ADR-BE-008.md)) e uma nova é criada;
- O front não pode montar URLs a partir de nomes; sempre precisa do id da API.

## Prós e Contras das Opções

### Opção 1 - Id sequencial

- Bom, porque é simples e compacto;
- Ruim, porque não resolve o upsert sozinho e expõe a contagem.

### Opção 2 - Chave natural como PK

- Bom, porque o upsert fica direto;
- Ruim, porque oferta nova pode não ter código e-MEC, e mudar a chave quebraria as FKs.

### Opção 3 - UUID + chaves naturais

- Bom, porque separa identidade interna de identidade de negócio;
- Ruim, porque exige manter as restrições de unicidade à parte.

## Links

- `sauf-api/docs/DECISOES.md`: D-19, D-25
- Relaciona-se com: [ADR-BE-008](ADR-BE-008.md) (ingestão), [ADR-SC-005](../scraper/ADR-SC-005.md) (chave do scraper)
