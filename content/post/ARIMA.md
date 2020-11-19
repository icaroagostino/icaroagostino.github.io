---
author: Ícaro
cover: img/ARIMA.jpg
date: "2020-03-23"
description: 'Aplicações em R para previsão de séries temporais utilizando modelagem ARIMA a partir do pacote forecast.'
title: Prevendo o futuro com modelos ARIMA
---
[Cover Photo by Lum3n.com from Pexels](https://www.pexels.com/photo/black-click-pen-on-white-paper-167682/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)

# **Previsão com modelos ARIMA**

Existem muitos conceitos relacionados a previsão, cientificamente falando uma das formas mais bem fundamentadas é a previsão baseada em séries temporais, em que você observa uma variável ao longo do tempo, em intervalos constantes, a partir disso consegue extrair padrões e extrapolar esses padrões para o futuro.

Nesse post vamos demonstrar de forma simplificada como ajustar um modelo de previsão utilizando a família de modelos **Autorregressivos Integrados de Médias Móveis (ARIMA)** em `R` para para previsão de séries temporais utilizando o pacote [**forecast**](https://pkg.robjhyndman.com/forecast/).

> `R` é uma linguagem de programação open source. Todos os recursos, bibliotecas, dados e implementações são gratuitos e desenvolvidos pela comunidade.
>
> Se você tem interesse de aprender a instalar o R e os passos básicos para iniciar nesse mundo sugiro o tutorial básico desenvolvido pelo pessoal do [curso-r](https://www.curso-r.com/): http://material.curso-r.com/instalacao/

*Vamos começar...*

![](https://media3.giphy.com/media/VMmRM3EjhjBII/giphy-downsized.gif#center)

## Carregando bibliotecas e dados

As bibliotecas [**forecast**](https://cran.r-project.org/web/packages/forecast/) e [**ggplot2**](https://cran.r-project.org/web/packages/ggplot2/) são necessárias, utilize os comandos `install.packages('nome da biblioteca')` e `library(nome da biblioteca)` para instalar e carregar as bibliotecas.

Para este exemplo vamos importar um banco direto da internet que está hospedado [aqui](https://github.com/icaroagostino/ARIMA/blob/master/dados/MA.txt), são dados mensais do saldo de emprego do estado do Maranhão.

```visual-basic
library(forecast) #Modelos de previsão (Hyndman and Khandakar, 2008)
library(ggplot2) #Elegant Graphics (Wickham, 2009)
dados <- read.table("https://raw.githubusercontent.com/icaroagostino/ARIMA/master/dados/MA.txt", header=T)
attach(dados) #tranformando em objeto
MA <- ts(MA, start = 2007, frequency = 12) #tranformando em Séries Temporal
```
## Visualização

Para uma visualização bastante completa de diversas características dos dados vamos usar a função `ggtsdisplay`, essa função vai plotar a série de dados e autocorrelação (Auto Correlation Function - ACF) e autocorrelação parcial (Partial AutoCorrelation Function - PACF), ou seja, o quanto a variável está correlacionada com ela mesma em instantes passados do tempo.

```visual-basic
ggtsdisplay(MA, main="Saldo de emprego - MA")
```
![](graff.png#center)

A série possui características de sazonalidade aditiva com tendência moderada negativa, além disso a análise ACF permite evidenciar a presença de autocorrelação temporal entre as observações.

## Ajuste do modelo

Para a estimação dos parâmetros e ajuste do modelo será utilizado a função `auto.arima()`, que utiliza o algorítimo desenvolvido e publicado por Hyndman e Khandakar (2008) que combina a aplicação de testes de raízes unitárias, minimização do AIC e MLE utilizando o procedimento stepwise.

A função pertence a biblioteca [**forecast**](https://pkg.robjhyndman.com/forecast/) que atualmente é mais completa para previsão de séries temporais com aplicações em R. O modelo ajustado terá a notação usual `ARIMA(p,d,q)(P,D,Q)[m]` sendo que `p` representa o número de parâmetros autorregressivos, `d` a ordem de integração, `q` o número de parâmetros de médias móveis.

Caso a série possua sazonalidade, além dos parâmetros já citados ainda terão `P` para o número de parâmetros autorregressivos sazonais, `D` a ordem de integração sazonal, `Q` o número de parâmetros de médias móveis sazonais, e `[m]` a ordem sazonal.

```visual-basic
ARIMA_fit <- auto.arima(MA)
ARIMA_fit #sai o modelo ajustado
```

```visual-basic
## Series: MA 
## ARIMA(1,0,1)(2,1,0)[12] with drift 
##
## Coefficients:
##          ar1      ma1     sar1     sar2     drift
##       0.7076  -0.3409  -0.7236  -0.4268  -24.3796
## s.e.  0.1315   0.1734   0.0875   0.0862   13.9307
##
## sigma^2 estimated as 2708743:  log likelihood=-1025.46
## AIC=2062.91   AICc=2063.68   BIC=2079.43
```

O modelo ajustado automaticamente considerou 1 parâmetro autorregressivo, 1 parâmetro de médias móveis, 2 parâmetros sazonais autorregressivos e 1 diferença sazonal de ordem 12 (meses).

Tais parâmetros podem e devem ser alterados a fim de buscar um melhor ajuste do modelo a partir do comando `Arima(x, order = c(0, 0, 0), seasonal = c(0, 0, 0))`, informe os parâmetros a serem estimados nos argumentos `order` e `seasonal`, lembrando que é possível estimar diversos modelos concorrentes e decidir o melhor modelo pelos critérios de AIC e BIC.

## Verificação dos resíduos

A próxima etapa é verificar a qualidade do ajuste do modelo, para isso vamos analisar os resíduos do modelos, olhando para os erros gerados no processo de estimação, felizmente o pacote **forecast** fornece funções para esse fim.

```visual-basic
checkresiduals(ARIMA_fit)
```

![](res.png#center)

Os resíduos gerados pelo modelo apresentaram características de ruído branco. Como podemos ver a sequência de erros oscila em torno de zero (gráfico superior). Olhando para o gráfico ACF, a maior parte dos valores estão dentro do intervalo de confiança (gráfico inferior esquerdo), demonstrando que a informação correlação temporal foi "capturada" pelo modelo. Além disso, a distribuição dos erros segue uma distribuição aproximadamente normal (gráfico inferior direito).

Todas essas características são importantes para serem analisadas no processo de estimação para garantir a qualidade da previsão que faremos na próxima etapa.

## Previsão

Finalmente vamos prever o futuro, para isso vamos usar a função `forecast` e na sequência plotar o gráfico de previsão com a função `autoplot`. O **horizonte** será de 12 meses.

```visual-basic
forecast(ARIMA_fit, h = 12) %>% autoplot
```
![](prev.png#center)

Como podemos ver, a previsão parece seguir o comportamento esperado para a série, sendo capaz de capturar o comportamento de tendência e sazonalidade dos dados.

## Chegamos ao fim

Nesse tutorial vimos de forma simplificada a utilização de modelos ARIMA para previsão de séries temporais, o exemplo demonstrado acima é totalmente reproduzível, basta instalar as bibliotecas necessárias e copiar os códigos acima para o seu Rstudio.

Se você está interessado em outros modelos de previsão, nesse link você vai encontrar um tutorial para utilização de [Redes Neurais Artificais](https://icaroagostino.github.io/post/ann/).

Se usar algum dos pacotes aqui demonstrados não esqueça de citar a publicação:

```visual-basic

  citation("forecast")
  
  ##
  ## To cite the forecast package in
  ## publications, please use:
  ##   
  ## To cite the forecast package in publications, please use:
  ##
  ## Hyndman R, Athanasopoulos G, Bergmeir C, Caceres G, Chhay L,
  ## O'Hara-Wild M, Petropoulos F, Razbash S, Wang E, Yasmeen F
  ## (2019). _forecast: Forecasting functions for time series and
  ## linear models_. R package version 8.9, <URL:
  ## http://pkg.robjhyndman.com/forecast>.
  ## 
  ## Hyndman RJ, Khandakar Y (2008). “Automatic time series
  ## forecasting: the forecast package for R.” _Journal of
  ## Statistical Software_, *26*(3), 1-22. <URL:
  ## http://www.jstatsoft.org/article/view/v027i03>.
  ##
  
  citatio("ggplot2")
  
  ##
  ## To cite ggplot2 in publications, please use:
  ## 
  ## H. Wickham. ggplot2: Elegant Graphics for Data Analysis.
  ## Springer-Verlag New York, 2016.
  ##
  
```

## Obs.:

Os dados utilizados nesse exemplo são públicos, para mais detalhes baixe o script [ARIMA.R](https://github.com/icaroagostino/ARIMA/blob/master/ARIMA.R) que está [neste](https://github.com/icaroagostino/ARIMA/) repositório.

contato: icaroagostino@gmail.com

---

Até o próximo post, bons estudos em `R` :D

![](https://media.giphy.com/media/VbnUQpnihPSIgIXuZv/giphy.gif#center)