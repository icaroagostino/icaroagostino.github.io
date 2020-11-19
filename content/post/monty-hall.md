---
author: Ícaro
cover: img/monty_hall.jpg
date: '2020-03-27'
description: 'Probabilidades são contra intuitivas.'
title: O Problema de Monty Hall
---

[Cover Photo by fotografierende from Pexels](https://www.pexels.com/photo/person-about-to-catch-four-dices-1111597/)

# **O Problema de Monty Hall**

Nesse post vamos abordar um aspecto interessante sobre probabilidades, especificamente o problema de Monty Hall, também conhecido por paradoxo de **Monty Hall**, trata-se de um problema matemático e um paradoxo que surgiu a partir de um concurso televisivo dos Estados Unidos chamado *Let’s Make a Deal*, exibido na década de 1970.[[1]](https://brilliant.org/wiki/monty-hall-problem/)

Para demonstrar os conceitos discutidos aqui vamos simular alguns cenários utilizando funções básicas em `R`. Se você já conhece `R` pode simplesmente ir copiando e colando no seu script os códigos, o exemplo é totalmente reproduzível `:)`. No final desse post você vai achar links para o código usado, assim como links sobre como começar a utilizar `R`.

*Vamos começar...*

![](https://media.giphy.com/media/CjmvTCZf2U3p09Cn0h/giphy.gif#center)

## O problema: escolha a sua sorte

O jogo consistia no seguinte: Monty Hall, o apresentador, apresentava três portas aos jogadores. Atrás de uma delas estava um prêmio (um carro) e, atrás das outras duas, dois bodes.

  - Na 1ª etapa o jogador escolhe uma das três portas (que ainda não é aberta)
  
  - Na 2ª etapa, Monty abre uma das outras duas portas que o jogador não escolheu, revelando que o carro não se encontra nessa porta e revelando um dos bodes
  
  - Na 3ª etapa Monty pergunta ao jogador se quer decidir permanecer com a porta que escolheu no início do jogo ou se ele pretende mudar para a outra porta que ainda está fechada para então a abrir. Agora, com duas portas apenas para escolher — pois uma delas já se viu, na 2.ª etapa, que não tinha o prêmio — e sabendo que o carro está atrás de uma das restantes duas, o jogador tem que tomar a decisão.
  
![](https://brilliant-staff-media.s3-us-west-2.amazonaws.com/tiffany-wang/gWotbuEdYC.png#center)
  
[Photo by brilliant.org](https://brilliant.org/wiki/monty-hall-problem/)

Qual é a estratégia mais lógica? Ficar com a porta escolhida inicialmente ou mudar de porta? Com qual das duas portas ainda fechadas o jogador tem mais probabilidades de ganhar? Por quê?

Intuitivamente pensamos que tanto faz mudar ou não, afinal temos duas portas, logo as chances são 50% para cada uma.

Agora vem um **spoiler**: A chances não são 50%, mas sim **~33.3% se você não mudar** de porta e **~66.6% se você mudar**.

![](https://media1.giphy.com/media/qvWavuCEwjEPu/giphy.gif#center)

## O Físico Turista e o twitter

Esse problema veio até mim por meio do vídeo (link no fim do post) do canal Físico Turista do [**Caio Gomes**](https://twitter.com/caiocgomes), após assistir esse vídeo fiquei pensativo sobre o assunto e resolvi implementar uma simulação em `R` para ver se de fato as coisas ocorriam como descritos pelo **Caio**.

Na época divulguei a implementação no twitter e o próprio **Caio** deu reply `:)`

{{< tweet 1216493961935757324 >}}

## Simulando a sorte?

Quando estudamos probabilidade, a ideia é sempre pensar o que é mais provável acontecer, **empiricamente** podemos fazer esse tipo análise repetindo de forma aleatória muitas vezes um experimento e no fim analisar como em média os resultados se comportam.

Aqui vamos tratar cada decisão como uma estratégia, e implementar ambas utilizando funções de probabilidades, cada estratégia será simulada **10 mil vezes**, no final vamos comparar os resultados graficamente.

  - **Estratégia 1** - Não muda de porta
  - **Estratégia 2** - Muda de porta

Primeiro vamos criar algumas variáveis para o experimento:

```visual-basic
n <- 10000 # Numero de repeticoes
vitorias <- c() # Vetor onde serao gravadas as vitorias
portas <- c(1:3) # Vetor de portas
```
### **Estratégia 1** - Não muda de porta.

Aqui de uma forma simples, apenas utilizando funções básicas do `R` vamos criar um loop com o número de repetições definidas anteriormente.

Vamos aleatoriamente sortear a `porta_premio` e depois a `escolha` do jogador. Depois comparamos se o jogador acertou de primeira, se sim computamos uma vitória, se não computamos a derrota de forma direta, já que nesse caso o jogador nunca mudará de porta.

```visual-basic

###################################
# Estrategia 1 - Nao muda a porta #
###################################

for (i in 1:n){
  
  porta_premio <- sample(x = portas, size = 1) # porta com premio
  escolha <- sample(x = portas, size = 1) # escolha do jogador
  
  # Se a escolha for igual a porta premiada vence [1]
  if (porta_premio == escolha){
    
    vitorias <- c(vitorias, 1)
    
  # Se a escolha for diferente da porta premiada perde [0]
  } else {
    
    vitorias <- c(vitorias, 0)
    
  }
}
```
Agora vamos salvar esses resultados para comparar no final.

```visual-basic
# Computa resultados
data_estrategia1 <-
  data.frame(rodada = c(1:n),
             perc.ganha = cumsum(vitorias)/c(1:n))
```

### **Estratégia 2** - Muda de porta.

Assim como no caso anterior vamos criar um loop com o número de repetições definidas anteriormente.

Vamos novamente aleatoriamente sortear a `porta_premio` e depois a `escolha1` do jogador. Depois o apresentador vai excluir uma das portas diferente da escolha do jogador.

Agora o jogador tem a `escolha1` e a porta que sobrou, nessa estratégia ele sempre muda para a que sobrou, aqui representada pela `escolha2`. Por fim, comparamos se a nova escolha é a `porta_premio`, se sim computamos uma vitória.

```visual-basic
###############################
# Estrategia 2 - Muda a porta #
###############################

vitorias <- c() # Vetor onde serao gravadas as vitorias

for (i in 1:n){
  
  porta_premio <- sample(x = portas, size = 1) # porta com premio
  escolha1 <- sample(x = portas, size = 1) # primeira escolha do jogador
  
  x <- portas[-c(porta_premio, escolha1)] # variavel auxliar
  
  porta_eliminada <- if(length(x) > 1) sample(x, size = 1) else x # elimina
  
  escolha2 <- portas[-c(escolha1,porta_eliminada)] # jogador muda de porta
  
  # Se a escolha 2 for igual a porta premiada vence [1]
  if (porta_premio == escolha2){
    
    vitorias <- c(vitorias, 1)
    
  # Se a escolha 2 for diferente da porta premiada perde [0]
  } else {
    
    vitorias <- c(vitorias, 0)
    
  }
  
}
```
Agora vamos salvar esses resultados para comparar no final.

```visual-basic
# Computa resultados
data_estrategia2 <-
  data.frame(rodada = c(1:n),
             perc.ganha = cumsum(vitorias)/c(1:n))
```
## Resultados contra intuitivos?

Chegou a hora de ver os resultados das simulações, para plotar ambas as estratégias, vamos utilizar o pacote [ggplot2](https://ggplot2.tidyverse.org/).

**Estratégia 1** - Não muda de porta.

```visual-basic
library(ggplot2)

ggplot(data_estrategia1, aes(x = rodada, y = perc.ganha)) +
  geom_line() +
  geom_hline(yintercept=1/3,
             color = "red")
```
![](e1.png#center)

**Estratégia 2** - Muda de porta.

```visual-basic
ggplot(data_estrategia2, aes(x = rodada, y = perc.ganha)) +
  geom_line() +
  geom_hline(yintercept=2/3,
             color = "red")
```
![](e2.png#center)

Mas o que aconteceu? por que mudar sempre de porta parece ser a melhor escolha?

> A verdade é que estatisticamente falando, a estratégia 2 **sim** é a melhor, ela não garante a vitória, mas têm a maior probabilidade de ganhar o jogo, agora vamos tentar entender porque isso acontece.

Uma informação que não levamos em consideração, e que muda a "sorte", é que quando o apresentador revela uma das portas que não foi escolhida ele possui uma informação: **ele sabe onde está o prêmio**. Mas o que isso muda?

## Sorte ou chance?

Como o show deve continuar o apresentador ao revelar umas das portas restantes ele sempre abrirá a porta onde o prêmio não está, assim o suspense continua e o jogo passa para a fase 2.

Sabendo disso quando analisamos novamente o cenário, temos duas possibilidades:

  - **Cenário 1**: você escolhe a porta certa de primeira (nesse caso a chance é de um terço), e o para o apresentador tanto faz qual ele revelará.
  
  - **Cenário 2**: você escolha aporta errada de primeira (nesse caso a chance é dois terços), e o apresentador será forçado a abrir uma porta onde o prêmio não está, logo sobra a sua escolha inicial e mais uma porta possível.
  
Sabendo disso quando aplicamos as duas estratégias temos o seguinte resultado:

  - **Estratégia 1**: se você escolher de primeira a porta certa, somente nesse caso você ganha, logo espera-se que isso ocorra em um terço das vezes (~33.3%).
  
  - **Estratégia 2**: se você não escolher de primeira a porta certa, você ganhará, pois o apresentador eliminará uma porta errada e como você escolheu uma errada inicialmente você muda e ganha o jogo, como a chance inicial de escolher a porta errada é maior, espera-se que isso ocorra em dois terços das vezes (~66,6%).
  
![](https://i.pinimg.com/originals/e4/5f/ec/e45fec5478e45d2668062daf8fd2dd5b.jpg#center)
  
Se você estiver interessado em uma solução analítica para o problema pode encontrar [aqui](https://brilliant.org/wiki/monty-hall-problem/).

## Chegamos ao fim

Espero que tenha gostado desse problema, se quiser acessar o script completo para reproduzir o exemplo basta clicar [aqui](https://github.com/icaroagostino/fun/blob/master/Monty_Hall.R).

> `R` é uma linguagem de programação open source. Todos os recursos, bibliotecas, dados e implementações são gratuitos e desenvolvidos pela comunidade.
>
> Se você tem interesse de aprender a instalar o R e os passos básicos para iniciar nesse mundo sugiro o tutorial básico desenvolvido pelo pessoal do [curso-r](https://www.curso-r.com/): http://material.curso-r.com/instalacao/

Parte do texto inicial utilizado nesse post foi retirado da wikipedia, link [aqui](https://pt.wikipedia.org/wiki/Problema_de_Monty_Hall)

Se você quiser ver o vídeo original do **Caio**, fica aqui o link:

{{< youtube nWYeCX4V5lM >}}

Até o próximo post, bons estudos em `R` :D

Você pode compartilhar esse post nas redes sociais utilizando os botões no fim dessa página.