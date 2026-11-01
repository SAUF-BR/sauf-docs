# ADR-BE-013 - Planos de assinatura definidos no código e regras de vigência

- Status: aceito (pagamento simulado até existir gateway, P-14)
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-21

## Contexto e Declaração do Problema

A área do assinante (simulados) depende de uma assinatura ativa ([ADR-006](../front/ADR-006.md)). A tela de assinaturas mostra os planos com preço, texto, selo e recursos. O time precisava decidir onde esses planos ficam, como a API responde quando o usuário não assina e quando a assinatura deixa de valer, antes de existir um gateway de pagamento.

Onde ficam os planos, e quais regras definem se uma assinatura está ativa?

## Fatores da Decisão

- Facilidade para alterar os planos, numa fonte só;
- Validação pelos testes;
- Ninguém com acesso sem ter pago;
- Resposta clara para "sem assinatura", diferente de erro.

## Opções Consideradas

- Opção 1: Planos num `enum Plano` no código; o banco guarda só a assinatura do usuário
- Opção 2: Tabela de planos no banco, editável
- Opção 3: Planos só no front

## Resultado da Decisão

Opção escolhida: "Opção 1".

- `enum Plano` em `assinatura/`: preço, texto, selo e recursos mudam no código e são publicados com a API. Uma constante que já tenha assinaturas não é renomeada nem apagada, porque o banco guarda o nome dela (D-54);
- Rotas: `GET /assinatura/planos`, `GET /assinatura`, `PUT /assinatura` (assinar ou trocar), `POST /assinatura/cancelar` e `POST /assinatura/reativar`;
- **Sem assinatura: `GET /api/v1/assinatura` responde `204 No Content`** (D-56);
- **Renovação (+1 mês) só com pagamento confirmado**. Sem confirmação, a assinatura vale até o `renovaEm`, inclusive, e depois conta como sem assinatura (D-57);
- Cancelar agenda o fim: o acesso continua até o `renovaEm`;
- Provisório (P-14): sem gateway, o `PUT /assinatura` age como pagamento confirmado;
- Provisório (D-55): até o login existir, todas as requisições usam um usuário fixo e `assinatura.usuario_id` fica sem FK.

### Consequências Positivas

- Os planos têm uma fonte só, e os testes garantem que estão coerentes;
- O front distingue "não assina" (204) de erro;
- A regra de vigência é explícita e testável.

### Consequências Negativas

- Mudar preço ou recurso exige publicar a API;
- Enquanto não houver gateway, qualquer usuário "assina" sem pagar;
- Uma migration futura precisa criar a FK de `assinatura.usuario_id` quando o login existir.

## Prós e Contras das Opções

### Opção 1 - Enum no código

- Bom, porque é simples, versionado e testado;
- Ruim, porque não permite edição sem deploy.

### Opção 2 - Tabela no banco

- Bom, porque permite editar sem deploy;
- Ruim, porque exige tela de administração e abre espaço para planos inconsistentes.

### Opção 3 - Só no front

- Bom, porque não exige nada no backend;
- Ruim, porque o backend precisa conhecer o plano para liberar os simulados.

## Links

- `sauf-api/docs/DECISOES.md`: D-54, D-55, D-56, D-57; P-14
- Pasta do projeto: `12-review-TR-282-assinaturas.md`
- Relaciona-se com: [ADR-006](../front/ADR-006.md), [ADR-004](../front/ADR-004.md)
