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

    ##                    european            eu        europe    commission        member        states      economic        policy        market         union          work 
    ##       5113048         59742         36230         28884         26388         21918         21109         20112         20033         18331         17780         15249 
    ##        energy     important        growth          make     countries         today     financial        people       support         world          time          year 
    ##         14909         14767         14550         14211         13889         13598         12860         12827         12806         11849         11794         11584 
    ##        future        social      national         years        public   development       economy        global         level        crisis      research    innovation 
    ##         11425         11385         11285         11020         10912         10145          9952          9892          9769          9664          9422          9376 
    ##    investment      citizens        change     political    challenges      services        ladies        sector          good     gentlemen           key        single 
    ##          8942          8888          8874          8674          8162          8114          8005          7987          7964          7921          7805          7774 
    ##       markets        ensure         trade       council international         rules         clear   cooperation          area          role        action       process 
    ##          7664          7557          7526          7519          7413          7397          7378          7332          7173          7052          6986          6928 
    ##        common   competition          part      business     framework          euro        system       climate   sustainable        issues         state       forward 
    ##          6872          6863          6862          6753          6606          6548          6525          6500          6490          6488          6485          6418 
    ##      security     companies      strategy          made        rights         areas     president    parliament        reform      progress       working      approach 
    ##          6385          6350          6320          6299          6166          6125          6078          6060          6026          5973          5942          5938 
    ##      continue           set     agreement      measures         means          jobs      policies         place        strong       efforts          high          open 
    ##          5909          5873          5859          5819          5814          5770          5588          5411          5356          5346          5331          5318 
    ##       country      europe's      regional          data 
    ##          5272          5221          5185          5139

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

    ## INFO  [00:36:56.215] epoch 1, loss 0.1916 
    ## INFO  [00:36:58.548] epoch 2, loss 0.1317 
    ## INFO  [00:37:00.845] epoch 3, loss 0.1153 
    ## INFO  [00:37:03.103] epoch 4, loss 0.1064 
    ## INFO  [00:37:05.378] epoch 5, loss 0.1008 
    ## INFO  [00:37:07.720] epoch 6, loss 0.0969 
    ## INFO  [00:37:10.009] epoch 7, loss 0.0940 
    ## INFO  [00:37:12.285] epoch 8, loss 0.0918 
    ## INFO  [00:37:14.570] epoch 9, loss 0.0901 
    ## INFO  [00:37:16.954] epoch 10, loss 0.0886

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
vectors. We transpose the `wv_context` object so that it has the same
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

    ##    crisis financial  response  economic situation    crises   current    facing      face  problems 
    ## 1.0000000 0.7524674 0.7041877 0.6894738 0.6672441 0.6656138 0.6565840 0.6522636 0.6470957 0.6435665

Crisis refers mostly to the Eurocrisis.

Let’s inspect the context of climate

``` r
find_similar_words("climate", word_vectors)
```

    ##      climate       change       global    challenge     tackling biodiversity       energy   challenges    combating       nature 
    ##    1.0000000    0.9387349    0.7071720    0.6666390    0.6227258    0.6177690    0.6135447    0.6096179    0.6071690    0.5971675

Global climate change needs to be addressed, that much is clear.

We can sum vectors to each find neigbors. Let’s add crisis + Ireland

``` r
crisis_Ireland <- word_vectors["crisis", , drop = FALSE] +
  word_vectors["Ireland", , drop = FALSE] 

cos_sim_crisis_Ireland <- sim2(x = word_vectors, y = crisis_Ireland, method = "cosine", norm = "l2")
head(sort(cos_sim_crisis_Ireland[,1], decreasing = TRUE), 10)
```

    ##    crisis    Greece   Ireland financial situation    facing      fact     Spain      time  economic 
    ## 0.8867139 0.7825902 0.7520153 0.6909402 0.6670793 0.6459119 0.6226135 0.6161054 0.6042114 0.6035676

It mostly lists other countries that where also struggling at the time.

What if we substract the Ireland vector from the crisis vector?

``` r
crisis_Ireland <- word_vectors["crisis", , drop = FALSE] -
  word_vectors["Ireland", , drop = FALSE] 

cos_sim_crisis_Ireland <- sim2(x = word_vectors, y = crisis_Ireland, method = "cosine", norm = "l2")
head(sort(cos_sim_crisis_Ireland[,1], decreasing = TRUE), 10)
```

    ##       crisis     response consequences       change       crises     systemic        worst     backdrop    financial     economic 
    ##    0.7520688    0.6512934    0.6181434    0.5858492    0.5835431    0.5672534    0.5561884    0.5376234    0.5320961    0.5296491

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

    ##        good   excellent     results encouraging    progress        work       start        made    positive       clear        news        hope        note      making 
    ##   0.7712044   0.6758385   0.6688791   0.6363904   0.6114265   0.6054392   0.5943078   0.5935627   0.5901991   0.5866238   0.5818353   0.5752110   0.5644145   0.5616752 
    ##    continue      follow        turn    practice       ideas    examples 
    ##   0.5593802   0.5577209   0.5576794   0.5539986   0.5519418   0.5504012

This includes some words that seem useful such as encouraging and
opportunity and forward. But also the word bad appears.

Let’s do the same for our ‘negative’ dictionary

``` r
cos_sim_negative <- sim2(x = word_vectors, y = negative, method = "cosine", norm = "l2")
head(sort(cos_sim_negative[,1], decreasing = TRUE), 20)
```

    ##            bad           poor          thing      beautiful    instructive        morally       negative           sick            ART           rich       harvests 
    ##      0.7412229      0.6223142      0.5717069      0.5187178      0.5179452      0.5163368      0.5093479      0.5033431      0.4990570      0.4948177      0.4913088 
    ##       scenario   Mastercard's    proportions authorisations      spillover      marriages          wrong   Mobilisation      weathered 
    ##      0.4881395      0.4842546      0.4810798      0.4755884      0.4718849      0.4695651      0.4677111      0.4645006      0.4643193

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
