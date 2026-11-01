# ADR-BE-004 - Postgres local em Docker para desenvolvimento, H2 só nos testes e Supabase só para a main

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-17

## Contexto e Declaração do Problema

A primeira decisão (D-28) foi usar um único projeto Supabase de desenvolvimento, porque o Docker não estava instalado. Com o time crescendo, isso virou risco: uma migration de uma branch ainda não mergeada seria aplicada no banco compartilhado e quebraria a API dos colegas ("applied migration not resolved locally"). Ao mesmo tempo, o Carlos pediu para tirar o H2, para a API não rodar num banco em memória sem querer, mas os testes automáticos dependiam dele.

Onde cada pessoa desenvolve, onde os testes rodam e o que vai para o Supabase?

## Fatores da Decisão

- Isolamento: o trabalho de uma branch não pode quebrar o banco dos outros;
- Testes rápidos e sem serviço externo (`mvn test`);
- Impossibilidade de a API subir num banco errado por engano;
- Pouca configuração para os colegas.

## Opções Consideradas

- Opção 1: Postgres 17 local em Docker (dev), H2 em modo PostgreSQL só no escopo de teste, Supabase só com o que está na `main`
- Opção 2: Um Supabase compartilhado para dev, como na D-28
- Opção 3: Testcontainers (Postgres real também nos testes), sem H2
- Opção 4: Um segundo projeto Supabase só para testes

## Resultado da Decisão

Opção escolhida: "Opção 1", porque isola o banco de cada pessoa, mantém o `mvn test` rápido e sem Docker obrigatório para os testes, e reserva o Supabase para o estado da `main`.

- `docker-compose.yml` com `postgres:17-alpine` na porta **5433** (para não conflitar com um Postgres instalado no Windows) e volume `sauf-pgdata`. Usuário e senha `sauf` são só de dev;
- O `.env` tem o bloco local ativo e o do Supabase comentado;
- O H2 está no `pom.xml` com `<scope>test</scope>` e não entra na API em execução;
- A classe `config/ExigirPostgres` faz a API **parar com mensagem clara** se `SAUF_DB_URL` estiver vazia ou não começar com `jdbc:postgresql://`;
- Para zerar o banco local: `docker compose down -v`, e depois repopular pelo scraper (censo → spider → igc → sisu).

### Consequências Positivas

- Migrations de branches não mergeadas nunca chegam ao banco compartilhado;
- A API não roda em banco em memória por engano;
- Os testes continuam rodando sem Docker e sem internet.

### Consequências Negativas

- Cada integrante precisa instalar o Docker Desktop e lembrar de abri-lo antes do `docker compose up -d`;
- O H2 em modo PostgreSQL não é um Postgres de verdade: algum comportamento específico pode passar nos testes e falhar no banco real;
- O banco local começa vazio e precisa ser repopulado depois de cada `down -v`.

## Prós e Contras das Opções

### Opção 1 - Docker local + H2 em teste + Supabase na main

- Bom, porque isola cada desenvolvedor;
- Bom, porque os testes são rápidos;
- Ruim, porque exige Docker para rodar a API localmente.

### Opção 2 - Supabase compartilhado

- Bom, porque não exige nada instalado;
- Ruim, porque uma branch pode quebrar o banco de todos.

### Opção 3 - Testcontainers

- Bom, porque os testes usam um Postgres real;
- Ruim, porque exige Docker até para o `mvn test` e deixa os testes mais lentos.

### Opção 4 - Supabase só para testes

- Bom, porque os testes usam um Postgres real sem Docker;
- Ruim, porque é mais um serviço externo para manter, e os testes dependem da internet.

## Links

- `sauf-api/docs/DECISOES.md`: D-28, D-52; README do `sauf-api`, seção "Banco local com Docker"
- Relaciona-se com: [ADR-BE-003](ADR-BE-003.md) (migrations)
- Relaciona-se com: [ADR-BE-002](ADR-BE-002.md) (papel do Supabase)
