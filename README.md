# UC04 -- Visualizar Tela Inicial

## Descrição

Este caso de uso tem como objetivo apresentar ao usuário uma visão geral
do sistema logo após o acesso. A tela inicial reúne informações
resumidas sobre os cursos e instituições cadastradas, permitindo que o
usuário tenha uma visão rápida das oportunidades disponíveis antes de
realizar pesquisas ou consultas mais específicas.

## Objetivo

Exibir informações atualizadas e resumidas de cursos e instituições
disponíveis na plataforma.

## Ator Principal

-   Usuário

## Pré-condições

-   O sistema deve estar disponível e operacional.
-   A base de dados deve estar acessível.
-   Devem existir informações cadastradas de cursos e/ou instituições.

## Pós-condições

-   A tela inicial é exibida com sucesso.
-   As informações apresentadas correspondem aos dados mais recentes
    disponíveis no sistema.

## Gatilho

-   O usuário acessa a aplicação, seja por meio da URL principal ou após
    autenticação (quando aplicável).

## Fluxo Principal

1.  O usuário acessa o sistema.
2.  O sistema inicia o carregamento das informações necessárias para a
    tela inicial.
3.  O sistema consulta os dados dos cursos disponíveis.
4.  O sistema consulta os dados das instituições cadastradas.
5.  O sistema organiza as informações para exibição.
6.  O sistema apresenta ao usuário a tela inicial contendo:
    -   Lista resumida de cursos;
    -   Informações básicas das instituições;
    -   Quantidade de cursos disponíveis (quando aplicável);
    -   Destaques ou informações relevantes cadastradas.
7.  O usuário visualiza os dados apresentados.

## Fluxos Alternativos

### A1. Falha na consulta dos dados

1.  Durante o carregamento das informações, ocorre uma falha de
    comunicação com a base de dados ou serviço responsável.
2.  O sistema interrompe o carregamento das informações.
3.  O sistema exibe uma mensagem informando que os dados não puderam ser
    carregados no momento.
4.  O usuário pode tentar atualizar a página posteriormente.

### A2. Nenhum dado cadastrado

1.  O sistema realiza a consulta dos cursos e instituições.
2.  Nenhum registro é encontrado.
3.  O sistema exibe uma mensagem informando que não há cursos ou
    instituições disponíveis no momento.

## Regras de Negócio

-   RN01: Apenas informações ativas devem ser exibidas na tela inicial.
-   RN02: Os dados apresentados devem refletir o estado atual do
    sistema.
-   RN03: O carregamento da tela inicial deve ocorrer de forma
    automática ao acessar a aplicação.
-   RN04: Em caso de indisponibilidade dos dados, o sistema deve
    informar o usuário sem expor detalhes técnicos do erro.

## Entradas

-   Acesso do usuário ao sistema.

## Saídas

-   Tela inicial contendo informações resumidas de cursos e
    instituições.
-   Mensagens de aviso ou erro, quando aplicável.

## Critérios de Aceitação

-   A tela inicial deve ser exibida após o acesso ao sistema.
-   Os cursos disponíveis devem ser apresentados corretamente.
-   As instituições cadastradas devem ser apresentadas corretamente.
-   O sistema deve informar ao usuário quando houver indisponibilidade
    temporária.
-   O sistema deve exibir mensagem adequada quando não existirem dados
    cadastrados.

    <img width="386" height="805" alt="image" src="https://github.com/user-attachments/assets/684d0d88-ee83-4f6a-bcec-eb051b13cecd" />

