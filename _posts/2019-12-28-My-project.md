---
layout: post
title: Testes A/B bayesianos no shiny
tags: [R, Estatística, talk, BI]

---


 Como prometido no post anterior, vou compartilhar aqui minha apresentação para quem não foi ao SatRday e entrar um pouco mais em detalhes que não cabem numa lightning talk. 
 
 ### Explicando e contextualizando o teste
 
 O objetivo do projeto é executar testes A/B bayesiano com shiny, para explicar o teste A/B,  vamos  pensar num contexto de mercado e imaginar uma rede de grande porte que quer lançar um novo produto de marca própria e tem a versão A e B desse produto. Para embasar a decisão final eles querem saber qual produto vai ter melhor aceitação por parte dos clientes, então eles recorrem aos clientes cadastrados no programa de fidelidade para executar um experimento, oferecendo amostras grátis da versão A para metade dessas pessoas e a versão B para a outra.

 
 Um comentário válido, apesar deste ser um exemplo ilustrativo, é sobre o quão conveniente seria num caso como este aplicar o experimento à pessoas cadastradas no programa de fidelidade. Isso porque é de se esperar que essas pessoas façam grande parte das compras, senão todas, nos estabelecimentos da rede, o que significa que os dados serão bem completos e é possível controlar outras variáveis de compra caso seja necessário. Ao mesmo tempo isso não significa que essas pessoas estão mais propensas a comprar algo por ser de marca própria da rede, eles precisam se convencer do custo benefício do produto primeiro. Dito isso, vejamos como se apresentaria a base de compras da rede.
 
 ![base_market](https://raw.githubusercontent.com/IanniMuliterno/iannimuliterno.github.io/master/img/base.png)
 
  Nesta base é possível identificar o item comprado, seu valor, unidades compradas e data de compra, além das variáveis identificadores de transação e cliente. Esse tipo de informação rende uma análise exploratória bacana, que vamos ver mais para frente, por hora é necessário preparar os dados para o "formato de entrada" de um teste A/B. Fazemos isso construindo uma variável que responda a seguinte pergunta : " Qual versão do produto mais fez as pessoas voltarem para compra-lo? ". 
  
  ```r
dt_teste <- dt %>% 
  group_by(user) %>%
  summarise(success = sum(volume[item == "Item_1"])) %>%
  mutate(success = ifelse(success > 0,1,0)) 
dt_teste
```

 ![binomial](https://raw.githubusercontent.com/IanniMuliterno/iannimuliterno.github.io/master/img/binomial2.png)

  Considerando que a base `dt` já está filtrada para retornar apenas resultados a partir do início do experimento, temos uma contagem de sucessos e fracassos por pessoa caso ela tenha comprado ou não o item de interesse, esta variável segue distribuição binomial e a probabilidade de sucesso `p` desta distribuição é conhecida como taxa de conversão. Nos moldes bayesianos podemos usar a relação beta-binomial tendo a taxa de conversão como uma variável que segue distribuição beta em que alpha é o número de sucessos e beta o número de fracassos.
  
  <p align="center">
   <img src="https://raw.githubusercontent.com/IanniMuliterno/iannimuliterno.github.io/master/img/P.png">
 </p>
 Fazendo essa associação, podemos usar a beta para fazer simulações da taxa de conversão. 
 
 ```r
 trials <- 1000000
success <- sum(dt_teste$success)
n_users <- nrow(dt_teste)

CR <- data.table( CR = rbeta(trials, success, n_users - success))

ggplot(CR, aes(x=CR)) + 
  geom_histogram(fill="blue",alpha = .3,bins = 30) + geom_vline(xintercept = mean(CR$CR))
 ```
 
  <p align="center">
   <img width="350" height="300" src="https://raw.githubusercontent.com/IanniMuliterno/iannimuliterno.github.io/master/img/hist_cr.png">
 </p>
 
 Podemos também escrever matematicamente a resposta para a nossa pergunta, como: " A probabilidade de que o `p` da versão B seja maior que o `p` da versão A."
 
  <p align="center">
   <img src="https://raw.githubusercontent.com/IanniMuliterno/iannimuliterno.github.io/master/img/question.png">
 </p>

 Essa função de forma fechada ainda não parece tão amigável assim, mas felizmente o R tem funções geradoras, para o nosso caso a `rbeta` vai ser a cavalaria, assim atingimos o mesmo resultado da equação acima através de simulação.
 
 ```r
 trials <- 100000
 prior.alpha <- 6
 prior.beta <- 9
 
 success_a <- 35
 n_users_a <- 50
 #CR = 0.7
 
 success_b <- 35
 n_users_b <- 40
 #CR = 0.875
 
 # 0.875/0.7 = 1.25, podemos confiar neste suposto lift de 25% ?
 
 # Calculando " A probabilidade de que o `p` da versão B seja maior que o `p` da versão A."
 
 a.samples <- rbeta(trials, success_a + prior.alpha, (n_users_a - success_a) + prior.beta)
 b.samples <- rbeta(trials, success_b + prior.alpha, (n_users_b - success_b) + prior.beta)
 
 p.b_superior <- sum(b.samples > a.samples)/trials
 p.b_superior
 # 0.50108
 ```
 
 Observe o exemplo no chunk acima e vamos voltar mais uma vez para o cenário da 'rede de mercado que quer lançar um produto', para entender porque usar a versão bayesiana desse teste é interessante. No contexto de mercado as pessoas estão ansiosas por resultados e a pessoa que está rodando o teste, pode dar uma espiada na base, duas semanas depois do experimento ter começado e ver que na versão A 35 de 50 pessoas recompraram e na versão B 35 de 40 pessoas recompraram e isso pode levar a crer que a versão B foi melhor e que realmente houve esse ganho relativo de 25%. Nos moldes frequentistas existe o risco de que o resultado do teste seja significativo e eles simplesmente acreditem nesse resultado, no molde bayesiano o resultado é que a probabilidade disso ser verdade é de 50.1%. a medida que a evidência for ficando mais forte, ou seja, se essa relação se mantiver a medida que mais pessoas são observadas, esses 50.1% vão aumentar, dando uma noção se realmente podemos confiar na performance que estamos vendo e parar o teste.
 
 ### Entendendo melhor o que pode acontecer na versão frequentista
  
  Suponha que você está tentando descobrir se taxas de conversão são estatisticamente diferentes com um teste frequentista, ou seja, você está olhando para um valor de significância. Porém você está ansioso por resultados e checa o tempo todo como anda seu experimento, a questão é que a significância representa a chance de que aquele resultado relevante que você está vendo aconteceu por acaso e isso é calculado levando em consideração que existe um valor fixo de amostra, se essa regra é violada, as chances de tomar uma decisão errada aumentam. 
  
 ### Passando para o shiny
