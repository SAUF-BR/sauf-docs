# ADR-BE-008 - Ingestão por rotas próprias com X-Api-Key, upsert idempotente e recusa de IES desconhecida

- Status: aceito (autenticação por X-Api-Key aguarda confirmação formal, P-06; inativação provisória, P-26)
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-08

## Contexto e Declaração do Problema

O scraper coleta os dados e envia para a API ([ADR-SC-002](../scraper/ADR-SC-002.md)). São cinco fluxos: Censo (instituições e cursos do Censo), lote do site de cada universidade, IGC, Sisu e, futuramente, outros importadores. Essas rotas escrevem no banco e não podem ficar abertas como as rotas de leitura do catálogo. O login de usuários ainda não existe, e o scraper não é um usuário.

Como o scraper envia os dados, como a API protege essas rotas e o que acontece quando um dado chega de novo, chega incompleto ou some?

## Fatores da Decisão

- Proteção simples das rotas de escrita, sem depender do login de usuários;
- Reenvio seguro (idempotência);
- Dados oficiais da instituição nunca criados pelo scraper do site;
- Rastreabilidade de recusas no log.

## Opções Consideradas

- Opção 1: Rotas `POST /api/v1/ingestao/*` com header `X-Api-Key` (segredo compartilhado), upsert por chave natural
- Opção 2: Scraper escreve direto no banco
- Opção 3: Rotas de ingestão protegidas pelo mesmo login dos usuários (usuário "robô")

## Resultado da Decisão

Opção escolhida: "Opção 1".

- Rotas: `POST /api/v1/ingestao/censo`, `/lotes`, `/igc` e `/sisu`;
- O `ApiKeyInterceptor` exige o header `X-Api-Key` igual a `SAUF_API_KEY` (mesmo valor no `.env` dos dois projetos). Sem ele: **401**;
- Upsert pelas chaves naturais ([ADR-BE-007](ADR-BE-007.md)). Rodar duas vezes não duplica;
- Resposta: `{criados, atualizados, inativados, rejeitados[]}` (o Sisu devolve também `semOferta`);
- **Ordem obrigatória**: o `censo` cadastra as instituições primeiro. Lote, IGC ou Sisu de uma instituição não cadastrada é **recusado com 422** e registrado no log (D-27), por exemplo "Instituição com código e-MEC 57 não está cadastrada. Rode o import do Censo antes.";
- Oferta que não aparece no lote seguinte da mesma instituição fica **inativa**, sem ser apagada (provisório, P-26);
- O `censo` não cria ofertas ([ADR-BE-009](ADR-BE-009.md)).

### Consequências Positivas

- O scraper não precisa conhecer o banco, só o contrato JSON;
- Reenviar é seguro, então o scraper pode rodar quantas vezes quiser;
- Dados de instituição só entram pela fonte oficial.

### Consequências Negativas

- Uma chave compartilhada não identifica quem enviou e precisa ser trocada à mão se vazar;
- O time precisa rodar os comandos na ordem certa (censo → spider → igc → sisu), o que já gerou um 422 no primeiro teste;
- A collection do Postman tem exemplos de ingestão que gravam uma IES fictícia (e-MEC 999999) no banco em que forem rodados.

## Prós e Contras das Opções

### Opção 1 - Rotas próprias com X-Api-Key

- Bom, porque é simples e independente do login;
- Ruim, porque o segredo é único para todos os clientes de ingestão.

### Opção 2 - Escrita direta no banco

- Bom, porque é mais rápido de fazer;
- Ruim, porque espalha regras entre Python e Java e contraria a [ADR-BE-002](ADR-BE-002.md).

### Opção 3 - Login de usuário robô

- Bom, porque reaproveitaria a autenticação;
- Ruim, porque o login ainda não existe e misturaria papéis de usuário e de sistema.

## Links

- `sauf-api/docs/DECISOES.md`: D-27, D-48; P-06, P-26; seção 3 (rotas)
- README do `sauf-scraper`: seção "Como popular a API"
- Relaciona-se com: [ADR-SC-002](../scraper/ADR-SC-002.md), [ADR-BE-007](ADR-BE-007.md), [ADR-BE-009](ADR-BE-009.md)
