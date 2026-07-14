# 06 — Decisões e Aprendizados

Os outros documentos contam o que funcionou. Este conta o que não funcionou, e é onde está o aprendizado.

---

## A hipótese que caiu

### O que eu esperava

O briefing trazia uma hipótese natural: cliente de maior renda compra carro mais caro. Parecia óbvio. Construí uma página inteira do dashboard em cima disso, com quatro KPIs, três visuais e medidas de renda mediana e concentração de clientes premium.

### O que os dados disseram

Ticket médio por faixa de renda:

| Faixa | Carros | Ticket |
|---|---|---|
| Alta | 5.931 | $28.338 |
| Muito Alta | 5.694 | $28.113 |
| Baixa | 6.072 | $27.983 |
| Média | 6.209 | $27.938 |

A diferença entre a faixa mais alta e a mais baixa é de **1,43%**. Isso é ruído.

Não confiei só nisso, porque podia ser artefato dos meus cortes de faixa. Testei de duas outras formas.

**Correlação de Pearson entre renda e preço: 0,0121.** Não é uma correlação fraca, é a ausência de correlação.

**Ticket médio por decil de renda**, quebrando a base em 10 grupos:

```
Decil  1: $28.578      Decil  6: $28.958
Decil  2: $27.949      Decil  7: $28.411
Decil  3: $27.694      Decil  8: $27.980
Decil  4: $27.910      Decil  9: $27.656
Decil  5: $27.706      Decil 10: $28.644
```

Diferença entre o decil mais pobre e o mais rico: **0,23%**.

Depois removi os 22% de registros com renda placeholder (13.500) e refiz tudo. Correlação: 0,0106. Nada mudou.

### A conclusão

Nesta base, renda e preço do carro são variáveis independentes. Comprovado por três métodos.

Mas tem uma ressalva importante: provavelmente isso não diz nada sobre o negócio, e sim sobre o dado. Vinte e dois por cento de valores idênticos e correlação zero com tudo são as assinaturas de uma coluna gerada aleatoriamente pelo criador do dataset.

A afirmação honesta é essa:

> Testei a hipótese com três métodos. O ticket é uniforme entre as faixas de renda. Mas 22% dos registros têm renda repetida no mesmo valor, o que sugere dado sintético. Não posso afirmar que isso reflete comportamento real de mercado. O que posso afirmar é que renda, nesta base, não serve como variável de segmentação.

Dizer "descobri que renda não importa na compra de carros" seria irresponsável.

### A decisão

Cortei a página. O dashboard tem duas páginas em vez de três.

Uma página inteira provando que não há nada ali não é entrega, é enchimento. Resposta negativa vira uma frase na apresentação, não 700×400 pixels de espaço nobre.

### Por que isso é bom para o projeto

A maioria dos portfólios só mostra o que confirma. Testar quatro hipóteses, ver todas falharem, e ter a disciplina de cortar em vez de forçar um insight: isso é o trabalho.

Cortar uma página foi uma decisão de análise, não uma redução de escopo.

---

## As outras hipóteses testadas

| Variável | Dispersão do ticket | Veredito |
|---|---|---|
| Gênero | 0,8% | Não segmenta |
| Transmissão | 1,2% | Não segmenta |
| Cor | 4,6% | Fraco demais |

Nenhuma entrou no dashboard.

Uma sobreviveu, e por pouco: **carroceria**, com 11,5% de dispersão (Sedan a $29.833 contra Hatchback a $27.127). Tem sinal real. Manteve o lugar na página 1.

Mas o gráfico de rosca mostra que o mix é uniforme: nenhuma carroceria passa de 28%. Ou seja, não há concentração a explorar.

Mantive o visual mesmo assim, porque a ausência de concentração também é resposta. Significa que mix de carroceria não é alavanca de receita, e o gerente não deve investir esforço aí.

Se me perguntarem por que esse gráfico está na tela, é essa a resposta. Não é "para mostrar o mix", porque isso não valeria o espaço.

---

## Os tropeços técnicos

### 1. O ALL() matou a legenda

**Problema:** o gráfico de linhas perdia a série de 2022 quando o slicer filtrava 2023.

**O que tentei:**

```dax
Receita Todos os Anos = CALCULATE ( [Receita Total]; ALL ( dCalendario[Ano] ) )
```

**O que aconteceu:** o `ALL` removeu o filtro do slicer, mas também removeu o filtro da legenda. As duas linhas viraram uma só, somando os dois anos por mês. O eixo saltou de $60M para $100M, e foi esse salto que me fez perceber o erro.

**A causa:** eu estava tratando um problema de interação de visual com uma ferramenta de cálculo. O `ALL` opera sobre o contexto de filtro inteiro, e não sabe distinguir de onde cada filtro veio.

**A solução certa não era DAX.** Era Editar Interações, configurando o gráfico de linhas para ignorar o slicer de Ano. Isso é responsabilidade do relatório, não do modelo.

Aprendi que saber em qual camada o problema mora (modelo, medida ou relatório) é metade do trabalho.

### 2. O gauge que mentia

O medidor de Receita vs Meta nasceu com escala automática: 0 a $742 milhões. A meta ($360M) caía no meio do arco, e a receita ($371M) parecia passar com folga.

Era mentira. A rede bateu a meta por 3%.

Fixei o máximo em $400M. Agora o arco vai de 0 a 400, a meta fica quase no fim, e a receita mal passa dela. A tela mostra a verdade: passou raspando.

Escala automática é um default, não uma decisão. Um gráfico que faz vitória apertada parecer folgada está enganando quem lê.

### 3. Formatação condicional com o tipo errado

Ao colorir as barras de crescimento (vermelho abaixo de 20%, âmbar acima), só uma barra ficava vermelha. Deveriam ser cinco.

A causa estava nos dropdowns da regra: o primeiro campo lia o valor como Porcentagem e o segundo como Número. Ao ler `0,2` como porcentagem, virava 0,2%, não 20%.

Troquei os quatro dropdowns para Número e as cinco lojas apareceram.

Como eu soube que estava certo: as barras vermelhas passaram a bater exatamente com o KPI "5 lojas abaixo da meta". Se o KPI diz 5 e o gráfico mostra 1, um dos dois está errado. Essa inconsistência entre KPI e visual é o erro mais grave que pode existir num painel.

### 4. A dimensão silenciosamente errada

Documentado em detalhe no [`03-etl-e-modelagem.md`](03-etl-e-modelagem.md). Resumo: `Cod_Concessionaria` era um código regional disfarçado, com 7 valores para 7 regiões. Quando usei ele como chave, a dimensão colapsou 28 lojas em 7 linhas.

Descobri por acaso, rodando um profiling para outra coisa. Se não tivesse descoberto, teria entregue uma página inteira construída sobre dados errados.

Nome de coluna não é contrato. Confira a cardinalidade antes de modelar.

---

## As decisões de design

### Um sistema, não uma paleta

A regra é uma só: **âmbar (`#F5A623`) significa "atual" ou "o que importa"**. Máximo de quatro usos por página. Todo o resto vive em grafite e aço.

Duas consequências práticas:

O gráfico de rosca não tem âmbar nenhum, porque não há líder. SUV ganha de Hatchback por dois pontos, o que é empate. Acender uma fatia comunicaria um destaque que não existe.

O KPI de crescimento é o único valor colorido da faixa superior. Ele é o número que exige reação.

### Planos de fundo carregam todo o chrome

Sidebar, cards, títulos, navegação: tudo desenhado num plano de fundo em SVG. Os visuais do Power BI por cima são transparentes, sem borda, sem título.

O ganho é consistência ao pixel. Todos os títulos de gráfico têm a mesma fonte, o mesmo tamanho, a mesma posição, porque são a mesma imagem.

### A régua de texto

Estabeleci um critério para decidir o que fica na tela:

> Se o texto explica como o número foi construído, fica. Se ele só reformula o título, sai.

Isso matou todos os subtítulos dos gráficos, que eram enchimento. Sobreviveram exatamente duas notas de método:

`META DE RECEITA → ano anterior + 20%`, porque a meta é uma premissa e quem lê precisa da regra.

`LOJAS ABAIXO DA META → de 28 concessionárias`, porque o denominador é informação, não enfeite.

### Por que não há frase de insight no fundo

Texto de insight fixo num fundo estático vira mentira assim que alguém aplica um filtro. Se o fundo afirma "setembro a dezembro concentra a receita" e o usuário filtra só Austin, o texto continua lá, afirmando algo que pode não valer mais.

O insight vive numa caixa de texto, que também é estática, mas que visivelmente é uma conclusão do analista e não um rótulo do sistema.

---

## O que eu faria diferente

**Teria feito o profiling de sinal antes de construir.** Rodar a dispersão de ticket por variável leva dez minutos. Eu construí uma página inteira sobre renda e só depois descobri que a variável não tinha sinal. A ordem certa é: profile, decide o que entra, constrói.

**Teria conferido a cardinalidade das chaves antes de modelar.** O bug da `dConcessionaria` teria sido pego em trinta segundos com um count distinct.

**Teria questionado a tese mais cedo.** O briefing pedia análise regional. Os dados mostram que região explica quase nada (5,3 pontos de amplitude) e que loja explica tudo (25 pontos). Se eu tivesse testado isso na primeira semana, teria economizado tempo, e a tese teria sido melhor desde o começo.

Esse último é o mais importante. A hipótese do cliente estava errada. Descobrir isso é o trabalho, não um desvio dele.

---

## Limitações que eu não consegui resolver

**A renda é quase certamente sintética.** Não afirmo nada sobre comportamento real de mercado a partir dela.

**A meta de 20% é minha.** Está sinalizada na tela, mas continua sendo uma premissa. Se o cliente real tivesse uma meta, o número mudaria e a leitura da página 2 junto.

**Não tenho custo nem margem.** Toda análise é de receita. "Cadillac tem ticket de $42 mil" não significa "Cadillac é mais lucrativa". O achado do segmento premium é uma hipótese comercial, não uma conclusão financeira.

**A geografia é implausível.** Todas as 28 lojas vendem nas 7 regiões, o que não acontece no mundo físico. Isso limita qualquer análise territorial mais séria e é uma característica do dataset, não uma escolha minha.

**A causa da sub-performance de Buddy Storbeck's não está nos dados.** O painel aponta o dedo. A investigação é em campo.
