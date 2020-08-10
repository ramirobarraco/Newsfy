## Newsfy

### Objective
Given a corpus of multiple news articles summaries we want to make clusters that represent each news, in other words, we want to know which articles talk about the same news. Is important to note that we don't want clusters about the same topics or categories, we want clusters of the same news or continuations of the same.

### Corpus
The corpus was taken from [Telam](https://cablera.telam.com.ar/cables) the Argentine national news agency. Here each article has a title, brief summary, categorie, date, themes and tags.
For Example:
categorie: Turismo
tags: [La Rioja, turismo, coronavirus, Plan 50 Destinos, Alberto Fernández, Matías Lammens, Ricardo Quintela, José Rosa, La Rioja, Argentina]
date: 08/08/2020 15:59
summary: El ministro de Turismo y Deportes, Matías Lammens, y el gobernador de La Rioja, Ricardo Quintela, pusieron en marcha el Plan Federal de Turismo y Culturas en la provincia, que se incorporó así al Plan 50 Destinos, en el cual el Estado nacional invertirá 60 millones de pesos.
themes: LA RIOJA-INFRAESTRUCTURA TURÍSTICA
title: La Rioja recibirá 60 millones de pesos de la Nación para el plan de Turismo y Culturas
By scrapping each of these articles and saved them in a dictionary with the id telam gives to each article as the key. Making a dictionary of documents each with all this data.

### Approaches

#### Naive approach

In this approach I made the matrix by assigning to each document a vector made by each word in the concatenation between the title and the summary. Each word with a value of 1. With [DictVectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.DictVectorizer.html) i transformed the matrix to a sparse one and using [KMeans](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.KMeans.html) i made 700 clusters.

#### [Doc2Vec](https://radimrehurek.com/gensim/models/doc2vec.html)

here i trained the doc2vec model  with the summary and title of each and then used it to generate 700 clusters with KMeans.

#### [Named Entities recognition](https://en.wikipedia.org/wiki/Named-entity_recognition)

This approach is similar to the naive one. The difference is that we made a new entry to each vector for the Named Entities.

#### [Mean Shift](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.MeanShift.html)

One problem that KMeans present us, is that it ask for a number of clusters and since the objective also tries in a way to answer how many news there are in the corpus, we end up having to eyeball it and try different numbers. MeanShift doesn't has this problem so i tried  using this method for clustering but it only accept continuous matrix so the naive and NER approach were impossible to try. So i tried with the trained doc2vec.

#### Word2vec similarities

Here each vector is composed by all the words of the summary of the documents and its value isn't 1 as the naive approach. Instead i used the mean of the similarity between the word and each word in the title. The words embeddings used were taken from the SBWCE<sup>1</sup>

### Metrics

Newsfy uses 2 metrics to give each cluster a score. We take the mean of the scores of all the clusters in an approach to decide the general score.

#### First metric

This metric was made for this purpose. Each article in the cluster has a list of Tags, Themes and Categories. This metris is the amount of times each tag appears in the cluster divided by the amount of articles in the cluster.

#### Second metric

Based in [Pointwise Mutual Information](https://en.wikipedia.org/wiki/Pointwise_mutual_information). For each cluster i made a list of the 5 closer clusters. Then with each cluster i took the ratio of tags that appeared in both. Then i took the mean of these 5 ratios and that is the score of the original cluster.

### Results


| Approach | First Metric | Second Metric |
| --- | --- | --- |
| Naive | 0.8429434495301747 | 10.391343667778111 |
| Doc2Vec | 0.42849429705985503 | 3.183148641174655 |
| NER | 0.8205863145044924 | 12.908410047923747 |
| MeanShift | 0.5384727817232402 | 6.972317144271971 |
| Word2Vec | 0.8651116625170725 | 9.981624780069618 |

### Problems

The most important problem was trying to implement metrics that made use of the tags i got from telam. Since there aren't lots of metrics that use tags instead of the words that belong to the cluster. Another problem was cleaning the corpus of short articles or documents with unknown characters.

### Conclusions And Future work

The most important conclusion is that a naive approach sometimes is the best way to tackle a problems since it was the easier to program and also one of the best results. NER was a little bit better but for spanish language sometimes is difficult to get all the entities right. Word2Vec is really good but you have to take the title of the articles out making the corpus smaller.

Work that could improve this could be to train the doc2vec model with more articles or use an already trained one, since the corpus i used only had 4000 summaries of articles. After this one could use Mean-Shift to get a better idea of how many news there are in the corpus and used that as an amount of clusters or just used Mean-Shift to get them.


###Clusters
Some of the best clusters.

Articles about a Golf tournament in Neuquen

>El bonaerense Gallegos es el mejor ubicado en el Golf de Neuquén . El bonaerense Andrés Gallegos es el argentino mejor ubicado, al cumplirse hoy la segunda ronda del Abierto Clásico de golf de Neuquén, que se lleva a cabo en la localidad de Chapelco, correspondiente al Tour Latinoamericano y con premios por 175 mil pesos.
>Argentinos Domínguez y Gallegos continúan a la expectativa luego de tercera ronda en Chapelco. Los argentinos Emilio Domínguez y Andrés Gallegos continúan a la expectativa, a tres golpes de la vanguardia, tras completarse hoy la tercera ronda del Abierto Clásico de golf de Neuquén, que se disputa en el Chapelco Golf Club y corresponde a una estación del Tour Latinoamericano del PGA.
>El puntano Domínguez se adjudicó el Abierto Clásico de Neuquén de golf. El golfista puntano Emilio 'Puma' Domínguez se adjudicó esta tarde el Abierto Clásico de golf de Neuquén, al vencer en dos hoyos desempates sobre el norteamericano Tom Whitney, luego de que ambos jugadores empataran en el primer puesto, al cabo de las cuatro rondas disputadas en el Chapelco Golf Club.

Cluster with the first article being about an argentinian journalist having a stroke in Bolivia and the secon article about him passing away.
>Periodista argentino sufrió un accidente cerebro vascular en Bolivia y su familia pide ayuda. Sebastián Moro, un periodista oriundo de Mendoza con residencia en la ciudad de La Paz, Bolivia, quedó en grave estado tras sufrir un accidente cerebro-vascular (ACV) y su familia pidió ayuda económica para afrontar gastos, informaron hoy a través de un comunicado.
>Falleció un periodista argentino que sufrió un accidente cerebro vascular en Bolivia . El periodista mendocino Sebastián Moro murió en la madrugada de hoy en la clínica Rengel de la capital boliviana donde estaba internado tras sufrir un accidente cerebro-vascular (ACV), informaron hoy sus familiares a través de un comunicado.

A series of articles about narcotrafic in Argentina
>Detienen en Santa Fe con 150 kilos de cocaína a una banda narco liderada por una mujer. Una mujer apodada "La Curandera" fue detenida en una mansión en el barrio Guadalupe, de la ciudad de Santa Fe, acusada de liderar una banda narco, a la que se le secuestraron casi 150 kilos de cocaína valuada en el mercado minorista entre 75 y 150 millones de pesos, informaron hoy fuentes oficiales.
>Detienen con 15 kilos de cocaína y más de 11 millones de pesos a banda narco liderada por un preso . Una presunta banda narco que operaba en la ciudad de Rosario y que era liderada por un ciudadano peruano que está preso en la cárcel de Piñero, fue desarticulada tras una serie de allanamientos en los que se logró secuestrar 15 kilos de cocaína, más de 5 millones de pesos y 100 mil dólares, informaron hoy fuentes judiciales.

Articles detailing the rate of interest of the Leliq
>La tasa de interés de las Leliq bajó 99 puntos básicos y cerró a 64,942%. El Banco Central de la República Argentina (BCRA) convalidó hoy una baja en la tasa de política monetaria de 99 puntos básicos respecto del cierre de ayer al finalizar a 64,942%.
>La tasa de interés de las Leliq bajó 92 puntos básicos y cerró a 64,013%. El Banco Central de la República Argentina (BCRA) convalidó hoy una baja en la tasa de política monetaria de 92 puntos básicos respecto del cierre de ayer al finalizar a 64,013%.
>La tasa de interés de las Leliq cerró a 63,209% y cayó 382 puntos básicos en la semana . El Banco Central de la República Argentina (BCRA) convalidó hoy una baja de 80 puntos básicos respecto del cierre de ayer al finalizar a 63,209% y en la semana retrocedió 382 puntos básicos.
>La tasa de interés de las Leliq bajó 16 puntos básicos y cerró a 63,047% . El Banco Central de la República Argentina (BCRA) convalidó hoy una baja en la tasa de política monetaria de 16 puntos básicos respecto del cierre del viernes al finalizar a 63,047%.
>La tasa de interés de las Leliq cerró a 63,003% y cayó 20 puntos básicos en la semana. El Banco Central de la República Argentina (BCRA) convalidó hoy una baja marginal de la tasa de Letras de Liquidez (Leliq) al finalizar a 63,003% y en la semana retrocedió 20 puntos básicos.

Articles about the elections held in Arroyito a city of Cordoba
>La ciudad de Arroyito cierra el calendario electoral cordobés el próximo domingo. El próximo domingo la ciudad cordobesa de Arroyito elegirá autoridades municipales, con la participación de cinco listas de candidatos que aspiran a suceder al intendente radical Mauricio Cravero, quien cumple su segundo mandato al frente de la localidad del este de la provincia.
>La ciudad cordobesa de Arroyito elige autoridades municipales. La ciudad de Arroyito, ubicada en el departamento de San Justo de la provincia de Córdoba, está votando hoy autoridades municipales, con la participación de cinco listas de candidatos que aspiran a suceder al intendente radical, Mauricio Cravero, quien cumple su segundo mandato que establece como límite la Carta Orgánica Municipal de esa localidad del este de la provincia.

Articles related about the politic state of bolivia during 2019
>Senadora boliviana ratifica la sesión parlamentaria que designará autoridades provisorias. La senadora boliviana Jeanine Añez ratificó para hoy la sesión extraordinaria de la Asamblea Legislativa que debe designar al sucesor de Evo Morales, quien renunció a la presidencia del país y denunció un golpe de Estado en su contra.
>Senadora boliviana ratifica la sesión parlamentaria que designará autoridades provisorias. La senadora boliviana Jeanine Añez ratificó para hoy la sesión extraordinaria de la Asamblea Legislativa que debe designar al sucesor de Evo Morales, quien renunció a la presidencia del país y denunció un golpe de Estado en su contra, aunque la bancada del MAS ya anticipó que no va a participar de una convocatoria que tacha de "trucha".


### References
1.[Cristian Cardellino: Spanish Billion Words Corpus and Embeddings (March 2016)]( https://crscardellino.github.io/SBWCE/).




