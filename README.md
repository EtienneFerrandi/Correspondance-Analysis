# Correspondance-Analysis

It's about to detect correspondances between homilies belonging to the _Quinquaginta_ collection. The different traditions of one sermon has been digitally edited according to the apparatus in the modern editions. The versions whose name starts by an "A" are from Augustine (for example, A_AU_s_142_DVD) and those by a "C" are from Caesarius (for example, C_AU_s_142Q). In this example, "DVD" means the De Verbis Domini collection and Q the Quinquaginta collection.

```ruby
library(FactoMineR)
library(R.temis)
library(tm)
library(wordcloud)
library(stylo)
library(stopwords)

stylo() #dresser une table des mots les plus fréquents dans les textes du corpus
X=read.csv("table_with_frequencies.csv", sep = "", header=TRUE)
X2=X[c(1:200), c(1:10)] #sélection des deux cent mots les plus fréquents et des dix premiers textes
AFC=CA(X2, graph = TRUE) #analyse factorielle de correspondances sur une table de fréquences de mots dans un corpus
plot.CA(AFC)

corpus=SimpleCorpus(DirSource("~/corpus/"), control = list(language = "lat"))
#on enlève tous les chiffres qui suivent les mots issus de la lemmatisation de Deucalion
corpus=tm_map(corpus, removeNumbers)
#on enlève tous les stopwords de la liste latin perseus du corpus de textes
corpus_stopwords=tm_map(corpus, removeWords, stopwords("la", source = "perseus"))
#création de la matrice TermDocument
dtm=TermDocumentMatrix(corpus_stopwords)
m <- as.matrix(dtm)
v <- sort(rowSums(m),decreasing=TRUE)
d <- data.frame(word = names(v),freq=v)
nuage=wordcloud(words = d$word, freq = d$freq, min.freq = 1,
          max.words=200, random.order=FALSE, rot.per=0.35, 
          colors=brewer.pal(8, "Dark2"))

corpus2=import_corpus(paths="~/corpus/", format = "txt", language="lat")
dtm2=build_dtm(corpus2) #création du tableau lexical (pas de suppression des stopwords)
inspect(dtm2)
AC=corpus_ca(corpus2, dtm2) #analyse de correspondance
explor(AC)
CAH=corpus_clustering(AC) #classification ascendante hiérarchique
plot(CAH$call$t$tree)

## ----Recherche de Cooccurrences--------------------------------
#graphe de mots sur le DTM ou Analyse de données relationnelles (SNA)
terms_graph(dtm, vertex.label.cex = 0.5, )

#Méthode Alceste (Analyse des Lexèmes co-occurents dans les énoncés simples d'un texte)
#Rainette (The Reinert Method for Textual Data Clustering)
library(rainette)
library(quanteda)
dfm <- as.dfm(dtm)
resrai <- rainette(dfm, k = 10, min_segment_size = 0)
rainette_explor(resrai, dfm, corpus_src = dfm)
```
