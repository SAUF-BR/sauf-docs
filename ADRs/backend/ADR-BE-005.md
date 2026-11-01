# ADR-BE-005 - Catálogo em dois níveis: curso genérico e oferta (uma página do site = uma oferta)

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-08

## Contexto e Declaração do Problema

O DER e o diagrama de classes antigos ligavam `Instituicao → Curso` direto. O protótipo, porém, mostra um card "Biomedicina" com "14 universidades ofertam na sua região", e ao abrir o card aparece a lista de universidades. Além disso, uma mesma página do site da UEM (ex.: Educação Física) cobre bacharelado e licenciatura em turnos diferentes, e o Censo tem um código e-MEC para cada combinação.

Como modelar o curso que o estudante procura, o curso que cada universidade oferece e as variações de grau e turno?

## Fatores da Decisão

- Fidelidade ao protótipo (catálogo agrupado e lista de universidades por curso);
- Visão geral para o usuário, sem repetir cursos iguais;
- Possibilidade de filtrar por grau e turno;
- Ligação com o Censo para área, vagas e gratuidade ([ADR-BE-009](ADR-BE-009.md)).

## Opções Consideradas

- Opção 1: Um nível só (`Instituicao → Curso`), como no DER antigo
- Opção 2: Dois níveis, com uma oferta por código e-MEC (D-32: Física vira duas ofertas, licenciatura e bacharelado)
- Opção 3: Dois níveis, com uma oferta por página do site e as variações numa lista de opções (D-34)

## Resultado da Decisão

Opção escolhida: "Opção 3 - Dois níveis, uma oferta por página". Pedido do Carlos (08/10/2026): *"é totalmente desnecessário listar 4 cursos IGUAIS, queremos dar uma visão geral sobre o curso ofertado pela UEM"*.

- **`curso`** é o curso genérico do catálogo (ex.: "Biomedicina", área "Saúde e bem-estar"), definido pelo Censo ([ADR-BE-006](ADR-BE-006.md));
- **`oferta_curso`** é o curso numa universidade, uma linha por página do site, com turno, duração, cidade, link (`url_origem`), `sobre` e `mercado_trabalho`;
- **`oferta_opcao`** guarda as opções que a página mostra, cada uma com `grau` e `turno` (ex.: licenciatura "Integral ou Noturno"; bacharelado "Integral");
- **`oferta_codigo_emec`** guarda, para uso interno, os códigos e-MEC da página ([ADR-BE-009](ADR-BE-009.md)).

Na API, `GET /cursos` lista os cursos genéricos (com `graus`, `totalUniversidades` e duração mínima/máxima) e `GET /cursos/{id}/ofertas` lista as ofertas, cada uma com `opcoes: [{grau, turno}]`. A D-32, de um curso por código e-MEC, valeu só no mesmo dia e foi substituída.

### Consequências Positivas

- O catálogo bate com o protótipo;
- O usuário vê cada curso de uma universidade uma vez só, com as opções e o link oficial;
- Filtros por grau e turno funcionam sobre a lista de opções.

### Consequências Negativas

- O front precisa trocar o `grau` único por `graus` (card) e `opcoes` (oferta), e separar curso genérico de oferta no tipo `Curso` (item 2 e 3 dos alinhamentos do `DECISOES.md`);
- O grau `bacharelado_e_licenciatura` do Censo deixa de ser usado nas ofertas (P-04);
- O DER e o diagrama de classes da documentação precisam ser atualizados (P-08). Material de apoio: `docs/der-banco-atual.md`.

## Prós e Contras das Opções

### Opção 1 - Um nível

- Bom, porque é o modelo do DER original;
- Ruim, porque não agrupa o mesmo curso entre universidades, como o protótipo pede.

### Opção 2 - Uma oferta por código e-MEC

- Bom, porque cada oferta casa 1:1 com o Censo;
- Ruim, porque a mesma página do site vira várias ofertas quase iguais.

### Opção 3 - Uma oferta por página, com opções

- Bom, porque mostra o curso como a universidade o apresenta;
- Bom, porque mantém grau e turno filtráveis;
- Ruim, porque a ligação com o Censo passa a ser de 1 oferta para N códigos.

## Links

- `sauf-api/docs/DECISOES.md`: D-17, D-32, D-34; P-04, P-08
- Relaciona-se com: [ADR-BE-006](ADR-BE-006.md), [ADR-BE-009](ADR-BE-009.md)
- Relaciona-se com: [ADR-SC-006](../scraper/ADR-SC-006.md) (tabela de casamento página × códigos)
