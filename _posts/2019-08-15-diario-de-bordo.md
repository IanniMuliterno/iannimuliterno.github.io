---
layout: post
title: Diário de bordo
image: /img/baby.jpeg
tags: [diário]
---  

## Diapers

 Hello again! Este é o início do meu diário de bordo, onde deixo as recordações de dias de tempestade na vida de um marujo que navega o grande oceano que são os dados. A intenção é lembrar do que fez com que o sol brilhasse no horizonte novamente. 
 
 agosto/2019  
 objetivo : atualizar relatório que analisa consumo de fraldas para pessoas com filhos e consumidores sem filhos (considerados 'gifters') porque provavelmente estão comprando para chá de bebê  
 previsão do tempo : nublado  
 sensação térmica : 2°C

- #### problema 1  
Havia um histograma de idade (relacionado ao gasto com fraldas)  que apresentava grande aumento em pessoas de uam determinada faixa etária, em relação a análise anterior e a proporção da mudança não parecia coerente.
 solução : pegar a base de saída usado no primeiro relatório e extrair as variáveis 'pessoa, ano, classificação (parent/gifter)' para replicar em todas as pessoas do novo relatório no período correspondente. Então o período anterior, que já havia sido analisado, tem seus resultados intocados e a nova classificação lida apenas com o periodo novo. 

- #### problema 2:  
ao verificar mais de perto os comportamentos de consumo, vimos que algumas pessoas tinham informações diferentes para um mesmo período no arquivo que iria para o Dashboard, mesmo que as bases sejam idênticas neste mesmo período.  
 solução: usar mesmas técnicas de tratamento de outliers aplicada no tratamento da base usada no primeiro relatório. 
 solução parte 2: uma parte do script novo utilizava inner join quando deveria utilizar left join, então o filtro que remove da base um padrão que vamos chamar de "tipo_indesejado", interfere na presença de algumas linhas (parte do codigo abaixo, com correção).


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
all <- merge(all,intervalo, all.x = T, by = c("pessoa","ano"))
```


- #### problema 3:  
divergências em alguns gráficos  
Solução: descobrimos um filtro não documentado:A não documentação deste filtro me custou alguns minutos explorando a base para descobrir que existia limite inferior e superior na variável em questão. Eis um relato da importância de documentar as coisas.


#### Dicas para o outros tripulantes ao lidar com um projeto que segue a mesma idéia (de atualizar um relatório).

1 - Se sua atualização precisa mostrar parte da informação antiga para refrescar a memória do cliente, importe tanto quanto for possível do relatório anterior, você não quer mudar o passado, só atualizar o relatório.  

2 - Se essa atualização envolve uma classificação, treine um modelo com a classificação antiga no conjunto de treino e as mesmas métricas para classificar as entradas novas (acrescente alguma métrica se quiser e achar viável).  

3 - pense bem se inner_join é a opção certa e se decidir que sim confira se não sumiu nada que deveria continuar existindo.

4 - Documente tudo o que você fizer, mesmo que você tenha memória fotográfica outras pessoas podem precisar entender o que foi feito.

5 - Leve as dicas 3 e 4 para sua vida.  


