# 01 — Contexto

## O cenário

A DriveWell Motors é uma rede de concessionárias fictícia que eu construí em cima de um dataset real. A ficção existe por um motivo prático: um dataset solto não tem cliente, e sem cliente não há decisão a tomar. Precisei de alguém que quisesse alguma coisa.

O cliente é o **Gerente Comercial** da rede. Ele não quer gráficos. Ele quer saber onde colocar esforço na segunda-feira.

## O briefing

Duas perguntas, e elas definem tudo:

> **Onde está o crescimento da receita?** Região, marca, tempo.
> **Onde a rede sub-performa?**

Repare que a segunda pergunta é a difícil. Todo dashboard sabe mostrar o que cresceu. Poucos apontam o dedo.

## A tese

A tese contratada é **performance comercial regional**. Isso funcionou como filtro para tudo: se um visual não ajudava a responder onde o dinheiro entra ou onde a rede falha, ele não entrava na tela.

Foi essa régua que me fez cortar uma página inteira mais tarde. Não porque a análise era ruim — porque ela não servia à tese.

## Critérios de aceite

O projeto só estaria pronto quando:

1. Os números do Power BI batessem com os do Excel (validação cruzada obrigatória)
2. O dashboard fosse legível em 10 segundos, sem alguém do lado explicando
3. Houvesse uma **recomendação de negócio**, não apenas diagnóstico
4. Eu conseguisse defender cada decisão numa apresentação de 5 minutos

O item 4 é o que mais moldou o projeto. Toda vez que eu ia adicionar alguma coisa, a pergunta era: *"se me perguntarem por que isso está aqui, eu tenho resposta?"*

Várias coisas não sobreviveram a essa pergunta.

## O que é fictício e o que é real

| Elemento | Origem |
|---|---|
| Vendas, clientes, veículos, datas | **Real** — dataset Car Sales Report (Kaggle) |
| Nome da rede, o cliente, o briefing | Fictício |
| A meta de +20% de crescimento | **Premissa minha** — não existe na base |

A meta merece destaque porque é o único número inventado que aparece no dashboard. Ela está sinalizada na própria tela ("ano anterior + 20%") justamente para que ninguém a confunda com dado.
