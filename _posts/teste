---
layout: post
title: Criando um mapa de disputas militarizadas nas Américas em Python
readtime: true
comments: true
social-share: true
---

O nosso objetivo é construir, em Python, um mapa com a localização de disputas militarizadas nas Américas ocorridas entre  1816 e 2010. Os dados relativos as disputas militarizadas são do [Correlates of War (COW)](https://correlatesofwar.org/). Vamos usar, especificamente, o banco de dados [Militarized Interstate Dispute Locations (MIDLOC)](https://correlatesofwar.org/data-sets/MIDLOC) que contém as coordenadas geográficas de cada disputa militarizada catalogada pelo projeto. O arquivo que vamos usar está em CSV e se chama `MIDLOCA_2.1`. 

O _Militarized Interstate Dispute Location_ é mantido por Paul Bezerra e Alex Braithwaite. Para mais informações, ver [Braithwaite (2010)](https://journals.sagepub.com/doi/10.1177/0022343309350008).

Como motivação, este é o resultado que esperamos obter:

![mapa](https://raw.githubusercontent.com/pedrodrocha/pedrodrocha.github.io/master/blog/midmap_americas.png)

Para fazer o mapa, precisamos importar quatro pacotes: `pandas`,`geopandas`, `matplotlib` e `shapely`. Se você ainda não possui eles instalados, é preciso fazê-lo pelo Terminal:
- _conda install geopandas pandas matplotlib shapely_
; ou
- _pip install geopandas pandas matplotlib shapely_.

Lembre também de ativar o comando mágico `%matplotlib inline` para visualizar o mapa no Jupyter Notebook.


```Python
# Importe os pacotes "pandas", "geopandas", "matplotlib" e "shapely"
import geopandas as gpd
import pandas as pd
import matplotlib.pyplot as plt
from shapely.geometry import Point

# Ative o comando %matplotlib inline
%matplotlib inline
```

Para facilitar, não vamos utilizar um shapefile exterior ao Python para a geometria das Américas. As geometrias serão importadas de um banco de dados geográfico nativo ao pacote `geopandas`, o `naturalearth_lowres`. 

Isso pode ser feito chamando  `gpd.datasets.get_path('naturalearth_lowres)`, dentro da função `read_file` próprio ao `geopandas`. 

Lembre de em um segundo momento recortar o banco de dados de modo a selecionar somente as geometrias correspondentes às Américas. Isso pode ser feito selecionando a coluna `continent` e filtrando para `North America` e `South America`.

```Python
# Importando o geodataframe interno ao geopandas
world = gpd.read_file(gpd.datasets.get_path('naturalearth_lowres')) # Importe o banco de dados

americas = world[(world['continent'] == 'North America') | (world['continent'] == "South America")] # Recorte o grid para as Américas
```

Com o grid das Américas pronto, precisamos agora importar o arquivo `MIDLOCA_2.1`. Ele está disponível no site do projeto [_Correlates of War (COW)_](https://correlatesofwar.org/data-sets/MIDLOC). Para importá-lo, utilizamos a função `read_csv` do pacote `pandas`. Com o argumento `usecols` selecione apenas as variáveis `dispnum`, `midloc2_xlongitude` e `midloc2_ylatitude`.

```Python
# Importando o arquivo MIDLOCA_2.1
midloc = pd.read_csv("midloca.csv", # Caminho para o arquivo no diretório
                     usecols=["dispnum","midloc2_xlongitude","midloc2_ylatitude"]) # Seleção de colunas do arquivo
```

Com o arquivo já ativo no Jupyter Notebook, podemos então começar a prepará-lo. Temos muito o que fazer antes de podermos plotar nosso mapa

Em _primeiro lugar_, precisamos nos livrar das linhas que contém elementos faltantes. Para isso, podemos utilizar `dropna()`. 


```Python
# Nos livrando dos NAs
print(len(midloc)) # Checando a dimensão antes de nos livrar dos NAs 
print(len(midloc.dropna())) # Checando a dimensão depois de nos livrar dos NAs 
midloc = midloc.dropna() # Nos livramos dos NAs, para valer
```

Ao compararmos o número de linhas antes e depois de retirar os elementos faltantes, percebemos que haviam 85 disputas militarizadas em nosso _dataset_ que não tinham coordenadas geográficas. Antes, ela eram 2315; após aplicarmos `dropna()`, ficamos com 2230.

Em _segundo lugar_, para facilitar nosso trabalho daqui em diante, vamos mudar o nome das variáveis do _dataset_ relativas as coordenadas geográficas. Para isso, podemos utilizar a função `rename()` do pacote `pandas`. Ela levará o argumento `columns` que por sua vez recebe um dicionário. No exemplo, a variável `midloc2_xlongitude` foi renomeada para `long` e a variável `midloc2_ylatitude` foi renomeada para `lat`. Você pode checar o resultado chamando `head()`.

```Python
# Mudando o nome das colunas
midloc = midloc.rename(columns={"midloc2_xlongitude":"long",
                                "midloc2_ylatitude":"lat"})
# Checando se funcionou 
midloc.head()
```

Em _terceiro lugar_, devemos transformar as nossas coordenadas geográficas em objetos geográficos. 

Existem 3 tipos de objetos geográficos básicos: _**pontos**_, _**linhas**_ e _**polígonos**_. No nosso caso, como as disputas militarizadas representam pontos únicos no espaço, transformaremos suas coordenadas geográficas em _**pontos**_. Isso pode ser feito com a função `Point()` do pacote `shapely`.

Para transformar as coordenadas de _todas_ as disputas militarizadas de nosso dataset em pontos, precisamos seguir os seguintes passos:
- Criar uma lista vazia para armazenar os pontos. Chamaremos ela de `geom`;
- Utilizar um for loop (`for`) com `zip`;
- No for loop, chamar `Point()` dentro de `.append()`, de modo a adicionar cada ponto a lista `geom`;
- Por fim, fora do for loop, criamos uma nova variável em nosso _dataset_ chamada `geometry` e armazenamos nela a lista de pontos que criamos.

Ao fim, podemos checar o resultado com `head()`.

```Python
# Use zip e for loop para criar pontos a partir das coordenadas geográficas
geom = [] # Crie uma lista vazia para armazenar os pontos
for x, y in zip(midloc['long'], midloc['lat']):
    geom.append(Point(x, y)) # Armazene os pontos na lista

midloc['geometry'] = geom # Crie uma nova variável no dataset para armazenar os pontos contidos na lista

# Checando se funcionou
midloc.head()
```

Em _quarto lugar_, precisamos transformar os nossos dados de disputas militarizadas em um dataframe geográfico. No momento, ele é um dataframe panda comum. Se você quiser, pode checar a sua classe chamando `type()`. Para transformá-lo, utilizaremos a função `gpd.GeoDataFrame()` do pacote `geopandas`. Ao fazê-lo, é preciso adicionar o argumento `crs` que recebe um _Sistema de Coordenadas de Referência_. O CRS de nosso dataframe geográfico deve ser o mesmo CRS do nosso grid de Estados, por isso o argumento `crs` deve receber`americas.crs`. Para checar a transformação, chame `type()`. Para checar se os dois dataframes possuem o mesmo CRS, use o operador booleano `==`.

```Python 
# Checando a classe do dataframe antes de transformá-lo
print(type(midloc))

# Transformando o DataFrame em um GeoDataFrame 
midloc = gpd.GeoDataFrame(midloc,crs=americas.crs) 

# Checando a classe do dataframe depois de transformá-lo
print(type(midloc))


## Checando se os dois GeoDataFrames possuem o mesmo CRS
midloc.crs == americas.crs 
```

Teoricamente, já estamos habilitados agora para plotar nosso mapa. No entanto, temos um problema: em nosso dataset de disputas militarizadas há datapoints localizados em TODOS os continentes. 

Como nosso objetivo é plotar somente para as Américas, devemos filtrá-los. No entanto, como vocês podem observar pelas colunas originárias do arquivo `MIDLOCA_2.1` , não há nenhuma variável representativa para os continentes a partir da qual a gente possa filtrar os datapoints.  Para fazê-lo, portanto, devemos recorrer a outra ferramenta, que não as usuais presentes no pacote `pandas`.

Antes, entretanto, uma demonstração do resultado que obteríamos se resolvêssemos plotar o nosso mapa sem filtrar para as Américas:

![mapa](https://raw.githubusercontent.com/pedrodrocha/pedrodrocha.github.io/master/blog/midmap_semSJ.png)
 
 Para construir esse mapa siga os seguintes passos:
- Crie uma figura de tamanho 15 por 10 com a função `plt.subplots()` do pacote `matplotlib`. Lembre de criar também um objeto chamado `ax`;
- Plote o grid de Estados. Eu passei o parâmetro `facecolor` para mudar a cor dos Estados, já que o padrão é azul. Ao plotar, não esqueça de definir o objeto (`ax`) dentro do qual o mapa deve ser inserido;
- Plote os pontos de disputas militarizadas. Eu passei o parâmetro `facecolor` para controlar a cor dos pontos, `markersize` para controlar o tamanho dos pontos e `label` para poder depois construir uma legenda;
- Dê um título para o mapa com `ax.set_title()`;
- Crie legendas com `ax.legend`. O parâmetro `loc` serve para controlar o local onde a legenda aparecerá no mapa;
- Você pode escolher retirar ou não as linhas das coordenadas com o argumento `plt.axis()`;
- Por fim, salve a imagem com `savefig()`.

```Python
# O mapa ficaria assim se a gente não fizesse uma junção espacial 

fig, ax = plt.subplots(figsize=(15,10)) # Crie uma figura

americas.plot(ax=ax, # Defina o objeto
              facecolor='grey') # Para mudar a cor do mapa

midloc.plot(ax=ax,
            facecolor='darkred', # para mudar a cor dos pontos
            markersize=5, # para mudar o tamanho dos pontos
            label = "Disputas Militarizadas") # para legenda

ax.set_title("MidMap - Américas, 1816-2010") # Título do gráfico

ax.legend(loc='lower right', # localização da legenda
          shadow = True) # Tinkering 

plt.axis('off') # Para remover as linhas

fig.savefig("midmap_semSJ.png",
            bbox_inches="tight") # Para remover espaços em branco em excesso
 ```


Em _quinto lugar_, portanto, como solução para o nosso problema, podemos realizar uma junção espacial entre nossos dois dataframes a partir da função `gpd.sjoin` nativa do pacote `geopandas`. Em uma junção espacial, observações de dois ou mais dataframes geográficos são combinadas a partir da relação espacial que elas possuem.

Uma junção espacial tem dois argumentos principais: `op` e `how`. 

O argumento `op` especifica as **condições** para o `geopandas`  juntar as observações. Em relação a nosso problema, a condição que passaremos é `within`. Portanto, as obsersvações só vão ser combinadas se os pontos representativos das disputas militarizadas estiverem dentro dos polígonos do grid de Estados.

O argumento `how` especifica o **tipo** de combinação que vai ocorrer e qual geometria vai ser retida no dataframe geográfico resultante. Em relação a nosso problema, a condição que passaremos é `inner`. Portanto, o `geopandas` usará a intersecção dos valores dos dois dataframes e retornará um dataframe geográfico com colunas somente do dataframe da esquerda.

Para mais informações sobre junções espaciais, consulte a [documentação online](https://geopandas.org/mergingdata.html#spatial-joins) do pacote geopandas.

```Python
# Performando o Spatial Join
join = gpd.sjoin(midloc, # Esquerda
                 americas, # Direita
                 op="within", # Condição
                 how="inner") # Tipo
# Checando a dimensão dos dois dataframes (antes e depois da junção) para ver se funcionou
print(len(midloc))
print(len(join))
```

Agora sim! Estamos prontos para plotar nosso mapa!! Para fazê-lo, siga as mesmas instruções dadas anteriormente para o mapa de teste (_note que passei um tamanho de figura diferente dessa vez_):

- Crie uma figura de tamanho 10 por 10 com a função `plt.subplots()` do pacote `matplotlib`. Lembre de criar também um objeto chamado `ax`;
- Plote o grid de Estados. Eu passei o parâmetro `facecolor` para mudar a cor dos Estados, já que o padrão é azul. Ao plotar, não esqueça de definir o objeto (`ax`) dentro do qual o mapa deve ser inserido;
- Plote os pontos de disputas militarizadas. Eu passei o parâmetro `facecolor` para controlar a cor dos pontos, `markersize` para controlar o tamanho dos pontos e `label` para poder depois construir uma legenda;
- Dê um título para o mapa com `ax.set_title()`;
- Crie legendas com `ax.legend`. O parâmetro `loc` serve para controlar o local onde a legenda aparecerá no mapa;
- Você pode escolher retirar ou não as linhas das coordenadas com o argumento `plt.axis()`;
- Por fim, salve a imagem com `savefig()`.

```Python
# Agora podemos plotar e ver o resultado
fig, ax = plt.subplots(figsize=(10,10)) # criando a figura e o objeto

americas.plot(ax=ax, # definindo o objeto apropriado
              facecolor='grey') # mudando a cor dos polígonos 

join.plot(ax=ax,
            facecolor='darkred', #  mudando a cor dos pontos
            markersize=5, # mudando o tamanho dos pontos
            label = "Disputas Militarizadas") # Para legenda

ax.set_title("Disputas Militarizadas - Américas, 1816-2010") # Título

ax.legend(loc='lower right', # Posicionamento da legenda
          shadow = True) # Tinkering
plt.axis('off') # Removendo coordenadas
fig.savefig("midmap_americas.png", # Salvando
            bbox_inches="tight") # Retirando espaços brancos em excesso
```

## Temos um mapa!
![mapa](https://raw.githubusercontent.com/pedrodrocha/pedrodrocha.github.io/master/blog/midmap_americas.png)


 © Copyright 2020, Pedro D. Rocha. Última atualização 04 de jul. 2020. 

![cc](https://raw.githubusercontent.com/pedrodrocha/pedrodrocha.github.io/master/blog/cc.png)

