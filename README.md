# Correspondance-Analysis

It's about to detect correspondances between homilies belonging to the _Quinquaginta_ collection. The different traditions of one sermon has been digitally edited according to the apparatus in the modern editions. The versions whose name starts by an "A" are from Augustine (for example, A_AU_s_142_DVD) and those by a "C" are from Caesarius (for example, C_AU_s_142Q). In this example, "DVD" means the De Verbis Domini collection and Q the Quinquaginta collection.

```ruby
library(FactoMineR)
library(R.temis)

corpus=import_corpus(paths = "~/corpus/", format = "txt", language = "lat")
dtm=build_dtm(corpus) #création du tableau lexical
inspect(dtm)
word_cloud(dtm, remove_stopwords = FALSE) #création d'un nuage de mot du corpus
AC=corpus_ca(corpus, dtm) #analyse de correspondance
explor(AC)
CAH=corpus_clustering(AC) #classification ascendante hiérarchique
plot(CAH$call$t$tree)*
```
