# ADR-BE-001 - Java 21, Spring Boot 4 e Maven para a API

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-06

## Contexto e Declaração do Problema

O SAUF-BR precisa de uma API que guarde o catálogo de cursos e universidades, receba os dados do scraper e sirva o frontend web ([ADR-001](../front/ADR-001.md)). O repositório `sauf-api` tinha uma versão antiga do projeto (MySQL, anúncios, senha do banco commitada), que não servia mais de base. O time precisava escolher a stack da nova versão.

Com qual linguagem, framework e ferramenta de build a API deve ser construída?

## Fatores da Decisão

- Familiaridade do time com a stack, que precisa ser defendida na banca;
- Suporte de longo prazo da versão escolhida;
- Ecossistema pronto para os requisitos do projeto: REST, JPA, validação, migrations versionadas e documentação das rotas;
- Facilidade de deploy em serviços comuns.

## Opções Consideradas

- Opção 1: Java 21 + Spring Boot 4 + Maven
- Opção 2: Java 21 + Spring Boot 3.x + Maven
- Opção 3: Node.js (NestJS/Express) com TypeScript, mesma linguagem do front

## Resultado da Decisão

Opção escolhida: "Opção 1 - Java 21 + Spring Boot 4 + Maven", porque o time já usa Java e Maven, o Java 21 é LTS e a linha 3.x do Spring Boot já saiu do suporte gratuito. O conjunto de dependências fica enxuto:

- Spring Web MVC, Spring Data JPA e Bean Validation;
- driver do PostgreSQL e Flyway ([ADR-BE-003](ADR-BE-003.md));
- springdoc-openapi, para o Swagger em `/api/v1/swagger`;
- H2 só no escopo de teste ([ADR-BE-004](ADR-BE-004.md)).

DTOs de entrada e saída são `record`s do Java, e o projeto não usa Lombok ([ADR-BE-012](ADR-BE-012.md)).

### Consequências Positivas

- Stack conhecida pelo time e fácil de explicar na banca;
- Versões com suporte durante todo o projeto;
- Swagger gerado a partir do código, sempre igual às rotas reais;
- Bean Validation dá validação declarativa nos DTOs, com erros por campo para o front ([ADR-008](../front/ADR-008.md)).

### Consequências Negativas

- O Spring Boot 4 é recente, então parte do conteúdo da internet ainda é da linha 3 (ex.: nomes de starters mudaram, como `spring-boot-starter-webmvc`);
- O Flyway ainda não verificou oficialmente a versão do H2 usada nos testes (aviso no log, sem efeito prático);
- Front e back em linguagens diferentes: os tipos do contrato precisam ser mantidos em dois lugares.

## Prós e Contras das Opções

### Opção 1 - Java 21 + Spring Boot 4 + Maven

- Bom, porque é a linha com suporte atual;
- Bom, porque o time já conhece Java, Spring e Maven;
- Ruim, porque tem menos material de referência do que a linha 3.

### Opção 2 - Java 21 + Spring Boot 3.x + Maven

- Bom, porque tem mais material de referência;
- Ruim, porque a linha 3.x já saiu do suporte gratuito.

### Opção 3 - Node.js com TypeScript

- Bom, porque front e back compartilhariam linguagem e tipos;
- Ruim, porque o time e o projeto da Escola de TI já estavam direcionados para Java com Spring Boot;
- Ruim, porque exigiria escolher e montar ORM, migrations e validação à parte.

## Links

- `sauf-api/docs/DECISOES.md`: D-05, D-06, D-07, D-08, D-09
- Relaciona-se com: [ADR-001](../front/ADR-001.md) (front como SPA consumindo a API)
- Relaciona-se com: [ADR-BE-012](ADR-BE-012.md) (organização do código)
- Documentação oficial: [Spring Boot](https://docs.spring.io/spring-boot/)
