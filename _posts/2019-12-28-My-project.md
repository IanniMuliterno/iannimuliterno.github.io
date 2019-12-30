

 Como prometido no post anterior, vou compartilhar aqui minha apresentação para quem não foi ao SatRday e entrar um pouco mais em detalhes que não cabem numa lightning talk. O objetivo do projeto é executar testes A/B bayesiano com shiny, para explicar o teste A/B,  vamos  pensar num contexto de mercado e imaginar uma rede de grande porte que quer lançar um novo produto de marca própria e tem a versão A e B desse produto. Para embasar a decisão final eles querem saber qual produto vai ter melhor aceitação por parte dos clientes, então eles recorrem aos clientes cadastrados no programa de fidelidade para executar um experimento, oferecendo amostras grátis da versão A para metade dessas pessoas e a versão B para a outra.

 
 Um comentário válido, apesar deste ser um exemplo ilustrativo, é sobre o quão conveniente seria num caso como este aplicar o experimento à pessoas cadastradas no programa de fidelidade. Isso porque é de se esperar que essas pessoas façam grande parte das compras, senão todas, nos estabelecimentos da rede, o que significa que os dados serão bem completos e é possível controlar outras variáveis de compra caso seja necessário. Ao mesmo tempo isso não significa que essas pessoas estão mais propensas a comprar algo por ser de marca própria da rede, eles precisam se convencer do custo benefício do produto primeiro. Dito isso, vejamos como se apresentaria a base de compras da rede.
 
 ![base_market](https://raw.githubusercontent.com/IanniMuliterno/iannimuliterno.github.io/master/img/base.png)
 
  Nesta base é possível saber identificar o item comprado, seu valor, unidades compradas e data de compra, além das variáveis identificadores de transação e cliente. Esse tipo de informação rende uma análise exploratória bacana, que vamos ver mais para frente, por hora é necessário preparar os dados para o "formato de entrada" de um teste A/B. Fazemos isso construindo uma variável que responda a seguinte pergunta : " Qual versão do produto mais fez as pessoas voltarem para compra-lo? ". 
  
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
 
 Podemos também escrever matematicamente a resposta para a nossa pergunta, como: " Qual a probabilidade de que o `p` da versão B seja maior que o `p` da versão A? "
