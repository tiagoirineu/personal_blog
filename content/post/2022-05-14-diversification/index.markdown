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
## Diversification

Translating a Python Colab script, shared by profesor Daniel, to R.


```r
library(tidyverse)
library(tidyquant)
library(tsibble)
```


```r
tickers <- c("ITUB3.SA", 'BBAS3.SA', "ABEV3.SA", "MGLU3.SA", "VIIA3.SA", "VALE3.SA", "PETR4.SA", "ABEV3.SA", "BBDC4.SA", "GGBR4.SA", "CSNA3.SA", "COGN3.SA")

data <-  tq_get( tickers, get = "stock.prices", from = "2019-04-29")  #Downloading the data from the above company

head(data)
```



```
## Rows: 9060 Columns: 8
```

```
## -- Column specification --------------------------------------------------------
## Delimiter: ","
## chr  (1): symbol
## dbl  (6): open, high, low, close, volume, adjusted
## date (1): date
```

```
## 
## i Use `spec()` to retrieve the full column specification for this data.
## i Specify the column types or set `show_col_types = FALSE` to quiet this message.
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


Building a plot showing the normalized prices.



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

```
## Warning in grid.Call(C_stringMetric, as.graphicsAnnot(x$label)): font family not
## found in Windows font database

## Warning in grid.Call(C_stringMetric, as.graphicsAnnot(x$label)): font family not
## found in Windows font database
```

```
## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database
```

```
## Warning in grid.Call.graphics(C_text, as.graphicsAnnot(x$label), x$x, x$y, :
## font family not found in Windows font database
```

```
## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

### Simulação de pesos para carteira de ativos





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

```r
mean_returns <- log_returns %>% 
    group_by(symbol) %>% 
    summarise(mean_return = mean(daily.returns))

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







```r
set.seed(1)
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
weights <- runif(nticks, min= 0, max = 1) #Selects 10 floats from a uniform distribution between 0 and 1

weights <- weights/sum(weights) # Normalizes the weights, so the elements sum up to 1
m1 <- matrix(weights, ncol = nticks)
```

```r
head(log_returns)
```

```
## # A tibble: 6 x 3
##   symbol   date       daily.returns
##   <chr>    <date>             <dbl>
## 1 ITUB3.SA 2019-04-29       0      
## 2 ITUB3.SA 2019-04-30       0      
## 3 ITUB3.SA 2019-05-02       0.00374
## 4 ITUB3.SA 2019-05-03      -0.00579
## 5 ITUB3.SA 2019-05-06      -0.0210 
## 6 ITUB3.SA 2019-05-07      -0.00595
```




```r
cov_returns <- log_returns %>% 
    pivot_wider(names_from = symbol, values_from = daily.returns) %>%
    select(-date) %>% 
    cov()
```



```r
cov_returns
```

```
##              ITUB3.SA     BBAS3.SA     ABEV3.SA     MGLU3.SA     VIIA3.SA
## ITUB3.SA 0.0004697341 0.0004419312 0.0002545130 0.0002950754 0.0004238628
## BBAS3.SA 0.0004419312 0.0007453423 0.0003161717 0.0004675760 0.0006801657
## ABEV3.SA 0.0002545130 0.0003161717 0.0004920664 0.0003113230 0.0003817922
## MGLU3.SA 0.0002950754 0.0004675760 0.0003113230 0.0014975509 0.0011473333
## VIIA3.SA 0.0004238628 0.0006801657 0.0003817922 0.0011473333 0.0019573467
## VALE3.SA 0.0002130779 0.0003378932 0.0001504379 0.0003606447 0.0003859131
## PETR4.SA 0.0003740808 0.0005946207 0.0003215342 0.0004999780 0.0006931320
## BBDC4.SA 0.0004572923 0.0006005140 0.0003071596 0.0003779936 0.0005310895
## GGBR4.SA 0.0003210784 0.0004586532 0.0002597558 0.0004519814 0.0005859306
## CSNA3.SA 0.0003490224 0.0005029187 0.0002789481 0.0005664861 0.0006572778
## COGN3.SA 0.0004400244 0.0006151901 0.0004326268 0.0007375932 0.0009787276
##              VALE3.SA     PETR4.SA     BBDC4.SA     GGBR4.SA     CSNA3.SA
## ITUB3.SA 0.0002130779 0.0003740808 0.0004572923 0.0003210784 0.0003490224
## BBAS3.SA 0.0003378932 0.0005946207 0.0006005140 0.0004586532 0.0005029187
## ABEV3.SA 0.0001504379 0.0003215342 0.0003071596 0.0002597558 0.0002789481
## MGLU3.SA 0.0003606447 0.0004999780 0.0003779936 0.0004519814 0.0005664861
## VIIA3.SA 0.0003859131 0.0006931320 0.0005310895 0.0005859306 0.0006572778
## VALE3.SA 0.0007103067 0.0004673612 0.0003035960 0.0005574439 0.0007103582
## PETR4.SA 0.0004673612 0.0010584294 0.0005211319 0.0005995512 0.0006403803
## BBDC4.SA 0.0003035960 0.0005211319 0.0006583671 0.0004171907 0.0004606396
## GGBR4.SA 0.0005574439 0.0005995512 0.0004171907 0.0009241007 0.0008675454
## CSNA3.SA 0.0007103582 0.0006403803 0.0004606396 0.0008675454 0.0014338514
## COGN3.SA 0.0003250605 0.0006128369 0.0005289011 0.0005014293 0.0006229606
##              COGN3.SA
## ITUB3.SA 0.0004400244
## BBAS3.SA 0.0006151901
## ABEV3.SA 0.0004326268
## MGLU3.SA 0.0007375932
## VIIA3.SA 0.0009787276
## VALE3.SA 0.0003250605
## PETR4.SA 0.0006128369
## BBDC4.SA 0.0005289011
## GGBR4.SA 0.0005014293
## CSNA3.SA 0.0006229606
## COGN3.SA 0.0015842363
```




```r
# Neste código, é calculado o índice de Sharpe, o retorno e o risco


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

### Defining the optimum portfolio as measured by the sharpe index




```r
all_weights <- as_tibble(all_weights) 
```

```
## Warning: The `x` argument of `as_tibble.matrix()` must have unique column names if `.name_repair` is omitted as of tibble 2.0.0.
## Using compatibility `.name_repair`.
## This warning is displayed once every 8 hours.
## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was generated.
```

```r
names(all_weights) <- tickers
```

```r
Sharpe <- as_tibble(sharpe)
names(Sharpe) <- "sharpe"
Sharpe
```

```
## # A tibble: 10,000 x 1
##      sharpe
##       <dbl>
##  1 -0.103  
##  2 -0.139  
##  3 -0.156  
##  4 -0.100  
##  5 -0.0800 
##  6 -0.136  
##  7 -0.123  
##  8 -0.00552
##  9 -0.00569
## 10 -0.219  
## # ... with 9,990 more rows
```

```r
df <- cbind(all_weights,Sharpe)

ordered_df <- df %>% 
    arrange(desc(sharpe))
```



```r
head(ordered_df)
```

```
##     ITUB3.SA   BBAS3.SA    ABEV3.SA    MGLU3.SA   VIIA3.SA   VALE3.SA
## 1 0.13252856 0.01230708 0.056065353 0.034909682 0.24259336 0.22452407
## 2 0.12737715 0.06776101 0.093144720 0.003120477 0.09373137 0.18683119
## 3 0.20247721 0.01540019 0.052897277 0.010888195 0.15953751 0.17107368
## 4 0.03379910 0.12159538 0.008037548 0.010781633 0.19279428 0.20974137
## 5 0.16256247 0.03173938 0.025036994 0.021391028 0.06983878 0.19735648
## 6 0.04063537 0.05727850 0.095400921 0.032135026 0.21627621 0.02781771
##     PETR4.SA   BBDC4.SA   GGBR4.SA  CSNA3.SA   COGN3.SA    sharpe
## 1 0.02415015 0.03144297 0.05008969 0.1778865 0.01350258 0.1452811
## 2 0.02407536 0.02168471 0.20258521 0.1739842 0.00570458 0.1402076
## 3 0.01223440 0.02083686 0.12228837 0.1929037 0.03946264 0.1297514
## 4 0.01289657 0.06127633 0.15427796 0.1086115 0.08618831 0.1232454
## 5 0.03595600 0.05193993 0.15358300 0.2025529 0.04804310 0.1204598
## 6 0.01927328 0.01339078 0.09585353 0.3846369 0.01730179 0.1165953
```


```r
ordered_df[1,1: 11]*100
```

```
##   ITUB3.SA BBAS3.SA ABEV3.SA MGLU3.SA VIIA3.SA VALE3.SA PETR4.SA BBDC4.SA
## 1 13.25286 1.230708 5.606535 3.490968 24.25934 22.45241 2.415015 3.144297
##   GGBR4.SA CSNA3.SA COGN3.SA
## 1 5.008969 17.78865 1.350258
```




























