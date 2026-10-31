# ADRs do SAUF-BR

Registro das decisões de arquitetura (Architecture Decision Records) do SAUF-BR: front (`sauf-web-front`), backend (`sauf-api`) e scraper (`sauf-scraper`).

Cada ADR registra **uma** decisão: o contexto, as opções avaliadas, a escolha e as consequências. Quando uma decisão muda, a ADR antiga não é apagada: ela passa para "substituído por" e uma ADR nova é criada.

## Convenções

- **Formato:** baseado no MADR, em português. Para uma ADR nova, copie o [template.md](template.md).
- **Numeração:**
  - front: `ADR-001`, `ADR-002`... (mantida como já era, para não quebrar referências como "ADR-010 do front");
  - backend: `ADR-BE-001`, `ADR-BE-002`...;
  - scraper: `ADR-SC-001`, `ADR-SC-002`...
- **Pastas:** `front/`, `backend/` e `scraper/`. Os links entre ADRs são relativos (ex.: `../backend/ADR-BE-010.md`).
- **Status:** `proposto` (em discussão), `aceito`, `rejeitado` ou `substituído por ADR-xxx`. Quando há uma ressalva, ela vai entre parênteses no próprio status.
- **Revisões:** ao comparar uma ADR com o código, a diferença é anotada numa seção "Revisão de AAAA-MM-DD" no fim da ADR, sem reescrever a decisão original.
- **Relação com o `DECISOES.md`:** o `sauf-api/docs/DECISOES.md` continua sendo o diário das decisões do dia a dia (D-xx) e das pendências (P-xx). As ADRs agrupam as decisões de arquitetura e apontam para os D-xx de origem.

## Front (`sauf-web-front`)

| ADR | Título | Status |
|---|---|---|
| [ADR-001](front/ADR-001.md) | Vite+React (SPA) em vez de Next.js | aceito |
| [ADR-002](front/ADR-002.md) | Organização de pastas por feature (domínio) em vez de por tipo de arquivo | aceito |
| [ADR-003](front/ADR-003.md) | React Query para estado de servidor | aceito |
| [ADR-004](front/ADR-004.md) | Estado global de autenticação com Zustand | proposto |
| [ADR-005](front/ADR-005.md) | Estratégia de autenticação e armazenamento do token JWT | aceito |
| [ADR-006](front/ADR-006.md) | Guards de rota para controle de acesso | proposto |
| [ADR-007](front/ADR-007.md) | Cliente HTTP centralizado com axios e interceptors | aceito |
| [ADR-008](front/ADR-008.md) | Validação de formulários (React Hook Form + Zod) | aceito |
| [ADR-009](front/ADR-009.md) | Estilização com CSS Modules em vez de styled-components | aceito |
| [ADR-010](front/ADR-010.md) | Contrato de comunicação com a API Java | proposto |
| [ADR-011](front/ADR-011.md) | Avaliação do Tailwind CSS para estilização | rejeitado |

> **Mudança de número:** na pasta `sauf-web-front/docs/ADRs`, o arquivo `ADR-009` era o Tailwind e o `ADR-011` era o CSS Modules, ao contrário da planilha de ADRs. Aqui a numeração segue a planilha: **ADR-009 = CSS Modules** e **ADR-011 = Tailwind**.

## Backend (`sauf-api`)

| ADR | Título | Status |
|---|---|---|
| [ADR-BE-001](backend/ADR-BE-001.md) | Java 21, Spring Boot 4 e Maven para a API | aceito |
| [ADR-BE-002](backend/ADR-BE-002.md) | API própria como única porta de acesso; Supabase só como banco Postgres | aceito |
| [ADR-BE-003](backend/ADR-BE-003.md) | Flyway para versionar o banco, com migrations nomeadas por data e hora | aceito |
| [ADR-BE-004](backend/ADR-BE-004.md) | Postgres local em Docker para desenvolvimento, H2 só nos testes e Supabase só para a main | aceito |
| [ADR-BE-005](backend/ADR-BE-005.md) | Catálogo em dois níveis: curso genérico e oferta (uma página do site = uma oferta) | aceito |
| [ADR-BE-006](backend/ADR-BE-006.md) | Curso genérico e áreas pela classificação Cine do Censo, com nome amigável | aceito (tabela de nomes amigáveis ainda não criada) |
| [ADR-BE-007](backend/ADR-BE-007.md) | UUID como chave primária e chaves naturais à parte | aceito |
| [ADR-BE-008](backend/ADR-BE-008.md) | Ingestão por rotas próprias com X-Api-Key, upsert idempotente e recusa de IES desconhecida | aceito (autenticação por X-Api-Key aguarda confirmação formal, P-06; inativação provisória, P-26) |
| [ADR-BE-009](backend/ADR-BE-009.md) | O site da universidade é a fonte de verdade; o Censo só complementa | aceito |
| [ADR-BE-010](backend/ADR-BE-010.md) | Contrato REST da API: /api/v1, formato da ADR-010, enums em minúsculas e campos sem fonte como null | aceito |
| [ADR-BE-011](backend/ADR-BE-011.md) | Nota MEC e notas de corte guardadas por ano e calculadas no backend | aceito |
| [ADR-BE-012](backend/ADR-BE-012.md) | Pacotes por domínio, DTOs como records e sem Lombok | aceito |
| [ADR-BE-013](backend/ADR-BE-013.md) | Planos de assinatura definidos no código e regras de vigência | aceito (pagamento simulado até existir gateway, P-14) |

## Scraper (`sauf-scraper`)

| ADR | Título | Status |
|---|---|---|
| [ADR-SC-001](scraper/ADR-SC-001.md) | Stack do scraper: Python 3.12, uv, Requests + BeautifulSoup e Pydantic | aceito |
| [ADR-SC-002](scraper/ADR-SC-002.md) | Scraper como container separado que envia lotes para a API (com dry-run em JSON) | aceito |
| [ADR-SC-003](scraper/ADR-SC-003.md) | Separação spider × parser × pipeline, parsers puros testados com fixtures e falha isolada por instituição | aceito |
| [ADR-SC-004](scraper/ADR-SC-004.md) | Coleta educada, dados oficiais do Censo e escopo restrito às IES com spider | aceito |
| [ADR-SC-005](scraper/ADR-SC-005.md) | Chave natural do curso, urlOrigem obrigatório e codigosEmec como lista | aceito |
| [ADR-SC-006](scraper/ADR-SC-006.md) | Tabela de casamento site × Censo versionada e confirmada à mão | aceito |
| [ADR-SC-007](scraper/ADR-SC-007.md) | Importadores gerais leem um arquivo oficial baixado à mão | aceito |
| [ADR-SC-008](scraper/ADR-SC-008.md) | Textos "Sobre" e "Mercado de Trabalho" como texto puro, com reconhecimento de títulos | aceito (formato provisório, D-37) |
| [ADR-SC-009](scraper/ADR-SC-009.md) | LGPD no scraper: não coletar dados pessoais e versionar fixtures anonimizadas | aceito |

## Questões em aberto

1. **[ADR-004](front/ADR-004.md), estado de autenticação:** o Zustand não foi instalado e hoje há duas fontes de sessão (React Query e Context). Opções: (a) criar o store em Zustand; (b) substituir a ADR por "sessão e assinatura no cache do React Query" e remover o `AuthProvider`; (c) manter o Context.
2. **[ADR-005](front/ADR-005.md), token:** a ADR decide cookie `HttpOnly`, mas o `client.ts` usa Bearer em `localStorage` (P-10). Seguir a ADR quando o login existir, ou substituí-la registrando o risco de XSS.
3. **[ADR-006](front/ADR-006.md):** o `SubscriberRoute` lê o Context (sempre `false`) em vez de consultar `GET /api/v1/assinatura`. Depende da questão 1.
4. **[ADR-010](front/ADR-010.md):** o backend já segue o contrato; falta passar de "proposto" para "aceito" com o time do front (P-19).
5. **Front:** o `.env.example` usa `VITE_API_URL=http://localhost:8080/api`, mas a API responde em `/api/v1` ([ADR-BE-010](backend/ADR-BE-010.md)).
6. **[ADR-BE-006](backend/ADR-BE-006.md):** como uma oferta sem código e-MEC encontra ou cria o curso genérico (P-05).
7. **[ADR-BE-012](backend/ADR-BE-012.md):** nome do pacote Java, `br.com.sauf_api` ou `br.com.saufbr.api` (P-22).
8. **Scraper, escolha das próximas IES:** ainda não há critério escrito (avaliação em `scraper/11-avaliacao-ies.md`: UFSC boa, UFPR fraca). Falta decidir se todas as IES precisam ser do Paraná. Quando decidido, vira uma ADR-SC nova.
