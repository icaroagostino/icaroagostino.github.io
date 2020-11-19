---
author: Ícaro
cover: img/coronabr.png
date: "2020-04-28"
description: coronabr é um pacote de R para fazer download e visualizar os dados dos casos diários de coronavírus (COVID-19) disponibilizados por diferentes fontes
title: Testando o pacote CoronaBr [R]
---

# **O pacote coronabr [R]**

Diversas iniciativas têm facilitado o acesso a dados relativos aos casos de COVID-19 ao redor do mundo. Eu mesmo fiz um post aqui no blog ensinando a usar a API [covid19api](https://covid19api.com/) (você pode acessar clicando [aqui](https://icaroagostino.github.io/post/rmd-cov19/)), que permite acessar dados relativos aos casos, mortes e recuperados por país. Outras iniciativas interessantes têm organizado os dados em maior granularidade, permitindo visualizar a situação por estados com foco no Brasil, como é o caso da iniciativa do [Brasil.io](https://brasil.io/dataset/covid19/caso/).

Recentemente um grupo de pesquisadores desenvolveu o pacote [`coronabr`](https://liibre.github.io/coronabr/index.html) que simplesmente centraliza diversos dos recursos que comentei. O pacote permite de forma simples e prática buscar dados de diversas fontes, baixar esses dados atualizados diretamente para sua seção do `R` já formatados como `data.frame` e prontos para sua análise. Além disso, essa ferramenta fornece visualizações gráficas com mapas e tendências, e já se encontra disponível para download e utilização no GitHub =)

Os pesquisadores responsáveis pelo pacote são: [Sara Mortara](http://www.twitter.com/mortarasara), [Andrea Sánchez-Tapia](https://twitter.com/SanchezTapiaA) e [Karlo Guidoni Martins](https://twitter.com/kguidonimartins), clicando [aqui](https://liibre.github.io/coronabr/about.html) você pode saber um pouco mais sobre eles.

Nas próximas seções vamos instalar o pacote `coronabr`, testar as principais funções e gerar algumas visualizações.

## Instalando __coronabr__ e baixando dados

O pacote está disponível no GitHub, isso significa que precisamos instalar via o pacote `remotes`. Caso você já tenha o pacote `remotes` pode pular essa etapa. No código abaixo instalamos o `remotes` e na sequência o `coronabr`:

```visual-basic
install.packages('remotes') # remotes

remotes::install_github("liibre/coronabr") # coronabr
```

No momento da instalação talvez seja requerido a atualização dos pacotes que são dependências para o `coronabr`, sugiro selecionar a opção `CRAN packages only`.

Após a instalação vamos carregar o pacote e baixar os dados do Brasil por estado.

```visual-basic
library(coronabr) # carregando pacote

dados <- get_corona_br(by_uf = TRUE) # baixando dados
View(dados) # vizualizando dados
```

Usando a função `View()` podemos ver que os dados já estão organizados em `data.frame` contendo diversas informações interessantes, como por exemplo a taxa de mortalidade por região.

![](dados.png#center)

Temos outras funções que baixam dados de diferentes fontes, segundo o site do projeto essas são as possibilidades:

[`get_corona_minsaude()`](https://liibre.github.io/coronabr/reference/get_corona_minsaude.html): Extrai dados de COVID-19 para o Brasil da página do Ministério da Saúde

[`get_corona_br()`](https://liibre.github.io/coronabr/reference/get_corona_br.html): Extrai dados de coronavírus para o Brasil de Brasil.io

[`format_corona_br()`](https://liibre.github.io/coronabr/reference/format_corona_br.html): Preenche as datas vazias nos dados de COVID-19 do Brasil.io

[`get_corona_jhu()`](https://liibre.github.io/coronabr/reference/get_corona_jhu.html): Extrai dados mundiais de coronavírus

Clicando nas funções acima você será direcionado para o site oficial com o help completo da função, contendo a descrição dos argumentos.

## Gráficos e mapas com o pacote __coronabr__

Agora com nossos dados já baixados vamos gerar alguns gráficos de forma bem simples. O primeiro é o histórico de todos os estados, para isso usamos as funções:

```visual-basic
dados <- format_corona_br(dados)
plot_uf(dados)
```
![](plot_uf.png#center)

Você pode customizar bastante o gráfico, selecionando o tipo de dado por casos confirmados ou mortes, mostrar dados proporcionais a população, entre outras possibilidades.

Outro gráfico interessante é um mapa mostrando a dispersão dos casos pelo Brasil, com a possibilidade de gerar esse gráfico como uma animação:

```visual-basic
map_corona_br(dados, anim = TRUE)
```

![](map.gif#center)

Também é possível gerar animações dos gráficos de tendências:

```visual-basic
plot_uf(dados, anim = TRUE)
```

![](casos.gif#center)

Infelizmente para esse último caso eu não consegui alterar a proporção do gráfico, prejudicando um pouco a visualização das datas. Vale ressaltar que o pacote está em constante desenvolvimento, provavelmente logo eles permitirão de forma simples alterar essa opção.

Lembrando que existem outras possibilidades de gráficos derivados das outras fontes de dados =)

## Chegamos ao fim

O pacote `coronabr` é bem direto ao ponto e eficiente, facilitando o fluxo de buscar dados atualizados de diferentes fontes e permitir a geração de gráficos, mapas e animações. Vale lembrar que você pode usar o pacote para baixar os dados já em `data.frame` e criar modelos preditivos.

Parabéns aos criadores do `coronabr`, acredito que essa ferramenta será muito útil para pesquisadores brasileiros na busca de entender como combater esse problema =)

Não deixe de visitar o site oficial do pacote, lá você terá todas as informações: https://liibre.github.io/coronabr/index.html

Se esse post foi útil para você me marque em um tweet, meu user é [`@icaroagostino`](https://twitter.com/icaroagostino). Você pode me seguir nas minhas redes (nos botões a seguir). Se quiser interagir comigo o twitter é a plataforma que mais uso para se comunicar com a comunidade de `R`.

{{< mysm >}}

---

> `R` é uma linguagem de programação open source. Todos os recursos, bibliotecas, dados e implementações são gratuitos e desenvolvidos pela comunidade.
>
> Se você tem interesse de aprender a instalar o R e os passos básicos para iniciar nesse mundo, sugiro o tutorial básico desenvolvido pelo pessoal do curso-r: http://material.curso-r.com/instalacao/

---

Até o próximo post, bons estudos em `R` :D

Você pode compartilhar esse post nas redes sociais utilizando os botões no fim dessa página.

