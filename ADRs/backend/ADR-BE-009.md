# ADR-BE-009 - O site da universidade é a fonte de verdade; o Censo só complementa

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-08

## Contexto e Declaração do Problema

Os dados de um curso vêm de duas fontes que discordam: o site da universidade (turno, duração, textos, link, cursos novos) e o Censo da Educação Superior do INEP (código e-MEC, grau, vagas, área, gratuidade). O Censo mais recente é de 2024 e sai com cerca de um ano de atraso; o site é atualizado pela própria universidade. A primeira ideia era o Censo definir a lista de cursos e o site só enriquecer, mas isso recusaria cursos novos, como Engenharia de Software da UEM.

Quando as fontes discordam, qual vale? E o Censo pode criar cursos que não estão no site?

## Fatores da Decisão

- Dados atualizados para o usuário ("essa universidade oferece, temos essas informações, acessar o site");
- Não perder cursos novos que ainda não estão no Censo;
- Aproveitar o que só o Censo tem (área, vagas, gratuidade);
- Regra única e simples de explicar.

## Opções Consideradas

- Opção 1: Censo define a lista de cursos; site só enriquece; curso fora do Censo é recusado
- Opção 2: Site define a lista e vale sobre o Censo; Censo só completa o que o site não mostra
- Opção 3: As duas fontes criam ofertas independentes

## Resultado da Decisão

Opção escolhida: "Opção 2". Nas palavras do Carlos (08/10/2026): *"Dê preferência ao que está no site da UEM, é o mais atualizado. Vamos usar o CSV como complemento apenas. 2024 para 2026 é muito tempo para confiarmos no senso."*

- **Instituições** vêm do Censo, só as que têm spider (D-22, [ADR-SC-004](../scraper/ADR-SC-004.md));
- **Ofertas** vêm do site. Oferta sem código e-MEC é **aceita** (D-23);
- **Quando discordam, vale o site** (D-24). Grau e turno vêm da página; os do Censo só entram quando a página não informa;
- **O Censo não cria ofertas** (D-33): os cursos dele ficam na tabela `censo_curso` e só completam as ofertas do site. Curso que está no Censo e não está no site **não aparece** no catálogo;
- **Os códigos e-MEC de cada página ficam numa lista interna** (`oferta_codigo_emec`), fora da API pública (D-35). Eles ligam a oferta ao Censo: curso genérico e área ([ADR-BE-006](ADR-BE-006.md)), vagas (soma) e gratuidade (verdadeiro se algum código for gratuito). Também ligam a oferta às notas de corte do Sisu ([ADR-BE-011](ADR-BE-011.md)).

A lista de códigos de cada página vem da tabela de casamento revisada à mão no scraper ([ADR-SC-006](../scraper/ADR-SC-006.md)).

### Consequências Positivas

- O catálogo mostra o que a universidade oferece hoje;
- Cursos novos aparecem, mesmo com menos informação;
- Área, vagas e notas continuam automáticas para quem tem código.

### Consequências Negativas

- Oferta sem código fica sem área, vagas e nota de corte até alguém revisar (P-05, "Sem informação" no front);
- A qualidade depende da tabela de casamento de cada universidade;
- Vagas do Censo são de 2024, mesmo quando o resto da oferta é atual.

## Prós e Contras das Opções

### Opção 1 - Censo manda

- Bom, porque a lista é oficial e com chave estável;
- Ruim, porque recusa cursos novos e usa dados com até dois anos de atraso.

### Opção 2 - Site manda, Censo complementa

- Bom, porque mostra o dado mais atual;
- Ruim, porque exige o casamento site × Censo.

### Opção 3 - Fontes independentes

- Bom, porque não exige casamento;
- Ruim, porque a mesma oferta aparece duas vezes no catálogo.

## Links

- `sauf-api/docs/DECISOES.md`: D-22, D-23, D-24, D-33, D-35
- `sauf-scraper/docs/MAPEAMENTO_CENSO.md`
- Relaciona-se com: [ADR-SC-004](../scraper/ADR-SC-004.md), [ADR-SC-006](../scraper/ADR-SC-006.md), [ADR-BE-006](ADR-BE-006.md)
