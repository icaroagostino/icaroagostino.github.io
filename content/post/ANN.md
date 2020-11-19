---
author: Ícaro
cover: img/ANN.jpg
date: "2020-04-04"
description: 'Aplicações em R para previsão de séries temporais utilizando Redes Neurais Artificiais (partir do pacote forecast).'
title: Prevendo o futuro com Redes Neurais Artificiais
---
[Photo by Soumil Kumar from Pexels](https://www.pexels.com/photo/photo-of-person-typing-on-computer-keyboard-735911/)

# **Previsão com Redes Neurais Artificiais**

Uma das abordagens que mais tem despertado interesse para previsão são as Redes Neurais Artificiais, em que você treina um modelo bio-inspirado capaz de aprender padrões não lineares e fazer predições.

Nesse post vamos demonstrar de forma simplificada como treinar um modelo de previsão utilizando **Redes Neurais Artificiais Autorregressivas (NNAR)** em `R` para para previsão de séries temporais. Existem diversas arquiteturas de Redes Neurais disponíveis, nesse tutorial vamos utilizar a implementação feed-forward utilizando o pacote [**forecast**](https://pkg.robjhyndman.com/forecast/). 

> `R` é uma linguagem de programação open source. Todos os recursos, bibliotecas, dados e implementações são gratuitos e desenvolvidos pela comunidade.
>
> Se você tem interesse de aprender a instalar o R e os passos básicos para iniciar nesse mundo sugiro o tutorial básico desenvolvido pelo pessoal do [curso-r](https://www.curso-r.com/): http://material.curso-r.com/instalacao/

*Vamos começar...*

![](https://media.giphy.com/media/13HBDT4QSTpveU/giphy.gif#center)

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

Para a estimação dos parâmetros e ajuste do modelo será utilizado a função `nnetar()`, que utiliza o algorítimo baseado na função [nnet()](https://cran.r-project.org/web/packages/nnet/) desenvolvido e publicado por Venables e Ripley (2002). A função pertence a biblioteca [**forecast**](https://pkg.robjhyndman.com/forecast/) que atualmente é mais completa para previsão de séries temporais com aplicações em R. 

Esta abordagem somente considera a arquitetura feed-forward networks com uma camada intermediária usando a notação `NNAR(p,k)` para séries sem sazonalidade e `NNAR(p,P,k)[m]` para séries com sazonalidade sendo que `p` representa o número de lags na camada de entrada, `k` o número de nós na camada intermediária da rede, `P` o número de lags sazonais e `[m]` a ordem sazonal.

```visual-basic
NNAR_fit <- nnetar(MA)
NNAR_fit #sai o modelo ajustado
```

```visual-basic
## Series: MA 
## Model:  NNAR(2,1,2)[12] 
## Call:   nnetar(y = MA)
## 
## Average of 20 networks, each of which is
## a 3-2-1 network with 11 weights
## options were - linear output units 
## 
## sigma^2 estimated as 3022826
```

O modelo ajustado automaticamente considerou 2 lags na camada de entrada, 1 lag sazonal de ordem 12 (meses) e 2 nós na camada intermediária, tais parâmetros podem e devem ser alterados a fim de buscar um melhor ajuste do modelo a partir do comando `nnetar(MA, p = 1, P = 1, size = 1)`, também é possível definir o número de repetições para o ajuste do modelo adicionando o argumento `repeats = 20`, o que acarretará em um provável aumento da acurácia, mas também exigirá maior tempo para o ajuste da rede caso `repeats > 20`.

## Verificação dos resíduos

A próxima etapa é verificar a qualidade do ajuste do modelo, para isso vamos analisar os resíduos do modelos, olhando para os erros gerados no processo de estimação, felizmente o pacote **forecast** fornece funções para esse fim.

```visual-basic
checkresiduals(NNAR_fit)
```

![](https://raw.githubusercontent.com/icaroagostino/ANN/master/img/Exemplo%20MA/res.png#center)

Os resíduos gerados pelo modelo apresentaram características de ruído branco. Como podemos ver a sequência de erros oscila em torno de zero (gráfico superior). Olhando para o gráfico ACF, a maior parte dos valores estão dentro do intervalo de confiança (gráfico inferior esquerdo), demonstrando que a informação correlação temporal foi "capturada" pelo modelo. Além disso, a distribuição dos erros segue uma distribuição aproximadamente normal (gráfico inferior direito).

Todas essas características são importantes para serem analisadas no processo de estimação para garantir a qualidade da previsão que faremos na próxima etapa.

## Previsão

Finalmente vamos prever o futuro, para isso vamos usar a função `forecast` e na sequência plotar o gráfico de previsão com a função `autoplot`. O **horizonte** será de 12 meses.

```visual-basic
forecast(NNAR_fit, h = 12) %>% autoplot
```
![](https://raw.githubusercontent.com/icaroagostino/ANN/master/img/Exemplo%20MA/prev.png#center)

Como podemos ver, a previsão parece seguir o comportamento esperado para a série, sendo capaz de capturar o comportamento de tendência e sazonalidade dos dados.

O modelo que ajustamos nesse tutorial é bem simples e inicial, pode ser melhorado com a inclusões de mais variáveis (com o parâmetro `xreg`), com maior tempo de treinamento (`repeats`) e com diferentes arquiteturas, você pode consultar mais parâmetros possíveis na função `nnet()` do pacote de mesmo nome.

Utilize esse tutorial como ponto de partida, crie modelos melhores e mais robustos e divirta-se no processo :)

## Chegamos ao fim

Nesse tutorial vimos de forma simplificada a utilização de Redes Neurais Artificiais para previsão de séries temporais, o exemplo demonstrado acima é totalmente reproduzível, basta instalar as bibliotecas necessárias e copiar os códigos acima para o seu Rstudio.

Se você está interessado em outros modelos de previsão, nesse link você vai encontrar um tutorial para utilização de modelos [ARIMA](https://icaroagostino.github.io/post/arima/).

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

Os dados utilizados nesse exemplo são públicos, para mais detalhes baixe o script [ANN.R](https://github.com/icaroagostino/ANN/blob/master/NNAR.R) que está [neste](https://github.com/icaroagostino/ANN/) repositório.

---

Até o próximo post, bons estudos em `R` :D

Você pode compartilhar esse post nas redes sociais utilizando os botões no fim dessa página.

![](https://media.giphy.com/media/102h4wsmCG2s12/giphy.gif#center)

