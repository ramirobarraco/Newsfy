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


###screenshot
Some of the best clusters.

==================
Luis Verón y Anahí Sánchez, la presencia argentina mañana en Londres. El bonaerense Luis Alberto Verón, campeón latino welter OMB (Organización Mundial de Boxeo) se medirá mañana con el campeón europeo de la categoría, el inglés Michael McKinson, en uno de los combates estelares de la velada a desarrollarse en el York Hall de Bethnal Green, Londres.
Los argentinos Luis Verón y Anahí Sánchez perdieron sus combates en Londres. El bonaerense Luis Alberto Verón, campeón latino welter OMB (Organización Mundial de Boxeo) perdió hoy por puntos con el campeón europeo de la categoría, el inglés Michael McKinson, en uno de los combates estelares de la velada desarrollada en el York Hall de Bethnal Green, Londres.
==================
Con dos partidos se inicia mañana la undécima fecha del Torneo Federal A de fútbol . Crucero del Norte recibirá mañana a Juventud Unida de Gualeguaychú y Juventud Unida Universitario de San Luis a Ferro de General Pico, en el inicio de la undécima fecha del Torneo Federal A de fútbol.
Con dos empates se inició la undécima fecha del Torneo Federal A de fútbol. Crucero del Norte y Juventud Unida de Gualeguaychú igualaron esta tarde 2 a 2 y Juventud Unida Universitario de San Luis y Ferro de General Pico lo hicieron 1 a 1, en el inicio de la undécima fecha del Torneo Federal A de fútbol.
==================
 El bonaerense Gallegos es el mejor ubicado en el Golf de Neuquén . El bonaerense Andrés Gallegos es el argentino mejor ubicado, al cumplirse hoy la segunda ronda del Abierto Clásico de golf de Neuquén, que se lleva a cabo en la localidad de Chapelco, correspondiente al Tour Latinoamericano y con premios por 175 mil pesos.
Argentinos Domínguez y Gallegos continúan a la expectativa luego de tercera ronda en Chapelco. Los argentinos Emilio Domínguez y Andrés Gallegos continúan a la expectativa, a tres golpes de la vanguardia, tras completarse hoy la tercera ronda del Abierto Clásico de golf de Neuquén, que se disputa en el Chapelco Golf Club y corresponde a una estación del Tour Latinoamericano del PGA.
El puntano Domínguez se adjudicó el Abierto Clásico de Neuquén de golf. El golfista puntano Emilio 'Puma' Domínguez se adjudicó esta tarde el Abierto Clásico de golf de Neuquén, al vencer en dos hoyos desempates sobre el norteamericano Tom Whitney, luego de que ambos jugadores empataran en el primer puesto, al cabo de las cuatro rondas disputadas en el Chapelco Golf Club.
==================
Periodista argentino sufrió un accidente cerebro vascular en Bolivia y su familia pide ayuda. Sebastián Moro, un periodista oriundo de Mendoza con residencia en la ciudad de La Paz, Bolivia, quedó en grave estado tras sufrir un accidente cerebro-vascular (ACV) y su familia pidió ayuda económica para afrontar gastos, informaron hoy a través de un comunicado.
Falleció un periodista argentino que sufrió un accidente cerebro vascular en Bolivia . El periodista mendocino Sebastián Moro murió en la madrugada de hoy en la clínica Rengel de la capital boliviana donde estaba internado tras sufrir un accidente cerebro-vascular (ACV), informaron hoy sus familiares a través de un comunicado.
==================
Detienen en Santa Fe con 150 kilos de cocaína a una banda narco liderada por una mujer. Una mujer apodada "La Curandera" fue detenida en una mansión en el barrio Guadalupe, de la ciudad de Santa Fe, acusada de liderar una banda narco, a la que se le secuestraron casi 150 kilos de cocaína valuada en el mercado minorista entre 75 y 150 millones de pesos, informaron hoy fuentes oficiales.
Detienen con 15 kilos de cocaína y más de 11 millones de pesos a banda narco liderada por un preso . Una presunta banda narco que operaba en la ciudad de Rosario y que era liderada por un ciudadano peruano que está preso en la cárcel de Piñero, fue desarticulada tras una serie de allanamientos en los que se logró secuestrar 15 kilos de cocaína, más de 5 millones de pesos y 100 mil dólares, informaron hoy fuentes judiciales.
==================
Piden prisión preventiva para el acusado del crimen de una joven trans en La Plata. El fiscal a cargo de la causa que investiga el asesinato a puñaladas y patadas de una joven trans cerca de la estación terminal de la ciudad de La Plata hace 10 días pidió hoy la prisión preventiva para el único acusado, informaron fuentes judiciales.
Prisión preventiva para el acusado del crimen de una joven trans en La Plata. La justicia platense dictó la prisión preventiva de único acusado de asesinar a puñaladas y patadas a una joven trans cerca de la estación terminal de la ciudad de La Plata el 26 de octubre último, informaron fuentes judiciales.
==================
La tasa de interés de las Leliq bajó 99 puntos básicos y cerró a 64,942%. El Banco Central de la República Argentina (BCRA) convalidó hoy una baja en la tasa de política monetaria de 99 puntos básicos respecto del cierre de ayer al finalizar a 64,942%.
La tasa de interés de las Leliq bajó 92 puntos básicos y cerró a 64,013%. El Banco Central de la República Argentina (BCRA) convalidó hoy una baja en la tasa de política monetaria de 92 puntos básicos respecto del cierre de ayer al finalizar a 64,013%.
La tasa de interés de las Leliq cerró a 63,209% y cayó 382 puntos básicos en la semana . El Banco Central de la República Argentina (BCRA) convalidó hoy una baja de 80 puntos básicos respecto del cierre de ayer al finalizar a 63,209% y en la semana retrocedió 382 puntos básicos.
La tasa de interés de las Leliq bajó 16 puntos básicos y cerró a 63,047% . El Banco Central de la República Argentina (BCRA) convalidó hoy una baja en la tasa de política monetaria de 16 puntos básicos respecto del cierre del viernes al finalizar a 63,047%.
La tasa de interés de las Leliq cerró a 63,003% y cayó 20 puntos básicos en la semana. El Banco Central de la República Argentina (BCRA) convalidó hoy una baja marginal de la tasa de Letras de Liquidez (Leliq) al finalizar a 63,003% y en la semana retrocedió 20 puntos básicos.
 ==================
La ciudad de Arroyito cierra el calendario electoral cordobés el próximo domingo. El próximo domingo la ciudad cordobesa de Arroyito elegirá autoridades municipales, con la participación de cinco listas de candidatos que aspiran a suceder al intendente radical Mauricio Cravero, quien cumple su segundo mandato al frente de la localidad del este de la provincia.
La ciudad cordobesa de Arroyito elige autoridades municipales. La ciudad de Arroyito, ubicada en el departamento de San Justo de la provincia de Córdoba, está votando hoy autoridades municipales, con la participación de cinco listas de candidatos que aspiran a suceder al intendente radical, Mauricio Cravero, quien cumple su segundo mandato que establece como límite la Carta Orgánica Municipal de esa localidad del este de la provincia.
==================
Senadora boliviana ratifica la sesión parlamentaria que designará autoridades provisorias. La senadora boliviana Jeanine Añez ratificó para hoy la sesión extraordinaria de la Asamblea Legislativa que debe designar al sucesor de Evo Morales, quien renunció a la presidencia del país y denunció un golpe de Estado en su contra.
Senadora boliviana ratifica la sesión parlamentaria que designará autoridades provisorias. La senadora boliviana Jeanine Añez ratificó para hoy la sesión extraordinaria de la Asamblea Legislativa que debe designar al sucesor de Evo Morales, quien renunció a la presidencia del país y denunció un golpe de Estado en su contra, aunque la bancada del MAS ya anticipó que no va a participar de una convocatoria que tacha de "trucha".
==================
Recordatorio de la agencia Télam para el lunes 11 de noviembre de 2019.. MACRI INAUGURA UN ENCUENTRO DEL CUERPO DE ABOGADOS DEL ESTADO
Recordatorio de la agencia Télam para el martes 12 de noviembre de 2019. BOLIVIA: LA ASAMBLEA LEGISLATIVA TRATA LA RENUNCIA DE MORALES Y LA CONTINUIDAD INSTITUCIONAL
Recordatorio de la agencia Télam para el sábado 16 de noviembre de 2019.. DEL CAÑO Y BREGMAN ENCABEZAN ACTO DE LA IZQUIERDA CON OBREROS DE CHILE EN FERRO.
Recordatorio de la agencia Télam para el lunes 18 de noviembre de 2019.. LAS DOS CTA Y ORGANIZACIONES SOCIALES MARCHAN CONTRA "EL GOLPE DE ESTADO" EN BOLIVIA
==================
Se realizará en Jujuy el II Encuentro de la Mesa de Abejas Nativas sin Aguijón. Los avances, experiencias y perspectivas en abejas nativas sin aguijón y los recursos poliníferos serán analizados en el II Encuentro de la Mesa de Abejas Nativas Sin Aguijón, que se llevará a cabo desde este jueves, hasta el viernes, en la Facultad de Ciencias Agrarias en San Salvador de Jujuy.
Encuentro en Jujuy sobre producción y uso de la miel de abejas nativas sin aguijón . La producción de abejas nativas sin aguijón, los recursos poliníferos y la miel que se aprovecha para consumo y uso medicinal fueron analizados hoy en el II Encuentro de la Mesa de Abejas Nativas Sin Aguijón (ANSA), que se realiza en Jujuy.
==================
La tasa de interés de las Leliq cerró en 63,009% promedio y alcanzó el mínimo dispuesto por el BCRA. La tasa promedio de Leliq, equivalente a la tasa de política monetaria, se ubicó hoy en 63,009% y de esta forma llegó al piso dispuesto por el Comité de Política Monetaria del Banco Central como meta mínima prevista para noviembre.
La tasa de Leliq cerró en 63,006%. La tasa promedio del día de las Leliq, equivalente a la tasa de política monetaria fue 63,006% por un total adjudicado fue de $VN 183.731 millones.
La tasa de Leliq cerró en 63,004% promedio. El Banco Central de la República Argentina convalidó hoy una tasa promedio de Leliq, equivalente a la tasa de política monetaria de 63,004% promedio para un total adjudicado de $136.206 millones.
==================
Un policía permanece atrincherado en su casa de Los Hornos y amenaza con dispararse. Un policía bonaerense de 46 años permanecía esta mañana atrincherado en su casa situada en la calle 160, entre 64 y 65, de la localidad de Los Hornos, partido de La Plata, desde donde realizó disparos al aire y amenaza con balearse, por lo que se implementó un operativo en la zona, informaron fuentes de la fuerza.
Un policía se atrincheró en su casa, amenazó con suicidarse y se entregó al Grupo Halcón . Un policía bonaerense de 46 años se atrincheró esta mañana en su casa de la localidad platense de Los Hornos y, luego de realizar tiros al aire y amenazar con suicidarse, se entregó ante efectivos del Gurpo Halcón, que rodearon la zona.
==================
Bomberos y brigadistas controlan dos focos de incendios en San Luis. El incendio que afecta al paraje Juan W. Gez, en inmediaciones a la ruta provincial 3, en la provincia de San Luis, se encuentra controlado con dos focos activos, sobre los que trabajan brigadistas y bomberos, informó hoy a Télam el jefe de San Luis Solidario, Damián Gómez.
Bomberos lograron sofocar dos importantes incendios sobre la ruta 3 en la provincia de San Luis. Bomberos, brigadistas y baqueanos lograron sofocar el incendio que se inició el jueves último sobre la ruta número 3 en San Luis, en cercanías a la localidad de Juan W. Gez, informaron hoy fuentes oficiales.
==================
Inglaterra quiere que el australiano Jones siga como entrenador del seleccionado. La Rugby Football Union (RFU), máximo organismo del rugby inglés, desea que el australiano Eddie Jones siga en el cargo como entrenador del seleccionado de Inglaterra hasta 2023, tras culminar como subcampeón mundial en Japón.
El actual subcampeón mundial Inglaterra jugara dos partidos ante Japón en 2020. El seleccionado de Inglaterra de rugby, actual subcampeón mundial, jugará dos partidos ante Japón como visitante en 2020 según lo anuncio hoy la Rugby Football Union (RFU), máximo organismo inglés del rugby.
El entrenador de Inglaterra lamenta no haber hecho cambios para la final . El entrenador del seleccionado inglés de rugby, el australiano Eddie Jones, lamenta hoy no haber "refrescado" al equipo para la final que perdió en la Copa del Mundo de Japón ante Sudáfrica.
==================
El Frente Amplio uruguayo anuncia a Mujica y Astori como futuros ministros si gana el balotaje. El candidato a la presidencia de Uruguay por el Frente Amplio, Daniel Martínez, anunció hoy que, si llega al gobierno, tendrá como ministros de Ganadería y de Relaciones Exteriores a José Mujica y Danilo Astori, dos de los principales dirigentes de la alianza de centroizquierda.
El Frente Amplio sumó a dos de sus mayores figuras para la recta final electoral: Mujica y Astori. A casi dos semanas del balotaje presidencial en Uruguay y con las encuestas en contra, el candidato del Frente Amplio (FA), Daniel Martínez, sumó hoy a su eventual gabinete a dos de las figuras de más peso del oficialismo que ya parecían retiradas de la gestión pública: el ex presidente José Mujica como ministro de Ganadería, Agricultura y Pesca, y Danilo Astori, como canciller.
==================
Hallan asesinada a puñaladas y envuelta en sábanas a una mujer en La Matanza. El cadáver semidesnudo de una mujer asesinada aparentemente a puñaladas, con una bolsa en la cabeza y envuelto en sábanas y frazadas fue hallado esta mañana en el cruce de las calles Torquins y José Hernández, del barrio San Pedro, de la localidad bonaerense de Altos de Laferrere, partido de La Matanza, informaron fuentes policiales.
Hallan asesinada a puñaladas y envuelta en sábanas a una mujer en La Matanza. El cadáver semidesnudo de una mujer asesinada aparentemente a puñaladas, con una bolsa en la cabeza y envuelto en sábanas y frazadas fue hallado hoy en el barrio San Pedro de la localidad bonaerense de Altos de Laferrere, partido de La Matanza, informaron fuentes policiales.
Hallan asesinada a puñaladas y envuelta en sábanas a una mujer en La Matanza. El cadáver semidesnudo de una mujer asesinada de 21 puñaladas, con una bolsa en la cabeza y envuelto en sábanas y frazadas fue hallado hoy en el barrio San Pedro del partido bonaerense de La Matanza, informaron fuentes judiciales y policiales.
==================
Aguacateros, del DT Casalánguida, no pudo por la Liga Mexicana de básquetbol. Aguacateros de Michoacán, del técnico argentino Nicolás Casalánguida, no pudo y perdió anoche frente a Astros de Jalisco, por 80-66, en un partido válido por la fase regular de la Liga Nacional de Baloncesto Profesional (LNBP) de México.
Buen trabajo de argentino Redivo en triunfo de Aguacateros de Michoacán en México. El escolta argentino Lucio Redivo diseñó anoche una muy buena labor para Aguacateros de Michoacán, que dirige el DT comodorense Nicolás Casalánguida y batió a Angeles de Puebla, por 107-76, en partido válido por la fase regular de la Liga Nacional de Baloncesto Profesional (LNBP) de México.
==================
Cuba ordena el "retiro inmediato" de sus colaboradores en Bolivia y denuncia arbitrarias detenciones. El gobierno cubano anunció que ordenó "el retorno inmediato" de colaboradores que cumplían misiones humanitarias o de cooperación en Bolivia, después de denunciar la detención de cuatro de ellos bajo la  "falsa acusación" de que "alientan o financian protestas".
Cuba inicia el retiro de sus ciudadanos de Bolivia tras denunciar detenciones arbitrarias. Un grupo de 226 médicos cubanos partió hoy rumbo a Cuba desde Santa Cruz, un día después de que el gobierno de la isla caribeña decidiera repatriar a sus colaboradores en Bolivia tras denunciar la detención de cuatro de ellos bajo la "falsa acusación" de que "alientan o financian protestas".
==================
El puntero Maipú de Mendoza recibe a Villa Mitre de Bahía Blanca por el Federal A. Deportivo Maipú de Mendoza, uno de los punteros de la Zona B del Torneo Federal de fútbol junto a Huracán Las Heras -que tendrá jornada libre-, recibirá mañana a Villa Mitre de Bahía Blanca, en el inicio de la duodécima fecha.
El puntero Maipú de Mendoza igualó con Villa Mitre de Bahía Blanca por el Federal A. Deportivo Maipú de Mendoza, puntero de la Zona B del Torneo Federal de fútbol, empató esta noche con Villa Mitre de Bahía Blanca 1 a 1, en el inicio de la duodécima fecha.
==================
La CIDH también quiere investigar violaciones de derechos humanos en Chile. La Comisión Interamericana de Derechos Humanos (CIDH) pidió hoy formalmente un permiso al gobierno de Chile para enviar una misión de observadores a ese país que investigue las denuncias contras las fuerzas de la represión durante las protestas sociales de las últimas semanas.
La CIDH también investigará denuncias sobre violaciones de DDHH en Chile. El gobierno de Chile envió hoy una invitación a la Comisión Interamericana de Derechos Humanos (CIDH) para que investigue las denuncias por la acción de la represión durante las protestas de las últimas semanas.
==================
Identifican a la mujer asesinada de 21 puñaladas y envuelta en frazadas en La Matanza. La mujer hallada semidesnuda y asesinada de 21 puñaladas, con una bolsa en la cabeza y envuelta en sábanas y frazadas en el partido bonaerense de La Matanza fue identificada por su ex marido, quien dijo la vio por última vez el domingo, cuando discutieron, informaron hoy fuentes judiciales y policiales.
Detienen a una mujer por encubrimiento de un femicidio en Isidro Casanova. Una mujer fue aprehendida hoy por el supuesto encubrimiento del femicidio de otra mujer hallada el lunes pasado -semidesnuda y asesinada de 21 puñaladas, con una bolsa en la cabeza y envuelta en sábanas y frazadas- en Isidro Casanova, en el partido bonaerense de La Matanza, informaron fuentes judiciales.
==================
Morales pide retorno a la "paz social" y vuelve a denunciar un golpe en su carta de renuncia. El Parlamento de Bolivia recibió hoy la carta de renuncia de Evo Morales a la Presidencia, en la que indica que con decisión busca "evitar" la violencia y expresa su deseo de que retorne la "paz social" al país.
Morales pidió retorno a la "paz social" y volvió a denunciar un golpe en su carta de renuncia. El parlamento de Bolivia recibió hoy la carta de renuncia de Evo Morales a la Presidencia, en la que indica que con decisión busca "evitar" la violencia, expresa su deseo de que retorne la "paz social" al país y vuelve a denunciar que fue víctima de un golpe de Estado.
==================
Con dos partidos, se completa la sexta fecha del campeonato de primera del fútbol femenino. Los dos partidos de la sexta fecha del campeonato de Primera División del fútbol femenino suspendidos ayer por el mal tiempo se jugarán mañana, informó hoy la Asociación del Fútbol Argentino.
Se completa la sexta fecha del campeonato de primera del fútbol femenino. Defensores de Belgrano jugará ante Sindicato Argentino de Televisión (SAT) y Huracán ante Excursionistas en dos partidos a jugarse hoy por la sexta fecha del campeonato de Primera División del fútbol femenino, que fueron suspendidos el pasado lunes por el mal tiempo.
Triunfos de SAT y Excursionistas por el campeonato de Primera División del fútbol femenino. Sindicato Argentino de Televisión (SAT) y Excursionistas ganaron hoy sus respectivos partidos pendientes de la sexta fecha del campeonato de Primera División del fútbol femenino, suspendidos el domingo por el mal tiempo.

### References
1.[Cristian Cardellino: Spanish Billion Words Corpus and Embeddings (March 2016)]( https://crscardellino.github.io/SBWCE/).




