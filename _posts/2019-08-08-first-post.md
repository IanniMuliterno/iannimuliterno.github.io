---



layout: post



title: Preditores, dropar ou n�o dropar? Transformar ou n�o transformar? (Primeiro post!)



image: /img/hello_world.jpeg



tags: [compara��o,modelagem,m�xima_verossimilhan�a]



---







Hello my friends, stay awhile and listen!







 Primeiro post \o/. Esse � um tema no qual coincidentemente esbarrei enquanto tamb�m esbarrava no belo e amig�vel ['beautiful-jekyll'](https://deanattali.com/beautiful-jekyll/). Combina��o de coincid�ncias que me fez (*finalmente*) come�ar a escrever.  







 Vamos a quest�o, recentemente me uni a um grupo de Devs e nossa primeira brincadeira oficial est� sendo trabalhar com uma base de dados de pre�os de casas que consiste em algumas centenas de linhas e 80 vari�veis. Depois de desbravar um pouco a base,observamos multicolinearidade (vulgo : *alguns preditores s�o muito semelhantes entre si* ) ent�o surgiram as ideias de como lidar. Criar uma outra vari�vel que contenha informa��o de ambas e dropar as originais depois? Dropar uma e deixar outra? Se sim, qual dropar? Estamos viajando e na verdade n�o precisa mexer em nada disso? Como saber o que � melhor?   







 Existem v�rias formas de decidir e a primeira que me veio a mente foi o teste da raz�o de verossimilhan�a, mas como se trata de um post sobre **como** tomar essa decis�o, vamos colocar mais algumas formas de como chegar a ela. Sempre de forma mais intuitiva, simples e direta que minha habilidade de comunica��o permitir. *Antes de seguir, vale ressaltar que existe uma discuss�o sobre 'multicolinearidade � motivo suficiente para se livrar de uma vari�vel?', como tudo na Estat�stica e suas vers�es gourmet a resposta � depende, ent�o sempre que houver um tema importante, mesmo que eu n�o avise, provavelmente tem uma discuss�o intermin�vel nos bastidores. Para prosseguir sem consci�ncia pesada, vamos convenientemente supor que 'sim, multicolinearidade � motivo suficiente para se livrar de uma vari�vel'*.  







 Saber qual a melhor vers�o do modelo, de acordo com as altera��es sugeridas acima, � uma �tima forma de decidir qual a melhor altera��o a se fazer. Ent�o eis aqui algumas m�tricas para este fim:







 * **AIC**    



   AIC = 2k - 2ln(![L_hat](https://raw.githubusercontent.com/IanniMuliterno/iannimuliterno.github.io/master/img/L_hat.png))   



  O crit�rio de informa��o de Akaike � uma m�trica de qualidade de modelo. 'k' � o n�mero de vari�veis no modelo e '![L_hat](https://raw.githubusercontent.com/IanniMuliterno/iannimuliterno.github.io/master/img/L_hat.png)' � a m�xima verossimilhan�a do modelo (que � de forma resumida, um valor que representa o qu�o bem o meu modelo explica os dados, vamos falar disso em outro post). O que acontece com essa m�trica quando dropamos vari�veis � que diminuimos o valor de 'k por�m tamb�m diminu�mos '![L_hat](https://raw.githubusercontent.com/IanniMuliterno/iannimuliterno.github.io/master/img/L_hat.png)' porque certamente nosso modelo vai ficar um pouco pior em explicar os dados. Observe que pela f�rmula do AIC, o menor valor poss�vel significa o melhor modelo poss�vel, ent�o se voc� dropa uma vari�vel e de repente seu AIC que era -1000 passou para -800, � sinal de que voc� n�o deveria estar dropando tal vari�vel. Por outro lado se o AIC ficar ainda menor ou se a diferen�a � �nfima (de -1000 para -990 por exemplo), � sinal de que o drop da vari�vel � aceit�vel.  







 * **BIC**  



   BIC = ln(n)k - 2ln(![L_hat](https://raw.githubusercontent.com/IanniMuliterno/iannimuliterno.github.io/master/img/L_hat.png))  



   O crit�rio de informa��o Bayesiano � bastante parecido com o AIC, e � avaliado da mesma forma, quanto menor melhor. A diferen�a na f�rmula � que o '2' � substituido por ln(n) onde n � o numero de observa��es, ou seja, o BIC � melhor em penalizar a complexidade do modelo. Em geral a an�lise do AIC concorda com a do BIC, quando elas discordam o comum � que o AIC esteja indicando que vers�es mais complexas s�o melhores. No fim das contas, se voc� decidir que um deles � interessante para seu caso, o s�bio � usar os dois. Se esses carinhas mais simples chamaram sua aten��o, aqui est� um breve post abordando [AIC vs BIC](https://www.methodology.psu.edu/resources/AIC-vs-BIC/), onde inclusive eles fazem a ousada afirma��o de que sua maior refer�ncia deve ser o AIC se um falso negativo � um problema mais grave para voc�, e BIC caso contr�rio. Fiquem a vontade para investigar e decidir se concordam com isso.  







 * **Cross-Validation**  



 Cross-validation � um m�todo que costuma ser aplicado em observa��es (ou inst�ncias) e consiste em dividir o banco em partes iguais, de modo que as vari�veis em cada 'fatia' do banco estejam distribu�das da mesma maneira. Ent�o separa uma dessas fatias, treina o modelo usando todas as outras observa��es e ent�o testa usando a parte que voc� separou, repetindo o processo at� que se tenha usado todas as fatias do banco como teste, deste modo � poss�vel prever o erro do modelo de forma mais consistente. (*existem varia��es do cross-validation, a que foi descrita aqui se chama 'leave-one-out'*).



 Usando este racioc�nio, � poss�vel fazer o *leave-one-out* com as vari�veis, comparando a taxa de erro dos modelos com este mesma taxa calculada utilizando o banco de treino completo. Isso � bem parecido com o que � preciso fazer quando queremos usar o AIC ou BIC como m�tricas, por�m o Cross-validation � mais indicado para uma 'busca as cegas'.







 * **Likelihood Ratio Test**  



 LR = -2\[ln(![L_hat](https://raw.githubusercontent.com/IanniMuliterno/iannimuliterno.github.io/master/img/L_hat.png)0)- ln(![L_hat](https://raw.githubusercontent.com/IanniMuliterno/iannimuliterno.github.io/master/img/L_hat.png))\]  



 O teste da raz�o da verossimilhan�a (*LR*), como o nome sugere, compara os valores da m�xima verossimilhan�a dos modelos. Para entender melhor o que est� acontecendo vamos admitir que ![L_hat](https://raw.githubusercontent.com/IanniMuliterno/iannimuliterno.github.io/master/img/L_hat.png)0 � a m�xima verossimilhan�a calculada com o modelo completo e ![L_hat](https://raw.githubusercontent.com/IanniMuliterno/iannimuliterno.github.io/master/img/L_hat.png) � a mesma m�trica calculada depois que o modelo sofre a altera��o que se quer testar. Multiplicar essa diferen�a por -2 permite que a m�trica seja associada a uma distribui��o qui-quadrado, logo para chegar a uma conclus�o sobre os modelos, LR � comparado a um certo valor. Este tal valor � retirado da distribui��o qui-quadrado e compara-lo com LR equivale a executar um teste de hip�tese, onde a hip�tese 0 � que n�o h� evid�ncias de que os modelos tem performances diferentes.  











 Sim, explicar � fundo o LR � um pouco tricky, precisar�amos ainda chafurdar um pouco em teste de hip�tese, em entender o b�sico da distribui��o qui-quadrado e saber o que acontece no c�lculo da m�xima verossimilhan�a. Felizmente, mesmo que o LR n�o seja o mais amig�vel dos m�todos, gra�as a softwares como o R n�o � necess�rio implementar o teste, apenas compreender o que acontece por tr�s dele. Se mesmo assim agora voc� quer saber *o que diabos acontece no c�lculo da m�xima verossimilhan�a*, ou voc� n�o se conforma porque eu n�o falei mais sobre o valor com o qual LR � comparado no fim das contas, voc� pode [clicar aqui](https://en.wikipedia.org/wiki/Likelihood-ratio_test) e [aqui](http://www.portalaction.com.br/confiabilidade/421-metodo-de-maxima-verossimilhanca).







 A presto!

