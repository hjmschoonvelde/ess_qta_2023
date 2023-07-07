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
library(quanteda.corpora)
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

    ## INFO  [12:52:38.696] epoch 1, loss 0.1917 
    ## INFO  [12:52:41.092] epoch 2, loss 0.1316 
    ## INFO  [12:52:43.437] epoch 3, loss 0.1152 
    ## INFO  [12:52:45.775] epoch 4, loss 0.1063 
    ## INFO  [12:52:48.103] epoch 5, loss 0.1007 
    ## INFO  [12:52:50.515] epoch 6, loss 0.0969 
    ## INFO  [12:52:52.907] epoch 7, loss 0.0940 
    ## INFO  [12:52:55.310] epoch 8, loss 0.0918 
    ## INFO  [12:52:57.741] epoch 9, loss 0.0901 
    ## INFO  [12:53:00.089] epoch 10, loss 0.0886

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

    ##    crisis financial  response  economic    crises   current  problems     worst 
    ## 1.0000000 0.7364213 0.7051089 0.6989131 0.6961694 0.6743098 0.6647155 0.6629262 
    ##      face    facing 
    ## 0.6512870 0.6504801

Crisis refers mostly to the Eurocrisis.

Let’s inspect the context of climate

``` r
find_similar_words("climate", word_vectors)
```

    ##       climate        change        global     challenge        energy 
    ##     1.0000000     0.9294916     0.7153259     0.6798152     0.6251943 
    ##    challenges      tackling    addressing environmental     combating 
    ##     0.6207178     0.6198490     0.6158034     0.6100744     0.6100212

Global climate change needs to be addressed, that much is clear.

We can sum vectors to each find neigbors. Let’s add crisis + Ireland

``` r
crisis_Ireland <- word_vectors["crisis", , drop = FALSE] +
  word_vectors["Ireland", , drop = FALSE] 

cos_sim_crisis_Ireland <- sim2(x = word_vectors, y = crisis_Ireland, method = "cosine", norm = "l2")
head(sort(cos_sim_crisis_Ireland[,1], decreasing = TRUE), 10)
```

    ##    crisis    Greece   Ireland financial situation  economic     shown     Spain 
    ## 0.8763837 0.7613849 0.7472700 0.6799355 0.6533613 0.6245232 0.6236872 0.6183308 
    ##   country    facing 
    ## 0.6173216 0.6126600

It mostly lists other countries that where also struggling at the time.

What if we substract the Ireland vector from the crisis vector?

``` r
crisis_Ireland <- word_vectors["crisis", , drop = FALSE] -
  word_vectors["Ireland", , drop = FALSE] 

cos_sim_crisis_Ireland <- sim2(x = word_vectors, y = crisis_Ireland, method = "cosine", norm = "l2")
head(sort(cos_sim_crisis_Ireland[,1], decreasing = TRUE), 10)
```

    ##     crisis     crises   response   systemic    dealing  disasters      worst 
    ##  0.7426447  0.7417806  0.6652384  0.6153838  0.5673196  0.5541403  0.5431474 
    ##      avoid  challenge responding 
    ##  0.5311781  0.5245836  0.5206163

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

    ##        good    positive    examples     results encouraging   excellent 
    ##   0.7808327   0.7062718   0.7018453   0.6528358   0.6370227   0.6347685 
    ##        news        hope       clear        note     message        give 
    ##   0.5807421   0.5783515   0.5768792   0.5750679   0.5698282   0.5654349 
    ##     pleased  discussion    evidence       words       start discussions 
    ##   0.5567510   0.5427146   0.5415881   0.5397098   0.5366197   0.5358276 
    ##   confident   extremely 
    ##   0.5358209   0.5338840

This includes some words that seem useful such as encouraging and
opportunity and forward. But also the word bad appears.

Let’s do the same for our ‘negative’ dictionary

``` r
cos_sim_negative <- sim2(x = word_vectors, y = negative, method = "cosine", norm = "l2")
head(sort(cos_sim_negative[,1], decreasing = TRUE), 20)
```

    ##           bad      negative      scenario  consequences    spillovers 
    ##     0.6881907     0.6704861     0.6004726     0.5620927     0.5554682 
    ##         worse         thing       effects         wrong       obvious 
    ##     0.5497586     0.5239028     0.5207938     0.5075231     0.5037331 
    ##          news      positive         trend         blood    disastrous 
    ##     0.4969956     0.4961535     0.4958657     0.4915971     0.4885373 
    ##        Posted externalities geo-political          poor           DAS 
    ##     0.4810521     0.4800516     0.4795765     0.4776578     0.4737017

Again we see a mix of useful and less useful words.

## Exercises

Estimate new word vectors but this time on a feature co-occurrence
matrix with a window size of 5 but with more weight given to words when
they appear closer to the target word (see the *count* and *weight*
arguments in `fcm()`. To estimate this model comment out the code chunk
below to run the model)

``` r
speeches_fcm_weighted <- fcm(tokens_speeches, 
                    context = "window", 
                    count = "weighted", 
                    weights = 1 / (1:5),
                    tri = TRUE)


glove <- GlobalVectors$new(rank = 50, 
                           x_max = 10)

wv_main_weighted <- glove$fit_transform(speeches_fcm_weighted, 
                                        n_iter = 10,
                                        convergence_tol = 0.01, 
                                        n_threads = 8)
```

    ## INFO  [12:53:04.556] epoch 1, loss 0.1901 
    ## INFO  [12:53:06.911] epoch 2, loss 0.1264 
    ## INFO  [12:53:09.234] epoch 3, loss 0.1075 
    ## INFO  [12:53:11.558] epoch 4, loss 0.0982 
    ## INFO  [12:53:13.879] epoch 5, loss 0.0923 
    ## INFO  [12:53:16.221] epoch 6, loss 0.0883 
    ## INFO  [12:53:18.572] epoch 7, loss 0.0853 
    ## INFO  [12:53:20.919] epoch 8, loss 0.0829 
    ## INFO  [12:53:23.274] epoch 9, loss 0.0811 
    ## INFO  [12:53:25.678] epoch 10, loss 0.0795

``` r
wv_context_weighted <- glove$components

word_vectors_weighted <- wv_main_weighted + t(wv_context_weighted)
```

2.  Compare the nearest neighbors for crisis in both the original and
    the new model. Are they any different?

``` r
find_similar_words("crisis", word_vectors)
```

    ##    crisis financial  response  economic    crises   current  problems     worst 
    ## 1.0000000 0.7364213 0.7051089 0.6989131 0.6961694 0.6743098 0.6647155 0.6629262 
    ##      face    facing 
    ## 0.6512870 0.6504801

``` r
find_similar_words("crisis", word_vectors_weighted)
```

    ##    crisis financial situation  economic  response   current    crises     worst 
    ## 1.0000000 0.7526049 0.7094093 0.6600743 0.6594098 0.6481856 0.6401994 0.6342556 
    ##    facing       hit 
    ## 0.6303885 0.6167291

3.  Inspect the nearest neighbors for Greece, Portugal, Spain and Italy
    and substract the vectors for Netherlands, Germany, Denmark and
    Austria

``` r
southern_northern  <- (word_vectors["Greece", , drop = FALSE] +
  word_vectors["Portugal", , drop = FALSE] +
  word_vectors["Spain", , drop = FALSE] +
  word_vectors["Italy", , drop = FALSE] -
  word_vectors["Netherlands", , drop = FALSE] -
  word_vectors["Germany", , drop = FALSE] -
  word_vectors["Denmark", , drop = FALSE] -
  word_vectors["Austria", , drop = FALSE])


cos_sim_southern_northern <- sim2(x = word_vectors, y = southern_northern, method = "cosine", norm = "l2")
head(sort(cos_sim_southern_northern[,1], decreasing = TRUE), 20)
```

    ##        Greece      solution        return    structural        reform 
    ##     0.6644773     0.5670495     0.5500280     0.5477975     0.5249152 
    ## consolidation       reforms      overcome          time          term 
    ##     0.5191139     0.5147756     0.5145297     0.5131185     0.5045437 
    ##        crisis        Cyprus          long    adjustment           job 
    ##     0.4980700     0.4959574     0.4914641     0.4889783     0.4872575 
    ##     stability       started     financial         Greek     difficult 
    ##     0.4849518     0.4819255     0.4808925     0.4802797     0.4753983

4.  And turn this vector around

``` r
northern_southern  <- (word_vectors["Netherlands", , drop = FALSE] +
  word_vectors["Germany", , drop = FALSE] +
  word_vectors["Denmark", , drop = FALSE] +
  word_vectors["Austria", , drop = FALSE] -
  word_vectors["Greece", , drop = FALSE] -
  word_vectors["Portugal", , drop = FALSE] -
  word_vectors["Spain", , drop = FALSE] -
  word_vectors["Italy", , drop = FALSE])


cos_sim_northern_southern <- sim2(x = word_vectors, y = northern_southern, method = "cosine", norm = "l2")
head(sort(cos_sim_northern_southern[,1], decreasing = TRUE), 20)
```

    ##            MP3            AAA  Stabilization          petty          pulls 
    ##      0.5811587      0.5743342      0.5552847      0.5522885      0.5518696 
    ## sophistication    precautions     worst-case           Cork       undercut 
    ##      0.5414238      0.5380800      0.5334127      0.5201880      0.5105606 
    ##         Device           Tech  intellectuals      pictorial         Munich 
    ##      0.5067531      0.5041397      0.5038774      0.4902376      0.4755115 
    ##        sulphur             PO           beet           heed         Forced 
    ##      0.4751914      0.4713660      0.4671099      0.4659723      0.4650465

5.  Inspect these word vectors further. If you receive a
    `subscript out of bounds` error, it means that the word does not
    appear in the corpus.
