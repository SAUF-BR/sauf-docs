# SAUF.BR — Documentação

**SAUF.BR** (Serviço de Admissão para Universidades e Faculdades) é uma plataforma de orientação acadêmica para quem vai entrar no ensino superior. Ela reúne num lugar só, a partir de dados oficiais, as informações que hoje estão espalhadas entre o site de cada universidade, o Censo da Educação Superior (INEP), o IGC e o Sisu (MEC):

- **cursos e universidades**, com turno, duração, vagas, gratuidade, texto do curso e link para a página oficial;
- **nota MEC** (IGC) da instituição e **nota de corte do Sisu** por curso, sempre com a fonte e o ano;
- **formas de ingresso** e **calendário de vestibulares**;
- **simulados** do ENEM e de vestibulares, para assinantes.

| Nível de acesso | O que pode fazer |
|---|---|
| Visitante (sem login) | Navegar por cursos, universidades, formas de ingresso e calendário |
| Logado (conta gratuita) | Perfil, favoritos, notificações e teste vocacional |
| Assinante (Basic, Plus ou Premium) | Simulados, acompanhamento e cronômetro |

Projeto de conclusão do curso de Engenharia de Software (ESOFT-N-A, 8º semestre), feito por Antonio Ferreira, Carlos Favarão, Gabrielle Morente, Heloísa Scarante, Karen Tanaka e Leonardo Souza.

## Arquitetura

```
                 HTTPS /api/v1                 JDBC
 Estudante ──▶ sauf-web-front ──────────▶ sauf-api ──────────▶ PostgreSQL 17
 (navegador)   React 19 (SPA)             Spring Boot 4        Docker (local) · Supabase
                                             ▲
                                             │ POST /api/v1/ingestao/** (X-Api-Key)
                                             │
 Fontes oficiais ──────────────────▶ sauf-scraper
 sites das IES, Censo, IGC, Sisu     Python (CLI)
```

- A **API é a única porta de entrada do banco**. O front nunca acessa o banco, e o Supabase é usado só como Postgres, sem o Auth dele.
- O **scraper envia lotes** e a API decide o que gravar.
- Quando as fontes discordam, **vale o site da universidade**. O Censo só completa o que o site não mostra.

Mais detalhes no [System Design](SAUF.BR%20—%20System%20Design.pdf) e nas [ADRs](ADRs/README.md).

## Repositórios

Todos ficam na organização [SAUF-BR](https://github.com/SAUF-BR) do GitHub.

| Repositório | O que é | Stack |
|---|---|---|
| [sauf-web-front](https://github.com/SAUF-BR/sauf-web-front) | Aplicação web que o estudante usa: telas de cursos, universidades, perfil, assinatura e simulados. Algumas telas ainda usam dados simulados (mock). | Vite 8, React 19, TypeScript, React Router 7, TanStack Query 5, axios, React Hook Form + Zod, Sass com CSS Modules |
| [sauf-api](https://github.com/SAUF-BR/sauf-api) | API REST com as regras de negócio. Serve o catálogo, as universidades, as notas e a assinatura, e recebe os dados do scraper. Os contratos de erro, de paginação e de datas são os mesmos para o sistema todo. | Java 21, Spring Boot 4, Maven, Spring Data JPA, Bean Validation, Flyway, PostgreSQL 17, springdoc (Swagger), JUnit + H2 nos testes |
| [sauf-scraper](https://github.com/SAUF-BR/sauf-scraper) | Coletor de dados em linha de comando. Lê o site de cada universidade (um *spider* por instituição) e importa o Censo, o IGC e o Sisu a partir de arquivos oficiais. Envia tudo para a API. | Python 3.12, uv, Requests, BeautifulSoup 4, Pydantic, openpyxl, pytest, ruff |
| **sauf-docs** (este) | Documentação do projeto: ADRs, diagramas, System Design, estudo de mercado e artefatos da disciplina. | Markdown, Mermaid e PDF |

## Como rodar

A ordem é: **banco → API → dados (scraper) → front**. Os comandos abaixo são para Windows (PowerShell). Cada repositório tem um README com o passo a passo completo.

### Pré-requisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (banco local);
- [JDK 21](https://adoptium.net/) e [Maven](https://maven.apache.org/);
- [uv](https://docs.astral.sh/uv/) (instala o Python 3.12 sozinho);
- [Node.js](https://nodejs.org/) **20.19 ou superior** (o Vite 8 não roda no Node 18) e npm.

### 1. Banco e API (`sauf-api`)

```powershell
cd sauf-api
copy .env.example .env        # preencha o SAUF_API_KEY
docker compose up -d          # Postgres 17 local na porta 5433
mvn spring-boot:run           # o Flyway cria as tabelas na primeira vez
```

- Swagger: <http://localhost:8080/api/v1/swagger>
- Teste rápido: <http://localhost:8080/api/v1/areas>
- Testes: `mvn test` (usa H2, não precisa do Docker)
- Para zerar o banco local: `docker compose down -v`

> O Supabase é compartilhado e só recebe o que já está na `main`. Cada pessoa testa as migrations no próprio Postgres local.

### 2. Dados (`sauf-scraper`)

O banco começa vazio. Para popular, o `SAUF_API_KEY` do `.env` do scraper precisa ser igual ao da API, e o `SAUF_API_BASE_URL` precisa ser `http://localhost:8080`, sem `/api/v1` no fim. Os microdados do Censo e as planilhas do IGC e do Sisu ficam em `fontes/`, que não vai para o Git: baixe-as à mão nos sites oficiais.

```powershell
cd sauf-scraper
uv sync                       # cria a .venv com as versões do uv.lock
copy .env.example .env

uv run sauf-scraper censo                       # 1º: instituições e cursos do Censo (INEP)
uv run sauf-scraper --ies uem                   # 2º: cursos do site da UEM
uv run sauf-scraper igc --arquivo fontes/IGC_2023.xlsx      # 3º: nota MEC (IGC)
uv run sauf-scraper sisu --arquivo fontes/<planilha-do-sisu>.xlsx   # 4º: notas de corte
```

Outros comandos úteis:

```powershell
uv run sauf-scraper --listar              # spiders registrados
uv run sauf-scraper --ies uem --dry-run   # grava o lote em output/ sem enviar para a API
uv run sauf-scraper --ies uem --mostrar   # mostra o resultado na tela
uv run pytest                             # testes
uv run ruff check . ; uv run ruff format .   # lint e formatação
```

Se a coleta da UEM devolver `422`, o `censo` ainda não foi rodado. Se devolver `401`, as chaves dos dois `.env` estão diferentes.

### 3. Front (`sauf-web-front`)

```powershell
cd sauf-web-front
npm install
copy .env.example .env
npm run dev                   # http://localhost:5173
```

> A API responde em `/api/v1`, então use `VITE_API_URL=http://localhost:8080/api/v1`. O `.env.example` do front ainda traz `/api`.

| Comando | O que faz |
|---|---|
| `npm run dev` | Servidor local com hot reload |
| `npm run build` | Build de produção em `dist/` |
| `npm run preview` | Serve o build de produção localmente |
| `npm run lint` | ESLint |

## O que tem neste repositório

| Arquivo ou pasta | Conteúdo |
|---|---|
| [ADRs/](ADRs/README.md) | Decisões de arquitetura do front (`ADR-0xx`), do backend (`ADR-BE-0xx`) e do scraper (`ADR-SC-0xx`), com índice, template e questões em aberto |
| [SAUF.BR — System Design.pdf](SAUF.BR%20—%20System%20Design.pdf) | Contexto, escopo, requisitos, arquitetura, tecnologias, modelagem, fluxo principal e segurança |
| [SAUF.BR — Estudo de Mercado (1).pdf](SAUF.BR%20—%20Estudo%20de%20Mercado%20(1).pdf) | Problema, mercado, público-alvo, concorrentes, diferenciais e SWOT |
| [diagramas-uml.md](diagramas-uml.md) | Casos de uso, diagrama de classes e DER, em Mermaid |
| [diagramas-atividades.md](diagramas-atividades.md) | Diagramas de atividades dos principais fluxos (cadastro, login, teste vocacional, assinatura, simulado...) |
| [FIGMA.md](FIGMA.md) | Link do protótipo no Figma |
| `SAUF_EspecificacaoCasosDeUso-V2.xlsx` | Especificação dos casos de uso |
| `Indicadores_SAUF.xlsx` | Indicadores das sprints (tarefas planejadas e concluídas) e horas por integrante, extraídos do Jira |
| [Artefatos antigos/](Artefatos%20antigos/) | Versões anteriores dos diagramas (casos de uso, classes, sequência, DER) e horas do Jira |

## Convenções do time

- Commits no formato `TR-<número> Descrição`, com o número do card do Jira.
- Toda decisão de arquitetura nova vira uma ADR em [ADRs/](ADRs/README.md), a partir do [template](ADRs/template.md).
- Segredos ficam só no `.env` de cada projeto, que nunca vai para o Git.
- Nenhum dado pessoal coletado pelo scraper (LGPD): nada de nomes, e-mails ou telefones das páginas das universidades.
