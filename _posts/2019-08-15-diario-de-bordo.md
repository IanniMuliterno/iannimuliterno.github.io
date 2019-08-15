---
layout: post
title: Diário de bordo
image: /img/baby.jpeg
tags: [diário]
---  

## Diapers

 agosto/2019  
 objetivo : atualizar relatório que analisa consumo de fraldas para pessoas com filhos e consumidores sem filhos (considerados 'gifters') porque provavelmente estão comprando para chá de bebê  
 previsão do tempo : nublado  
 sensação térmica : 2°C

- #### problema 1  
histograma de idade (apenas o de gasto)  com grande aumento em pessoas com mais de 55 anos (mesmo depois de lidar com todos os outros problemas essa diferença permanece mas não desaparece quando se usa a classificação nova de parents)  
 solução : pegar a base de saída usado no primeiro relatorio e extrair 'id_cpf, ano, cluster' para replicar em todos os cpfs do novo relatorio nos anos correspondentes, sendo que a nova classificação lida apenas com o periodo novo 

- #### problema 2:  
ao verificar mais de perto comportamentos por cpfs vimos que algumas pessoas tinham informações diferentes para um mesmo periodo no arquivo que iria para o BI, mesmo que as extrações sejam identicas neste mesmo periodo.  
 solução: usar mesmo limite maximo de quantidade das fraldas na limpeza de variaveis  
 solução parte 2: uma parte do script novo utilizava inner join quando deveria utilizar left join, então o filtro de absorvente, quando se carrega o banco 'suporte' no script "~/melhorar_definicao_parents_baseada_antiga.R", interfere na presença de algumas linhas (parte do codigo abaixo)


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
all <- merge(all,intervalo, all.x = T, by = c("id_cpf","ano"))
```


- #### problema 3:  
divergências em alguns gráficos  
Solução: descobrimos um filtro não registrado, valor_unitario/unidade deveria ser maior que 0.2 e menor que 5


Dicas para o outros tripulantes.

1 - para cpfs iguais e anos iguais copie a classificação antiga, você não quer mudar o passado, só atualizar o relatório.  

2 - treine um modelo com a classificação antiga no conjunto de treino e as mesmas métricas para classificar as pessoas que ficaram de fora no passo anterior (acrescente alguma métrica se quiser e achar viável).  

3 - pense bem se inner_join é a opção certa e se decidir que sim confira se não sumiu nada que deveria continuar existindo.  

4 - Leve a dica anterior para sua vida.  


