# 07 — O Dashboard

Duas páginas. Nenhum visual entrou sem responder a uma pergunta de negócio.

---

## A regra que governa tudo

**Âmbar (`#F5A623`) significa "atual / o que importa". Máximo de quatro usos por página.**

Todo o resto vive em grafite (`#3E4756`) e aço (`#4A7FB5`). Vermelho (`#D64550`) é reservado para "abaixo da meta" — nunca decorativo.

Num painel escuro, o que mais chama atenção é o que mais contrasta. Se tudo brilha, nada brilha. A disciplina do âmbar é o que faz o olho ir direto ao que importa.

---

## Página 1 — Visão Executiva

**Eyebrow:** DESEMPENHO COMERCIAL DA REDE

### Os quatro KPIs

`Receita Total` · `Carros Vendidos` · `Ticket Médio` · `Crescimento YoY`

**Só o crescimento é âmbar.** Os outros três são leitura de contexto; o YoY é o número que exige reação. Se ficar negativo, formatação condicional o vira vermelho.

Nenhum dos quatro tem linha de apoio embaixo. *"Receita total → acumulado do período"* é o rótulo dito de novo com outras palavras — enchimento. Foi cortado.

### Receita mensal — 2023 vs 2022

**Gráfico de linhas.** `NomeMes` no eixo, `Receita Total` no valor, **`Ano` na legenda** (é a legenda que cria as duas séries).

2023 em âmbar, espessura 2,5. 2022 em aço, espessura 2. O ano corrente acende, o anterior recua.

**A decisão que salvou este gráfico:** ele **ignora o slicer de Ano**, via *Editar Interações*. Sem isso, filtrar 2023 apagava a série de 2022 — e um gráfico chamado "2023 vs 2022" que mostra um ano só não cumpre o que promete.

É o visual mais importante da página. É nele que se vê que as duas linhas andam coladas até agosto.

### Receita por região

**Barras horizontais**, ordenadas decrescente. Só **Austin em âmbar**; as outras seis em grafite.

Um alerta que eu registro por honestidade: Austin lidera porque é o **maior mercado**, não a melhor operação (o ticket dela é praticamente igual ao de Middletown). O âmbar ali comunica "líder em volume", não "líder em desempenho". É uma escolha, e ela é discutível.

### Ticket médio por marca

**Barras horizontais**, Top 5 por `Ticket Médio` — **não por receita.**

Esse detalhe é a razão de o gráfico existir. Filtrando por receita, ele mostraria Chevrolet, Ford e Dodge — o mesmo que todo mundo já sabe. Filtrando por ticket, ele mostra **Cadillac, Saab, Lexus, Buick, Lincoln** — marcas que não aparecem em lugar nenhum do resto da página.

**O gráfico existe para contradizer o de cima.** É isso que o torna útil.

Volume e ticket ficam no tooltip, para quem quiser o detalhe sem poluir a tela.

### Mix por carroceria

**Rosca. E ela não tem âmbar.**

O mix é uniforme: 28,1% / 25,6% / 23,3% / 18,7% / 13,4%. Não há líder — SUV ganha de Hatchback por dois pontos, o que é empate. Acender uma fatia comunicaria um destaque que não existe.

**Por que o gráfico está aí, então?** Porque a ausência de concentração é resposta: significa que **mix de carroceria não é alavanca de receita**, e o gerente não deve investir esforço aí.

Se me perguntarem "o que esse gráfico mostra?", a resposta não é *"o mix"* — é *"que não há alavanca aqui"*. Se a resposta fosse a primeira, o gráfico não valeria o espaço.

### Insight

**Caixa de texto** com barra âmbar à esquerda. Duas frases e uma ação:

> **Crescemos 23,6% — mas só de setembro a dezembro.**
> Até agosto, 2023 andou lado a lado com 2022. O ano inteiro depende de quatro meses.
> **Ação:** atacar o vale de janeiro a agosto.

**É o único elemento da página que contradiz o próprio KPI.** O cartão grita "+23,59%" em âmbar; o insight, logo abaixo, diz "calma". Essa tensão é o que faz alguém parar de passar o olho e começar a ler.

---

## Página 2 — Concessionárias

**Eyebrow:** ONDE A REDE SUB-PERFORMA

O título da página é uma promessa. Todo visual dela precisa cumpri-la.

### Os quatro KPIs

`Receita Total` · `Meta de Receita` · `% da Meta` · `Lojas Abaixo da Meta`

**Dois deles têm nota de método** — e essas notas sobreviveram ao corte porque **explicam como o número foi construído**, não reformulam o título:

- `META DE RECEITA` → *ano anterior + 20%* — a meta é uma premissa minha; quem lê precisa da regra
- `LOJAS ABAIXO DA META` → *de 28 concessionárias* — "5 lojas abaixo" é diferente de 5 em 28 ou 5 em 100

**A régua:** se o texto explica o método, fica. Se só reformula o título, sai.

### Receita vs meta

**Medidor (gauge).** Valor = `Receita Total`, alvo = `Meta Receita`.

**O ajuste que fez esse visual valer:** o máximo do eixo está **fixado em $400M**.

Com escala automática, o Power BI usava 0 a $742M — e a meta ($360M) caía no meio do arco, fazendo a vitória parecer confortável. **Mentira: a rede bateu por 3%.**

Com o máximo em $400M, a meta fica quase no fim e a receita mal passa dela. A tela mostra a verdade: **passou raspando.**

Formatação condicional: âmbar se `% da Meta ≥ 1`, vermelho se abaixo.

### Crescimento por concessionária

**Barras horizontais, ordenadas CRESCENTE** — as piores no topo.

Isso é deliberado. Num painel de gestão, **o problema vem primeiro** — não por pessimismo, mas porque é o que exige ação. Os campeões não precisam de reunião na segunda-feira.

Formatação condicional: abaixo de 20% (a meta) em **vermelho**, acima em âmbar.

**O teste de integridade:** as cinco barras vermelhas batem exatamente com o KPI "5 lojas abaixo da meta". Se o KPI dissesse 5 e o gráfico mostrasse 1, um dos dois estaria errado — e essa é a inconsistência mais grave que pode existir num painel.

*(Ela aconteceu. A regra de formatação estava lendo `0,2` como porcentagem em vez de número. Ver [`06`](06-decisoes-e-aprendizados.md).)*

### Detalhe por concessionária

**Matriz** com `Concessionaria` nas linhas e cinco medidas nas colunas: receita, volume, ticket, YoY, % da meta.

A coluna `% da Meta` tem **barras de dados** — leitura de dois segundos.

**Validação cruzada embutida:** a linha Total da matriz devolve `$371.185.120` · `13.261` · `23,59%` · `102,99%`, e bate com os quatro KPIs no topo da página. **O painel não se contradiz.**

---

## As decisões de construção

### Planos de fundo carregam todo o chrome

Sidebar, cards, títulos, navegação, rótulos — tudo está desenhado num **PNG de fundo** (2560×1440, 2× o canvas). Os visuais do Power BI por cima são **transparentes, sem borda, sem título**.

**Ganhos:**
- Tipografia e espaçamento consistentes ao pixel — nada de "quase alinhado"
- Os títulos dos gráficos ficam todos com a mesma fonte, mesmo tamanho, mesma posição

### O menu funciona sem bookmark

Cada página carrega um fundo com **seu item de navegação já aceso** em âmbar. Por cima, botões **invisíveis** (sem texto, sem preenchimento, sem borda) com ação de *Navegação de página*.

Resultado: o estado ativo do menu funciona sozinho. Sem indicador, sem bookmark, sem estado selecionado configurado. Trocou de página, o menu acompanha.

Não usei o **Navegador de Página** nativo justamente porque ele desenha o próprio texto — e duplicaria o que já está no fundo.

### Zero visuais customizados

Nada de AppSource. Tudo nativo.

**Por quê:** visual customizado é dependência externa. Quem abrir o `.pbix` sem ele instalado vê um retângulo de erro. Em ambiente corporativo, o admin frequentemente bloqueia. E vários não renderizam em exportação para PDF.

Considerei o Chiclet Slicer para os filtros. **Não trouxe** — o slicer nativo em estilo Bloco atendia, e não vi motivo para adicionar dependência.

*"Ficar mais bonito" nunca é motivo suficiente.*

### Canvas 1280×720

Proporção 16:9. Cabe em qualquer tela sem scroll, e é o formato que o Power BI Service renderiza melhor.

---

## O que eu deixaria diferente

**A matriz de detalhe está com estilo claro** (fundo branco, texto escuro), destoando do tema escuro do resto. Foi uma escolha deliberada de contraste — mas ela quebra a regra de tom único que eu mesmo estabeleci, e num painel escuro o elemento mais claro rouba o destaque de quem devia tê-lo (o gauge e o gráfico de crescimento, que carregam a mensagem).

Registro aqui porque é uma decisão discutível, e prefiro nomeá-la a fingir que não existe.
