# ADR-BE-012 - Pacotes por domínio, DTOs como records e sem Lombok

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-06
- História Técnica: D-09, D-12, D-29 do `docs/DECISOES.md` do `sauf-api`; P-22 em aberto

## Contexto e Declaração do Problema

O código antigo do `sauf-api` era organizado por camada (`models/`, `controllers/`, `repositories/`). A nova API tem domínios bem separados (catálogo, ingestão, assinatura) e será mantida por várias pessoas, que precisam explicar o código na banca. Além disso, o Lombok era uma opção comum para reduzir getters e setters.

Como organizar os pacotes Java e quanto de "mágica" de geração de código aceitar?

## Fatores da Decisão

- Encontrar rápido tudo que pertence a uma funcionalidade;
- Menos conflitos entre pessoas em funcionalidades diferentes;
- Código explícito, fácil de revisar e de explicar;
- Coerência com o front ([ADR-002](../front/ADR-002.md)).

## Opções Consideradas

- Opção 1: Pacotes por domínio, DTOs como `record`, entidades com getters/setters explícitos, sem Lombok
- Opção 2: Pacotes por camada (`controller/`, `service/`, `repository/`, `model/`)
- Opção 3: Pacotes por domínio com Lombok

## Resultado da Decisão

Opção escolhida: "Opção 1".

- Um pacote por domínio em `br.com.sauf_api`: `area`, `curso`, `oferta`, `instituicao`, `censo`, `sisu`, `ingestao`, `assinatura`. Cada um tem entidade, repositório, serviço, controller e DTOs;
- `comum/` guarda o que é transversal: `ErroResposta`, `Pagina<T>`, `TratadorDeErros`, exceções, conversão de enums (`EnumComValor`) e `UsuarioLogado`;
- `config/` guarda configuração (`WebConfig` com CORS, `OpenApiConfig`, `ExigirPostgres`);
- DTOs de entrada e saída são `record`s (D-09), com nomes de negócio (`CursoResumo`, `LoteScraperRequest`, `UniversidadeDetalhe`);
- Sem Lombok (D-29);
- Pouco comentário: só onde algo é específico e difícil de entender (D-12).

O nome do pacote (`br.com.sauf_api` ou `br.com.saufbr.api`) ainda está em aberto (P-22).

### Consequências Positivas

- Mesma lógica de organização no front e no back;
- Revisões de PR ficam concentradas num pacote;
- Nenhum código gerado escondido: o que está no arquivo é o que roda.

### Consequências Negativas

- Entidades JPA ficam mais longas (getters e setters à mão);
- Exige disciplina para decidir o que vai para `comum/`;
- Trocar o nome do pacote fica mais caro a cada arquivo novo.

## Prós e Contras das Opções

### Opção 1 - Por domínio, records, sem Lombok

- Bom, porque é explícito e agrupado por funcionalidade;
- Ruim, porque tem mais linhas nas entidades.

### Opção 2 - Por camada

- Bom, porque é o padrão de muitos tutoriais;
- Ruim, porque espalha uma funcionalidade por vários pacotes.

### Opção 3 - Por domínio com Lombok

- Bom, porque reduz código repetitivo;
- Ruim, porque esconde código gerado e exige plugin na IDE de todos.

## Links

- `sauf-api/docs/DECISOES.md`: D-09, D-12, D-29; P-22
- Relaciona-se com: [ADR-002](../front/ADR-002.md) (organização por feature no front)
