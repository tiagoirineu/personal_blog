---
title: Diversification
author: Tiago Irineu
date: '2022-05-14'
slug: Markowitiz
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2022-05-14T23:45:13-03:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---
## Diversificação


### Introdução

Abaixo, traduzo para o R um script em Python compartilhado pelo professor Daniel Bergmann, no curso Fundamentos de métodos quantitativos aplicados a Risco, parte da especialização em Gestão de Risco pela Saint Paul e B3.

Você pode acessar o código original [aqui](https://colab.research.google.com/drive/1IyTmX8v9SWo6QvsgEv6YmhUhpVeR6L_A#scrollTo=dKORsyf9JAvs).

É realizado um exercício de diversificação de carteira.

* Iremos acessar as informações reais de um conjunto de ações listada na bolsa e calcular o log-retorno diário. 

* Simular 10.000 carteiras diferentes e no fim representar graficamente qual seria a carteira com maior sharpe, ou seja, com a melhor razão risco/retorno.

### Diversification

Primeiro, precisamos carregar os pacote que iremos utilizar durante a análise. [Tidyverse](https://r4ds.had.co.nz/) para trabalhar com os dados e [tidyquant](https://business-science.github.io/tidyquant/) para baixar os dados financeiros e acessar algumas funções específicas para este tipo de dados.


```r
library(tidyverse)
library(tidyquant)
```

Para baixar os dados utilizamos a função *tq_get*. Identificamos as empresas para as quais queremos baixar os dados a partir dos tickers, os códigos pelos quais elas são representadas na bolsa.


```r
tickers <- c("ITUB3.SA", 'BBAS3.SA', "ABEV3.SA", "MGLU3.SA", "VIIA3.SA", "VALE3.SA", "PETR4.SA", "ABEV3.SA", "BBDC4.SA", "GGBR4.SA", "CSNA3.SA", "COGN3.SA")

data <-  tq_get( tickers, get = "stock.prices", from = "2019-04-29")  #Downloading the data from the above company

head(data)
```



```
## # A tibble: 6 x 8
##   symbol   date        open  high   low close volume adjusted
##   <chr>    <date>     <dbl> <dbl> <dbl> <dbl>  <dbl>    <dbl>
## 1 ITUB3.SA 2019-04-29  29.5  29.5  29.1  29.4 108200     26.7
## 2 ITUB3.SA 2019-04-30  29.7  29.7  29.1  29.4 228400     26.7
## 3 ITUB3.SA 2019-05-02  29.4  29.6  29.2  29.5 309200     26.8
## 4 ITUB3.SA 2019-05-03  29.7  29.8  29.2  29.3 200500     26.6
## 5 ITUB3.SA 2019-05-06  29.1  29.1  28.7  28.7 236900     26.1
## 6 ITUB3.SA 2019-05-07  28.7  28.8  28.1  28.5 417500     25.9
```


Abaixo, visualizamos os dados.

Normalizamos os preçoos de todas as ações, de tal forma,que visualizamos a performance dos papéis a partir da data inicial.Isto permite ter uma visão da performance relativa de cada ação.


```r
data %>% 
    group_by(symbol) %>% 
    mutate(norm_price = close/close[1]*100) %>% 
    ggplot(aes(date, norm_price)) +
    geom_line(aes(color = symbol), size = 0.8) +
    labs(y = "Preços normalizados", x = NULL) +

# Agora, irei aplicar algumas modificações estéticias. Apenas por gosto pessoal.
    theme_bw( base_family = "Roboto Condensed") +
 #   guides(color = FALSE) +
  #  geom_text(data = filter(data, date == last(date)),aes(label = symbol)) +
    theme(panel.grid =  element_blank(),
          axis.title.y = element_text(face = "bold")
    )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

### Simulação de pesos para carteira de ativos

Nosso objetivo é simular diferentes pesos e chegar a uma carteira hipotética que maximiza o índice Sharpe. 

Para isto, iremos simular 10.000 carteiras diferentes, calcular retorno, volatilidade, e o indíce Sharpe para cada uma delas e por fim fazer um gráfico de pizza apresentando a carteira com o maior índice de Sharpe.


#### Calculando o log returns das ações


```r
# We create a grouped dataset, so every function is applied over each ticker (company)

log_returns <- data %>% 
    group_by(symbol) %>% 
    #Daily log returns for the close price
    tq_transmute(select = close, mutate_fun = dailyReturn, type = "log") %>% 
    ungroup()

str(log_returns)
```

```
## tibble [8,305 x 3] (S3: tbl_df/tbl/data.frame)
##  $ symbol       : chr [1:8305] "ITUB3.SA" "ITUB3.SA" "ITUB3.SA" "ITUB3.SA" ...
##  $ date         : Date[1:8305], format: "2019-04-29" "2019-04-30" ...
##  $ daily.returns: num [1:8305] 0 0 0.00374 -0.00579 -0.02105 ...
```

Abaixo, calculamos os retornos médios das ações.


```r
mean_returns <- log_returns %>% 
    group_by(symbol) %>% 
    summarise(mean_return = mean(daily.returns))
# Abaixo, vamos checar os valores médios
mean_returns
```

```
## # A tibble: 11 x 2
##    symbol   mean_return
##    <chr>          <dbl>
##  1 ABEV3.SA   -0.000308
##  2 BBAS3.SA   -0.000451
##  3 BBDC4.SA   -0.000434
##  4 COGN3.SA   -0.00178 
##  5 CSNA3.SA    0.000285
##  6 GGBR4.SA    0.000872
##  7 ITUB3.SA   -0.000435
##  8 MGLU3.SA   -0.000320
##  9 PETR4.SA    0.000298
## 10 VALE3.SA    0.000579
## 11 VIIA3.SA   -0.000521
```


#### Simulação de 10.000 carteiras 

Iremos simular 10.000 combinações diferentes dos 11 ativos para os quais baixamos as informações.

Abaixo, iniciamos alguns vetores e matrizes que iremos utilizar para guardar as informações das simulações que iremos realizar. 



```r
set.seed(1) #define o parâmetro para a simulação. Importante para garantir reprodutibilidade da análise.


nticks = length(tickers)
nsample = 10000
port_returns = c()
port_vol = c()
all_weights <- matrix(nrow=nsample, ncol = nticks)
exp_return <- rep(0, nsample)
exp_var <- rep(0, nsample)
exp_std <- rep(0, nsample)
sharpe <- matrix(0, nrow=10000, ncol=1)
```
 

```r
cov_returns <- log_returns %>% 
    pivot_wider(names_from = symbol, values_from = daily.returns) %>%
    select(-date) %>% 
    cov()
```

No código abaixo, realizamos de fato a simulação das carteiras e guardamos os resultados calculados para o indíce de sharpe, retornos, e volatilidade.


```r
for (i in 1:nsample) {
    
    # Create, aleatory weights and normalize then to sum to 1
    weights <- runif(nticks, min= 0,1)
    weights <- weights/sum(weights)
    
    all_weights[i,] <- weights
    
    # Multiply by 250 so the returns and volatility are annualized
    exp_return[i] <- sum(weights*mean_returns$mean_return)*250
    
    exp_var[i] = weights%*%((cov_returns*250)%*%weights)
    
    exp_std[i] = sqrt(exp_var[1]) #Standard deviation as the risk
    sharpe[i,] <- exp_return[i]/exp_std[i] #Return/risk
    
    # Return and volatility vectors
    
    port_returns <- append(port_returns, exp_return[i])
    port_vol <-  append(port_vol, exp_std[i])
    
}
```

### Encontrando o portfolio que maximiza o índice de Sharpe

Abaixo, combinamos os pesos com os seus sharpes equivalentes. A partir disto, vamos criar um ordered arranjado. 


```r
all_weights <- as_tibble(all_weights) 
names(all_weights) <- tickers

Sharpe <- as_tibble(sharpe)
names(Sharpe) <- "sharpe"

df <- cbind(all_weights,Sharpe)

#Ordena o dataset de acordo com o índice de sharpe. Colocando o portfolio com o maior índice como o primeio
ordered_df <- df %>% 
    arrange(desc(sharpe))
```


Abaixo, selecionamos o portfolio com o maior indíce de Sharpe e o visualizamos com um gráfico de pizza.


```r
tibble(ordered_df[1,1: 11]*100) %>% 
    pivot_longer(cols = tickers, names_to = "empresa", values_to = "percentual") %>% 
    ggplot(aes(x="", y = percentual, fill = empresa)) + 
    geom_bar(stat= "identity", color = "white") +
    coord_polar("y", start = 0) +
    theme_void() +
    labs(title = "Portfolio com o maior índice Sharpe")
```

```
## Note: Using an external vector in selections is ambiguous.
## i Use `all_of(tickers)` instead of `tickers` to silence this message.
## i See <https://tidyselect.r-lib.org/reference/faq-external-vector.html>.
## This message is displayed once per session.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />

























