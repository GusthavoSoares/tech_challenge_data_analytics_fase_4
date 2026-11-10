# Tech Challenge Fase 4: Amazon Product Performance & Customer Insights

PosTech FIAP Data Analytics.

A empresa quer saber **quais produtos, categorias e estratégias comerciais merecem maior
atenção, e quais oportunidades os dados revelam.** A cadeia que a liderança quer enxergar
é `preço → desconto → engajamento → satisfação`.

Base: [Amazon Sales Dataset no Kaggle](https://www.kaggle.com/datasets/karkavelrajaj/amazon-sales-dataset),
1.465 linhas e 16 colunas, preços em rupia indiana.

## Entregáveis

**Vídeo executivo:** <https://youtu.be/ZIk8-Yih-8w>

**Painel:** [`dashboard/tc4_amazon.pbix`](dashboard/tc4_amazon.pbix)

## Como abrir o painel

Baixe o repositório (botão **Code**, depois **Download ZIP**), extraia e abra
`dashboard/tc4_amazon.pbix` no Power BI Desktop.

O modelo é de importação, então os dados estão gravados dentro do próprio `.pbix` e as
14 páginas abrem com os números já carregados. Se o Power BI pedir a origem dos dados,
aponte para a pasta `results/` que veio no ZIP.

São 6 páginas de leitura no menu da esquerda, mais uma lista de produtos que se alcança
pelo botão da Visão executiva, e 7 páginas de dica que aparecem ao passar o mouse sobre
os indicadores com selo dourado.

### Para atualizar a partir dos CSV

Cada consulta aponta para a pasta da máquina onde o painel foi montado, e são 8, uma por
tabela.

Em **Página Inicial > Transformar dados**, abra cada consulta, clique na engrenagem ao
lado de **Origem** e troque a pasta pelo `results/` que veio no ZIP. Depois
**Fechar e Aplicar**.

Isso só é necessário para recarregar os dados do zero. Para ver e navegar o painel, o
arquivo já vem com tudo dentro.

## Estrutura

```
.
├── data/
│   └── amazon.csv                              base baixada do Kaggle
├── notebooks/
│   ├── tc_4_amazon.ipynb                       extração e tratamento inicial
│   ├── tc_4_amazon_novo.ipynb                  limpeza textual e NLP
│   └── tc_4_amazon_modelagem_powerbi.ipynb     modelagem em star schema
├── results/                                    as 9 tabelas que alimentam o painel
└── dashboard/
    └── tc4_amazon.pbix                         o painel, com os dados dentro
```

## Os três notebooks

| Notebook | O que faz |
|---|---|
| `tc_4_amazon.ipynb` | extração e tratamento inicial da base, com pandas e numpy |
| `tc_4_amazon_novo.ipynb` | limpeza textual e engenharia de atributos com nltk e spacy. Escreve `tabela_atributos_produto.csv` e `tabela_termos_frequencia.csv` |
| `tc_4_amazon_modelagem_powerbi.ipynb` | consolida tudo em star schema e escreve as 9 tabelas de `results/`. É o que o painel lê |

Os dois primeiros rodam no Colab e leem de `/content/amazon.csv`. O terceiro roda em
qualquer lugar, basta apontar o caminho da base na primeira célula.

## O que o terceiro notebook resolve

Rode e confira: ele imprime a prova de cada decisão enquanto executa.

**A categoria é hierarquia, não lista.** O campo `category` vem como
`Electronics|Headphones|In-Ear`. Virou uma coluna por nível, até o 7. Se virasse linha,
a soma de `rating_count` saltaria de 23,8 milhões para 98,9 milhões, uma inflação de
**4,2 vezes**. O notebook calcula as duas e mostra lado a lado.

**92 IDs de produto se repetem.** Resolvidos por desempate declarado: menos nulos vence,
empate decide pelo maior `rating_count`. Quem sobrou e por quê está em
`results/relatorio_duplicados.csv`. Restam 1.351 produtos.

**`review_id` não é chave.** São 10.598 linhas para 9.265 ids distintos, e 759 ids
aparecem em produtos diferentes. A tabela de fato usa `sk_avaliacao` como chave
substituta.

**O corte de percepção é percentil, não número redondo.** O `rating` desta base vai de
2,0 a 5,0 com mediana 4,10. Um corte fixo em 3,0 marcaria **6 produtos de 1.351** como
negativos, o que inviabiliza comparar. Os percentis 25 e 75 desta base dão 3,90 e 4,30,
e separam 340 abaixo, 591 na média e 419 acima.

**Termo exclusivo de uma faixa é nome de marca.** A primeira versão da comparação
devolvia `boult`, `iqoo`, `sandisk`. A versão final só compara termo presente nas duas
faixas, por frequência relativa, o que faz aparecer o que interessa: `noise`, `battery`,
`bass`.

Antes de exportar, o notebook roda asserts. Se algum falhar, não exporta.

## As tabelas em `results/`

| Tabela | Linhas | O que é |
|---|---|---|
| `fProdutos` | 1.351 | fato no grão de produto |
| `dCategoria` | 211 | hierarquia de categoria em colunas |
| `fAvaliacoes` | 10.598 | uma linha por avaliação com texto |
| `dTermosAvaliacao` | 114.500 | termo por produto, com a faixa de percepção |
| `dTermoComparado` | 840 | o que separa quem reclama de quem elogia |
| `dMoeda` | 3 | desconectada, alimenta a segmentação de moeda |
| `dRecomendacao` | 6 | **texto autoral**, leitura nossa sobre os números |
| `dFonte` | 15 | **texto autoral**: premissas, limites, caminho do dado |
| `relatorio_duplicados` | 92 | os IDs repetidos e como cada um foi resolvido |

As duas marcadas como autorais **não saem da base**. São interpretação nossa, escritas à
mão na seção 7.5 do notebook. Estão versionadas junto porque o painel precisa delas, mas
quem citar precisa saber que são leitura, não medição.

## O que a análise encontrou

Tudo conferido na base, nada estimado.

| Achado | Número |
|---|---|
| Concentração do catálogo | 97,2% dos produtos em 3 categorias de nível 1 (1.313 de 1.351) |
| Desconto contra nota | Pearson -0,16, Spearman -0,15. A nota ponderada cai de 4,16 na faixa até 20% para 4,01 na de 80% ou mais |
| Desconto contra engajamento | +0,00. Desconto maior não traz mais avaliação |
| Tráfego desperdiçado | 112 produtos com avaliações acima da mediana e nota no quartil inferior |
| Reputação parada | 76 produtos com nota no quartil superior e menos de mil avaliações |
| O que separa reclamação de elogio | `average` +13,8 pp, `bad` +13,2 pp, `noise` +9,9 pp, `battery` +8,8 pp, `bass` +8,6 pp |
| Preço contra satisfação | a nota ponderada varia só 0,10 ponto entre as seis faixas de preço (4,113 a 4,216) |

## Premissas declaradas

**Câmbio.** A base só traz rupia e não traz cotação. A conversão é taxa fixa, declarada
na tabela `dMoeda`, e serve para ordem de grandeza.

**`rating_count` é proxy de engajamento, não de venda.** A base não traz volume vendido,
então qualquer leitura de receita é inferência.

**Duas contagens de avaliação convivem.** `rating_count` soma 23,8 milhões declaradas; o
arquivo traz 10.598 linhas de texto. A análise de termos usa só a segunda, e nenhuma
conclusão dela vale para as 23,8 milhões.

## Integrantes

| Nome | RM |
|---|---|
| Alexandre de Souza Amorim | 372077 |
| Allison Lima | 372648 |
| Caio Rodrigues Bosnic Barbosa | 372603 |
| Gusthavo Lourenço Rios Soares | 371911 |
