# ADR-SC-006 - Tabela de casamento site × Censo versionada e confirmada à mão

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-08
- História Técnica: D-32 (substituída), D-34, D-35 do `docs/DECISOES.md` do `sauf-api`; `scraper/04-casamento-uem.md`

## Contexto e Declaração do Problema

Para completar uma oferta do site com área, vagas, gratuidade e nota de corte, é preciso saber quais códigos e-MEC do Censo correspondem a cada página. Os nomes não batem sempre ("Tecnologia em Alimentos" × "Alimentos"), uma página pode ter vários códigos (bacharelado e licenciatura, diurno e noturno) e alguns cursos não têm par. Na UEM, dos 67 cursos do site, o casamento automático achou 46 diretos, 9 ambíguos e 12 sem par.

Como ligar cada página do site aos códigos do Censo de forma confiável?

## Fatores da Decisão

- Nenhum código errado no catálogo;
- Revisão possível por uma pessoa, sem programar;
- Histórico das decisões de casamento;
- Repetível quando sair um Censo novo.

## Opções Consideradas

- Opção 1: Comando `casar` gera sugestões; uma tabela CSV por IES, versionada, com coluna `confirmado` revisada à mão; o spider só usa códigos confirmados
- Opção 2: Casamento automático por nome a cada execução
- Opção 3: Tabela de casamento no banco, editada por uma tela

## Resultado da Decisão

Opção escolhida: "Opção 1".

- **`sauf-scraper casar --ies uem`** compara o site com o Censo. Regra: mesmo nome e mesma cidade do campus; desempate pelo grau do site e, se preciso, pelo turno;
- O resultado vai para **`spiders/dados/<ies>_codigos.csv`** (separador `;`), com uma linha por par página × código e as colunas `chave`, `nome_site`, `cidade`, `turno_site`, `grau_site`, `status`, `codigo_emec`, `nome_censo`, `grau_censo`, `vagas_diurno`, `vagas_noturno`, `sugestao` e `confirmado`;
- Uma pessoa revisa os ambíguos e troca `confirmado` para `sim`. O spider só envia os códigos confirmados (`codigos_confirmados`);
- Vários códigos confirmados na mesma página vão juntos em `codigosEmec` de **uma** oferta (D-34, D-35). A D-32 ("um curso por código") foi substituída: *"é totalmente desnecessário listar 4 cursos IGUAIS"* (Carlos, 08/10/2026);
- A tabela fica no Git, junto do spider.

### Consequências Positivas

- Só entra código que uma pessoa conferiu;
- A revisão é feita numa planilha, por qualquer pessoa do time;
- O Git guarda quem mudou cada casamento e quando.

### Consequências Negativas

- Trabalho manual a cada IES nova e a cada Censo novo;
- Enquanto a revisão não termina, parte das ofertas fica sem área e sem nota (na UEM, 15 códigos do Sisu ainda não estão na tabela, adiado em 26/10);
- Cada IES nova precisa da sua própria tabela.

## Prós e Contras das Opções

### Opção 1 - CSV versionado e confirmado à mão

- Bom, porque é confiável e auditável;
- Ruim, porque depende de revisão humana.

### Opção 2 - Automático a cada execução

- Bom, porque não exige trabalho manual;
- Ruim, porque erra em nomes diferentes e em páginas com vários códigos, sem ninguém ver.

### Opção 3 - Tabela no banco com tela

- Bom, porque a revisão ficaria dentro do sistema;
- Ruim, porque exige tela de administração e tira o histórico do Git.

## Links

- `sauf-scraper/src/sauf_scraper/spiders/dados/uem_codigos.csv`, `sauf-scraper/src/sauf_scraper/censo/casamento.py`
- `sauf-scraper/docs/MAPEAMENTO_CENSO.md`
- Pasta do projeto: `scraper/04-casamento-uem.md`, `scraper/09-o-que-falta.md`
- Relaciona-se com: [ADR-BE-009](../backend/ADR-BE-009.md), [ADR-BE-006](../backend/ADR-BE-006.md), [ADR-SC-005](ADR-SC-005.md)
