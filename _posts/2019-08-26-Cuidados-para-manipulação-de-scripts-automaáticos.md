---
layout: post
title: Cuidados para manipulação de scripts automáticos
tags: [diário]
---  
##

 Este é o primeiro post do 'diário de bordo', posts com essa tag serão todos motivados por algo que me aconteceu enquanto navegava o oceano dos dados, algo que me levou a pensar 'caramba isso aqui vale um registro'. Dito isso, prossigamos.  
 
 Em empresas de TI, é comum que certos processos entrem na rotina e precisem ser executados com frequência, então para se livrar da repetição manual, é interessante tornar o script automático. Em geral isso consiste em aceitar entradas dinâmicas, preparar warnings para alertar ocorrências estranhas entre outros cuidados e dá trabalho até que o script esteja realmente sem arestas, certo?! As vezes precisamos revisitar aquele belo e polido script, para acrescentar ou otimizar coisas e é claro, é necessário repetir o processo de aparar arestas, de modo que ele não quebre e te faça querer cometer atrocidades quando descobrir que aquele script que leva 12 horas para rodar e você iniciou no dia anterior, quebrou no meio da noite enquanto você estava dormindo e sonhando que iria ter o resultado no dia seguinte. No diário de hoje deixo uma nota para mim e a quem interessar, sobre algo que chamamos de 'smoke test', um teste que garante que sua manipulação não quebrou nada, para não esquecer o passo a passo necessário para polir com sucesso seu script. 
 
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

#Este 'if' é mais uma boa prática, caso o usuário cometa erros ao setar os argumentos, o script para na hora, e retorna um aviso.
#O aviso abaixo orienta o usuario sobre quais argumentos usar e a ordem em que devem ser informados. É ativado quando o 
#usuário informa uma quantidade incorreta de argumentos. Algo simples, normalmente um script precisa de vários avisos e checkups. 
if (length(args) != 3) {
  message("Os argumentos são: <path_file_input> <parametro2> <path_saida> \n O script assume entrada zipada")
  quit()
}


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
Último ponto, o que vai sair do seu script? Seja a saída um modelo, imagens ou bases, é interessante que haja uma rotina no R que garanta que, caso o script esteja rodando pela segunda vez, sua saída já existente seja copiada para uma pasta auxiliar ( que eu costumo chamar de histórico) pois você pode precisar do seu resultado anterior. Abaixo, complemento o exemplo dado no passo um, com uma forma de fazer isso no R.

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

#Este 'if' é mais uma boa prática, caso o usuário cometa erros ao setar os argumentos, o script para na hora, e retorna um aviso.
#O aviso abaixo orienta o usuario sobre quais argumentos usar e a ordem em que devem ser informados. É ativado quando o 
#usuário informa uma quantidade incorreta de argumentos. Algo simples, normalmente um script precisa de vários avisos e checkups. 
if (length(args) != 3) {
  message("Os argumentos são: <path_file_input> <parametro2> <path_saida> \n O script assume entrada zipada")
  quit()
}

# argumentos para smoke test
# dados_entrada = ~/minha_menor_base.csv
# parametro2 = 2019
# path_saida = "~/output_teste/"

# captura a data e hora em que a saida foi concluída.
mytime <- format(Sys.time(), format = "%Y_%m_%d_%H_%M_%S")

# Essa linha sobrescreve o arquivo de saída sempre que é executada
fwrite(arquivo_saida,paste0(path_saida,"saida",".txt"))

# Essa linha deixa o arquivo_saida salvo no diretorio 'historico' informando a data em que ele foi gerado.
# Como a variável 'mytime' está sempre mudando, os arquivos no historico não serão sobrescritos
fwrite(arquivo_saida,paste0(path_saida,"historico/saida_",mytime,".txt"))
```

Essas foram minhas observações, por enquanto ter esses 5 passos em mente tem funcionado, mas *by all means* compartilhem boas práticas que vocês acham que deveriam estar citadas aqui.  

