---
author: Ícaro
cover: img/Rmd-cov19.jpg
date: "2020-03-29"
description: 'Nesse post aprendemos a criar relatórios dinâmicos em Rmarkdown utilizando como exemplo dados do COVID-19 no Brasil'
title: 'Relatório dinâmico com Rmarkdown: COVID-19 no Brasil'
---

[Cover Photo by Lukas from Pexels](https://www.pexels.com/photo/person-writing-on-notebook-669615/)

# **Criando relatórios dinâmicos em R**

Nesse post vamos aprender a criar relatórios dinâmicos em `R` utilizando `Rmarkdown`. Para isso vamos consultar dados online utilizando uma API externa e utilizar esses dados para de forma automática gerar um relatório em `.html` ou `.pdf`.

Se você trabalha em uma empresa e precisa por exemplo fazer relatórios de vendas semanais, usar o Rmarkdown para gerar relatórios dinâmicos pode **facilitar sua vida**, mesmo que você não tenha uma API para consultar os dados automaticamente, é possível automatizar da mesma forma leitura de planilhas e outros tipos de arquivos.

![](f1.png#center)

> `R` é uma linguagem de programação open source. Todos os recursos, bibliotecas, dados e implementações são gratuitos e desenvolvidos pela comunidade.
>
> Se você tem interesse de aprender a instalar o R e os passos básicos para iniciar nesse mundo sugiro o tutorial básico desenvolvido pelo pessoal do [curso-r](https://www.curso-r.com/): http://material.curso-r.com/instalacao/

Para esse exemplo vamos utilizar dados relativos a casos do COVID-19 no Brasil, coletar esses dados online, gerar gráficos automaticamente e criar um relatório.

*Vamos começar...*

![](https://media.giphy.com/media/AElZM8kDYlxHGGQ6kB/giphy.gif#center)

## Dados online do COVID-19

Na data que esse post está sendo realizado passamos por um momento difícil no mundo, com uma pandemia global causada pelo novo Corona Vírus (COVID-19). Diversos esforços estão sendo realizados para disponibilizar dados para que cientistas possam estudar e entender o espalhamento dessa doença.

Recentemente uma aplicação foi desenvolvida que permite acessar dados atualizados de casos no mundo inteiro, os dados são obtidos de forma dinâmica da API [covid19api](https://covid19api.com/) que utiliza como fonte dados da [Johns Hopkins CSSE](https://github.com/CSSEGISandData/COVID-19).

Essa API permite diversos tipos de consultas, aqui vamos utilizar a opção por país, consultando o número de casos confirmados, mortes e recuperados por dia. Para ver todas as opções de consultas dessa API clique [aqui](https://documenter.getpostman.com/view/10808728/SzS8rjbc?version=latest)

## Rotina em `R` para consultar dados via API e gerar gráficos

Antes de criar nosso relatório vamos desenvolver um rotina em `R` para consultar os dados online, tratar e gerar gráficos automaticamente.

Vamos iniciar carregando as bibliotecas necessárias:

  - [`httr`](https://github.com/r-lib/httr): permite fazer requisições http, vamos utilizar essa biblioteca para consultar os dados.
  
  - [`jasonlite`](https://github.com/jeroen/jsonlite): permite manipular arquivos `.json`, por padrão API's trocam arquivos `.json`, não se preocupe de você não sabe ou nunca manipulou esse tipo de arquivo, vamos converter esses dados para `data.frame` automaticamente após receber.
  
  - [`ggplot2`](https://ggplot2.tidyverse.org/): biblioteca para criação de gráficos.
  
  - [`plotly`](https://plotly.com/r/): biblioteca para gráficos interativos.

```visual-basic
library(httr)     # Cominucacao com API's
library(jsonlite) # Manipulacao de .json
library(ggplot2)  # graficos
library(plotly)   # graficos interativos
```
Agora vamos consultar os dados na plataforma, vamos utilizar a consulta por país utilizando as 3 opções: Nº de casos, Nº de mortes e Nº de recuperados.

Para isso vamos utilizar a função `GET` da biblioteca `httr`. A url básica para acessar esses dados é:
  
  - `api.covid19api.com/country/{país}/status/{tipo de dado}`

(substitua no link o nome do país e o tipo de dados para: confirmed, recovered ou deaths)

As três consultas serão realizadas com os códigos abaixo, após receber os dados vamos fazer a conversão utilizando funções da biblioteca `jasonlite` que vai nos retornar um `data.frame`.

```visual-basic
# Consultando dados de nº de casos confirmados
cov_br_conf <- GET(url = "https://api.covid19api.com/country/brazil/status/confirmed",
                   encode = 'json') %>%
  content %>%
  toJSON %>%
  fromJSON

# Consultando dados de nº de recuperados
cov_br_rec <- GET(url = "https://api.covid19api.com/country/brazil/status/recovered",
                  encode = 'json') %>%
  content %>%
  toJSON %>%
  fromJSON

# Consultando dados de nº de mortes
cov_br_dt <- GET(url = "https://api.covid19api.com/country/brazil/status/deaths",
                 encode = 'json') %>%
  content %>%
  toJSON %>%
  fromJSON
```
Agora vamos converter as datas para um formato que o `R` entende, depois vamos criar um `data.frame` unificado e remover a primeira linha (a consulta retorna como primeira linha o dia 23 de janeiro quando não tínhamos casos no Brasil ainda):

```visual-basic
# Converte as datas
datas <- cov_br_conf$Date %>% unlist %>% as.Date()

# Cria um data.frame unificado
cov_br <- data.frame(
  
  dia = datas,
  confirmados = cov_br_conf$Cases %>% unlist,
  mortes      = cov_br_dt$Cases   %>% unlist,
  recuperados = cov_br_rec$Cases  %>% unlist
  
)

# Remove a primeira linha
cov_br <- cov_br[-1,]
```
Vamos ter uma tabela final assim:

![](f2.png#center)

Agora vamos gerar nossos gráficos utilizando `ggplot2`. Aqui não vou entrar em muitas explicações de como criar os gráficos, mudar cores e etc, sugiro esse [link](https://r4ds.had.co.nz/data-visualisation.html) caso você estiver interessado em aprender a usar o `ggplot2`.

Outro link útil é a [The R Graph Gallery](https://www.r-graph-gallery.com/), lá você tem vários templates de gráficos prontos com o código disponível, foi lá que peguei o modelo dos gráficos desse tutorial.

```visual-basic
# Grafico do nº de casos confirmados
graph_conf <- cov_br %>%
  ggplot(aes(x=dia)) +
  geom_area( aes(y = confirmados), fill  = "#f25a5a", alpha = 0.4) +
  geom_line( aes(y = confirmados), color = "#f25a5a", size = 1) +
  geom_point(aes(y = confirmados), color = "#f25a5a", size = 2)

# Grafico do nº de mortes
graph_mort <- cov_br %>%
  ggplot(aes(x=dia)) +
  geom_area( aes(y = mortes), fill  = "#f28f5a", alpha = 0.4) +
  geom_line( aes(y = mortes), color = "#f28f5a", size = 1) +
  geom_point(aes(y = mortes), color = "#f28f5a", size = 2)

# Grafico do nº de recuperados
graph_rec <- cov_br %>%
  ggplot(aes(x=dia)) +
  geom_area( aes(y = recuperados), fill  = "#69b37b", alpha = 0.4) +
  geom_line( aes(y = recuperados), color = "#69b37b", size = 1) +
  geom_point(aes(y = recuperados), color = "#69b37b", size = 2)
```
Vamos dar uma olhada em como nossos gráficos ficaram:

```visual-basic
graph_conf
```
![](f3.png#center)

```visual-basic
graph_mort
```
![](f4.png#center)

```visual-basic
graph_rec
```
![](f5.png#center)

O nosso código em `R` está pronto, estamos consultando dados online, transformando os dados e gerando automaticamente gráficos. Agora podemos construir nosso relatório, a seguir vou criar um exemplo bem simples utilizando o código acima para consultar os dados.

Lembrando que aqui está tudo **automatizado**, essa rotina sempre vai buscar os **últimos dados atualizados**, você não precisa se preocupar em modificar manualmente esse código, desde que a API continue nos fornecendo os mesmos padrões de dados, a rotina funcionará perfeitamente.

Antes de prosseguir crie uma pasta no seu computador e salve a rotina que criamos (exclua a última parte onde plotamos os gráficos, isso será feito direto no relatório), se você quer baixar direto essa rotina clique [aqui](https://github.com/icaroagostino/API_cov19/blob/master/tutorial/cov19.R).

## Criando um relatório dinâmico com Rmarkdown

`Markdown` é uma linguagem simples de marcação, em que podemos escrever documentos que são renderizados em diversos formatos como `.pdf`, `.html` e até `.docx`.

Já o `Rmarkdown` é uma ferramenta que permite que códigos em `R` possam ser executados junto de documentos, evitando que você precise rodar sua análise, salvar gráficos e resultados no seu computador e depois copiar esses resultados para seu relatório. Com `Rmarkdown` seu código em `R` é executado junto com a renderização do documento em um único arquivo.

A IDE [RStudio](https://rstudio.com/) possui suporte nativo para `Rmardown`, basta clicar no botão de criação de um novo documento e a opção estará lá:

![](f6.png#center)

Agora temos que escolher o formato do nosso relatório, nesse caso vamos selecionar `.html`, posteriormente podemos trocar o formato de saída sem problemas.

![](f7.png#center)

No cabeçalho em cima podemos adicionar algumas informações básicas para o relatório, como título, autor, data e etc. Ao criarmos o documentos alguns códigos de exemplo virão juntos, podemos excluir esses códigos e deixar nosso arquivo desta forma:

```visual-basic
---
title: "Casos de COVID-19 no Brasil"
author: "Ícaro Agostino"
date: "29/03/2020"
output: html_document
---
```

Para incluir nosso código em `R` podemos utilizar o atalho `Ctrl`+`Alt`+`i` ou apertar o botão insert. Perceba que é possível incluir não somente códigos em `R` como também `python`, `SQL` dentre outras linguagens.

![](f8.png#center)

Após criar nosso `chunk` podemos chamar a nossa rotina em `R`, utilizando a função `source`. Aqui é importante incluir `include=FALSE` ao lada do `r` no topo do `chunk`, isso garante que o código seja executado sem mostrar nenhum resultado no nosso relatório.

*Obs.: se você estiver copiando os códigos para seu exemplo, exclua a `#` no começo e final de cada `chunk`.*

```visual-basic
#```{r include=FALSE}
source("cov19.R")
#```
```
Atente para o nome do arquivo `.R` e o caminho que ele está no seu computador, caso você não tenha salvo o script na sua máquina como fizemos acima, podemos chamar ele direto do github, nesse caso escreva o `chunk` assim:

```visual-basic
#```{r include=FALSE}
source(https://raw.githubusercontent.com/icaroagostino/API_cov19/master/tutorial/cov19.R")
#```
```
Nós podemos escrever livremente o texto que deve ir no relatório, podemos destacar títulos de seções utilizando `#` para seções primárias, `##` para secundárias e assim por diante. Para inserir link no nosso relatório basta escrever `[texto aqui](link aqui)`.

Vamos começar com um título e algumas informações básicas:

```visual-basic
## Número de casos de COVID-19 no Brasil

Esse relatório dinâmico mostra o número de casos de **COVID-19 no Brasil**, 
os dados são obtidos de forma dinâmica da API [covid19api](https://covid19api.com/) 
que utiliza como fonte dados da 
[Johns Hopkins CSSE](https://github.com/CSSEGISandData/COVID-19).

Última atualização: **`r cov_br$dia[NROW(cov_br)] %>% format(format="%d/%m/%Y")`**.
```

Perceba que na última linha, para a última atualização da data, utilizei um pequeno código que vai consultar na nossa tabela a última informação e automaticamente alterar no texto, automatizando esse passo.

Agora vamos chamar nossos gráficos, para isso vamos converter os gráficos estáticos gerados pelo `ggplot2` em gráficos iterativos utilizando a função `ggplotly` do pacote `plotly`. Esse passo só é necessário caso você deseja renderizar seu relatório em `.html` para ter gráficos interativos, caso for renderizar em `.pdf` pode chamar direto os gráficos `graph_conf`, `graph_mort` e `graph_rec` dentro dos `chunks` abaixo.

```visual-basic
### Evolução diária do número de casos confirmados no Brasil (COVID-19)

#```{r echo=FALSE, message=FALSE, warning=FALSE}
ggplotly(graph_conf)
#```

### Evolução diária do número de mortes confirmadas no Brasil (COVID-19)

#```{r echo=FALSE, message=FALSE, warning=FALSE}
ggplotly(graph_mort)
#```

### Evolução diária do número de recuperados confirmados no Brasil (COVID-19)

#```{r echo=FALSE, message=FALSE, warning=FALSE}
ggplotly(graph_rec)
#```
```
Pronto, nosso relatório esta pronto, vamos pedir para renderizar clicando botão `Knit`.

![](f9.png#center)

Todo o código para gerar esse relatório está [aqui](https://github.com/icaroagostino/API_cov19/blob/master/tutorial/cov19.Rmd).

## Vendo os resultados

Aqui podemos ver como ficou a renderização do relatório em imagem, para ver a renderização em `.pdf` [clique aqui](content/exemplo.pdf) e em `.html` [clique aqui](content/exemplo.html).

---

[![](exemplo/exemplo-1.png#center)](content/exemplo.html)

---

Esse exemplo foi renderizado no dia **29/03/2020** com dados até **28/03/2020**, se você rodar depois desta data os dados estarão diferentes e atualizados.

Uma versão extendida desse relatório incluindo novos casos diário pode ser encontrada [aqui](https://github.com/icaroagostino/API_cov19).

## Chegamos ao fim

Espero que tenha gostado desse tutorial, lembrando que esse exemplo pode ser **extendido a diversas situações** onde temos que gerar relatórios padronizados em que novos dados chegam com o passar do tempo.

Se quiser acessar os scripts completos para reproduzir o exemplo basta clicar [aqui](https://github.com/icaroagostino/API_cov19/tree/master/tutorial).

Para aprender mais sobre `Rmarkdown` sugiro o livro [**R Markdown: The Definitive Guide**](https://bookdown.org/yihui/rmarkdown/), é dos criadores do pacote e 100% livre para ler online (como quase tudo em `R` :D). Outro material muito bom foi desenvolvido pelo pessoal do [RStudio](https://rmarkdown.rstudio.com/lesson-1.html) e pode ser encontrado [aqui](https://rmarkdown.rstudio.com/lesson-1.html).

Até o próximo post, bons estudos em `R` :D

Você pode compartilhar esse post nas redes sociais utilizando os botões no fim dessa página.
