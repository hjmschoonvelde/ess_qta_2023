QTA Day 10: Word embeddings
================
22 July, 2022

The goal of today’s lab session is to develop an understanding for word
embeddings. We’ll train a word embeddings model using the **text2vec**
library (Selivanov, Bickel & Wang, 2022) on a set of speeches of
European Commissioners and we’ll inspect these embeddings.

NB: Keep in mind that this lab session is meant for practice purposes
only. The word vectors that we’ll inspect require careful validation.

Let’s load the required libraries first.

``` r
library(quanteda)
library(quanteda.textstats)
library(tidyverse)
library(text2vec)
```

## Preparing the data

Let’s read in the Commission speeches

``` r
load("european_commission.Rdata")

dim(commission_speeches)
```

    ## [1] 6140    2

``` r
names(commission_speeches)
```

    ## [1] "speaker" "text"

We’ll tokenise the speeches.

``` r
corpus_speeches <- corpus(commission_speeches)
tokens_speeches <- tokens(corpus_speeches,
                          what = "word",
                          remove_punct = TRUE, 
                          remove_symbols = TRUE, 
                          remove_numbers = TRUE,
                          remove_url = TRUE,
                          remove_separators = TRUE,
                          split_hyphens = FALSE,
                          ) %>%
  tokens_remove(stopwords(source = "smart"), padding = TRUE)
```

NB: The next few steps draw on
[this](https://quanteda.io/articles/pkgdown/replication/text2vec.html)
**quanteda** tutorial.

We’ll select those features that occur at least 10 times

``` r
feats <- dfm(tokens_speeches) %>%
    dfm_trim(min_termfreq = 10) %>%
    featnames()
```

``` r
tokens_speeches <- tokens_select(tokens_speeches, feats, padding = TRUE)
```

Let’s inspect which features occur most often

``` r
tokens_speeches %>%
  dfm() %>%
  topfeatures(n = 100,
              decreasing = TRUE,
              scheme = c("count")
)
```

    ##                    european            eu        europe    commission 
    ##       5113048         59742         36230         28884         26388 
    ##        member        states      economic        policy        market 
    ##         21918         21109         20112         20033         18331 
    ##         union          work        energy     important        growth 
    ##         17780         15249         14909         14767         14550 
    ##          make     countries         today     financial        people 
    ##         14211         13889         13598         12860         12827 
    ##       support         world          time          year        future 
    ##         12806         11849         11794         11584         11425 
    ##        social      national         years        public   development 
    ##         11385         11285         11020         10912         10145 
    ##       economy        global         level        crisis      research 
    ##          9952          9892          9769          9664          9422 
    ##    innovation    investment      citizens        change     political 
    ##          9376          8942          8888          8874          8674 
    ##    challenges      services        ladies        sector          good 
    ##          8162          8114          8005          7987          7964 
    ##     gentlemen           key        single       markets        ensure 
    ##          7921          7805          7774          7664          7557 
    ##         trade       council international         rules         clear 
    ##          7526          7519          7413          7397          7378 
    ##   cooperation          area          role        action       process 
    ##          7332          7173          7052          6986          6928 
    ##        common   competition          part      business     framework 
    ##          6872          6863          6862          6753          6606 
    ##          euro        system       climate   sustainable        issues 
    ##          6548          6525          6500          6490          6488 
    ##         state       forward      security     companies      strategy 
    ##          6485          6418          6385          6350          6320 
    ##          made        rights         areas     president    parliament 
    ##          6299          6166          6125          6078          6060 
    ##        reform      progress       working      approach      continue 
    ##          6026          5973          5942          5938          5909 
    ##           set     agreement      measures         means          jobs 
    ##          5873          5859          5819          5814          5770 
    ##      policies         place        strong       efforts          high 
    ##          5588          5411          5356          5346          5331 
    ##          open       country      europe's      regional          data 
    ##          5318          5272          5221          5185          5139

We’ll create a feature-co-occurrence matrix using the `fcm()` function
which calculates co-occurrences of features within a user-defined
context. We’ll choose a window size of 5, but other choices are
available

``` r
speeches_fcm <- fcm(tokens_speeches, 
                    context = "window", 
                    window = 5,
                    tri = TRUE)

dim(speeches_fcm)
```

    ## [1] 21448 21448

Let’s see what `speeches_fcm()` looks like.

``` r
speeches_fcm[1:5,1:5]
```

    ## Feature co-occurrence matrix of: 5 by 5 features.
    ##            features
    ## features    Dear President Committee Regions Erasmus
    ##   Dear       382       195        26       9       2
    ##   President    0       408        53      19       1
    ##   Committee    0         0        52     236       2
    ##   Regions      0         0         0       6       1
    ##   Erasmus      0         0         0       0      34

*Dear* and *President* co-occur 163 times in the corpus. *Dear* and
*Regions* only 5 times.

## Fitting a GloVe model

We’ll now fit a GloVe vector model. GloVe is an unsupervised learning
algorithm for obtaining vector representations for words. Training is
performed on the feature co-occurrence matrix, which represents
information about global word-word co-occurrence statistics in a corpus.

``` r
glove <- GlobalVectors$new(rank = 50, 
                           x_max = 10)

wv_main <- glove$fit_transform(speeches_fcm, 
                               n_iter = 10,
                               convergence_tol = 0.01, 
                               n_threads = 8)
```

    ## INFO  [09:41:58.504] epoch 1, loss 0.1918 
    ## INFO  [09:42:00.999] epoch 2, loss 0.1318 
    ## INFO  [09:42:03.469] epoch 3, loss 0.1155 
    ## INFO  [09:42:05.943] epoch 4, loss 0.1066 
    ## INFO  [09:42:08.408] epoch 5, loss 0.1010 
    ## INFO  [09:42:10.881] epoch 6, loss 0.0971 
    ## INFO  [09:42:13.373] epoch 7, loss 0.0942 
    ## INFO  [09:42:15.849] epoch 8, loss 0.0920 
    ## INFO  [09:42:18.339] epoch 9, loss 0.0902 
    ## INFO  [09:42:20.804] epoch 10, loss 0.0888

``` r
dim(wv_main)
```

    ## [1] 21448    50

The model learns two sets of word vectors - main and context. They are
essentially the same.

``` r
wv_context <- glove$components
dim(wv_context)
```

    ## [1]    50 21448

Following recommendations in the **text2vec** package we sum these
vector. We transpose the `wv_context` object so that it has the same
number of rows and columns as `wv_main`

``` r
word_vectors <- wv_main + t(wv_context)

dim(word_vectors)
```

    ## [1] 21448    50

We now have 50-dimension word_vectors for all 27882 tokens in our
corpus.

## Inspecting the GloVe model

Now it’s tme to inspect these word embeddings. For example, we find the
nearest neighbors of a word (or a set of words) of interest. Nearest
neighbors are those words that are most closely located in the vector
space. We can find those using by calculating cosine similarities
between the word vector of a target word and all other word vectors.

We’ll use a custom function
([source](https://s-ai-f.github.io/Natural-Language-Processing/Word-embeddings.html))
to finds these similar words It takes in three arguments: the target
word, the word_vectors object, and the number of neighbors we want to
inspect.

``` r
find_similar_words <- function(word, word_vectors, n = 10) {
  similarities <- word_vectors[word, , drop = FALSE] %>%
    sim2(word_vectors, y = ., method = "cosine")
  
  similarities[,1] %>% sort(decreasing = TRUE) %>% head(n)
}
```

The Commissioner speeches span the time period 2007–2014, a time of
upheaval in the EU. Let’s take a look at the nearest neighbors of
‘crisis’. The `drop = FALSE` argument ensure that crisis is not
converted to a vector.

``` r
find_similar_words("crisis", word_vectors)
```

    ##    crisis financial  economic  response      face     worst    facing    crises 
    ## 1.0000000 0.7205389 0.6955636 0.6710442 0.6611584 0.6571256 0.6538330 0.6466988 
    ##  problems      past 
    ## 0.6428712 0.6351051

Crisis refers mostly to the Eurocrisis.

Let’s inspect the context of climate

``` r
find_similar_words("climate", word_vectors)
```

    ##       climate        change        global     challenge    challenges 
    ##     1.0000000     0.9431015     0.6961250     0.6813582     0.6579412 
    ## environmental    addressing        energy      tackling     combating 
    ##     0.6310504     0.6287525     0.6233663     0.6198157     0.5976436

Global climate change needs to be addressed, that much is clear.

We can sum vectors to each find neigbors. Let’s add crisis + Ireland

``` r
crisis_Ireland <- word_vectors["crisis", , drop = FALSE] +
  word_vectors["Ireland", , drop = FALSE] 

cos_sim_crisis_Ireland <- sim2(x = word_vectors, y = crisis_Ireland, method = "cosine", norm = "l2")
head(sort(cos_sim_crisis_Ireland[,1], decreasing = TRUE), 10)
```

    ##    crisis    Greece   Ireland     Spain      past situation financial  Portugal 
    ## 0.8951058 0.7853493 0.7851357 0.6635981 0.6369862 0.6258102 0.6251408 0.6198187 
    ##     Italy   country 
    ## 0.6156742 0.6143161

It mostly lists other countries that where also struggling at the time.

What if we substract the Ireland vector from the crisis vector?

``` r
crisis_Ireland <- word_vectors["crisis", , drop = FALSE] -
  word_vectors["Ireland", , drop = FALSE] 

cos_sim_crisis_Ireland <- sim2(x = word_vectors, y = crisis_Ireland, method = "cosine", norm = "l2")
head(sort(cos_sim_crisis_Ireland[,1], decreasing = TRUE), 10)
```

    ##      crisis    response    systemic      crises      threat      severe 
    ##   0.7287263   0.6648250   0.6450068   0.6324673   0.5729786   0.5623114 
    ##   financial      manage exacerbated    economic 
    ##   0.5555145   0.5541356   0.5516986   0.5449746

This time we get more general crisis terms.

Inspecting a word embeddings model like so can be useful for a few
different tasks:

1.  As a list of potential terms for dictionary construction;
2.  As an input to downstream QTA tasks;
3.  As a source for visualization.

Let’s take this first task an example. Perhaps we want to develop a
sentiment dictionary for Commissioner speeches, but we are less trusting
of off-the-shelf sentiment dictionaries because we suspect that these
may not capture how sentiment is expressed in Commissioner speeches. One
way to go is use a small seed dictionary of positive and negative words,
and use word embeddings to inspect what other words are close in the
embedding space to these seed words.

For example, we may take as positive words a small set of positive seed
words: *good*, *nice*, *excellent*, *positive*, *fortunate*, *correct*,
*superior*. And as negative words a small set of negative seed words:
*bad*, *nasty*, *poor*, *negative*, *wrong*, *unfortunate*

Let’s start by calculating the average vector for good.

``` r
positive <- (word_vectors["good", , drop = FALSE] +
  word_vectors["nice", , drop = FALSE] +
  word_vectors["excellent", , drop = FALSE] +
  word_vectors["positive", , drop = FALSE] + 
  word_vectors["fortunate", , drop = FALSE] + 
  word_vectors["correct", , drop = FALSE] + 
  word_vectors["superior", , drop = FALSE]) /7
```

And for bad

``` r
negative <- (word_vectors["bad", , drop = FALSE] +
  word_vectors["nasty", , drop = FALSE] +
  word_vectors["poor", , drop = FALSE] +
  word_vectors["negative", , drop = FALSE] + 
  word_vectors["wrong", , drop = FALSE] + 
  word_vectors["unfortunate", , drop = FALSE]) /6
```

We can now inspect the neighbors of our ‘positive’ seed dictionary.

``` r
cos_sim_positive <- sim2(x = word_vectors, y = positive, method = "cosine", norm = "l2")
head(sort(cos_sim_positive[,1], decreasing = TRUE), 20)
```

    ##        good   excellent     results    positive    examples        news 
    ##   0.7982570   0.6488449   0.6400226   0.6361492   0.6173310   0.6129839 
    ##         lot        turn      follow       words encouraging        past 
    ##   0.6033134   0.5972825   0.5747383   0.5700175   0.5543929   0.5506568 
    ##      making  discussion    progress       thing        made        work 
    ##   0.5504474   0.5366199   0.5365522   0.5275053   0.5273586   0.5235043 
    ##     pleased        make 
    ##   0.5216753   0.5206701

This includes some words that seem useful such as encouraging and
opportunity and forward. But also the word bad appears.

Let’s do the same for our ‘negative’ dictionary

``` r
cos_sim_negative <- sim2(x = word_vectors, y = negative, method = "cosine", norm = "l2")
head(sort(cos_sim_negative[,1], decreasing = TRUE), 20)
```

    ##          bad        thing       handle        worst     negative      abusive 
    ##    0.7051604    0.6384524    0.5483712    0.5345217    0.5334316    0.5176171 
    ##        wrong        worse     hardship    downturns consequences         poor 
    ##    0.5099475    0.5099219    0.4963512    0.4929460    0.4900107    0.4866847 
    ##       stolen         news        broke     breached     excesses      morally 
    ##    0.4854249    0.4827202    0.4810448    0.4806523    0.4762984    0.4594585 
    ##        times        timid 
    ##    0.4581795    0.4468920

Again we see a mix of useful and less useful words.

## Exercises

Estimate new word vectors but this time on a feature co-occurrence
matrix with a window size of 5 but with more weight given to words when
they appear closer to the target word (see the *count* and *weight*
arguments in `fcm()`. To estimate this model comment out the code chunk
below to run the model)

``` r
#speeches_fcm_weighted <- fcm(tokens_speeches, 
#                    context = "window", 
#                    count = "weighted", 
#                    weights = 1 / (1:5),
#                    tri = TRUE)


#glove <- GlobalVectors$new(rank = 50, 
#                           x_max = 10)

#wv_main_weighted <- glove$fit_transform(speeches_fcm_weighted, 
#                                        n_iter = 10,
#                                        convergence_tol = 0.01, 
#                                        n_threads = 8)

#wv_context_weighted <- glove$components

#word_vectors_weighted <- wv_main_weighted + t(wv_context_weighted)
```

2.  Compare the nearest neighbors for crisis in both the original and
    the new model. Are they any different?

``` r
#find_similar_words("crisis", word_vectors)
#find_similar_words("crisis", word_vectors_weighted)
```

3.  Inspect the nearest neighbors for Greece, Portugal, Spain and Italy
    and substract the vectors for Netherlands, Germany, Denmark and
    Austria

``` r
#southern_northern  <- (word_vectors["Greece", , drop = FALSE] +
#  word_vectors["Portugal", , drop = FALSE] +
#  word_vectors["Spain", , drop = FALSE] +
#  word_vectors["Italy", , drop = FALSE] -
#  word_vectors["Netherlands", , drop = FALSE] -
#  word_vectors["Germany", , drop = FALSE] -
#  word_vectors["Denmark", , drop = FALSE] -
#  word_vectors["Austria", , drop = FALSE])


#cos_sim_southern_northern <- sim2(x = word_vectors, y = southern_northern, method = "cosine", norm = "l2")
#head(sort(cos_sim_southern_northern[,1], decreasing = TRUE), 20)
```

4.  And turn this vector around

``` r
#northern_southern  <- (word_vectors["Netherlands", , drop = FALSE] +
#  word_vectors["Germany", , drop = FALSE] +
# word_vectors["Denmark", , drop = FALSE] +
# word_vectors["Austria", , drop = FALSE] -
# word_vectors["Greece", , drop = FALSE] -
#  word_vectors["Portugal", , drop = FALSE] -
#  word_vectors["Spain", , drop = FALSE] -
#  word_vectors["Italy", , drop = FALSE])


#cos_sim_northern_southern <- sim2(x = word_vectors, y = northern_southern, method = "cosine", norm = "l2")
#head(sort(cos_sim_northern_southern[,1], decreasing = TRUE), 20)
```

5.  Inspect these word vectors further. If you receive a
    `subscript out of bounds` error, it means that the word does not
    appear in the corpus.
