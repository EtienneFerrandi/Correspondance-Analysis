# Correspondance-Analysis

It's about to detect correspondances between homilies belonging to the _Quinquaginta_ collection. The different traditions of one sermon has been digitally edited according to the apparatus in the modern editions. The versions whose name starts by an "A" are from Augustine (for example, A_AU_s_142_DVD) and those by a "C" are from Caesarius (for example, C_AU_s_142Q). In this example, "DVD" means the De Verbis Domini collection and Q the Quinquaginta collection.

```ruby
library(R.temis)
library(tm)
library(RColorBrewer)
library(stylo)
library(stopwords)
library(factoextra)

corpus=SimpleCorpus(DirSource("~/quinqua/"))

#on enlève tous les chiffres qui suivent les mots issus de la lemmatisation de Deucalion
corpus1=tm_map(corpus, removeNumbers)

#on enlève tous les stopwords de la liste latine perseus du corpus de textes
corpus_stopwords=tm_map(corpus1, removeWords, stopwords("la", source = "perseus"))

#création d'une liste des 25 mots les plus fréquents
tm=build_dtm(corpus_stopwords)
df=as.data.frame(frequent_terms(tm))
liste=row.names(df)

#création de la matrice TermDocument avec une limitation du nombre de mots aux plus fréquents
dtm=R.temis::build_dtm(corpus_stopwords, dictionary = liste)
inspect(dtm)

#AC avec les mots les plus fréquents
AC=corpus_ca(corpus_stopwords, dtm) 
#AC sans sélection des mots les plus fréquents
AC2=corpus_ca(corpus_stopwords, tm) 

#test du khi2 pour évaluer le niveau de dépendance entre les catégories des lignes et celles des colonnes
summary(AC)

#axes à considérer pour l'AFC
axes=as.data.frame(get_eigenvalue(AC2))

#biplot général
fviz_ca_biplot(AC, repel = TRUE, axes = c(1,2))

#classification ascendante hiérarchique des sermons du corpus avec un nombre de classes fixé entre 8 et 12
Classification=HCPC(AC2, min=8, max= 12, consol = TRUE, metric = "manhattan", method = "ward", graph = FALSE)

#affichage des clusters avec visualisation de leur centre
fviz_cluster(Classification, repel=TRUE, show.clust.cent = TRUE, palette = "jco", main = "Clusters sermons Quinquaginta")

#affichage sous forme de dendrogramme
fviz_dend(Classification, palette = "jco", rect = TRUE, rect_fill = TRUE, rect_border = "jco", labels_track_height = 0.8  )

```
