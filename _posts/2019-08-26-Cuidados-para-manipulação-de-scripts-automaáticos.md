---
layout: post
title: Cuidados para manipulação de scripts automáticos
tags: [diário]
---  
##

 Segundo registro do diário de bordo. Em empresas de TI, é comum que certos processos entrem na rotina e precisem ser executados com frequência, então para se livrar da repetição manual, é interessante tornar o script automático. Em geral isso consiste em aceitar entradas dinâmicas, preparar warnings para alertar ocorrências estranhas entre outros cuidados e dá trabalho até que o script esteja realmente sem arestas, certo?! As vezes precisamos revisitar aquele belo e polido script, para acrescentar ou otimizar coisas e é claro, é necessário repetir o processo de aparar arestas, de modo que ele não quebre e te faça querer cometer atrocidades quando descobrir que aquele script que leva 12 horas para rodar e você iniciou no dia anterior, quebrou no meio da noite enquanto você estava dormindo e sonhando que iria ter o resultado no dia seguinte. No diário de hoje deixo uma nota para mim e a quem interessar, sobre algo que chamamos de 'smoke test', um teste que garante que sua manipulação não quebrou nada, para não esquecer o passo a passo necessário para polir com sucesso seu script. 
 
 agosto/2019  
 objetivo : Manipular scripts automáticos  
 previsão do tempo : ensolarado (porém frio)  
 sensação térmica : 15°C

- #### Passo 1  
Um script automático é como uma função gigante, certo?! Temos parâmetros de entrada que, no meu caso, sempre envolvem uma base de dados, então sempre deixe comentado o *path* de uma base teste, a menor possível para quando você for fazer sua rodada de teste apenas remover o comentário e rodar tudo sabendo que vai ter um teste efetivo e pronto o mais rápido possível, como no exemplo abaixo.

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

- #### Passo 2
Se você precisa automatizar um script, provavelmente é porque sua máquina não será a única que vai rodar ele e isso pede pelo menos 3 cuidados. O primeiro é em relação a versão, o ideal é que todas as máquinas estejam com a mesma versão do software utilizado e que esteja documentado a versão em que o script foi escrito. Problemas de versão podem dar mais dor de cabeça do que você espera.

- #### Passo 3
O segundo problema em rodar coisas em muitas máquinas diferentes é em relação a bibliotecas, neste caso vamos falar de um problema que considero muito trollador das galáxias. Quando eu testava um script, costumava limpar o environment e começar rodando do zero, obviamente uma boa prática, mas não suficiente. O ideal é reiniciar a sessão, o R tem a opção "restart R session", se não houver um atalho desse tipo no seu software, basta fechar e abrir novamente. Por quê? Porque se você rodar uma biblioteca, ela continua carregada mesmo que o environment seja limpo e isso pode te fazer esquecer de declarar aquela biblioteca nova no início do script. 

Aliás, este problema está dentro do anterior, já que algumas bibliotecas rodam apenas em versões específicas. 

- #### Passo 4  
Terceiro cuidado para ter multiplas maquinas rodando seu script. Se o script usa paralelização em algum momento, tenha certeza de que o cálculo de cores utilizados será dinâmico, outras máquinas podem não ter a mesma quantidade de cores que a sua.

- #### Passo 5    
 #cuidado com fwrites + criação de bases historicas
