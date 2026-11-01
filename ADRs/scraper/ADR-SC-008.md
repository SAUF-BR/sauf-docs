# ADR-SC-008 - Textos "Sobre" e "Mercado de Trabalho" como texto puro, com reconhecimento de títulos

- Status: aceito (formato provisório, D-37)
- Decisores: Carlos Eduardo Souza Favarão
- Data: 2026-10-09

## Contexto e Declaração do Problema

A tela de detalhe mostra o texto "Sobre o Curso" e o bloco "Onde você pode trabalhar". Esses textos vêm do HTML da página da universidade, que varia muito: às vezes o título está em negrito, às vezes é uma linha de texto, às vezes se chama "Campo de atuação". Guardar HTML cru exigiria sanitizar no front e traria lixo de layout.

Em que formato o scraper envia esses textos, e como reconhece cada seção?

## Fatores da Decisão

- Segurança no front (nada de HTML vindo de terceiros);
- Manter parágrafos e quebras de linha;
- Funcionar com variações de título entre páginas;
- Não cortar frases normais por engano.

## Opções Consideradas

- Opção 1: Texto puro com `\n` e `\n\n`, campos separados (`sobre` e `mercadoTrabalho`), títulos reconhecidos por regra
- Opção 2: HTML sanitizado
- Opção 3: Um campo só com o texto inteiro da página

## Resultado da Decisão

Opção escolhida: "Opção 1".

- **Texto puro, sem HTML**: `\n` entre linhas e `\n\n` entre parágrafos; `null` quando a página não tem a seção (D-37);
- **`sobre`** e **`mercadoTrabalho`** em campos separados (D-46). Só o site preenche; o Censo não mexe; o lote seguinte sobrescreve;
- **Títulos reconhecidos** (D-47): "Mercado de Trabalho" e "Campo de atuação", em negrito ou como linha exata de texto, com ou sem `:`, maiúsculas ou minúsculas, com ou sem acento. Frases como "O campo de atuação do biomédico é…" não são cortadas. Se a página tiver as duas seções, elas vão juntas no `mercadoTrabalho`;
- Blocos com dado pessoal (coordenação, depoimentos) são removidos antes ([ADR-SC-009](ADR-SC-009.md)).

### Consequências Positivas

- O front só renderiza texto, sem risco de XSS;
- O mesmo formato serve para todas as IES;
- O front esconde o bloco quando o campo vem `null`.

### Consequências Negativas

- Perde negrito, listas e links do texto original;
- Cada IES nova pode trazer um título diferente, que precisa entrar na regra;
- O formato é provisório e pode mudar (D-37).

## Prós e Contras das Opções

### Opção 1 - Texto puro em campos separados

- Bom, porque é seguro, simples e previsível;
- Ruim, porque perde a formatação.

### Opção 2 - HTML sanitizado

- Bom, porque mantém a formatação;
- Ruim, porque exige sanitização nos dois lados e traz estilos do site.

### Opção 3 - Um campo só

- Bom, porque não exige reconhecer títulos;
- Ruim, porque o front não consegue montar os blocos do protótipo.

## Links

- `sauf-scraper/src/sauf_scraper/parsers/uem.py`
- `sauf-api/docs/DECISOES.md`: D-37, D-46, D-47
- Relaciona-se com: [ADR-SC-009](ADR-SC-009.md), [ADR-BE-010](../backend/ADR-BE-010.md)
