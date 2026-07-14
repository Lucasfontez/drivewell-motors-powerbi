# 02, Dados e Profiling

## O dataset

**Car Sales Report**: Kaggle, autoria de *missionjee*.

| | |
|---|---|
| Linhas | 23.906 |
| Período | 2022–2023 |
| Formato | Tabela única, flat, sem chaves |
| Granularidade | Uma linha = uma venda |

Nenhum relacionamento, nenhuma dimensão. Tudo denormalizado num CSV só. Isso significa que o trabalho de modelagem era inteiro meu, o que, para um projeto de portfólio, é bom: é exatamente o que se faz na vida real.

## Dicionário

| Coluna | Tipo | O que é |
|---|---|---|
| `Data` | data | Data da venda |
| `Gênero` | texto | Gênero do comprador |
| `Renda` | número | Renda anual declarada |
| `Concessionaria` | texto | Nome da loja |
| `Cod_Concessionaria` | texto | **Não é o que parece**: ver abaixo |
| `Região` | texto | Região da venda |
| `Marca` | texto | Fabricante do veículo |
| `Modelo` | texto | Modelo |
| `Motor` | texto | Tipo de motor |
| `Transmissão` | texto | Manual / automática |
| `Cor` | texto | Cor do veículo |
| `Carroceria` | texto | SUV, Sedan, Hatchback, Passenger, Hardtop |
| `Valor Venda` | número | Preço da venda |

## O que estava quebrado

### 1. Encoding no campo Motor

O campo `Motor` vinha com caracteres corrompidos, um `Â` fantasma grudado nos valores. Tentei substituição direta (find & replace) e **não funcionou**.

O motivo: era **double-encoding**. O CSV foi salvo em UTF-8, lido como Latin-1, e salvo de novo em UTF-8. O caractere quebrado não era um caractere, era uma sequência de bytes que o Power Query interpretava de forma inconsistente. Replace não pega isso porque o que você vê na tela não é o que está no byte.

**Solução:** reconstruí a coluna inteira por lógica, em vez de tentar consertar o texto.

```m
Table.TransformColumns(
    Fonte,
    {{"Motor", each if Text.Contains(_, "Double") 
                    then "DoubleOverhead Camshaft" 
                    else "Overhead Camshaft"}}
)
```

Como só existiam dois valores possíveis, testei pela presença de "Double" e reescrevi ambos do zero. O texto original virou irrelevante.

Lição: quando o encoding está duplamente corrompido, não tente limpar, reconstrua a partir de um sinal que sobreviveu à corrupção.

### 2. As faixas de renda estavam desbalanceadas

Minha primeira tentativa de segmentar renda usou cortes arbitrários (algo como 200k / 500k / 1M). Resultado: **49% dos clientes caíram na faixa "Muito Alta"**. Uma faixa que contém metade da base não é uma faixa, é ruído.

Voltei e calculei os quartis reais:

| Quartil | Valor |
|---|---|
| Q1 | 386.000 |
| Mediana | 735.000 |
| Q3 | 1.175.000 |

Recalibrei os cortes para **400k / 750k / 1,2M**, e as quatro faixas ficaram com ~25% cada:

| Faixa | Carros |
|---|---|
| Baixa | 6.072 |
| Média | 6.209 |
| Alta | 5.931 |
| Muito Alta | 5.694 |

Lição: nunca defina faixas por intuição. Deixe a distribuição dos dados definir os cortes.

### 3. A renda tem 22% de dado sintético

O valor **13.500** aparece **5.273 vezes**: em 1 a cada 5 registros. Não é coincidência; é placeholder.

Isso não invalidou a análise (a mediana é robusta a concentração de valores), mas **invalidou qualquer conclusão de negócio** que eu tirasse dali. Está declarado como limitação no README.

### 4. `Cod_Concessionaria` não é código de concessionária

Este foi o pior, e eu só descobri depois de já ter construído a dimensão errada. Está documentado em [`03-etl-e-modelagem.md`](03-etl-e-modelagem.md).

## O que o profiling revelou sobre o sinal dos dados

Antes de construir qualquer visual, testei quais variáveis tinham **poder de discriminação**: medido pela dispersão do ticket médio entre categorias.

| Variável | Dispersão do ticket | Veredito |
|---|---|---|
| **Marca** | **111,3%** | Sinal forte |
| **Carroceria** | **11,5%** | Sinal real |
| Cor | 4,6% | Ruído |
| Região | 1,8% | Ruído |
| Renda | 1,4% | Ruído |
| Transmissão | 1,2% | Ruído |
| Gênero | 0,8% | Ruído |

**Duas variáveis de sete têm sinal.** Isso definiu o que entrou no dashboard, e o que ficou de fora.

Fazer esse teste *antes* de construir economizou o retrabalho de montar visuais que não diriam nada. Ou quase: eu já tinha construído uma página inteira sobre renda quando rodei o teste. Ver [`06-decisoes-e-aprendizados.md`](06-decisoes-e-aprendizados.md).
