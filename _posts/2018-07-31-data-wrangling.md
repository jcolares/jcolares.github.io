---
title: Preparação de dados com R e Rstudio
---
A atividade de coletar e preparar dados para análise consome cerca de 50% a 80% do tempo do cientista de dados. Este guia rápido tem o objetivo de agilizar esse processo, compilando no mesmo local uma lista de técnicas de manipulação de dados úteis. Você também pode usá-lo como um tutorial, se você seguir os exemplos desde o início.

Quase tudo aqui veio do excelente webinar [Data Wrangling with R e RStudio](https://www.rstudio.com/resources/webinars/data-wrangling-with-r-and-rstudio/), de Garret Grolemund, que você poderá conferir se precisar de mais detalhes.

# Pré Requisitos
## R e RStudio
Se você ainda não tem o R ou o RStudio instalados em sua máquina, siga as instruções [neste site](http://leg.ufpr.br/~fernandomayer/aulas/ce083-2016-2/R-instalacao.html) para colocar tudo em ordem antes de começar.

## Bibliotecas necessárias
Boa parte das atividades utiliza funções dos pacotes [tidyr](https://cran.r-project.org/web/packages/tidyr/README.html), [readr](https://cran.r-project.org/web/packages/readr/README.html) e [dplyr](https://www.r-project.org/nosvn/pandoc/dplyr.html). Você precisa instalá-los para conseguir executar o código de exemplo.

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
Na importação do nosso arquivo 19 linhas não puderam ser importada e um *warning* com trechos mais ou menos assim foi exibido na tela:
```
Warning: 19 parsing failures.
1 32431 Código Autorização (DI) an integer B      'github/data-wrangling/VRA_do_MES_012017.csv' file 
2 32432 Código Autorização (DI) an integer B      'github/data-wrangling/VRA_do_MES_012017.csv' row 
3 32433 Código Autorização (DI) an integer B      'github/data-wrangling/VRA_do_MES_012017.csv' col 
4 32434 Código Autorização (DI) an integer B      'github/data-wrangling/VRA_do_MES_012017.csv' expected 
5 32435 Código Autorização (DI) an integer B      'github/data-wrangling/VRA_d [... truncated]
```
Isso aconteceu porque em algumas das linhas, no campo *Código Autorização (DI)*, que é do tipo numérico, foi inserido um caractere "B".
Existem duas interpretações e duas soluções correspondentes para esse problema: Se a letra "B" foi digitada por engano e puder ser descartada, não é necessário fazer nada (as demais colunas dessas linhas foram importadas normalmente). Se a letra B é uma entrada válida para esse campo e não deve ser descartada, então a coluna deve ser convertida para alfanumérica de forma que possa acomodar letras em seu conteudo. Nesse caso, precisamos mudar seu tipo. Para fazer isso, recarregue o arquivo, desta vez acrescentando o parãmetro adicional, como no exemplo a seguir:
```
vra <- read_delim("github/data-wrangling/VRA_do_MES_012017.csv", ";", 
                  locale = locale(encoding = "ISO-8859-1")
                  col_types = cols("Código Autorização (DI)"= col_character()))
View(vra)                  
```
## Tratamento de datas
As colunas que contém datas e horas não são automaticamente interpretadas como tal. No nosso exemplo, as colunas *Partida Prevista*, *Partida Real*, *Chegada Prevista* e *Chegada Real* são tratadas como se seu conteúdo fosse texto simples. A princípio não há problema nisso e, por esse motivo, não foi exibido nenhum alerta durante a importação, mas se você posteriormente precisar fazer cálculos com essas datas, será necessário converter esses campos para o formato apropriado. Para fazer isso, acrescente os parâmetros adicionais como no exemplo abaixo:
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
# Agrupando dados de diferentes arquivos
Os dados que estamos utilizando estão divididos em um arquivo CSV para cada mês. Queremos unir todos esses arquivos em um único dataframe para podermos fazer análises anuais. Se as colunas dos arquivos são semelhantes, podemos concatená-los com os comandos abaixo: 
```
vra02 <- read_delim("github/data-wrangling/VRA_do_MES_022017.csv", ";", 
                  locale = locale(encoding = "ISO-8859-1"),
                  col_types = cols("Código Autorização (DI)"= col_character(),
                                   "Partida Prevista" = col_datetime(format = "%d/%m/%Y %H:%M"),
                                   "Partida Real" = col_datetime(format = "%d/%m/%Y %H:%M"),
                                   "Chegada Prevista" = col_datetime(format = "%d/%m/%Y %H:%M"),
                                   "Chegada Real" = col_datetime(format = "%d/%m/%Y %H:%M")
                                   )
                  )
vra03 <- read_delim("github/data-wrangling/VRA_do_MES_032017.csv", ";", 
                  locale = locale(encoding = "ISO-8859-1"),
                  col_types = cols("Código Autorização (DI)"= col_character(),
                                   "Partida Prevista" = col_datetime(format = "%d/%m/%Y %H:%M"),
                                   "Partida Real" = col_datetime(format = "%d/%m/%Y %H:%M"),
                                   "Chegada Prevista" = col_datetime(format = "%d/%m/%Y %H:%M"),
                                   "Chegada Real" = col_datetime(format = "%d/%m/%Y %H:%M")
                                   )
                  )
vra <- rbind(vra, vra02, vra03)
```
Se uma das colunas for diferente em um dos arquivos, o rbind() acima não funcionará. Em nosso exemplo, isso ocorre com a coluna "Código Justificativa", que no vra03 tem o nome "CódigoJustificativa". Para renomeá-la, digite o comando abaixo:
```
colnames(vra03)[colnames(vra03)=="CódigoJustificativa"] <- "Código Justificativa"

```
Agora é possível fazer o rbind():
```
vra <- rbind(vra, vra02, vra03)
```
## Liberando espaço na memória
Considerando que muitas vezes estaremos lidando com grandes volumes de dados, não faz muito sentido ficar ocupando espaço precioso de memória com coisas que não pretendemos mais usar. Para apagar um dataframe da memória, use o comando abaixo:
```
rm(vra02)
rm(vra03)
```


