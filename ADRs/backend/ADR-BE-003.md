# ADR-BE-003 - Flyway para versionar o banco, com migrations nomeadas por data e hora

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-17 (Flyway desde 2026-10-06; V1 consolidada em 2026-10-09)
- História Técnica: D-08 e D-52 do `docs/DECISOES.md` do `sauf-api`

## Contexto e Declaração do Problema

O esquema do banco muda a cada decisão do catálogo (oferta por página, textos do site, notas de corte, endereço, assinaturas). Essas mudanças precisam ser reproduzíveis no banco de cada integrante e no Supabase. Com 6 pessoas trabalhando em branches paralelas, o primeiro formato de nome (`V4`, `V5`...) já gerou colisão: duas branches criaram a mesma versão, e o Flyway recusou subir com "more than one migration with version".

Como versionar o esquema do banco, e como nomear as migrations para evitar colisões entre branches?

## Fatores da Decisão

- Esquema reproduzível a partir do código, em qualquer banco;
- Histórico auditável das mudanças;
- Nenhuma colisão de versão entre branches paralelas;
- Regras simples para o time seguir.

## Opções Consideradas

- Opção 1: Flyway com versão sequencial (`V4__...`, `V5__...`)
- Opção 2: Flyway com data e hora no nome (`V<AAAA>_<MM>_<DD>_<HHMM>__descricao.sql`)
- Opção 3: Deixar o Hibernate criar e alterar as tabelas (`ddl-auto=update`)

## Resultado da Decisão

Opção escolhida: "Opção 2 - Flyway com data e hora no nome". O Flyway lê `_` como separador de versão, então `V2026_10_17_1015__nota_corte_sisu.sql` vira a versão `2026.10.17.1015`, e duas branches praticamente nunca geram o mesmo número.

Regras:
- `V1__estrutura_inicial.sql`, `V2__mercado_trabalho.sql` e `V3__indicador_ies.sql` ficam como estão. Como `2026.x` é maior que `3`, a ordem continua certa;
- Migration já aplicada **nunca é editada**: qualquer mudança vira um arquivo novo. Editar quebra o checksum do Flyway;
- `outOfOrder` fica desligado;
- Cada um testa no seu Postgres local, e o Supabase só recebe o que está na `main` ([ADR-BE-004](ADR-BE-004.md));
- O Hibernate só valida o esquema, sem criar tabelas.

Antes de o Supabase receber a primeira migration, as cinco migrations de rascunho foram juntadas numa só `V1` (pedido do Carlos em 09/10/2026), porque não havia banco para preservar.

### Consequências Positivas

- Qualquer banco vazio chega ao esquema atual só com `mvn spring-boot:run`;
- As mudanças do banco ficam no histórico do Git, junto do código que as usa;
- Branches paralelas não disputam o mesmo número de versão.

### Consequências Negativas

- Uma migration criada antes, mas mergeada depois de outra já aplicada, é recusada no banco local ("fora de ordem"). Aconteceu com a da TR-282 (assinaturas, 21/10) e a da fase C (endereço, 26/10). A solução foi recriar o banco local (`docker compose down -v`) e repopular;
- Os nomes ficam mais longos e menos legíveis que `V4`;
- Não dá para "corrigir" uma migration aplicada: o conserto sempre é outra migration (ex.: a coluna `oferta_curso.nota_corte_sisu` sem uso ainda existe e só sai numa migration futura).

## Prós e Contras das Opções

### Opção 1 - Versão sequencial

- Bom, porque os nomes são curtos e a ordem é óbvia;
- Ruim, porque branches paralelas criam o mesmo número, e quem mergeia por último precisa renumerar.

### Opção 2 - Data e hora no nome

- Bom, porque a colisão fica quase impossível;
- Bom, porque a versão diz quando a migration foi criada;
- Ruim, porque merges fora de ordem exigem recriar o banco local.

### Opção 3 - `ddl-auto=update`

- Bom, porque não exige escrever SQL;
- Ruim, porque não versiona nada, não remove colunas e pode alterar o banco de produção de forma imprevisível.

## Links

- `sauf-api/docs/DECISOES.md`: D-08, D-52; README do `sauf-api`, seção "Migrations"
- Relaciona-se com: [ADR-BE-004](ADR-BE-004.md) (banco local e Supabase)
- Documentação oficial: [Flyway - Versioned migrations](https://documentation.red-gate.com/flyway)
