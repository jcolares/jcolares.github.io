---
title: Preparação de dados com R 
---
A atividade de coletar e preparar dados para análise consome cerca de 50% a 80% do tempo do cientista de dados. Este guia rápido tem o objetivo de agilizar esse processo, compilando no mesmo local uma lista de técnicas de manipulação de dados úteis. Você também pode usá-lo como um tutorial, se seguir os exemplos desde o início.

## Pré-requisitos
### R e RStudio
Se você ainda não tem o R ou o RStudio instalados em sua máquina, siga as instruções [neste site](http://leg.ufpr.br/~fernandomayer/aulas/ce083-2016-2/R-instalacao.html) para colocar tudo em ordem antes de começar.

### Bibliotecas necessárias
Boa parte das atividades utiliza funções dos pacotes abaixo. Você precisará instalá-los para conseguir executar o código de exemplo:
- [tidyr](https://cran.r-project.org/web/packages/tidyr/README.html) 
- [readr](https://cran.r-project.org/web/packages/readr/README.html)
- [readxl](https://readxl.tidyverse.org/reference/read_excel.html)
- [dplyr](https://www.r-project.org/nosvn/pandoc/dplyr.html). 

### Dataset de exemplo
O dataset utilizado para todos os exemplos é o VRA, que contém o histórico de voos registrados pela ANAC. Ele pode ser obtido [nesse endereço](http://www.anac.gov.br/assuntos/dados-e-estatisticas/historico-de-voos). Só é necessário baixar os arquivos correspondentes aos meses de [janeiro](http://www.anac.gov.br/assuntos/dados-e-estatisticas/base-historica-1/vra/2017/VRA_do_MS_012017.zip), [fevereiro](http://www.anac.gov.br/assuntos/dados-e-estatisticas/base-historica-1/vra/2017/VRA_do_MS_022017.zip) e [março](Março) de 2017 e descompactar todos na mesma pasta.
Além dos dados dos voos, precisaremos de alguns arquivos XLS Apenas baixe esses arquivos na mesma pasta que os anteriores:
- [Nomes das empresas aéreas](http://www.anac.gov.br/assuntos/dados-e-estatisticas/vra/glossario_de_empresas_aereas.xls)
- [Nomes dos aeroportos.](http://www.anac.gov.br/assuntos/dados-e-estatisticas/vra/glossario_de_aerodromo.xls)
- [Nomes dos Dígitos Identificadores (tipos de autorização)](http://www.anac.gov.br/assuntos/dados-e-estatisticas/vra/glossario_de_digito_identificador.xls)
- [Nomes dos Tipos de Linha (Natureza)](http://www.anac.gov.br/assuntos/dados-e-estatisticas/vra/glossario_de_tipo_de_linha.xls)
- [Nomes das justificativas de atrasos](http://www.anac.gov.br/assuntos/dados-e-estatisticas/vra/glossario_de_justificativas.xls)

## Carregando dados de arquivos
### Arquivos XLS (ou XLSX)
Para carregar uma planilha, utilize o pacote *readxl* e a função *read_excel()*. Veja a linha de comando e, logo abaixo, a descrição dos parâmetros utilizados:
```
library(readxl)
aerodromos <- read_excel("github/data-wrangling/glossario_de_aerodromo.xls", skip = 3, 
  col_names = TRUE )
```
O primeiro parâmetro é o caminho (e o nome) do arquivo. 

*skip = 3* serve para indicar que as primeiras 3 linhas da planilha devem ser ignoradas.

*col_names = TRUE* indica que a primeira das linhas importadas contém os nomes das colunas. 

*sheet = NULL* esse parâmetro indica o nome da planilha dentro do arquivo. Se não for informado, como no nosso caso, é lida a primeira planilha.

### Visualizando os dados
Para visualizar os dados, use o comando abaixo. Atenção para a inicial que deve ser maiúscula.
```
View(aerodromos)
```
Carregue as demais planilhas com os comandos abaixo:
```
DI <- read_excel("github/data-wrangling/glossario_de_digito_identificador.xls", 
  col_names = TRUE, skip = 3)
empresas <- read_excel("github/data-wrangling/glossario_de_empresas_aereas.xls", 
  col_names = TRUE, skip = 3)
justificativas <- read_excel("github/data-wrangling/glossario_de_justificativas.xls", 
  col_names = TRUE, skip = 3)
```

### Arquivos CSV (ou TXT)
Para carregar um arquivo CSV, utilize o pacote *readr* e a função *read_delim()*, especificando o ";" como delimitador de colunas do arquivo:
```
library(readr)
vra <- read_delim("github/data-wrangling/VRA_do_MES_012017.csv", ";")
View(vra)
```
### Arquivos RData
Os arquivos RData são criados no R e podem conter um ou mais objetos. Para carregá-los, use o comando *load()*:
```
load("github/data-wrangling/vra.RData")
```
## Selecionando a codificação correta
Eventualmente, podem ocorrer erros de codificação do arquivo caso você esteja no Linux e seu CSV tiver sido criado no Windows ou vice-versa. Nesses casos, você pode acrescentar um parâmetro que evita esse problema:
```
vra <- read_delim("github/data-wrangling/VRA_do_MES_012017.csv", ";", 
                  locale = locale(encoding = "ISO-8859-1"))
View(vra)
```
Os tipos de encoding mais comuns são o "ISO-8859-1" e o "UTF-8".

## Correção de problemas na carga de dados
### Tratamento de erros 
Na importação do nosso arquivo 19 linhas não puderam ser importadas e um *warning* foi exibido na tela com trechos semelhantes a esses:
```
Warning: 19 parsing failures.
1 32431 Código Autorização (DI) an integer B      
'github/data-wrangling/VRA_do_MES_012017.csv' 
```
Isso aconteceu porque em algumas das linhas, no campo *Código Autorização (DI)*, que é do tipo numérico, foi inserido um caractere "B".
Existem duas interpretações e duas soluções correspondentes para esse problema: Se a letra "B" foi digitada por engano e puder ser descartada, não é necessário fazer nada (as demais colunas dessas linhas foram importadas normalmente). Se a letra B é uma entrada válida para esse campo e não deve ser descartada, então a coluna deve ser convertida para alfanumérica de forma que possa acomodar letras em seu conteudo. Nesse caso, precisamos mudar seu tipo. Para fazer isso, recarregue o arquivo, desta vez acrescentando um parâmetro adicional, como no exemplo a seguir:
```
vra <- read_delim("github/data-wrangling/VRA_do_MES_012017.csv", ";", 
                  locale = locale(encoding = "ISO-8859-1")
                  col_types = cols("Código Autorização (DI)"= col_character()))
View(vra)                  
```
### Tratamento de datas
As colunas que contém datas e horas não são automaticamente interpretadas como tal. No nosso exemplo, as colunas *Partida Prevista*, *Partida Real*, *Chegada Prevista* e *Chegada Real* são tratadas como se seu conteúdo fosse texto simples. A princípio não há problema nisso e, por esse motivo, não foi exibido nenhum alerta durante a importação, mas se você posteriormente precisar fazer cálculos com essas datas, será necessário converter esses campos para o formato apropriado. Para fazer isso, acrescente parâmetros adicionais como no exemplo abaixo:
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
No arquivo original de justificativas (glossario_de_justificativas.xls) o caractere "/" foi erroneamente representado como "¿". na variável *Descrição da Justificativa*. Para substituir todos os "¿" por "/", execute o comando abaixo:
```
justificativas$"Descrição Justificativa" <- gsub('¿', '/', justificativas$"Descrição Justificativa")
```

## Combinando múltiplos arquivos
### Unindo arquivos
Os dados que estamos utilizando estão segmentados em um arquivo CSV para cada mês. Queremos unir todos esses arquivos em um único dataframe para podermos fazer análises abrangendo o período completo. Quando os nomes e tipos das colunas dos arquivos são semelhantes, podemos concatená-los como nos comandos abaixo: 
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
Considerando que muitas vezes estaremos lidando com grandes volumes de dados, não faz muito sentido ficar ocupando espaço precioso de memória com coisas que não pretendemos mais usar. Para apagar um objeto da memória, siga o exemplo abaixo:
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

Se você não estiver familiarizado com o conceito de join, antes de prosseguir dê uma lida [nesse artigo](https://www.devmedia.com.br/inner-cross-left-rigth-e-full-joins/21016). Caso contrário, veja a seguir como fazer as junções no R:

### Nomes de colunas
Quando os dois datasetes têm colunas de nomes iguais, elas são mescladas automaticamente. Do contrário, use a notação by.x=nome-da-coluna, by.y=nome-da-coluna

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

## Juntando os dataframes do dataset de exemplo
Para criar, a partir do *vra*, um dataframe que contenha não apenas os códigos, mas também as descrições dos aeródromos, justificativas e etc, execute os outer joins abaixo:
```
vra <- merge(x = vra, y = empresas, by = "ICAO Empresa Aérea", all.x = TRUE)
vra <- merge(x = vra, y = DI, by = "Código Autorização (DI)", all.x = TRUE)
vra <- merge(x = vra, y = justificativas, by = "Código Justificativa", all.x = TRUE)
View(vra)
```
Os outer joins abaixo são ambos baseados no mesmo dataframe, então renomearemos alguns campos na sequência:
```
vra <- merge(x = vra, 
             y = aerodromos, 
             by.x = "ICAO Aeródromo Origem", 
             by.y = "ICAO Aeródromo", 
             all.x = TRUE)
colnames(vra)[colnames(vra)=="Descrição Aeródromo"] <- "Descrição Aeródromo Origem"
colnames(vra)[colnames(vra)=="Cidade"] <- "Cidade Origem"
colnames(vra)[colnames(vra)=="UF"] <- "UF Origem"
colnames(vra)[colnames(vra)=="País"] <- "País Origem"
colnames(vra)[colnames(vra)=="Continente"] <- "Continente Origem"
View(vra)

vra <- merge(x = vra, 
             y = aerodromos, 
             by.x = "ICAO Aeródromo Destino", 
             by.y = "ICAO Aeródromo", 
             all.x = TRUE)
colnames(vra)[colnames(vra)=="Descrição Aeródromo"] <- "Descrição Aeródromo Destino"
colnames(vra)[colnames(vra)=="Cidade"] <- "Cidade Destino"
colnames(vra)[colnames(vra)=="UF"] <- "UF Destino"
colnames(vra)[colnames(vra)=="País"] <- "País Destino"
colnames(vra)[colnames(vra)=="Continente"] <- "Continente Destino"
View(vra)
```


## Manipulando os dataframes
### Criando colunas calculadas
É provável que, em alguns casos, surja a necessidade de criar uma coluna adicional contendo o resultado de um cálculo entre outras colunas. Quando isso ocorrer, você pode utilizar a função *mutate()*. Em nosso cenário de exemplo, precisamos de uma nova coluna com a quantidade de minutos que o vôo atrasou na saída e uma outra com quantidade em minutos de atraso na chegada. Os comandos a seguir criam essas novas colunas. Observe que para calcular a diferença entre os horários de chegada e partida, utilizamos ainda a função *difftime()* com o parâmetro *units="mins"* que fornece a diferença desejada em minutos:
```
vra <- mutate(vra, "Atraso Partida" = difftime(vra$"Partida Real" , 
                                               vra$"Partida Prevista", 
                                               units = "mins"))
vra <- mutate(vra, "Atraso Chegada" = difftime(vra$"Chegada Real" , 
                                               vra$"Chegada Prevista",
                                               units = "mins"))
```
### Removendo uma coluna
Para remover uma ou mais colunas atribua a ela NULL como no exemplo abaixo:
```
vra$`Atraso Chegada` <- NULL
```

Para remover uma ou mais colunas, você também pode utilizar a função *select()*. Ela retorna um novo dataset apenas com as colunas especificadas:
```
select(vra,"ICAO Empresa Aérea","Número Voo","Código Autorização (DI)",
           "Código Tipo Linha","ICAO Aeródromo Origem" ,"ICAO Aeródromo Destino",
           "Partida Prevista","Partida Real","Chegada Prevista","Chegada Real",
           "Situação Voo","Código Justificativa")
```
O comando acima não armazena o resultado fornecido. Para fazer isso, atribua-o a um objeto. Pode ser o mesmo objeto original ou pode ser outro:
```
vra1 <- select(vra,"ICAO Empresa Aérea","Número Voo","Código Autorização (DI)",
           "Código Tipo Linha","ICAO Aeródromo Origem" ,"ICAO Aeródromo Destino",
           "Partida Prevista","Partida Real","Chegada Prevista","Chegada Real",
           "Situação Voo","Código Justificativa")
```
### Filtrando linhas
Assim como as colunas, você pode remover linhas indesejadas do seu dataset. Para isso, utilize a função *filter()*.
Por exemplo, para ver (e remover) as linhas com o comando abaixo:
```
filter(vra, `ICAO Aeródromo Origem`=="CYYZ")
```
Para remover, armazene no objeto original todas as linhas diferentes das que foram listadas anteriormente.
```
vra1 <- filter(vra, `ICAO Aeródromo Origem`!="CYYZ")
```
## Conclusão 
O objetivo da preparação de dados (data wrangling) é deixar os dados prontos para a fase de análise exploratória. Esse post mostra apenas algumas transformações mais comuns, mas existem diversas outras. Os pacotes dplyr e tidyr possuem diversas outras funções específicas para essa etapa do processamento.

Uma boa forma de aprender outras técnicas é assistindo o ótimo webinar [Data Wrangling with R e RStudio](https://www.rstudio.com/resources/webinars/data-wrangling-with-r-and-rstudio/), de Garret Grolemund. Vale a pena dar uma olhada.


