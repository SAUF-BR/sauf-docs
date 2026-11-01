# Diagramas de Atividades

Convertidos do board Miro "Meu primeiro board" (grupo "Diagrama de atividades", 13 frames). As raias (swimlanes) do board viraram `subgraph`; o início é `((Início))`, o fim é `(((Fim)))` e as decisões são losangos.

1. [Cadastrar Estudante](#1-cadastrar-estudante)
2. [Realizar Login](#2-realizar-login)
3. [Esqueci Minha Senha](#3-esqueci-minha-senha)
4. [Realizar Teste Vocacional](#4-realizar-teste-vocacional)
5. [Visualizar Resultado do Teste Vocacional](#5-visualizar-resultado-do-teste-vocacional)
6. [Favoritar Curso](#6-favoritar-curso)
7. [Favoritar Universidade](#7-favoritar-universidade)
8. [Assinatura](#8-assinatura)
9. [Realizar Pagamento](#9-realizar-pagamento)
10. [Emitir Nota Fiscal](#10-emitir-nota-fiscal)
11. [Alterar Plano de Assinatura](#11-alterar-plano-de-assinatura)
12. [Cancelar Assinatura](#12-cancelar-assinatura)
13. [Simulado](#13-simulado)

---

## 1. Cadastrar Estudante

```mermaid
flowchart LR
    subgraph Estudante["Estudante"]
        I((Início))
        A1["Acessar #quot;Criar Conta#quot;"]
        A2["Preencher os dados"]
        A3["Selecionar Continuar"]
        D2{"Aceita?"}
        A4["Aceitar e criar a conta"]
        F2(((Fim)))
    end

    subgraph Sistema["Sistema"]
        S1["Exibir Formulário de Cadastro"]
        S2["Validar os dados informados"]
        D1{"Email já cadastrado?"}
        S3["Exibir erro #quot;email em uso#quot;"]
        F1(((Fim)))
        S4["Exibir termos de uso"]
        S5["Criar conta na base"]
        S6["Enviar e-mail de confirmação"]
        S7["Autenticar e redirecionar"]
        F3(((Fim)))
    end

    I --> A1 --> S1 --> A2 --> A3 --> S2 --> D1
    D1 -->|SIM| S3 --> F1
    D1 -->|NÃO| S4 --> D2
    D2 -->|NÃO| F2
    D2 -->|SIM| A4 --> S5 --> S6 --> S7 --> F3
```

---

## 2. Realizar Login

```mermaid
flowchart LR
    subgraph Estudante["Estudante"]
        I((Início))
        A1["Acessar Tela de Login"]
        D0{" "}
        A2["Informar E-mail e senha"]
        A3["Selecionar Entrar"]
        A4["Selecionar entrar com Google"]
    end

    subgraph Sistema["Sistema"]
        S1["Validar credenciais"]
        D1{"Credenciais válidas?"}
        S2["Exibir #quot;E-mail ou senha inválidos#quot;"]
        F1(((Fim)))
        S3["Autenticar e criar sessão"]
        S4["Redirecionar página de origem"]
        F2(((Fim)))
    end

    subgraph Google["Google"]
        G1["Autenticar Google"]
    end

    I --> A1 --> D0
    D0 -->|E-MAIL| A2 --> A3 --> S1 --> D1
    D0 -->|GOOGLE| A4 --> G1 --> S3
    D1 -->|NÃO| S2 --> F1
    D1 -->|SIM| S3 --> S4 --> F2
```

---

## 3. Esqueci Minha Senha

```mermaid
flowchart LR
    subgraph Estudante["Estudante"]
        I((Início))
        A1["Selecionar esqueci minha senha"]
        A2["Informar E-mail"]
        A3["Acessar link do e-mail"]
        A4["Informar nova senha"]
    end

    subgraph Sistema["Sistema"]
        S1["Gerar token de recuperação"]
        S2["Enviar e-mail com o link"]
        S3["Salvar nova senha"]
        S4["Redirecionar para o login"]
        F(((Fim)))
    end

    I --> A1 --> A2 --> S1 --> S2 --> A3 --> A4 --> S3 --> S4 --> F
```

---

## 4. Realizar Teste Vocacional

```mermaid
flowchart LR
    subgraph Estudante["Estudante"]
        I((Início))
        A1["Acessar teste vocacional"]
        A2["Selecionar começar teste"]
        A3["Selecionar resposta"]
        A4["Finalizar teste"]
    end

    subgraph Sistema["Sistema"]
        S1["Exibir apresentação do teste"]
        D1{"Estudante autenticado?"}
        S2["Solicitar login/cadastro"]
        F1(((Fim)))
        S3["Exibir primeira pergunta"]
        S4["Registrar resposta"]
        D2{"Última pergunta?"}
        S5["Exibir próxima pergunta"]
        S6["Calcular resultado vocacional"]
        S7["Exibir resultado"]
        F2(((Fim)))
    end

    I --> A1 --> S1 --> A2 --> D1
    D1 -->|NÃO| S2 --> F1
    D1 -->|SIM| S3 --> A3 --> S4 --> D2
    D2 -->|NÃO| S5 --> A3
    D2 -->|SIM| A4 --> S6 --> S7 --> F2
```

---

## 5. Visualizar Resultado do Teste Vocacional

```mermaid
flowchart LR
    subgraph Estudante["Estudante"]
        I((Início))
        A1["Acessar resultado vocacional"]
    end

    subgraph Sistema["Sistema"]
        S1["Buscar resultado do estudante"]
        D1{"Resultado existe?"}
        S2["Exibir áreas e cursos recomendados"]
        F1(((Fim)))
        S3["Convidar a responder o teste vocacional"]
        F2(((Fim)))
    end

    I --> A1 --> S1 --> D1
    D1 -->|SIM| S2 --> F1
    D1 -->|NÃO| S3 --> F2
```

---

## 6. Favoritar Curso

```mermaid
flowchart LR
    subgraph Estudante["Estudante"]
        I((Início))
        A1["Selecionar #quot;Favoritar#quot; no curso"]
    end

    subgraph Sistema["Sistema"]
        D1{"Estudante autenticado?"}
        S1["Solicitar login/cadastro"]
        F1(((Fim)))
        D2{"Já favoritado?"}
        S2["Remover favorito"]
        S3["Registrar favorito (CursoFavorito)"]
        S4["Atualizar ícone de favorito"]
        F2(((Fim)))
    end

    I --> A1 --> D1
    D1 -->|NÃO| S1 --> F1
    D1 -->|SIM| D2
    D2 -->|SIM| S2 --> S4
    D2 -->|NÃO| S3 --> S4
    S4 --> F2
```

---

## 7. Favoritar Universidade

```mermaid
flowchart LR
    subgraph Estudante["Estudante"]
        I((Início))
        A1["Selecionar #quot;Favoritar#quot; na universidade"]
    end

    subgraph Sistema["Sistema"]
        D1{"Estudante autenticado?"}
        S1["Solicitar login/cadastro"]
        F1(((Fim)))
        D2{"Já favoritado?"}
        S2["Remover favorito"]
        S3["Registrar favorito (UniversidadeFavorita)"]
        S4["Atualizar ícone de favorito"]
        F2(((Fim)))
    end

    I --> A1 --> D1
    D1 -->|NÃO| S1 --> F1
    D1 -->|SIM| D2
    D2 -->|SIM| S2 --> S4
    D2 -->|NÃO| S3 --> S4
    S4 --> F2
```

---

## 8. Assinatura

```mermaid
flowchart LR
    subgraph Estudante["Estudante"]
        I((Início))
        A1["Acessar página de planos/assinaturas"]
        A2["Selecionar um dos planos disponíveis"]
        A3["Preencher dados de pagamento"]
        A4["Visualizar a confirmação de assinatura"]
        A5["Visualizar mensagem de erro"]
        F(((Fim)))
    end

    subgraph Sistema["Sistema"]
        S1["Exibir os planos disponíveis"]
        D1{"Estudante autenticado?"}
        S2["Solicitar login/cadastro"]
        F1(((Fim)))
        S3["Criar sessão de pagamento"]
        S4["Ativar a assinatura"]
        S5["Exibir mensagem de sucesso"]
        S6["Exibir mensagem de erro e sugere uma nova tentativa"]
    end

    subgraph Gateway["Gateway de Pagamento"]
        G1["Exibir formulário de pagamento"]
        G2["Processar as informações de pagamento"]
        D2{"Pagamento aprovado?"}
        G3["Retornar confirmação de pagamento"]
        G4["Retornar falha de pagamento"]
    end

    I --> A1 --> S1 --> A2 --> D1
    D1 -->|NÃO| S2 --> F1
    D1 -->|SIM| S3 --> G1 --> A3 --> G2 --> D2
    D2 -->|SIM| G3 --> S4 --> S5 --> A4 --> F
    D2 -->|NÃO| G4 --> S6 --> A5 --> F
```

---

## 9. Realizar Pagamento

```mermaid
flowchart LR
    subgraph Estudante["Estudante"]
        I((Início))
        A1["Selecionar plano + forma de pagamento"]
    end

    subgraph Gateway["Gateway Pagamento"]
        G1["Processar transação"]
        D1{"Pagamento aprovado?"}
    end

    subgraph Sistema["Sistema"]
        S1["Ativar assinatura"]
        S2["Exibir confirmação do pagamento"]
        S3["Emitir nota fiscal"]
        F1(((Fim)))
        S4["Exibir erro: pagamento recusado"]
        F2(((Fim)))
    end

    I --> A1 --> G1 --> D1
    D1 -->|SIM| S1 --> S2 --> S3 --> F1
    D1 -->|NÃO| S4 --> F2
```

---

## 10. Emitir Nota Fiscal

```mermaid
flowchart LR
    subgraph Sistema["Sistema"]
        I((Início))
        S1["Pagamento aprovado"]
        S2["Disponibilizar Nota para Download"]
        F1(((Fim)))
        S3["Exibir #quot;não foi possível emitir agora#quot;"]
        F2(((Fim)))
    end

    subgraph Gateway["Gateway Pagamento"]
        G1["Gerar nota no serviço fiscal"]
        D1{"Emissão: sucesso?"}
    end

    I --> S1 --> G1 --> D1
    D1 -->|SIM| S2 --> F1
    D1 -->|NÃO| S3 --> F2
```

---

## 11. Alterar Plano de Assinatura

```mermaid
flowchart LR
    subgraph Estudante["Estudante"]
        I((Início))
        A1["Selecionar novo plano"]
    end

    subgraph Sistema["Sistema"]
        D1{"Upgrade?"}
        S1["Cobrar diferença"]
        S2["Ativar novo plano"]
        S3["Agendar troca para próxima cobrança"]
        S4["Exibir confirmação"]
        F(((Fim)))
    end

    I --> A1 --> D1
    D1 -->|SIM| S1 --> S2 --> S4
    D1 -->|NÃO| S3 --> S4
    S4 --> F
```

---

## 12. Cancelar Assinatura

```mermaid
flowchart LR
    subgraph Estudante["Estudante"]
        I((Início))
        A1["Selecionar cancelar assinatura"]
        D1{"Confirmar?"}
        F2(((Fim)))
    end

    subgraph Sistema["Sistema"]
        S1["Pedir confirmação"]
        S2["Cancelar assinatura"]
        S3["Exibir confirmação"]
        F1(((Fim)))
    end

    I --> A1 --> S1 --> D1
    D1 -->|SIM| S2 --> S3 --> F1
    D1 -->|NÃO| F2
```

---

## 13. Simulado

```mermaid
flowchart LR
    subgraph Estudante["Estudante"]
        I((Início))
        A1["Acessar simulados"]
        A2["Iniciar simulado"]
        A3["Responder questão"]
        A4["Finalizar simulado"]
    end

    subgraph Sistema["Sistema"]
        D1{"Assinatura ativa?"}
        S1["Redirecionar para Assinatura"]
        F1(((Fim)))
        S2["Exibir questão"]
        S3["Registrar resposta"]
        D2{"Última questão?"}
        S4["Calcular pontuação"]
        S5["Exibir resultado do simulado"]
        F2(((Fim)))
    end

    I --> A1 --> D1
    D1 -->|NÃO| S1 --> F1
    D1 -->|SIM| A2 --> S2 --> A3 --> S3 --> D2
    D2 -->|NÃO| S2
    D2 -->|SIM| A4 --> S4 --> S5 --> F2
```
