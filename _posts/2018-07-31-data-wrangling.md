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
## Sintaxe Básica
Para carregar um arquivo CSV, utilize a biblioteca reader e a função read_delim(), especificando o ";" como delimitador de colunas do arquivo:
```
library(readr)
vra <- read_delim("github/data-wrangling/VRA_do_MES_012017.csv", ";")
```
## Visualizando os dados
Para visualizar os dados, use o comando abaixo. Atenção para a inicial que deve ser maiúscula.
```
View(vra)
```
# Selecionando a codificação correta
Eventualmente, podem ocorrer erros de codificação do arquivo caso você esteja no Linux e seu CSV tiver sido criado no Windows ou vice-versa. Nesses casos, você pode acrescentar um parâmetro para evitar esse problema:
```
vra <- read_delim("github/data-wrangling/VRA_do_MES_012017.csv", ";", 
                  locale = locale(encoding = "ISO-8859-1"))
View(vra)
```
Os tipos de encoding mais comuns são o "ISO-8859-1" e o "UTF-8".

# Alterando o tipo de dados da coluna
## Tratamento de erros 
Na importação do nosso arquivo 19 linhas não puderam ser importada e o seguinte *warning* foi exibido na tela:
```
Warning: 19 parsing failures.
row # A tibble: 5 x 5 col     row col                     expected   actual file                                          expected   <int> <chr>                   <chr>      <chr>  <chr>                                         actual 1 32431 Código Autorização (DI) an integer B      'github/data-wrangling/VRA_do_MES_012017.csv' file 2 32432 Código Autorização (DI) an integer B      'github/data-wrangling/VRA_do_MES_012017.csv' row 3 32433 Código Autorização (DI) an integer B      'github/data-wrangling/VRA_do_MES_012017.csv' col 4 32434 Código Autorização (DI) an integer B      'github/data-wrangling/VRA_do_MES_012017.csv' expected 5 32435 Código Autorização (DI) an integer B      'github/data-wrangling/VRA_d [... truncated]
```
Isso se dá porque em algumas das linhas, no campo *Código Autorização (DI)*, que é numérico, foi inserido um caractere "B".
Existem duas interpretações e duas soluções correspondentes para esse problema: Se a letra "B" foi digitada por engano e puderem ser descartados, não é necessário fazer nada (as demais colunas são importadas normalmente). Se a letra B é uma entrada válida para esse campo, então ele deve ser convertido para alfanumérico de forma que possa acomodar letras em seu conteudo. Nesse caso, precisamos mudar o tipo da coluna. Para isso, recarregue o arquivo, desta vez acrescentando o parãmetro adicional, como no exemplo a seguir:
```
vra <- read_delim("github/data-wrangling/VRA_do_MES_012017.csv", ";", 
                  locale = locale(encoding = "ISO-8859-1")
                  col_types = cols("Código Autorização (DI)"= col_character()))
View(vra)                  
```
## Tratamento de datas
As colunas que contém datas e horas não são automaticamente interpretadas como datetime. No nosso exemplo, as colunas *Partida Prevista*, *Partida Real*, *Chegada Prevista* e *Chegada Real* são tratadas como se seu conteúdo fosse texto simples. A princípio, não há problema nisso e, por esse motivo, não foi exibido nenhum alerta durante a importação, mas se você posteriormente precisar fazer cálculos com essas datas, será necessário converter esses campos para o formato apropriado. Para fazer isso, acrescente os parâmetros adicionais como no exemplo abaixo:
```
vra <- read_delim("github/data-wrangling/VRA_do_MES_012017.csv", ";", 
                  locale = locale(encoding = "ISO-8859-1"),
                  col_types = cols("Código Autorização (DI)"= col_character(),
                                   "Partida Prevista" = col_datetime(format = "%d/%m/%Y %H:%M"),
                                   "Partida Real" = col_datetime(format = "%d/%m/%Y %H:%M"),
                                   "Chegada Prevista" = col_datetime(format = "%d/%m/%Y %H:%M"),
                                   "Chegada Real" = col_datetime(format = "%d/%m/%Y %H:%M")
                                   )
                  )
View(vra)
```



