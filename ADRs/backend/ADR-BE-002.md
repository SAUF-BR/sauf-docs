# ADR-BE-002 - API própria como única porta de acesso; Supabase só como banco Postgres

- Status: aceito
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-06 (login próprio decidido em 2026-10-21)
- História Técnica: D-01 a D-04 e D-55 do `docs/DECISOES.md` do `sauf-api`

## Contexto e Declaração do Problema

O Supabase oferece, além do Postgres, uma API automática sobre as tabelas (Data API), autenticação (Supabase Auth) e chaves para o front acessar o banco direto. Isso poderia dispensar parte da API Java. Por outro lado, as regras de negócio do SAUF (precedência entre site e Censo, cálculo das notas de corte, assinaturas) precisam ficar num lugar só, testado e fácil de apresentar.

Qual papel o Supabase deve ter, e por onde o front e o scraper acessam os dados?

## Fatores da Decisão

- Regras de negócio concentradas num único lugar;
- Arquitetura clara para documentar (C4) e defender na banca;
- Segurança: nenhuma tabela exposta direto na internet, nenhum segredo no front;
- Independência de fornecedor: o time nunca tinha usado Supabase.

## Opções Consideradas

- Opção 1: Supabase só como Postgres gerenciado; front e scraper falam só com a API; login próprio na API
- Opção 2: Front lê o catálogo direto pela Data API do Supabase e usa o Supabase Auth; a API Java fica só com a ingestão
- Opção 3: Misto: API para tudo, mas login pelo Supabase Auth

## Resultado da Decisão

Opção escolhida: "Opção 1 - Supabase só como Postgres", porque mantém as regras, o login e o acesso aos dados na API própria, com um único contrato documentado no Swagger.

- O front (`sauf-web-front`) fala **só** com a API (D-02);
- O scraper também só envia dados pela API ([ADR-SC-002](../scraper/ADR-SC-002.md));
- A **Data API do Supabase fica desligada** ("Enable Data API" desmarcado), para as tabelas não ficarem expostas;
- A conexão é por variáveis de ambiente (`SAUF_DB_URL`, `SAUF_DB_USER`, `SAUF_DB_PASSWORD`), sem nada específico do Supabase no código. Qualquer Postgres serve (D-03);
- Segredos ficam no `.env`, fora do Git (D-04);
- Cadastro, login, logout e `GET /me` serão da própria API (D-55). Até o login existir, um usuário provisório fixo (`comum/UsuarioLogado.java`) atende as rotas de assinatura.

### Consequências Positivas

- Uma única fonte de regras, coberta pelos testes da API;
- O front não guarda nenhuma chave de banco;
- Trocar o Supabase por outro Postgres (ou por um container local, [ADR-BE-004](ADR-BE-004.md)) não muda o código;
- O desenho C4 fica simples: front → API → banco, scraper → API.

### Consequências Negativas

- O login precisa ser implementado do zero (Spring Security, hash de senha, cookie da [ADR-005](../front/ADR-005.md), CSRF), o que o Supabase Auth daria pronto;
- Toda tela nova exige uma rota nova na API, mesmo para leituras simples;
- O plano gratuito do Supabase pausa o projeto após 7 dias sem uso.

## Prós e Contras das Opções

### Opção 1 - Supabase só como Postgres

- Bom, porque concentra regras e acesso na API;
- Bom, porque não depende de recursos específicos do Supabase;
- Ruim, porque o login é trabalho do time.

### Opção 2 - Front direto no Supabase

- Bom, porque as leituras do catálogo sairiam sem código no backend;
- Ruim, porque as regras ficariam divididas entre API, políticas de RLS do banco e front;
- Ruim, porque expõe as tabelas na internet e acopla o front ao Supabase.

### Opção 3 - API + Supabase Auth

- Bom, porque o login viria pronto;
- Ruim, porque a API teria que validar tokens de um serviço externo, e o modelo de usuário ficaria dividido entre os dois.

## Links

- `sauf-api/docs/DECISOES.md`: D-01, D-02, D-03, D-04, D-55; pendência P-10
- Relaciona-se com: [ADR-005](../front/ADR-005.md) (token em cookie HttpOnly)
- Relaciona-se com: [ADR-BE-004](ADR-BE-004.md) (bancos de desenvolvimento)
- Guia: `docs/guia-supabase.md` (pasta do projeto)
