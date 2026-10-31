# ADR-BE-006 - Curso genérico e áreas pela classificação Cine do Censo, com nome amigável

- Status: aceito (tabela de nomes amigáveis ainda não criada)
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-08
- História Técnica: D-18, D-31, D-36 do `docs/DECISOES.md` do `sauf-api`; P-05 em aberto

## Contexto e Declaração do Problema

Para agrupar as ofertas no catálogo ([ADR-BE-005](ADR-BE-005.md)) é preciso decidir quando duas ofertas de universidades diferentes são "o mesmo curso" e em qual área ele fica. Os nomes nos sites variam ("Ciências Contábeis", "Contabilidade"), e o Censo tem uma classificação oficial (Cine/Unesco) com rótulos técnicos, às vezes separando bacharelado e licenciatura em áreas diferentes (ex.: "Física" em Ciências naturais e "Física formação de professor" em Educação).

Qual critério agrupa as ofertas em cursos genéricos, e qual nome e área o card mostra?

## Fatores da Decisão

- Agrupamento sem duplicatas e explicável;
- Nomes que o estudante reconhece;
- Áreas padronizadas para o filtro do catálogo;
- Pouco trabalho manual para 5 universidades.

## Opções Consideradas

- Opção 1: Rótulo Cine puro (nome e área oficiais)
- Opção 2: Nome normalizado vindo do site
- Opção 3: Rótulo Cine para agrupar e definir a área, com uma tabela pequena revisada à mão para o nome exibido

## Resultado da Decisão

Opção escolhida: "Opção 3 - Cine + nome amigável" (escolha do Carlos no card de 06/10/2026, registrada como D-31).

- **Áreas** = as 11 áreas gerais do Censo (`CO_CINE_AREA_GERAL` / `NO_CINE_AREA_GERAL`), criadas pela `V1` (D-18);
- **Curso genérico** = rótulo Cine (`CO_CINE_ROTULO`) da oferta, obtido pelos códigos e-MEC ligados a ela;
- **Bacharelado e "formação de professor" ficam juntos**, com a área do bacharelado (D-36, opção "b"). Quando só existe a licenciatura, o curso usa o nome sem o sufixo e a área dela;
- **Nome exibido**: enquanto a tabela revisada não existe, usa o rótulo Cine sem o sufixo (ex.: "Engenharia civil"). A ingestão nunca sobrescreve `curso.nome` depois de criado, então a revisão manual pode editar direto.

Caso ainda em aberto (P-05): oferta do site **sem código e-MEC** (ex.: Engenharia de Software da UEM, criada depois do Censo 2024). Provisoriamente, ela entra ligada a um curso genérico com o nome do site, **sem área** e com `pendente_revisao = true`. Hoje são 9 páginas da UEM nessa situação; o front mostra a área como "Sem informação".

### Consequências Positivas

- Agrupamento oficial e estável entre universidades;
- Filtro de área funciona sem curadoria;
- O usuário procura "Física" e acha bacharelado e licenciatura juntos.

### Consequências Negativas

- Depende dos códigos e-MEC das ofertas: sem eles, o curso fica sem área ([ADR-BE-009](ADR-BE-009.md));
- Os nomes técnicos do Cine aparecem até a tabela amigável existir;
- As áreas do Censo têm nomes diferentes dos usados nos mocks do front (ex.: "Saúde e bem-estar" em vez de "Saúde").

## Prós e Contras das Opções

### Opção 1 - Rótulo Cine puro

- Bom, porque é oficial e sem duplicatas;
- Ruim, porque os nomes são técnicos e as licenciaturas viram cards separados na área Educação.

### Opção 2 - Nome normalizado do site

- Bom, porque o nome é amigável;
- Ruim, porque a área e as variações de nome entre universidades ficam todas por conta do time.

### Opção 3 - Cine + nome amigável

- Bom, porque une o agrupamento oficial a nomes legíveis;
- Ruim, porque exige manter uma tabela de nomes.

## Links

- `sauf-api/docs/DECISOES.md`: D-18, D-31, D-36; P-05; seção 4 (comportamentos provisórios)
- `sauf-scraper/docs/MAPEAMENTO_CENSO.md`
- Relaciona-se com: [ADR-BE-005](ADR-BE-005.md), [ADR-BE-009](ADR-BE-009.md)
