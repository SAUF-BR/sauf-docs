# ADR-BE-014 - Testes unitários e de integração na API: JUnit 5, MockMvc e H2

- Status: aceito
- Decisores: Karen
- Data: 2026-10-31
- História Técnica: D-28 do `docs/DECISOES.md`; [ADR-BE-004](ADR-BE-004.md)

## Contexto e Declaração do Problema

A API concentra as regras de negócio do SAUF-BR (catálogo, ingestão, notas do Sisu e IGC, assinaturas), e um erro nelas aparece em todas as telas do front. Por isso, é preciso garantir com testes automatizados que essas regras continuem funcionando a cada mudança.

Quais testes automatizados o projeto faz e com quais tecnologias?

## Fatores da Decisão

- Testar a regra de negócio onde ela mora (no backend);
- Rodar de forma simples e rápida, com um comando só;
- Usar o que o Spring Boot já traz;
- Caber no prazo do projeto.

## Opções Consideradas

- Opção 1: Testes só no back: unitários com JUnit 5 + AssertJ e de integração com Spring Boot Test + MockMvc + H2 em modo PostgreSQL
- Opção 2: Testes no back e no front (Vitest + Testing Library)
- Opção 3: Testes de integração com Postgres real via Testcontainers

## Resultado da Decisão

Opção escolhida: "Opção 1", porque cobre as regras de negócio onde elas estão, é simples de rodar e não exige dependências novas.

- **Unitário** (`<Classe>Test`): testa uma classe sozinha, sem subir o Spring nem o banco, com JUnit 5 e AssertJ; Mockito só quando for preciso isolar uma dependência;
- **Integração** (`<Dominio>ApiTest`): testa uma rota de ponta a ponta com `@SpringBootTest`, `@AutoConfigureMockMvc`, `@ActiveProfiles("test")` e `@Transactional`, conferindo status e JSON com MockMvc;
- O banco dos testes é o H2 em modo PostgreSQL (`src/test/resources/application-test.yml`), onde as migrations do Flyway também rodam;
- Toda rota nova tem pelo menos um teste de integração com o caminho feliz e os erros principais, no formato da [ADR-BE-010](ADR-BE-010.md);
- Cada teste cria os próprios dados e o `@Transactional` desfaz tudo no fim;
- Os testes rodam com `./mvnw test`.

### Consequências Positivas

- Uma mudança que quebra uma rota aparece no `./mvnw test`;
- Os testes são rápidos e não dependem de nenhum serviço externo;
- O teste de integração também confere o contrato que o front consome.

### Consequências Negativas

- O H2 não é um Postgres de verdade: algum comportamento pode passar no teste e falhar no banco real;
- Testes de integração são mais lentos, porque sobem a aplicação;
- O front fica sem testes automatizados.

## Prós e Contras das Opções

### Opção 1 - Só no back, com H2

- Bom, porque testa a regra onde ela mora e é simples de rodar;
- Ruim, porque o H2 não é idêntico ao Postgres.

### Opção 2 - Back e front

- Bom, porque também pegaria erros de tela;
- Ruim, porque dobra o esforço e não cabe no prazo.

### Opção 3 - Testcontainers

- Bom, porque testa com Postgres real;
- Ruim, porque deixa o ambiente de testes mais pesado e complexo.

## Links

- Relaciona-se com: [ADR-BE-004](ADR-BE-004.md) (H2 só nos testes)
- Relaciona-se com: [ADR-BE-010](ADR-BE-010.md) (formato do erro e da paginação)
- Relaciona-se com: [ADR-BE-015](ADR-BE-015.md) (onde os testes ficam)
