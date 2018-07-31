---
title: Preparação de dados com R e Rstudio
---
A atividade de coletar e preparar dados para análise consome cerca de 50% a 80% do tempo do cientista de dados. Este guia rápido tem o objetivo de agilizar esse processo, compilando no mesmo local uma lista de técnicas de manipulação de dados úteis. Você também pode usá-lo como um tutorial, se você seguir os exemplos desde o início.

Quase tudo aqui veio do excelente webinar [Data Wrangling with R e RStudio](https://www.rstudio.com/resources/webinars/data-wrangling-with-r-and-rstudio/), de Garret Grolemund, que você poderá conferir se precisar de mais detalhes.

# Pré Requisitos
## R e RStudio
Se você ainda não tem o R ou o RStudio instalados em sua máquina, siga as instruções [neste site](http://leg.ufpr.br/~fernandomayer/aulas/ce083-2016-2/R-instalacao.html) para colocar tudo em ordem antes de começar.

## Bibliotecas necessárias
Boa parte das atividades utiliza funções dos pacotes [tidyr](https://cran.r-project.org/web/packages/tidyr/README.html) e [dplyr](https://www.r-project.org/nosvn/pandoc/dplyr.html). Você precisa instalá-los para conseguir executar o código de exemplo.

## Dataset de exemplo
O dataset utilizado para todos os exemplos é o VRA, que contém o histórico de voos registrados pela ANAC. Ele pode ser obtido [nesse endereço](http://www.anac.gov.br/assuntos/dados-e-estatisticas/historico-de-voos). Só é necessário baixar os arquivos correspondentes aos meses de [janeiro](http://www.anac.gov.br/assuntos/dados-e-estatisticas/base-historica-1/vra/2017/VRA_do_MS_012017.zip), [fevereiro](http://www.anac.gov.br/assuntos/dados-e-estatisticas/base-historica-1/vra/2017/VRA_do_MS_022017.zip) e [março](Março) de 2017 e descompactar todos na mesma pasta.

# Carregando dados para o R
Para carregar um arquivo CSV, utilize a biblioteca reader e a função read_delim(), especificando o ";" como delimitador de colunas do arquivo:
```
library(readr)
vra <- read_delim("github/data-wrangling/VRA_do_MES_012017.csv", ";")
```
Eventualmente, podem ocorrer erros de codificação do arquivo caso você esteja no Linux e seu CSV tiver sido criado no Windows ou vice-versa. Nesses casos, você pode acrescentar um parâmetro para evitar esse problema:
```
vra <- read_delim("github/data-wrangling/VRA_do_MES_012017.csv", ";", locale = locale(encoding = "ISO-8859-1"))
```
Para visualizar os dados, use o comando:
```
view(vra)
```
