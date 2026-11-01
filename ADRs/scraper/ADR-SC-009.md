# ADR-SC-009 - LGPD no scraper: não coletar dados pessoais e versionar fixtures anonimizadas

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-26

## Contexto e Declaração do Problema

As páginas de curso trazem dados pessoais junto com o conteúdo útil: nome e e-mail da coordenação (UEM) e depoimentos de estudantes com nome completo (UFSC). O catálogo não precisa de nenhum deles. Além disso, os testes usam HTML real salvo como fixture, e essas fixtures ficam no Git. Os microdados de PROUni são por bolsista.

O que o scraper pode coletar e guardar, e como tratar as fixtures?

## Fatores da Decisão

- Minimização de dados (LGPD, art. 6º, III);
- Nenhum dado pessoal no banco nem no repositório;
- Testes reproduzíveis para todo o time, com as fixtures no Git;
- Regra simples de seguir em cada spider novo.

## Opções Consideradas

- Opção 1: Parser remove os blocos com dado pessoal; fixtures anonimizadas e versionadas no Git
- Opção 2: Coletar tudo e filtrar no backend
- Opção 3: Fixtures fora do Git, cada pessoa com as suas

## Resultado da Decisão

Opção escolhida: "Opção 1".

- **Não coletar**: nomes e e-mails da coordenação, depoimentos de estudantes e qualquer outro dado de pessoa física. O parser corta esses blocos antes de extrair o texto;
- Dados agregados oficiais (Censo, IGC, Sisu) são por instituição e curso, não por pessoa. Se um dia entrar uma fonte por pessoa (como PROUni), guardamos só o agregado por curso;
- **Fixtures vão para o Git, sempre anonimizadas** (26/10/2026, `CONTRIBUTING.md`): nenhum nome, e-mail ou telefone real de pessoa no HTML salvo;
- O checklist de PR do `CONTRIBUTING.md` inclui a verificação de LGPD ("nada de nomes, e-mails ou telefones de pessoas").

### Consequências Positivas

- Nenhum dado pessoal sai do site da universidade;
- Todo o time roda os mesmos testes;
- Regra explícita para quem escrever um spider novo.

### Consequências Negativas

- Anonimizar cada fixture é um passo manual e pode ser esquecido; a revisão do PR precisa conferir;
- Se o site mudar a estrutura do bloco pessoal, o corte pode falhar sem aviso. O teste do parser precisa verificar que o bloco não aparece.

## Prós e Contras das Opções

### Opção 1 - Cortar no parser e anonimizar fixtures

- Bom, porque o dado pessoal nunca entra no sistema;
- Ruim, porque exige atenção em cada spider e cada fixture.

### Opção 2 - Filtrar no backend

- Bom, porque centraliza a regra;
- Ruim, porque o dado pessoal trafega e pode ficar em logs e no dry-run.

### Opção 3 - Fixtures fora do Git

- Bom, porque nada pessoal vai para o repositório;
- Ruim, porque os testes deixam de ser reproduzíveis e a CI não roda.

## Links

- `sauf-scraper/CONTRIBUTING.md`
- Pasta do projeto: `lgpd/`, `scraper/11-avaliacao-ies.md`
- Relaciona-se com: [ADR-SC-003](ADR-SC-003.md), [ADR-SC-008](ADR-SC-008.md)
