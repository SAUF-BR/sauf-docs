# ADR-BE-011 - Nota MEC e notas de corte guardadas por ano e calculadas no backend

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-17

## Contexto e Declaração do Problema

O protótipo mostra a "nota MEC" da universidade, o corte do Sisu de cada oferta e uma "Média do corte SISU" no card do curso genérico. As fontes oficiais são o IGC do INEP (por instituição e por ano) e as planilhas de inscrições e notas de corte do Sisu (por curso, turno, modalidade de concorrência e edição). Uma oferta pode ter vários códigos e turnos, e a lista de ofertas no front é paginada.

Onde guardar esses indicadores, qual valor mostrar e quem calcula a média?

## Fatores da Decisão

- Um número comparável por oferta e por universidade;
- Histórico por ano, sem apagar anos anteriores;
- Cálculo correto mesmo com a lista paginada no front;
- O rótulo deve dizer o ano do dado, nunca um ano fixo.

## Opções Consideradas

- Opção 1: Guardar por ano em tabelas próprias e expor o valor do ano mais recente, calculado no backend
- Opção 2: Guardar só o valor atual na oferta e na instituição
- Opção 3: Mandar as notas cruas para o front calcular menor corte e média

## Resultado da Decisão

Opção escolhida: "Opção 1".

**Nota MEC (D-49):**
- É a **faixa do IGC (1 a 5)** da instituição, guardada por ano em `indicador_ies` (com o IGC contínuo);
- A API mostra em `notaMec` a faixa do ano mais recente importado. `'SC'` (sem conceito) vira `null`.

**Notas de corte (D-45, D-50, D-53):**
- Só **ampla concorrência** (`TP_MOD_CONCORRENCIA = 'A'`). Corte `0` (poucos inscritos) vira `null`;
- Guardadas em `nota_corte_sisu` por (código e-MEC do curso, turno, ano);
- **Oferta**: `notaCorteSisu` = o **menor** corte entre os códigos e turnos da oferta, com `notaCorteSisuTurno` e `notaCorteSisuAno`;
- **Curso genérico**: `notaCorteSisuMedia` = **média simples** das ofertas com nota, cada par (código, turno) contando uma vez, arredondada em 2 casas, com `notaCorteSisuAno`. Substituiu o antigo `notaCorteSisuMin`;
- Sempre o **ano mais recente** que tem nota (hoje, Sisu 2025). Sem nota: `null`, e o front mostra "Ingresso por vestibular próprio".

### Consequências Positivas

- Uma regra só para todas as telas, testada no backend;
- O histórico fica guardado para uma evolução futura;
- A média não depende de quantas ofertas o front carregou.

### Consequências Negativas

- Na UEM, o Sisu cobre só parte das vagas (cerca de 20%). A tela precisa dizer "Corte SISU", nunca "a nota de corte da UEM";
- Ofertas sem código e-MEC ficam sem nota de corte (15 códigos do Sisu da UEM ainda não estão na tabela de casamento);
- A coluna antiga `oferta_curso.nota_corte_sisu` ficou sem uso e só sai numa migration futura.

## Prós e Contras das Opções

### Opção 1 - Por ano, calculado no backend

- Bom, porque é correto e reaproveitável;
- Ruim, porque exige tabelas e consultas a mais.

### Opção 2 - Só o valor atual

- Bom, porque é simples;
- Ruim, porque perde o histórico e não resolve ofertas com vários códigos.

### Opção 3 - Cálculo no front

- Bom, porque a API fica mais simples;
- Ruim, porque a lista é paginada ("Ver as 14") e a média sairia errada; o filtro e a ordenação por nota também precisam do número no backend.

## Links

- `sauf-api/docs/DECISOES.md`: D-45, D-49, D-50, D-53
- Pasta do projeto: `scraper/07-fase-a-igc.md`, `scraper/08-fase-b-sisu.md`
- Relaciona-se com: [ADR-SC-007](../scraper/ADR-SC-007.md) (importadores de arquivo), [ADR-BE-009](ADR-BE-009.md)
