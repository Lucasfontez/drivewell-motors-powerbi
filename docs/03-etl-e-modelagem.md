# 03, ETL e Modelagem

## Power Query, as etapas

Todas as transformações foram feitas no Power Query, em etapas reprodutíveis. Nada de célula editada na mão: se o dado mudar, o pipeline roda de novo.

1. **Importação** do CSV
2. **Correção do encoding** no campo `Motor` (ver [`02`](02-dados-e-profiling.md))
3. **Renomeação** de colunas para português e padrão consistente
4. **Tipagem** explícita, data como data, valores como número decimal
5. **Tratamento de nulos**: verificados e resolvidos
6. **Coluna de Faixa de Renda**: criada com cortes por quartil
7. **Criação das dimensões** por referência à tabela fato

## O modelo estrela

```
                    dCalendario
                         │
                         │ 1:*
                         ▼
   dVeiculos ────1:*──► fVendas ◄──1:*──── dConcessionaria
                       (23.906)
```

| Tabela | Linhas | Papel |
|---|---|---|
| `fVendas` | 23.906 | Fato, uma linha por venda |
| `dCalendario` | ~730 | Dimensão de tempo |
| `dVeiculos` | — | Marca, modelo, motor, transmissão, cor, carroceria |
| `dConcessionaria` | **28** | Nome da loja |

**Todos os relacionamentos são 1:N, direção única.** Sem bidirecional. Sem ambiguidade.

### Por que a Região ficou no fato

Esta é a decisão de modelagem menos óbvia do projeto, e a que mais rende pergunta.

A intuição diz: *"região é atributo de loja, coloque na dimensão de concessionária"*. **Está errado**: e eu só descobri porque investiguei.

Neste dataset, **todas as 28 concessionárias vendem em todas as 7 regiões**. Não existe hierarquia geográfica. Uma loja não *tem* uma região; ela vende em várias.

Portanto, `Região` é atributo da **venda**, não da **loja**. Ela fica no fato, como dimensão degenerada.

Criar uma `dRegiao` separada seria tecnicamente possível, mas não traria ganho nenhum, só complexidade. A regra que segui: **não normalize o que não tem hierarquia.**

### A dCalendario

Criada do zero no Power Query, com M:

```m
let
    DataInicio = #date(2022, 1, 1),
    DataFim = #date(2023, 12, 31),
    Dias = Duration.Days(DataFim - DataInicio) + 1,
    Lista = List.Dates(DataInicio, Dias, #duration(1,0,0,0)),
    Tabela = Table.FromList(Lista, Splitter.SplitByNothing(), {"Data"}),
    Tipada = Table.TransformColumnTypes(Tabela, {{"Data", type date}}),
    Ano = Table.AddColumn(Tipada, "Ano", each Date.Year([Data]), Int64.Type),
    NumMes = Table.AddColumn(Ano, "NumMes", each Date.Month([Data]), Int64.Type),
    NomeMes = Table.AddColumn(NumMes, "NomeMesAbreviado", each Date.ToText([Data], "MMM"), type text)
in
    Tipada
```

**Detalhe que quebra e ninguém lembra:** `NomeMesAbreviado` é texto, então o Power BI ordena alfabeticamente — *abr, ago, dez, fev...*. Precisa marcar a coluna para **Classificar por `NumMes`**, senão o gráfico de linhas fica sem sentido.

---

## O bug da dimensão de concessionária

Este é o erro mais instrutivo do projeto, e eu quase entreguei com ele.

### O que eu fiz

Construí a `dConcessionaria` do jeito óbvio: referenciei o fato, mantive `Cod_Concessionaria`, `Concessionaria` e `Região`, e removi duplicatas usando o código como chave.

**Resultado: 7 linhas.**

Eu aceitei. Sete concessionárias parecia plausível, afinal, eram sete regiões.

### O que estava errado

Ao rodar um profiling da base para outra análise, notei que existiam **28 nomes distintos** de concessionária. Mas minha dimensão tinha 7.

Investiguei:

```
Concessionárias distintas (por nome): 28
Códigos distintos (Cod_Concessionaria): 7
Regiões distintas: 7
```

**`Cod_Concessionaria` não identifica a loja. Ele identifica a região.** São 7 códigos para 7 regiões, um código regional com nome enganoso.

Quando removi duplicatas por esse código, o Power Query colapsou 28 lojas em 7 linhas, mantendo um nome arbitrário para cada grupo. **A dimensão estava silenciosamente errada, e qualquer análise por concessionária estaria mentindo.**

### A correção

```m
// dConcessionaria, a chave é o NOME, não o código
let
    Fonte = fVendas,
    SoNome = Table.SelectColumns(Fonte, {"Concessionaria"}),
    Unicos = Table.Distinct(SoNome)
in
    Unicos
// → 28 linhas
```

Descartei a dimensão antiga. Relacionei `dConcessionaria[Concessionaria]` → `fVendas[Concessionaria]`, 1:N, direção única.

### A lição

**Nome de coluna não é contrato.** `Cod_Concessionaria` sugere uma coisa e é outra. Se eu tivesse conferido a cardinalidade antes de modelar — *"quantos valores distintos tem essa chave, e quantos deveria ter?"* — teria pego na hora.

O check que eu não fiz e que agora faço sempre:

> Antes de usar uma coluna como chave de dimensão, compare a contagem de valores distintos dela com a contagem de valores distintos do atributo que ela supostamente identifica. Se não baterem, a coluna não é a chave.

Custou uma página do dashboard construída sobre dados errados. Barato, dado que descobri antes de entregar.

## Validação

O modelo só foi aceito depois de bater com o Excel:

| Métrica | Excel | Power BI |
|---|---|---|
| Receita Total | $671.525.465 | ✅ |
| Carros Vendidos | 23.906 | ✅ |
| Crescimento 2023 | +23,6% | ✅ |

Se não batesse, o erro estaria no modelo, não na fórmula.
