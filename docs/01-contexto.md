# 01 — Contexto

## Por que existe um cliente fictício

O dataset é real. A DriveWell Motors não é.

Inventei a rede porque um dataset solto não tem cliente, e sem cliente não existe decisão a tomar. Eu ia acabar montando gráficos bonitos sem nenhuma pergunta por trás. Precisava de alguém querendo alguma coisa.

O cliente é o Gerente Comercial da rede. Ele não quer gráficos. Quer saber onde colocar esforço na segunda-feira.

## O briefing

Duas perguntas:

> **Onde está o crescimento da receita?** Região, marca, tempo.
> **Onde a rede sub-performa?**

A segunda é a difícil. Todo dashboard sabe mostrar o que cresceu. Poucos apontam o dedo.

## A tese

A tese contratada foi **performance comercial regional**.

Ela virou o filtro de tudo: se um visual não ajudava a responder onde o dinheiro entra ou onde a rede falha, não entrava na tela. Foi essa régua que me fez cortar uma página inteira mais tarde, quando descobri que a análise que eu tinha construído não servia à tese.

Vale dizer desde já: **a tese estava parcialmente errada, e os dados provaram isso.** O briefing assumia que o problema era regional. Não é. Está documentado em [`05-analise.md`](05-analise.md).

## Critérios de aceite

O projeto só estaria pronto quando:

1. Os números do Power BI batessem com os do Excel
2. O dashboard fosse legível em 10 segundos, sem ninguém explicando
3. Houvesse uma recomendação de negócio, não só diagnóstico
4. Eu conseguisse defender cada decisão numa apresentação de 5 minutos

O item 4 foi o que mais moldou o projeto. Toda vez que eu ia adicionar alguma coisa, a pergunta era: se me perguntarem por que isso está aqui, eu tenho resposta?

Várias coisas não sobreviveram a essa pergunta.

## O que é real e o que é invenção

| Elemento | Origem |
|---|---|
| Vendas, clientes, veículos, datas | Real (dataset Car Sales Report, Kaggle) |
| Nome da rede, o cliente, o briefing | Fictício |
| A meta de +20% de crescimento | Premissa minha |

A meta merece destaque porque é o único número inventado que aparece no dashboard. Ela está sinalizada na própria tela ("ano anterior + 20%") para que ninguém a confunda com dado.
