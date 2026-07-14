# 06 — Decisões e Aprendizados

Este é o documento que eu mais recomendo ler. Os outros contam o que funcionou. Este conta o que não funcionou — e é onde está o aprendizado real.

---

## A hipótese que caiu (e custou uma página)

### O que eu esperava

O briefing tinha uma hipótese natural: **cliente de maior renda compra carro mais caro.** Parece óbvio. Eu construí uma página inteira do dashboard para explorar isso — quatro KPIs, três visuais, medidas de renda mediana e concentração de clientes premium.

### O que os dados disseram

**Ticket médio por faixa de renda:**

| Faixa | Carros | Ticket |
|---|---|---|
| Alta | 5.931 | $28.338 |
| Muito Alta | 5.694 | $28.113 |
| Baixa | 6.072 | $27.983 |
| Média | 6.209 | $27.938 |

**Dispersão entre a maior e a menor faixa: 1,43%.** Isso é ruído.

Não confiei no agrupamento em faixas — podia ser artefato dos meus cortes. Testei de duas outras formas:

**Correlação de Pearson entre renda e preço: `0,0121`.**
Não é fraca. É **inexistente**.

**Ticket médio por decil de renda** (quebrando a base em 10 grupos):

```
Decil  1: $28.578      Decil  6: $28.958
Decil  2: $27.949      Decil  7: $28.411
Decil  3: $27.694      Decil  8: $27.980
Decil  4: $27.910      Decil  9: $27.656
Decil  5: $27.706      Decil 10: $28.644
```

**Diferença entre o decil mais pobre e o mais rico: +0,23%.**

Removi os 22% de registros com renda placeholder (13.500) e refiz tudo. Correlação: **0,0106**. Nada muda.

### A conclusão

**Nesta base, renda e preço do carro são variáveis independentes.** Comprovado por três métodos.

Mas — e isso é crucial — **provavelmente não é o negócio. É o dado.** Vinte e dois por cento de valores idênticos e correlação zero com tudo são as assinaturas clássicas de uma coluna gerada aleatoriamente pelo criador do dataset.

**A afirmação honesta é:**

> *"Testei a hipótese com três métodos. O ticket é uniforme entre as faixas de renda. Mas 22% dos registros têm renda repetida no mesmo valor, o que sugere dado sintético — então não posso afirmar que isso reflete comportamento real de mercado. Renda, nesta base, não serve como variável de segmentação."*

Dizer *"descobri que renda não importa na compra de carros"* seria irresponsável.

### A decisão

**Cortei a página.** O dashboard tem duas páginas em vez de três.

Uma página inteira provando que não há nada ali não é entrega — é enchimento. Resposta negativa vira **uma frase na apresentação**, não 700×400 pixels de espaço nobre.

### Por que isso é uma boa notícia para o projeto

A maioria dos portfólios júnior só mostra o que confirma. Testar quatro hipóteses, ver todas falharem, e ter a disciplina de cortar em vez de forçar um insight — isso é o trabalho.

**Cortar uma página é uma decisão de análise, não uma redução de escopo.**

---

## As outras hipóteses testadas

| Variável | Dispersão do ticket | Veredito |
|---|---|---|
| Gênero | 0,8% | Não segmenta |
| Transmissão | 1,2% | Não segmenta |
| Cor | 4,6% | Fraco demais |

Nenhuma entrou no dashboard.

**Uma sobreviveu, e por pouco:** **Carroceria**, com 11,5% de dispersão (Sedan $29.833 vs Hatchback $27.127). Tem sinal real. Manteve o lugar na página 1 — mas o gráfico de rosca mostra que **o mix é uniforme** (nenhuma carroceria passa de 28%), então não há concentração a explorar.

Mantive o visual mesmo assim, porque **a ausência de concentração também é resposta**: significa que mix de carroceria não é alavanca de receita, e o gerente não deve investir esforço aí.

Se me perguntarem *"por que esse gráfico está aí?"*, essa é a resposta. Não é *"para mostrar o mix"* — isso não valeria o espaço.

---

## Os tropeços técnicos

### 1. `ALL()` matou a legenda

**Problema:** o gráfico de linhas perdia a série de 2022 quando o slicer filtrava 2023.

**O que tentei:** `CALCULATE([Receita Total]; ALL(dCalendario[Ano]))`

**O que aconteceu:** `ALL` removeu o filtro do slicer *e* o filtro da legenda. As duas linhas viraram uma só, somando os dois anos por mês. O eixo saltou de $60M para $100M — foi o sintoma que denunciou.

**A causa raiz:** eu estava tratando um problema de **interação de visual** com uma ferramenta de **cálculo**. `ALL` opera no contexto de filtro inteiro; ele não sabe distinguir de onde cada filtro veio.

**A solução certa:** **Editar Interações** — configurar o gráfico para ignorar aquele slicer específico. É configuração de relatório, não de modelo.

**Lição:** nem todo problema de visual se resolve com DAX. Saber *qual camada* resolve o problema (modelo, medida, ou relatório) é metade do trabalho.

### 2. O gauge que mentia

O medidor de "Receita vs Meta" nasceu com escala automática: **0 a $742 milhões**. A meta ($360M) ficava no meio do arco, e a receita ($371M) parecia passar com folga confortável.

**Mentira.** A rede bateu a meta por **3%**.

Fixei o máximo em **$400M**. Agora o arco vai de 0 a 400, a meta fica quase no fim, e a receita mal passa dela. A tela mostra a verdade: **passou raspando.**

**Lição:** escala automática é um default, não uma decisão. Um gráfico que faz vitória apertada parecer folgada está enganando quem lê.

### 3. Formatação condicional com o tipo errado

Ao colorir as barras de crescimento (vermelho abaixo de 20%, âmbar acima), só **uma** barra ficava vermelha, quando deveriam ser cinco.

**A causa:** os dropdowns da regra estavam misturados — o primeiro campo lia o valor como **Porcentagem**, o segundo como **Número**. Ao ler `0,2` como porcentagem, virava **0,2%**, não 20%.

Trocando os quatro dropdowns para **Número**, as cinco lojas apareceram.

**Como eu soube que estava certo:** as barras vermelhas passaram a bater exatamente com o KPI "5 lojas abaixo da meta". **Se o KPI diz 5 e o gráfico mostra 1, um dos dois está errado** — e essa inconsistência é o erro mais grave que existe num painel.

### 4. A dimensão silenciosamente errada

Documentado em [`03-etl-e-modelagem.md`](03-etl-e-modelagem.md). Resumo: `Cod_Concessionaria` era um código *regional* disfarçado, e minha dimensão colapsou 28 lojas em 7. Descobri por acaso, rodando um profiling para outra coisa.

**Lição:** nome de coluna não é contrato. Confira a cardinalidade antes de modelar.

---

## As decisões de design

### Um sistema, não uma paleta

Regra única, aplicada em tudo: **âmbar (`#F5A623`) significa "atual / o que importa".** Máximo de quatro usos por página. Todo o resto vive em tons de grafite e aço.

Consequências práticas:
- O gráfico de rosca (mix por carroceria) **não tem âmbar** — porque não há líder. SUV ganha de Hatchback por 1,1 ponto, o que é empate. Acender uma fatia comunicaria um destaque que não existe.
- O KPI de crescimento é o único valor colorido da faixa superior.

### Planos de fundo como chrome

Todo o "chrome" do dashboard (sidebar, cards, títulos, navegação) está desenhado num **plano de fundo** em PNG. Os visuais do Power BI por cima são **transparentes, sem borda, sem título**.

Ganhos:
- Tipografia e espaçamento consistentes ao pixel
- O estado ativo do menu funciona **sem bookmark** — cada página carrega o fundo com seu item aceso
- Os botões de navegação são invisíveis; quem aparece é o desenho

### A régua de texto

Estabeleci um critério para decidir o que fica na tela:

> **Se o texto explica como o número foi construído, fica. Se ele só reformula o título, sai.**

Isso matou todos os subtítulos dos gráficos (eram enchimento) e sobreviveram exatamente duas notas de método:
- `META DE RECEITA` → *ano anterior + 20%* — a meta é uma premissa; quem lê precisa da regra
- `LOJAS ABAIXO DA META` → *de 28 concessionárias* — o denominador é informação, não enfeite

### Por que não há frase de insight no fundo

Texto de insight fixo num fundo estático **vira mentira** assim que alguém aplica um filtro. Se o fundo afirma *"setembro a dezembro concentra a receita"* e o usuário filtra só Austin, o texto continua lá — afirmando algo que pode não valer mais.

O insight vive numa **caixa de texto** (que também é estática, mas visivelmente é uma conclusão do analista, não um rótulo do sistema) e, principalmente, na apresentação.

---

## O que eu faria diferente

**1. Teria feito o profiling de sinal ANTES de construir.**
Rodar a dispersão de ticket por variável leva dez minutos. Eu construí uma página inteira sobre renda e *depois* descobri que a variável não tinha sinal. A ordem certa é: profile → decida o que entra → construa.

**2. Teria conferido a cardinalidade das chaves antes de modelar.**
O bug da `dConcessionaria` teria sido pego em trinta segundos com um `COUNT DISTINCT`.

**3. Teria questionado a tese mais cedo.**
O briefing pedia análise *regional*. Os dados mostram que região não explica quase nada (5,3 pontos de amplitude) e que loja explica tudo (25 pontos). Se eu tivesse testado isso na primeira semana, teria economizado tempo — e a tese teria sido melhor desde o começo.

**Esse último ponto é o mais importante:** a hipótese do cliente estava errada. Descobrir isso *é* o trabalho, não um desvio dele.

---

## Limitações honestas

- **Dataset parcialmente sintético.** A renda é quase certamente gerada. Não afirmo nada sobre comportamento real de mercado a partir dela.
- **A meta de 20% é minha.** Está sinalizada na tela, mas é uma premissa, não um dado.
- **Sem custo ou margem.** Toda análise é de receita. "Ticket alto" não significa "mais lucrativo".
- **A geografia é implausível.** Todas as 28 lojas vendem nas 7 regiões, o que não acontece no mundo físico. Limita qualquer análise territorial.
- **A causa da sub-performance de Buddy Storbeck's não está nos dados.** O painel aponta o dedo; a investigação é em campo.
