---
title: Preparação de dados com R e Rstudio
---
# Preparação de dados com R e Rstudio

A atividade de coletar e preparar dados para análise consome cerca de 50% a 80% do tempo do cientista de dados. Este guia rápido tem o objetivo de economizar tempo, compilando no mesmo local uma lista de técnicas de manipulação de dados úteis, mas ele também pode ser lido como um tutorial.

Quase tudo aqui veio do excelente vídeo "Data Wrangling with R e RStudio" de Garret Grolemund, que você pode assistir se precisar de mais detalhes.

## Pré Requisitos
### R e RStudio
Se você ainda não tem o R ou o RStudio instalados em sua máquina, siga as instruções [neste site](http://leg.ufpr.br/~fernandomayer/aulas/ce083-2016-2/R-instalacao.html) para colocar tudo em ordem antes de começar.

### Bibliotecas necessárias
Boa parte das atividades utiliza funções das bibliotecas tidyr e dplyr. Você precisa baixar e instalá-las para conseguir executar o código de exemplo.

### Dataset de exemplo
O dataset utilizado para todos os exemplos é o VRA, que contém o histórico de voos registrados pela ANAC. Ele pode ser obtido [nesse endereço](http://www.anac.gov.br/assuntos/dados-e-estatisticas/historico-de-voos). Para executar os exemplos, só é necessário baixar os arquivos correspondentes aos meses de [janeiro](http://www.anac.gov.br/assuntos/dados-e-estatisticas/base-historica-1/vra/2017/VRA_do_MS_012017.zip), [fevereiro](http://www.anac.gov.br/assuntos/dados-e-estatisticas/base-historica-1/vra/2017/VRA_do_MS_022017.zip) e [março](Março) de 2017 e descompactar todos na mesma pasta.



