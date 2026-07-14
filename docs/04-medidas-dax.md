# 04 — Medidas DAX

Oito medidas. Todas em uso — nenhuma órfã no modelo. Medida sem visual é código morto: quem abrir o arquivo perde tempo tentando entender para que serve.

---

## As bases

```dax
Receita Total = SUM ( fVendas[Valor Venda] )
```

```dax
Carros Vendidos = COUNTROWS ( fVendas )
```

`COUNTROWS` em vez de `COUNT(coluna)` — conta linhas do fato direto, sem depender de nenhuma coluna específica ser não-nula.

```dax
Ticket Médio = DIVIDE ( [Receita Total]; [Carros Vendidos] )
```

**`DIVIDE`, não `/`.** O operador de divisão estoura com erro quando o denominador é zero; `DIVIDE` devolve em branco. Num dashboard filtrado, o denominador *vai* ser zero em algum recorte.

Repare que o ticket é construído sobre as duas medidas anteriores, não sobre `AVERAGE(Valor Venda)`. As duas dão o mesmo número aqui, mas a versão com `DIVIDE` é explícita sobre o que está sendo dividido por quê — e sobrevive a mudanças de granularidade.

---

## Inteligência de tempo

```dax
Receita Ano Anterior = 
CALCULATE ( 
    [Receita Total]; 
    SAMEPERIODLASTYEAR ( dCalendario[Data] ) 
)
```

Depende da `dCalendario` estar marcada como tabela de datas e ser contínua (sem buracos). É por isso que ela foi criada por código, e não extraída do fato.

```dax
Crescimento YoY % = 
IF (
    HASONEVALUE ( dCalendario[Ano] );
    DIVIDE ( 
        [Receita Total] - [Receita Ano Anterior]; 
        [Receita Ano Anterior] 
    )
)
```

### A guarda `HASONEVALUE` — por que ela existe

Sem ela, essa medida **mente**.

Com o slicer de ano limpo, o contexto abrange 2022 e 2023. `Receita Total` devolve $671,5M (os dois anos). Mas `SAMEPERIODLASTYEAR` desloca todo o período um ano para trás — para 2021–2022 — e só 2022 tem dado: $300,3M.

A conta vira `(671,5 − 300,3) / 300,3` = **123,59%**.

O DAX está correto. A pergunta é que não faz sentido: **você está comparando dois anos de receita contra um ano.** YoY sem contexto de ano não existe.

`HASONEVALUE` faz a medida devolver em branco quando não há um ano único selecionado. **Um cartão em branco é honesto. Um cartão dizendo 123,59% é uma mentira que parece um sucesso.**

Essa foi a lição mais importante de DAX do projeto: *medida que funciona no caso feliz não é medida pronta. Teste o caso em que o usuário não filtra nada.*

---

## Meta e atingimento

```dax
Meta Receita = [Receita Ano Anterior] * 1.20
```

**A meta é uma premissa minha, não um dado.** Não existe meta na base.

Por que 20%:
- A rede cresceu **23,6%** — então 20% é atingível, mas não de graça
- É redondo, fácil de explicar e defender
- **Separa bem:** com 20%, cinco lojas ficam abaixo. Com 15%, quase nenhuma. Com 25%, a rede inteira falha.

Uma meta que ninguém erra não é meta. Uma que todo mundo erra também não.

O card no dashboard mostra "ano anterior + 20%" embaixo do número — porque quem lê precisa saber a regra, senão o valor cai do céu.

```dax
% da Meta = DIVIDE ( [Receita Total]; [Meta Receita] )
```

```dax
Lojas Abaixo da Meta = 
COUNTROWS (
    FILTER (
        VALUES ( dConcessionaria[Concessionaria] );
        [% da Meta] < 1
    )
)
```

### Como essa medida funciona

`VALUES` devolve as concessionárias visíveis no contexto atual. `FILTER` itera sobre elas, **avaliando `[% da Meta]` uma vez para cada loja** — cada iteração tem seu próprio contexto de filtro, com uma loja só. `COUNTROWS` conta quantas sobraram.

É o padrão de **iteração com contexto de linha convertido em contexto de filtro**. É o conceito de DAX que mais custa a entender, e é o que separa uma medida que agrega de uma que *avalia por entidade*.

O resultado — **5 lojas** — bate exatamente com as cinco barras vermelhas do gráfico ao lado. Se não batesse, seria sinal de que a medida e o visual estão usando contextos diferentes.

---

## Validação cruzada

A linha **Total** da matriz de detalhe devolve:

| | |
|---|---|
| Receita | $371.185.120 |
| Carros | 13.261 |
| Ticket | $27.990,73 |
| YoY | 23,59% |
| % da Meta | 102,99% |

E bate com os quatro cartões de KPI no topo da mesma página. **Consistência interna** — o painel não se contradiz.

---

## O que eu tentei e não funcionou

**Problema:** o gráfico de linhas "2023 vs 2022" perdia a série de 2022 quando o slicer filtrava 2023.

**Primeira tentativa — errada:**

```dax
Receita Todos os Anos = 
CALCULATE ( [Receita Total]; ALL ( dCalendario[Ano] ) )
```

`ALL` removeu o filtro do slicer, mas **também removeu o filtro da legenda**. As duas séries colapsaram numa só, e o gráfico passou a mostrar a soma dos dois anos por mês. O eixo saltou de $60M para $100M — o sintoma que denunciou o erro.

**A causa:** `ALL` opera sobre o contexto de filtro inteiro. Ele não distingue "filtro que veio do slicer" de "filtro que veio da legenda do próprio visual". Ferramenta errada para o problema.

**A solução certa não era DAX.** Era **Editar Interações** (guia Formato, com o slicer selecionado): configurar o gráfico de linhas para ignorar o slicer de Ano.

**A lição:** nem todo problema de visual se resolve com medida. Filtro de interação é responsabilidade do *relatório*, não do *modelo*. Tentar resolver com DAX foi tratar sintoma de UI com remédio de cálculo.
