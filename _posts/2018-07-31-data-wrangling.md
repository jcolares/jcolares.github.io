---
title: Preparação de dados com R e Rstudio
---
A atividade de coletar e preparar dados para análise consome cerca de 50% a 80% do tempo do cientista de dados. Este guia rápido tem o objetivo de agilizar esse processo, compilando no mesmo local uma lista de técnicas de manipulação de dados úteis. Você também pode usá-lo como um tutorial, se você seguir os exemplos desde o início.

Quase tudo aqui veio do excelente webinar [Data Wrangling with R e RStudio](https://www.rstudio.com/resources/webinars/data-wrangling-with-r-and-rstudio/), de Garret Grolemund, que você poderá conferir se precisar de mais detalhes.

## Pré Requisitos
### R e RStudio
Se você ainda não tem o R ou o RStudio instalados em sua máquina, siga as instruções [neste site](http://leg.ufpr.br/~fernandomayer/aulas/ce083-2016-2/R-instalacao.html) para colocar tudo em ordem antes de começar.

### Bibliotecas necessárias
Boa parte das atividades utiliza funções dos pacotes abaixo. Você precisará instalá-los para conseguir executar o código de exemplo:
[tidyr](https://cran.r-project.org/web/packages/tidyr/README.html) 
[readr](https://cran.r-project.org/web/packages/readr/README.html)
[readxl](https://readxl.tidyverse.org/reference/read_excel.html)
[dplyr](https://www.r-project.org/nosvn/pandoc/dplyr.html). 

### Dataset de exemplo
O dataset utilizado para todos os exemplos é o VRA, que contém o histórico de voos registrados pela ANAC. Ele pode ser obtido [nesse endereço](http://www.anac.gov.br/assuntos/dados-e-estatisticas/historico-de-voos). Só é necessário baixar os arquivos correspondentes aos meses de [janeiro](http://www.anac.gov.br/assuntos/dados-e-estatisticas/base-historica-1/vra/2017/VRA_do_MS_012017.zip), [fevereiro](http://www.anac.gov.br/assuntos/dados-e-estatisticas/base-historica-1/vra/2017/VRA_do_MS_022017.zip) e [março](Março) de 2017 e descompactar todos na mesma pasta.
Além dos dados dos voos, precisaremos de alguns arquivos XLS Apenas baixe esses arquivos na mesma pasta que os anteriores:
[Nomes das empresas aéreas](http://www.anac.gov.br/assuntos/dados-e-estatisticas/vra/glossario_de_empresas_aereas.xls)
[Nomes dos aeroportos.](http://www.anac.gov.br/assuntos/dados-e-estatisticas/vra/glossario_de_aerodromo.xls)
[Nomes dos Dígitos Identificadores (tipos de autorização)](http://www.anac.gov.br/assuntos/dados-e-estatisticas/vra/glossario_de_digito_identificador.xls)
[Nomes dos Tipos de Linha (Natureza)](http://www.anac.gov.br/assuntos/dados-e-estatisticas/vra/glossario_de_tipo_de_linha.xls)
[Nomes das justificativas de atrasos](http://www.anac.gov.br/assuntos/dados-e-estatisticas/vra/glossario_de_justificativas.xls)

## Carregando dados para o R
### Arquivos XLS (ou XLSX)
Para carregar uma planilha, utilize o pacote readxl e a função read_excel(). Veja a linha de comando e, logo abaixo, a descrição dos parâmetros utilizados:
```
library(readxl)
aerodromos <- read_excel("github/data-wrangling/glossario_de_aerodromo.xls", skip = 3, col_names = TRUE )
```
O primeiro parâmetro é o caminho (e o nome) do arquivo. 
*skip = 3* serve para indicar que as primeiras 3 linhas da planilha devem ser ignoradas
*col_names = TRUE* indica que a primeira das linhas importadas contém os nomes das colunas
*sheet = NULL* esse parâmetro indica o nome da planilha dentro do arquivo. Se não for informado, como no nosso caso, é lida a primeira planilha.

### Visualizando os dados
Para visualizar os dados, use o comando abaixo. Atenção para a inicial que deve ser maiúscula.
```
View(aerodromos)
```
Carregue as demais planilhas com os comandos abaixo:
```
DI <- read_excel("github/data-wrangling/glossario_de_digito_identificador.xls", col_names = TRUE, skip = 3)
empresas <- read_excel("github/data-wrangling/glossario_de_empresas_aereas.xls", col_names = TRUE, skip = 3)
justificativas <- read_excel("github/data-wrangling/glossario_de_justificativas.xls", col_names = TRUE, skip = 3)
```

### Arquivos CSV (ou TXT)
Para carregar um arquivo CSV, utilize o pacote readr e a função read_delim(), especificando o ";" como delimitador de colunas do arquivo:
```
library(readr)
vra <- read_delim("github/data-wrangling/VRA_do_MES_012017.csv", ";")
View(vra)
```
## Selecionando a codificação correta
Eventualmente, podem ocorrer erros de codificação do arquivo caso você esteja no Linux e seu CSV tiver sido criado no Windows ou vice-versa. Nesses casos, você pode acrescentar um parâmetro para evitar esse problema:
```
vra <- read_delim("github/data-wrangling/VRA_do_MES_012017.csv", ";", 
                  locale = locale(encoding = "ISO-8859-1"))
View(vra)
```
Os tipos de encoding mais comuns são o "ISO-8859-1" e o "UTF-8".

## Alterando o tipo de dados da coluna
### Tratamento de erros 
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
### Tratamento de datas
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
### Corrigindo caracteres indesejados
No arquivo original de justificativas () o caractere "/" foi erroneamente representado como "¿". na variável *Descrição da Justificativa*. Para substituir todos os "¿" por "/", execute o comando abaixo:
```
justificativas$"Descrição Justificativa" <- gsub('¿', '/', justificativas$"Descrição Justificativa")
```

## Combinando múltiplos arquivos
### Unindo arquivos
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
### Renomeando colunas
Se uma das colunas tiver o nome ou tipo de dados diferente em um dos arquivos, o rbind() acima não funcionará. Um exemplo disso ocorre com a coluna "Código Justificativa", que no vra03 tem o nome "CódigoJustificativa". Para renomeá-la, digite o comando abaixo:
```
colnames(vra03)[colnames(vra03)=="CódigoJustificativa"] <- "Código Justificativa"

```
Agora é possível fazer o rbind():
```
vra <- rbind(vra, vra02, vra03)
```
Obs.: Também é possível unir dois ou mais arquivos horizontalmente com a função *cbind()*, caso todas as suas colunas sejam diferentes. Isso pode ser útil para juntar em uma mesma tabela dados complementares. Porém fique atento para que cada arquivo tenha as linhas correspondentes entre si.

### Liberando espaço na memória
Considerando que muitas vezes estaremos lidando com grandes volumes de dados, não faz muito sentido ficar ocupando espaço precioso de memória com coisas que não pretendemos mais usar. Para apagar um dataframe da memória, use o comando abaixo:
```
rm(vra02)
rm(vra03)
```
### Organizando o layout dos dataframes
Vamos agora executar os comandos abaixo e deixar os dataframes organizados para os próximos passos:

**empresas**
```
colnames(empresas)[colnames(empresas)=="Sigla OACI"] <- "ICAO Empresa Aérea"
colnames(empresas)[colnames(empresas)=="Nome Empresas"] <- "Empresa"
colnames(empresas)[colnames(empresas)=="Nacional ou Estrangeira"] <- "Nacionalidade"
empresas <- select(empresas, "ICAO Empresa Aérea", "Empresa", "Nacionalidade")
View(empresas)
```
**DI** 
```
colnames(DI)[colnames(DI)=="Código"] <- "Código Autorização (DI)"
colnames(DI)[colnames(DI)=="Descrição"] <- "Tipo de Voo"
View(DI)
```
**justificativas** 
```
colnames(justificativas)[colnames(justificativas)=="Sigla Justificativa"] <- "Código Justificativa"
View(justificativas)
```
**aerodromos** 
```
colnames(aerodromos)[colnames(aerodromos)=="Aeródromo"] <- "ICAO Aeródromo"
colnames(aerodromos)[colnames(aerodromos)=="Descrição"] <- "Descrição Aeródromo"
View(aerodromos)
```
## Join de dataframes

Veja as regras gerais:

### Nomes de colunas
Quando os dois datasetes têm colunas de nomes iguais, elas são mescladas automáticamente. Do contrário, use a notação by.x=nome-da-coluna, by.y=nome-da-coluna

### Inner join
teste <- merge(df1, df2)

### Left outer join
teste <- merge(x = df1, y = df2, by = "nome-da-coluna", all.x = TRUE)

### Right outer join
teste <- merge(x = df1, y = df2, by = "nome-da-coluna", all.y = TRUE)

### Left outer join
teste <- merge(x = df1, y = df2, by = "nome-da-coluna", all.x = TRUE)

### Cross join
teste <- merge(x = df1, y = df2, by = NULL)

## Mesclando os dataframes do dataset de exemplo
Para criar a partir de vra um dataframe que contenha não apenas os códigos, mas também as descrições dos aeródromos, justificativas e etc, execute os outer joins abaixo:
```
vra <- merge(x = vra, y = empresas, by = "ICAO Empresa Aérea", all.x = TRUE)
vra <- merge(x = vra, y = DI, by = "Código Autorização (DI)", all.x = TRUE)
vra <- merge(x = vra, y = justificativas, by = "Código Justificativa", all.x = TRUE)
View(vra)
```
Os outer joins abaixo são ambos baseados no mesmo dataframe, então renomearemos alguns campos na sequência:
```
vra <- merge(x = vra, y = aerodromos, by.x = "ICAO Aeródromo Origem", by.y = "ICAO Aeródromo", all.x = TRUE)
colnames(vra)[colnames(vra)=="Descrição Aeródromo"] <- "Descrição Aeródromo Origem"
colnames(vra)[colnames(vra)=="Cidade"] <- "Cidade Origem"
colnames(vra)[colnames(vra)=="UF"] <- "UF Origem"
colnames(vra)[colnames(vra)=="País"] <- "País Origem"
colnames(vra)[colnames(vra)=="Continente"] <- "Continente Origem"
View(vra)

vra <- merge(x = vra, y = aerodromos, by.x = "ICAO Aeródromo Destino", by.y = "ICAO Aeródromo", all.x = TRUE)
colnames(vra)[colnames(vra)=="Descrição Aeródromo"] <- "Descrição Aeródromo Destino"
colnames(vra)[colnames(vra)=="Cidade"] <- "Cidade Destino"
colnames(vra)[colnames(vra)=="UF"] <- "UF Destino"
colnames(vra)[colnames(vra)=="País"] <- "País Destino"
colnames(vra)[colnames(vra)=="Continente"] <- "Continente Destino"
View(vra)
```


## Manipulando os dataframes
### Criando colunas calculadas
É provável que, em muitos casos, surja a necessidade de criar uma coluna adicional contendo o resultado de um cálculo entre duas outras colunas. Quando isso ocorrer, você pode utilizar a função *mutate()*. Por exemplo, precisamos de uma nova coluna com a quantidade de minutos que o vôo atrasou na saída e uma outra com quantidade em minutos de atraso na chegada. Os comandos a seguir criam essas novas colunas. Observe que para calcular a diferença entre os horários de chegada e partida, utilizamos ainda a função *difftime()*, que dá a diferença em segundos, e dividimos o resultado por 60, para termos a diferença entre os horários em minutos:
```
vra <- mutate(vra, "Atraso Partida" = difftime(vra$"Partida Real" , vra$"Partida Prevista") / 60)
vra <- mutate(vra, "Atraso Chegada" = difftime(vra$"Chegada Real" , vra$"Chegada Prevista") / 60)
```
### Removendo uma coluna
Para remover uma coluna, utilize a função *select()*. Ela retorna um novo dataset apenas com as colunas especificadas. Ex: o comando abaixo retorna o dataset sem as colunas "Atraso Partida" e "Atraso Chegada":
```
select(vra,"ICAO Empresa Aérea","Número Voo","Código Autorização (DI)","Código Tipo Linha","ICAO Aeródromo Origem" ,"ICAO Aeródromo Destino","Partida Prevista","Partida Real","Chegada Prevista","Chegada Real" ,"Situação Voo","Código Justificativa")
```
para remover as colunas e armazenar o resultado, atribua-o a um objeto:
```
vra <- select(vra,"ICAO Empresa Aérea","Número Voo","Código Autorização (DI)","Código Tipo Linha","ICAO Aeródromo Origem" ,"ICAO Aeródromo Destino","Partida Prevista","Partida Real","Chegada Prevista","Chegada Real" ,"Situação Voo","Código Justificativa")
```
### Filtrando linhas
Assim como as colunas, você pode remover linhas indesejadas do seu dataset. Para isso, utilize a função *filter()*
Por exemplo, para ver (e remover) as linhas com o comando abaixo:
```
filter(vra, `ICAO Aeródromo Origem`=="CYYZ")
```
Para remover, armazene no objeto original todas as linhas diferentes das que foram listadas
```
vra1 <- filter(vra, `ICAO Aeródromo Origem`!="CYYZ")
```
