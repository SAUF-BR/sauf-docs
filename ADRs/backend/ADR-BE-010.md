# ADR-BE-010 - Contrato REST da API: /api/v1, formato da ADR-010, enums em minúsculas e campos sem fonte como null

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-09
- História Técnica: D-13, D-14, D-15, D-16, D-21, D-30 do `docs/DECISOES.md` do `sauf-api`; P-11, P-19

## Contexto e Declaração do Problema

O front começou com mocks e a [ADR-010](../front/ADR-010.md) propôs um contrato de erro, paginação e datas. O backend precisava implementar as rotas reais sem quebrar o que o front já fazia, e o protótipo tem campos que ainda não têm fonte de dados (mensalidade, imagens, logo). O Carlos também notou que o Swagger mostrava filtros que não funcionavam, como o de nota de corte.

Qual contrato a API segue, e o que fazer com campos e filtros que ainda não têm dado?

## Fatores da Decisão

- Tratamento centralizado de erros e paginação no front ([ADR-007](../front/ADR-007.md));
- Evolução do contrato sem quebrar o front;
- Honestidade do contrato: nada que pareça funcionar sem funcionar;
- Compatibilidade com os mocks do front.

## Opções Consideradas

- Opção 1: Seguir a ADR-010 do front, com prefixo de versão, enums em minúsculas, campos sem fonte como `null` documentados e filtros sem dado removidos
- Opção 2: Formato padrão do Spring (`Page` serializado direto e erro padrão do Spring Boot)
- Opção 3: Só expor campos que já têm dado

## Resultado da Decisão

Opção escolhida: "Opção 1".

- **Prefixo `/api/v1/...`** em todas as rotas (D-13). O front precisa trocar `VITE_API_URL` para terminar em `/api/v1`;
- **`/universidades`** na API, `instituicao` no banco (D-14);
- **Erro** `{ status, error, message, fields }` (`ErroResposta`, gerado pelo `TratadorDeErros`), **paginação** `{ content, page, size, totalElements, totalPages }` (`Pagina<T>`, `page` a partir de 0, `size` padrão 12 e máximo 50) e **datas** ISO 8601 UTC (D-15);
- **Enums em minúsculas no JSON** e nos filtros (`presencial`, `ead`, `bacharelado`, `tecnologo`, `publica`), maiúsculas no banco (D-30);
- **Swagger** em `/api/v1/swagger` e OpenAPI em `/api/v1/docs` (D-16);
- **Campos sem fonte** (`mensalidade`, `imagemUrl`, `logoUrl`) continuam no contrato, vindo `null`, com a descrição "Ainda sem fonte de dados; vem null (P-11)" no Swagger (D-21). Nota MEC e notas de corte já têm fonte ([ADR-BE-011](ADR-BE-011.md));
- **Filtros sem dado saem da API**: o `notaCorteMax` de `GET /cursos` foi removido (09/10/2026), porque devolvia lista vazia. Volta quando fizer sentido;
- Códigos e-MEC não aparecem na API pública ([ADR-BE-009](ADR-BE-009.md)).

### Consequências Positivas

- O front trata erro e paginação num lugar só;
- Uma v2 pode coexistir com a v1;
- O front já desenha as telas com os campos futuros, sem receber filtros enganosos.

### Consequências Negativas

- Se o front mandar `notaCorteMax`, a API ignora o parâmetro sem erro e devolve a lista sem esse filtro;
- O contrato precisa ser mantido igual nos dois lados; mudanças exigem combinar com o front;
- A [ADR-010](../front/ADR-010.md) ainda está como "proposto" no front (P-19).

## Prós e Contras das Opções

### Opção 1 - ADR-010 + extensões

- Bom, porque é o formato que o front já espera;
- Ruim, porque exige DTOs próprios (`Pagina<T>`, `ErroResposta`) em vez do padrão do Spring.

### Opção 2 - Padrão do Spring

- Bom, porque não exige código;
- Ruim, porque o `Page` do Spring tem muitos campos internos e não é um formato estável entre versões.

### Opção 3 - Só campos com dado

- Bom, porque o contrato fica mínimo;
- Ruim, porque o front teria que mudar de novo a cada fonte nova.

## Links

- `sauf-api/docs/DECISOES.md`: D-13 a D-16, D-21, D-30; P-11, P-19; seção 6 (alinhamentos com o front)
- Implementa: [ADR-010](../front/ADR-010.md)
- Relaciona-se com: [ADR-007](../front/ADR-007.md), [ADR-BE-011](ADR-BE-011.md)
- Collection do Postman: `sauf-api/docs/postman/`
