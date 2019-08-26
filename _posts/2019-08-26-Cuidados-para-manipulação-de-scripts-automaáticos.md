---
layout: post
title: Cuidados para manipulação de scripts automáticos
tags: [diário]
---  
 #cuidado com libraries e versoes (rodadas em computadores diferentes)
 #cuidado com fwrites + criação de bases historicas
##

 Segundo registro do diário de bordo. Em empresas de TI é comum que certos processos entrem na rotina e precisem ser executados com frequência, então para se livrar da repetição manual, é interessante tornar o script automático. Em geral isso consiste em aceitar entradas dinâmicas, preparar warnings para alertar ocorrências estranhas entre outros cuidados e dá trabalho até que o script esteja realmente sem arestas, certo?! As vezes precisamos revisitar aquele belo e polido script, para acrescentar ou otimizar coisas e é claro, é necessário repetir o processo de aparar arestas, de modo que ele não quebre e te faça querer cometer atrocidades quando descobrir que aquele script que leva 12 horas para rodar e você iniciou no dia anterior, quebrou no meio da noite enquanto você estava dormindo sonhando que iria ter o resultado no dia seguinte. No diário de hoje deixo uma nota para mim e a quem interessar, sobre 'smoke test', para não esquecer o passo a passo necessário para polir com sucesso seu script. 
 
 agosto/2019  
 objetivo : Manipular scripts automáticos
 previsão do tempo : ensolarado (porém frio)
 sensação térmica : 15°C

- #### Passo 1  
Um script automático é como uma função gigante, certo?! Temos parâmetros de entrada que, no meu caso, sempre envolvem uma base de dados, então sempre deixe comentado o path de uma base teste, a menor possível para quando você for fazer sua rodada de teste apenas remover o comentário e rodar tudo sabendo que vai ter um teste efetivo e pronto o mais rápido possível, como no exemplo abaixo.

```r
library(tidyverse, quietly = TRUE)
library(data.table, quietly = TRUE)

#----------------------------------------------------------------------
# Argumentos
# este comando permite que o script seja chamado por um bash, recebendo dele os argumentos setados (recomendo checar o help do commandArgs)
arg=commandArgs(trailingOnly = T)
dados_entrada = arg[1]
parametro2 = arg[2]
path_saida = arg[3]

# argumentos para smoke test
# dados_entrada = ~/minha_menor_base.csv
# parametro2 = 2019
# path_saida = "~/output_teste/"

```

- #### passo 2:  



```r
# adicionar informação de intervalo entre compras antes de passar na rede
suporte <- fread("~/fraldas_suporte.csv") %>%  
  dplyr::mutate(ano_mes = format(as.Date(ano_mes_dia), format=c("%Y-%m")),
                ano = lubridate::year(ano_mes_dia),
                ano_mes_dia = as.Date(ano_mes_dia)) %>% 
  dplyr::filter(!(stringr::str_detect(tipo,  pattern = "tipo_indesejado"))) %>%
  setDT()

#função que checa intervalo médio (em dias) entre compras efetuadas (agrupado por pessoa e ano)
intervalo <- invoice_interval(suporte) 
intervalo[,int_compra:= ifelse(is.nan(int_compra),365,int_compra)] #(valor nan significa apenas 1 compra naquele ano, então nan foi substituido por 365)

all <- fread("~/treino_parent_06_2019.txt")
# o problema estava aqui
all <- merge(all,intervalo, all.x = T, by = c("id_cpf","ano"))
```


- #### problema 3:  
divergências em alguns gráficos  
Solução: descobrimos um filtro não registrado, valor_unitario/unidade deveria ser maior que 0.2 e menor que 5




