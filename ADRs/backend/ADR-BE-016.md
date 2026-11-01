# ADR-BE-014 - Stripe como gateway de pagamento das assinaturas

- Status: proposto
- Decisores: a definir pelo time
- Data: 2026-10-31
- História Técnica: estudo de mercado SAUF.BR (slides 9, 13 e 14); [ADR-BE-013](ADR-BE-013.md); D-54, D-55, D-57 e P-14 do `sauf-api/docs/DECISOES.md`

## Contexto e Declaração do Problema

O estudo de mercado define que a receita vem de três assinaturas mensais de baixo valor — Basic R$ 8, Plus R$ 12 e Premium R$ 15 — e que a arquitetura trata "pagamento" como um serviço separado, acoplado à API; web e mobile não falam com o gateway (slide 9). O [ADR-BE-013](ADR-BE-013.md) deixou os planos prontos no código, mas assinar hoje é um "pagamento simulado" (P-14): o `PUT /assinatura` confirma a assinatura sem cobrar.

O público-alvo é jovem (15 a 24 anos) e muito sensível a preço: 63% dos inscritos no Enem 2025 pediram isenção da taxa, parte é menor de idade e parte não tem cartão de crédito. O ticket é tão baixo que qualquer taxa fixa alta inviabiliza o Basic.

Qual gateway e qual integração usar para transformar o pagamento simulado em pagamento real, sem perder o posicionamento de preço baixo e sem bloquear quem paga por Pix?

## Fatores da Decisão

- **Margem no ticket baixo**: a taxa (percentual + tarifa fixa) precisa caber em R$ 8 sem comer o Basic;
- **Pix como caminho principal**: parte do público não tem cartão; o gateway precisa cobrir Pix bem, de preferência com recorrência;
- **Escopo de projeto acadêmico**: equipe pequena, sem CNPJ, operando em modo teste; prioridade é entregar o fluxo ponta a ponta com menos código possível;
- **Desacoplamento**: a API continua dona das regras, planos e login (slide 9); o gateway é um fornecedor que pode trocar;
- **Fonte da verdade**: o status da assinatura precisa ficar coerente mesmo quando a cobrança falha ou acontece fora do nosso app (renovação automática, cancelamento no gateway, estorno);
- **LGPD e menores**: o produto é usado por adolescentes; a solução precisa caber nas regras brasileiras atuais (LGPD, Enunciado CD/ANPD nº 1/2023 e Lei nº 15.211/2025, "ECA Digital").

## Opções Consideradas

- **Opção 1: Stripe** (Checkout hospedado + Customer Portal + Billing + webhooks);
- **Opção 2: Mercado Pago** (Preapproval para recorrência, forte em Pix nativo e no público jovem);
- **Opção 3: Pagar.me** (produto brasileiro da Stone, com Recorrência própria para cartão/Pix/boleto);
- **Opção 4: Asaas** (forte em Pix e boleto, foco em cobrança simples e microticket).

## Resultado da Decisão

Opção escolhida: **"Opção 1 – Stripe"**, com a integração **Stripe Checkout hospedado + Customer Portal + Billing + webhooks**.

Decisões específicas que vão junto:

- **Mix de meios de pagamento no MVP: cartão de crédito recorrente + Pix avulso por ciclo**.
  Cartão é a única recorrência verdadeiramente universal hoje. Pix "avulso por ciclo" quer dizer que a API gera uma cobrança Pix a cada período e, quando o Stripe nos avisa que o cliente pagou (via webhook — ver abaixo), a assinatura renova por mais um mês. **Boleto fica de fora**: a tarifa fixa de R$ 3,45 come 43% do Basic.
- **Pix Automático na conta Stripe BR fica como pendência "a confirmar"**. O Stripe anunciou Pix recorrente via mandato, mas a documentação atual deixa claro que o fluxo é para contas Stripe fora do Brasil. Enquanto não houver confirmação do comercial Stripe de que a conta BR cobre Pix Automático, usamos o modelo "Pix avulso por ciclo". **Gatilho de reavaliação**: se, após a confirmação do Stripe, a resposta for negativa ou com prazo longo, reabrir a discussão e considerar Mercado Pago (ver "Alternativas").
- **Integração: Stripe Checkout hospedado + Customer Portal**, não Payment Element embutido. Ou seja: quando o usuário clica em "assinar", é redirecionado para uma página do próprio Stripe (hospedada por eles), paga, e volta para o nosso site. Para gerenciar a assinatura (trocar cartão, cancelar, baixar fatura), abrimos o Customer Portal, também do Stripe. Isso reduz quase a zero o código de interface de pagamento, deixa a maior parte da conformidade PCI (regra de proteção de dados de cartão) com o Stripe e já traz Pix e boleto configuráveis.
- **Modelo de produto na Stripe: um `Product` ("Assinatura SAUF.BR") com três `Price`s** (Basic, Plus, Premium). Isso é o que habilita o botão nativo de "trocar de plano" dentro do Customer Portal.
- **Mobile é "reader app" no MVP**: a venda acontece só pela web. O app não vende e não menciona preço de assinatura; só consome o conteúdo liberado. Esse padrão é aceito pela Apple e pela Google e evita a comissão das lojas (15% a 30% + regras de link externo), que inviabilizaria o ticket. **A confirmar com a documentação atual das lojas antes de publicar o app.**
- **Upgrade de plano é imediato, com cobrança proporcional** (Basic → Plus/Premium). "Cobrança proporcional" é o `proration_behavior=always_invoice` do Stripe: calcula o valor pelos dias restantes do ciclo e já cobra. Casa com D-57 (só vale como ativa com pagamento confirmado).
- **Downgrade é agendado para o fim do ciclo**, sem gerar crédito. Para ticket de R$ 8, emitir crédito dá mais trabalho que valor. Usa o estado `CANCELAMENTO_AGENDADO` que já existe.
- **Cancelamento usa `cancel_at_period_end=true` do Stripe**: o acesso continua até `renovaEm`. Já está modelado hoje.
- **Reembolso: integral até 7 dias da primeira cobrança** (CDC art. 49, direito de arrependimento em compra online). Depois disso, cancelamento no fim do ciclo e sem reembolso proporcional. Operação feita via Dashboard ou API do Stripe.
- **Falha de cobrança: ativar Smart Retries + e-mails nativos do Stripe**. "Smart Retries" é a política padrão do Stripe que reenvia a cobrança nos melhores dias (padrão: 8 tentativas em 2 semanas). Enquanto está tentando, a assinatura fica no estado `past_due` do Stripe — ou seja, "cobrança falhou e estamos tentando de novo". **O acesso continua liberado durante as retries** e só é cortado se o Stripe desistir e cancelar a assinatura.
- **Webhooks como fonte da verdade do status**. "Webhook" é uma chamada HTTP que o Stripe faz para a nossa API toda vez que algo acontece com a assinatura (pagou, falhou, cancelou). O endpoint `POST /api/v1/webhooks/stripe` fica fora do `X-Api-Key` da ingestão ([ADR-BE-008](ADR-BE-008.md)) porque o Stripe assina cada webhook com HMAC-SHA256 (um código verificador baseado em segredo compartilhado); a verificação dessa assinatura é o controle de acesso dele. Eventos mínimos tratados:
  - `checkout.session.completed` — primeira compra concluída; cria/atualiza `assinatura`, guarda `stripe_customer_id` e `stripe_subscription_id`;
  - `invoice.payment_succeeded` — renovação paga; estende `renovaEm` e status `ATIVA`;
  - `invoice.payment_failed` — cobrança falhou; status `PAST_DUE`;
  - `customer.subscription.updated` — troca de plano, cancelamento agendado, mudança de status;
  - `customer.subscription.deleted` — assinatura encerrada no Stripe;
  - `charge.refunded` — reembolso efetivado.
- **Idempotência obrigatória no handler de webhook**. "Idempotente" quer dizer: se o Stripe reenviar o mesmo evento duas vezes (acontece), nosso código processa só uma. Nova tabela `stripe_event (id UUID PK = event.id, type, received_at, processed_at)`. O handler faz `INSERT ... ON CONFLICT DO NOTHING`; se o id já existe, responde 200 e sai sem reprocessar.
- **Postgres é a fonte da verdade do produto; Stripe é a fonte da verdade do pagamento**. Em caso de divergência, confia no Stripe e reconcilia via `subscriptions.retrieve`.
- **Menores de idade: simplificação com risco aceito**. No MVP, qualquer usuário cadastrado pode assinar. No checkout, um checkbox obrigatório — "Sou maior de 18 anos ou tenho autorização do meu responsável" — é registrado na assinatura com data/hora e versão do termo. **Não** criamos o fluxo "estudante × responsável" nem a coluna `responsavel_id` neste momento. Riscos aceitos e seus gatilhos de reavaliação estão descritos abaixo em "Riscos aceitos".

### Mudanças na modelagem

Sobre o que já existe hoje em `assinatura/` (ver [ADR-BE-013](ADR-BE-013.md)):

- **Tabela `assinatura` ganha**:
  - `stripe_customer_id` (nullable até a primeira compra);
  - `stripe_subscription_id` (unique, nullable);
  - `cancelar_no_fim_do_periodo` (boolean), para refletir o estado do Stripe;
  - `aceite_termo_maior_idade` (boolean) e `aceite_versao_termo` (texto curto), registrando o checkbox do checkout.
- **Enum `StatusAssinatura` passa a ter**: `INCOMPLETA`, `ATIVA`, `PAST_DUE`, `CANCELAMENTO_AGENDADO`, `CANCELADA`. Mapeia um-para-um com `subscription.status` do Stripe. "Incompleta" é a janela entre criar a sessão de checkout e o Stripe confirmar a cobrança.
- **Enum `Plano` continua no código (D-54)**, mas cada valor passa a conhecer também o `stripePriceId` por ambiente (test/live). Configuração via `application.yml` e variáveis de ambiente; o `Price` não vai para o banco.
- **Nova tabela `stripe_event`** para idempotência (descrito acima).
- **Função de autorização**: `assinatura.cobre(RecursoPlano)` passa a considerar status efetivo: `ATIVA` libera; `PAST_DUE` com `renovaEm >= hoje` libera (acesso durante retries); `CANCELAMENTO_AGENDADO` libera até `renovaEm`; demais bloqueiam.
- **`UsuarioLogado` continua provisório (D-55)**. Nada no módulo de assinatura muda quando o login real existir: basta `usuarioId` passar a vir dele. A nova coluna de aceite do termo não exige dados que o cadastro atual não tenha.

### Impacto nos custos (ordem de grandeza)

Taxas usadas: cartão nacional **3,99% + R$ 0,39** e Pix **1,19%**, conforme `stripe.com/br/pricing` consultado em 2026-10-31. **A confirmar no contrato comercial**: taxa adicional de Billing para assinaturas (comum em outras geografias, ~0,5%), que a tabela pública BR não lista separadamente.

| Plano | Via cartão | Via Pix |
|---|---|---|
| Basic R$ 8  | ~R$ 0,71 (~8,9%) | ~R$ 0,10 (~1,2%) |
| Plus R$ 12  | ~R$ 0,87 (~7,3%) | ~R$ 0,14 (~1,2%) |
| Premium R$ 15 | ~R$ 0,99 (~6,6%) | ~R$ 0,18 (~1,2%) |

Chargeback (contestação do cliente no banco): **R$ 55,00** por caso. Em ticket de R$ 8–15, um único chargeback consome ~4 a 7 assinaturas pagas. Esse número volta no "risco aceito" de menores.

### Consequências Positivas

- Pagamento real sem reescrever a modelagem: encaixa nos estados que o [ADR-BE-013](ADR-BE-013.md) já previu (`ATIVA`, `CANCELAMENTO_AGENDADO`) e só acrescenta `PAST_DUE`, `INCOMPLETA` e `CANCELADA`;
- Checkout, Portal e dunning (sequência automática de retries e e-mails em caso de falha) prontos; o time escreve só o handler de webhook, o mapeamento de `Price` e a função de autorização;
- A certificação PCI do nosso lado fica no mínimo (SAQ-A: cartão não trafega pelo nosso servidor);
- Pix via Stripe cobre o público sem cartão por 1,19% (bem abaixo do cartão), compatível com o ticket baixo do slide 14;
- Mobile como reader app evita 15% a 30% de comissão das lojas no MVP;
- A API continua sendo a única porta de acesso ([ADR-BE-002](ADR-BE-002.md)); o front e o app não conhecem o Stripe;
- Trocar de gateway no futuro é limitado ao módulo `assinatura/` e ao handler de webhook.

### Consequências Negativas

- **Dependência de fornecedor estrangeiro**: Stripe define as regras, os preços e o roadmap. Pix Automático no Brasil ainda não está confirmado para contas BR;
- **Sem CNPJ, só modo teste**: o MVP não processa dinheiro de verdade. Produção depende de abrir conta real (fora deste ADR);
- **Boleto fica de fora**: perde-se o público que paga só por boleto. É um custo deliberado de preço baixo;
- **Checkout hospedado**: o usuário sai do nosso domínio para pagar e volta. Pode gerar atrito de confiança no primeiro acesso;
- **Reader app no mobile**: perde-se conversão via loja (botão de assinar dentro do app). Aceito no MVP.

### Riscos aceitos

- **Validade do contrato com menor (Código Civil)**: um menor de 16 anos é absolutamente incapaz; entre 16 e 18, relativamente incapaz. O checkbox "sou maior / tenho autorização" **não** cria um contrato juridicamente seguro: se o responsável não autorizou, o contrato pode ser anulado. **Gatilho de reavaliação**: implementar o fluxo "estudante × responsável" com cadastro do responsável no Customer do Stripe.
- **Chargeback por pais que não reconhecem a cobrança**: cada caso custa R$ 55 ao projeto, independentemente do valor da assinatura. **Gatilho de reavaliação**: se a taxa de chargeback passar de 1% das transações do mês, implementar o fluxo responsável e revisar a comunicação do checkout.
- **LGPD + ANPD sobre menores**: o Enunciado CD/ANPD nº 1/2023 esclarece que o tratamento de dados de crianças e adolescentes pode ser feito por qualquer das bases legais do art. 7º ou 11 da LGPD, desde que respeite e priorize o **melhor interesse** da criança/adolescente (art. 14 da LGPD); o consentimento do responsável deixou de ser a única base legal admitida. Para o SAUF.BR, isso significa que o cadastro de um adolescente pode se apoiar em outra base (ex.: execução de contrato), **desde que a coleta seja mínima** e as proteções do produto sejam razoáveis para o perfil.
- **Lei nº 15.211/2025 ("ECA Digital")**: em vigor desde 17/03/2026, obriga plataformas digitais a adotar verificação de idade, segurança por padrão no nível máximo para contas de menores, ferramentas de supervisão, proibição de perfilamento publicitário com dados de menores e remoção rápida de conteúdo prejudicial. Em contas de pessoas até 16 anos, a lei exige vínculo a um responsável para acesso a redes sociais. O SAUF.BR não é rede social, mas tem features sociais (ex.: ranking do plano Premium). **Decisão deste ADR**: no MVP, nenhuma feature social expõe menores a outros usuários além de ranking anonimizado. Classificação do SAUF.BR frente ao ECA Digital, verificação de idade formal e vínculo com responsável ficam como pendência (ver "Pendências") e devem ser revistos antes de produção.

## Prós e Contras das Opções

### Opção 1 - Stripe (Checkout + Portal + Billing + webhooks)

- Bom, porque Checkout hospedado e Customer Portal cobrem o ciclo todo (comprar, trocar de plano, cancelar, baixar fatura) sem UI nossa;
- Bom, porque documentação e SDK Java (`stripe-java`) são de referência; webhook e idempotência são padrão da indústria;
- Bom, porque Smart Retries + e-mails de dunning já estão prontos (não precisamos escrever cobrança por falha);
- Ruim, porque Pix Automático recorrente em conta BR ainda não está confirmado na documentação atual (a confirmar com o comercial Stripe);
- Ruim, porque é fornecedor estrangeiro, exposto a mudança de preço e regras de câmbio em produção.

### Opção 2 - Mercado Pago

- Bom, porque é brasileiro, muito presente no bolso do público jovem e tem Pix recorrente nativo via `preapproval`;
- Bom, porque tem forte reconhecimento de marca no checkout, o que reduz atrito;
- Ruim, porque a experiência de desenvolvedor (DX), as libs Java e as features de Customer Portal/dunning são menos maduras que o Stripe;
- Ruim, porque exigiria escrever mais UI nossa (gestão de cartão, histórico de faturas), o que empurra o escopo acadêmico para fora do MVP.

### Opção 3 - Pagar.me (Stone)

- Bom, porque é brasileiro, cobre cartão, Pix e boleto com recorrência nativa, e a Stone tem presença consolidada no mercado;
- Bom, porque pricing e integração local evitam câmbio e documentação em inglês;
- Ruim, porque documentação e exemplos públicos são menos completos que Stripe/Mercado Pago;
- Ruim, porque não há equivalente pronto ao Customer Portal — teríamos que construir a tela de gestão de assinatura.

### Opção 4 - Asaas

- Bom, porque é forte em Pix e boleto recorrente, com API simples e barata para microticket;
- Bom, porque atende bem o cenário "sem cartão";
- Ruim, porque não tem Checkout hospedado comparável ao Stripe (mais código a escrever);
- Ruim, porque features de SaaS billing (proration automática, dunning com ML) são mais limitadas.

## Pendências

Pendências a abrir no `sauf-api/docs/DECISOES.md` (números a atribuir pelo responsável do diário):

- **Pix Automático na conta Stripe BR**: confirmar com o comercial Stripe se e quando o fluxo de mandato Pix fica disponível para contas BR. Enquanto pendente, vale o modelo "Pix avulso por ciclo".
- **Billing fee adicional para assinatura (BR)**: a tabela pública `stripe.com/br/pricing` não lista separadamente a tarifa de Billing comum em outras geografias (~0,5%). Confirmar no contrato antes de produção.
- **Regras atuais da Apple e da Google para "reader app"**: validar com a versão vigente das App Review Guidelines e da Google Play Policy antes de publicar o app em produção.
- **LGPD / Lei 15.211/2025 ("ECA Digital")**: avaliar se o SAUF.BR é considerado plataforma com público infantojuvenil para fins da Lei 15.211/2025 e, se for, mapear verificação de idade, segurança por padrão, vínculo com responsável para contas até 16 anos e proibição de perfilamento publicitário.

## Links

- `sauf-api/docs/DECISOES.md`: D-54, D-55, D-57; P-14
- Estudo de mercado SAUF.BR, slides 9 (arquitetura), 13 (receita) e 14 (planos)
- Relaciona-se com: [ADR-BE-002](ADR-BE-002.md), [ADR-BE-008](ADR-BE-008.md), [ADR-BE-013](ADR-BE-013.md), [ADR-006](../front/ADR-006.md)
- Documentação externa (consultada em 2026-10-31):
  - [Stripe BR — Preços](https://stripe.com/br/pricing)
  - [Stripe Docs — Accept a recurring Pix payment](https://docs.stripe.com/payments/pix/accept-a-recurring-payment)
  - [Stripe Changelog — Pix recurring payments support (2026-04-22)](https://docs.stripe.com/changelog/dahlia/2026-04-22/pix-recurring-payments-support)
  - [Stripe Docs — Webhooks](https://docs.stripe.com/webhooks)
  - [Stripe Docs — Upgrade and downgrade subscriptions](https://docs.stripe.com/billing/subscriptions/upgrade-downgrade)
  - [Stripe Docs — Customer management (Customer Portal)](https://docs.stripe.com/customer-management)
- Base legal (Brasil):
  - [Enunciado CD/ANPD nº 1, de 22 de maio de 2023](https://bibliotecadigital.mj.gov.br/handle/1/10215)
  - [Lei nº 15.211/2025 ("ECA Digital"), em vigor desde 17/03/2026](https://ecadigital.com.br/eca-digital)
  - CDC, Lei nº 8.078/1990, art. 49 (direito de arrependimento em compra online)
