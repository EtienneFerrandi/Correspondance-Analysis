# Correspondance-Analysis

It's about to detect correspondances between homilies belonging to the _Quinquaginta_ collection. The different traditions of one sermon has been digitally edited according to the apparatus in the modern editions. The versions whose name starts by an "A" are from Augustine (for example, A_AU_s_142_DVD) and those by a "C" are from Caesarius (for example, C_AU_s_142Q). In this example, "DVD" means the De Verbis Domini collection and Q the Quinquaginta collection.

```ruby
---
title: "ADT"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

```{r}
library(FactoMineR)
library(R.temis)
library(tm)
library(RColorBrewer)
library(stylo)
library(stopwords)
```
Objectif : création d'un nuage de mots du corpus de sermons lemmatisés et sans les stopwords typiques en latin (à partir de la liste fournie par perseus)
```{r}
corpus=SimpleCorpus(DirSource("~/Documents/Documents/Stylometry/corpus/"))
#on enlève tous les chiffres qui suivent les mots issus de la lemmatisation de Deucalion
corpus1=tm_map(corpus, removeNumbers)
#on enlève tous les stopwords de la liste latin perseus du corpus de textes
corpus_stopwords=tm_map(corpus1, removeWords, stopwords("la", source = "perseus"))
#création de la matrice TermDocument
dtm=R.temis::build_dtm(corpus_stopwords)
inspect(dtm)
#création d'un nuage de mots du corpus
word_cloud(dtm,colors=brewer.pal(8, "Dark2"), n=100, min.freq=20)
```
On réalise une analyse factorielle de correspondance sur les sermons du corpus. On évalue les trois documents qui contribuent le plus à l'axe 1 de l'AFC.
```{r}
AC=corpus_ca(corpus_stopwords, dtm) 
explor(AC)
contributive_docs(corpus_stopwords,AC, 1, ndocs=3)
```
On recherche des coocurrences de termes au sein du corpus
```{r}
cut=split_documents(corpus_stopwords, 1) #découpage du corpus en paragraphes
dtm_cut=build_dtm(cut)
cooc_terms(dtm_cut, "deus")
# Recherche de concordances entre "deus" et le mot qui coocurre le plus avec lui
concordances(corpus_stopwords,dtm,c("deus","debeo"))
#graphe de mots sur le DTM ou Analyse de données relationnelles (SNA)
terms_graph(dtm_cut, vertex.label.cex = 0.5)
```
On réalise une classification ascendante hiéarchique pour déterminer quels clusters sont définis au sein du corpus.
```{r}
CAH=corpus_clustering(AC)
#au vu du dendrogramme, on fixe un nbre de classes; ici 8
clusTLE<-corpus_clustering(AC,8)
## #Afficher le dendrogramme
plot(clusTLE$call$t$tree)
## #description des classes
clusTLE$desc.var
## #documents specifiques des classes
corpus2<-add_clusters(corpus,clusTLE)
characteristic_docs(corpus2,dtmsmo_d2l,meta(corpus2)$cluster)
```
On crééé deux tableaux, l'un qui contient les 100 termes les plus fréquents et un autre, le tableau lexical des termes du corpus. Enfin, on mesure les termes spécifiques des textes du corpus.
```{r}
df=frequent_terms(dtm, variable = NULL, n = 100)

corpus2=import_corpus("~/Documents/Documents/Stylometry/corpus", format = 'txt', language = 'lat')
corpus2=R.temis::set_corpus_variables(corpus2,csv)
dtm2=R.temis::build_dtm(corpus2)
View(meta(corpus2))
lexical_summary(dtm2, corpus2,"Auteur",unit = "global")

specific_terms(dtm)
```
On applique la méthode Reinert, 1983, grâce au package Rainette (The Reinert Method for Textual Data Clustering) qui permet dans un premier temps de réduire le vocabulaire par lemmatisation automatique et choix de catégories de mots analysables. Ensuite, de découper le corpus en parties de texte appelés unités de contexte (UC), de construire un Tableau Lexical Entier (TLE) croisant les UC et le vocabulaire lemmatisé. Ensuite, on fait une classification (CDH) sur le TLE et on calcule les spécificités lexicales des classes. Enfin, on procède à l'interprétation et position des mondes lexicaux (représentations mentales d'un objet) sur l'arbre de classification
```{r}
library(rainette)
library(quanteda)
dfm <- as.dfm(dtm)
resrai <- rainette(dfm, k = 10, min_segment_size = 0)
rainette_explor(resrai, dfm, corpus_src = dfm)
```


```
