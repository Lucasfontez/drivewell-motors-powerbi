# 05 — Análise

## Os números da rede

| Métrica | 2022 | 2023 | Total |
|---|---|---|---|
| Receita | $300,3M | $371,2M | $671,5M |
| Carros | 10.645 | 13.261 | 23.906 |
| Ticket médio | — | $27.991 | $28.090 |
| Crescimento | — | **+23,6%** | — |

Validados contra o Excel. Se o Power BI divergisse, o erro estaria no modelo.

---

## Achado 1 — O crescimento é enganoso

**+23,6% parece uma boa notícia. Não é.**

O gráfico de receita mensal com as duas séries sobrepostas mostra o que o número anual esconde: **de janeiro a agosto, as duas linhas andam praticamente coladas.** 2023 mal se descola de 2022. É a partir de setembro que a linha de 2023 abre distância — e não volta mais.

Uma conta de guardanapo confirma: jan–ago dos dois anos somados vale **$329,6M**. O ano inteiro dos dois vale $671,5M. Logo, **setembro a dezembro sozinho vale $341,9M** — mais que os oito primeiros meses juntos.

**Quatro meses faturam mais que oito.**

### O "e daí"

A rede não cresceu. Ela teve um fim de ano melhor — e ficou **mais dependente dele**, não menos. Se dezembro falhar em 2024, o ano inteiro falha.

Isso muda a conversa. Se o gerente acredita que cresceu 23% de forma saudável, ele não faz nada. Se entende que oito meses estão parados, ele age.

**Ação:** atacar o vale de janeiro a agosto. É lá que existe capacidade ociosa.

### Sobre a sazonalidade

O serrilhado (setembro alto, outubro despencando, novembro e dezembro subindo) aparece **nos dois anos**. Isso significa que é padrão estrutural de mercado, não evento pontual — dá para planejar em cima dele.

Se aparecesse em só um ano, seria ruído ou erro de dado, e não poderia ser tratado como sazonalidade.

---

## Achado 2 — Volume ≠ faturamento

O ranking de receita por marca mostra **Chevrolet, Ford e Dodge** no topo. O ranking de ticket médio mostra outra coisa completamente:

| Marca | Ticket médio |
|---|---|
| Cadillac | $42.236 |
| Saab | $37.407 |
| Lexus | $34.051 |
| Buick | $32.185 |
| Lincoln | $32.083 |
| — | — |
| *Chevrolet* | *~$26.200* |

**Nenhuma das cinco primeiras em ticket aparece no top 5 de receita.** E vice-versa.

A dispersão de ticket entre marcas é de **111%** — a maior de qualquer variável testada no dataset. Marca é, de longe, o melhor discriminador de valor que existe nesses dados.

### O "e daí"

A DriveWell tem **dois negócios dentro dela**:

- Um de **volume** — Chevrolet, Ford, Dodge — que gira muito carro a ~$26 mil
- Um **premium** — Cadillac, Saab, Lexus, Buick — que gira pouco carro a $34–42 mil

E ninguém gerencia o segundo como linha separada. Eles estão no mesmo balcão, com a mesma abordagem comercial, o mesmo treinamento de vendedor, a mesma meta.

**Não é uma marca fora da curva. É um segmento inteiro operando num patamar diferente.**

---

## Achado 3 — A sub-performance é de loja, não de região

Aqui a análise contradiz a tese contratada, e isso é o mais interessante.

O briefing pedia **performance comercial regional**. A hipótese implícita era que alguma região estaria puxando o resultado para baixo.

**Não está.**

### Crescimento por região

| Região | 2022 | 2023 | YoY |
|---|---|---|---|
| Scottsdale | $42,6M | $53,4M | +25,5% |
| Austin | $52,2M | $65,0M | +24,7% |
| Aurora | $39,5M | $49,2M | +24,6% |
| Greenville | $39,3M | $48,9M | +24,5% |
| Janesville | $47,6M | $58,7M | +23,3% |
| Pasco | $39,7M | $48,4M | +21,9% |
| Middletown | $39,6M | $47,6M | +20,2% |

**Amplitude: 5,3 pontos.** Todas cresceram, todas na mesma faixa. Austin lidera em receita absoluta ($65M), mas isso é **escala de mercado**, não desempenho — o ticket médio dela ($28.326) é praticamente igual ao de Middletown ($27.623). Diferença de 2,5%.

A região não explica nada.

### Crescimento por concessionária

| Loja | YoY |
|---|---|
| Iceberg Rentals | +32,4% |
| Saab-Belle Dodge | +32,0% |
| ... | ... |
| C & M Motors | **+18,9%** |
| Ryder Truck Rental | **+17,4%** |
| Motor Vehicle Branch | **+17,2%** |
| Star Enterprises | **+17,1%** |
| **Buddy Storbeck's Diesel** | **+7,5%** |

**Amplitude: 25 pontos.** Cinco vezes maior que a variação regional.

### O "e daí" — e a nuance que importa

Cinco lojas ficaram abaixo da meta de 20%. Mas **elas não são o mesmo problema**:

- Quatro estão em **17% a 19%** — coladas na meta. Um empurrão resolve.
- Uma está em **7,5%** — dezesseis pontos abaixo da rede. Isso não é "um pouco abaixo". É outro patamar.

**Buddy Storbeck's não precisa de um empurrão. Precisa de uma investigação.**

**Ação:** não convoque cinco gerentes. Convoque um.

---

## O que a análise NÃO conseguiu responder

**Por que Buddy Storbeck's está a 7,5%?** Os dados não têm a resposta. Não há informação de headcount, de estoque, de mix de produto por loja, de tempo de operação. O dashboard aponta o dedo — mas a causa exige ir a campo.

Isso é honesto e é o que um analista deve dizer. Um painel que *finge* explicar a causa com os dados que tem é pior que um que aponta e cala.
