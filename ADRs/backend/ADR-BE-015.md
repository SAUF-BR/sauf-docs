# ADR-BE-015 - Testes organizados em subpastas por domínio

- Status: proposto
- Decisores: Karen
- Data: 2026-10-31
- História Técnica: [ADR-BE-012](ADR-BE-012.md)

## Contexto e Declaração do Problema

O código da API está organizado em um pacote por domínio, conforme a [ADR-BE-012](ADR-BE-012.md). À medida que novos testes são criados pelos integrantes da equipe, torna-se necessário definir uma organização que facilite sua localização e manutenção.

Onde cada classe de teste deve ser posicionada?

## Fatores da Decisão

- Facilidade na localização dos testes de cada domínio;
- Coerência com a organização adotada no código-fonte;
- Redução de conflitos entre integrantes que atuam em domínios distintos.

## Opções Consideradas

- Opção 1: Subpastas por domínio, espelhando os pacotes de `src/main/java`
- Opção 2: Subpastas por tipo de teste (`unitario/` e `integracao/`)
- Opção 3: Todos os testes na pasta raiz

## Resultado da Decisão

Opção escolhida: "Opção 1", pois segue o mesmo critério da ADR-BE-012, de modo que a localização do código indica também a localização de seu respectivo teste.

- O teste deve estar no mesmo pacote do código que verifica (por exemplo, `src/test/java/br/com/sauf_api/assinatura/AssinaturaApiTest.java`);
- Testes que envolvem mais de um domínio devem ser posicionados no domínio da regra que verificam;
- O tipo do teste é identificado pelo nome da classe, e não pela pasta, conforme a [ADR-BE-014](ADR-BE-014.md);
- O teste geral da aplicação (`SaufApiApplicationTests`) permanece na pasta raiz.

```
src/test/java/br/com/sauf_api/
├── SaufApiApplicationTests.java
├── assinatura/
├── config/
├── curso/
├── ingestao/
├── instituicao/
└── sisu/
```

### Consequências Positivas

- Os testes e o código-fonte compartilham a mesma organização;
- Por estarem no mesmo pacote, os testes podem acessar classes e métodos que não são públicos.

### Consequências Negativas

- Testes que envolvem mais de um domínio exigem a definição de onde serão posicionados.

## Prós e Contras das Opções

### Opção 1 - Subpastas por domínio

- Bom, porque espelha a organização do código-fonte;
- Ruim, porque testes que envolvem mais de um domínio exigem uma definição adicional.

### Opção 2 - Subpastas por tipo de teste

- Bom, porque separa os testes mais rápidos dos mais lentos;
- Ruim, porque distribui os testes de um mesmo domínio em pastas diferentes.

### Opção 3 - Todos os testes na pasta raiz

- Bom, porque é uma organização simples;
- Ruim, porque a pasta raiz cresce a cada novo domínio, dificultando a localização dos testes.

## Links

- Relaciona-se com: [ADR-BE-012](ADR-BE-012.md) (pacotes por domínio)
- Relaciona-se com: [ADR-BE-014](ADR-BE-014.md) (tipos de teste e tecnologias)
