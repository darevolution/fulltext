

```
  _____     .__  .__   __                   __
_/ ____\_ __|  | |  |_/  |_  ____ ___  ____/  |_
\   __\  |  \  | |  |\   __\/ __ \\  \/  /\   __\
 |  | |  |  /  |_|  |_|  | \  ___/ >    <  |  |
 |__| |____/|____/____/__|  \___  >__/\_ \ |__|
                                \/      \/
```

__Get full text articles from (almost) anywhere__

[![Build Status](https://api.travis-ci.org/ropensci/fulltext.png)](https://travis-ci.org/ropensci/fulltext)
[![Build status](https://ci.appveyor.com/api/projects/status/y487h3ec5wc2s20m/branch/master?svg=true)](https://ci.appveyor.com/project/sckott/fulltext/branch/master)
[![codecov.io](https://codecov.io/github/ropensci/fulltext/coverage.svg?branch=master)](https://codecov.io/github/ropensci/fulltext?branch=master)
[![rstudio mirror downloads](http://cranlogs.r-pkg.org/badges/fulltext)](https://github.com/metacran/cranlogs.app)
[![cran version](http://www.r-pkg.org/badges/version/fulltext)](https://cran.r-project.org/package=fulltext)

rOpenSci has a number of R packages to get either full text, metadata, or both from various publishers. The goal of `fulltext` is to integrate these packages to create a single interface to many data sources.

`fulltext` makes it easy to do text-mining by supporting the following steps:

* Search for articles
* Fetch articles
* Get links for full text articles (xml, pdf)
* Extract text from articles / convert formats
* Collect bits of articles that you actually need
* Download supplementary materials from papers

Additional steps we hope to include in future versions:

* Analysis enabled via the [tm](https://cran.r-project.org/package=tm) package and friends, and via [Spark-R](https://amplab-extras.github.io/SparkR-pkg/) to handle especially large jobs
* Visualization

Data sources in `fulltext` include:

* [Crossref](http://www.crossref.org/) - via the `rcrossref` package
* [Public Library of Science (PLOS)](https://www.plos.org/) - via the `rplos` package
* [Biomed Central](http://www.biomedcentral.com/)
* [arXiv](https://arxiv.org) - via the `aRxiv` package
* [bioRxiv](http://biorxiv.org/) - via the `biorxivr` package
* [PMC/Pubmed via Entrez](http://www.ncbi.nlm.nih.gov/) - via the `rentrez` package
* Many more are supported via the above sources (e.g., _Royal Society Open Science_ is
available via Pubmed)
* We __will__ add more, as publishers open up, and as we have time...See the [master list here](https://github.com/ropensci/fulltext/issues/4#issuecomment-52376743)

Authorization: A number of publishers require authorization via API key, and some even more
draconian authorization processes involving checking IP addresses. We are working on supporting
all the various authorization things for different publishers, but of course all the OA content
is already easily available.

We'd love your feedback. Let us know what you think in [the issue tracker](https://github.com/ropensci/fulltext/issues)

Article full text formats by publisher:  [https://github.com/ropensci/fulltext/blob/master/vignettes/formats.Rmd](https://github.com/ropensci/fulltext/blob/master/vignettes/formats.Rmd)

## Installation

Stable version from CRAN


```r
install.packages("fulltext")
```

Development version from GitHub


```r
devtools::install_github("ropensci/fulltext")
```

Load library


```r
library('fulltext')
```

## Search

`ft_search()` - get metadata on a search query.


```r
ft_search(query = 'ecology', from = 'plos')
#> Query:
#>   [ecology] 
#> Found:
#>   [PLoS: 36692; BMC: 0; Crossref: 0; Entrez: 0; arxiv: 0; biorxiv: 0; Europe PMC: 0; Scopus: 0; Microsoft: 0] 
#> Returned:
#>   [PLoS: 10; BMC: 0; Crossref: 0; Entrez: 0; arxiv: 0; biorxiv: 0; Europe PMC: 0; Scopus: 0; Microsoft: 0]
```

## Get full text links

`ft_links()` - get links for articles (xml and pdf).


```r
res1 <- ft_search(query = 'ecology', from = 'entrez', limit = 5)
ft_links(res1)
#> <fulltext links>
#> [Found] 5 
#> [IDs] ID_28166832 ID_28166755 ID_28007834 ID_27974440 ID_21935658 ...
```

Or pass in DOIs directly


```r
ft_links(res1$entrez$data$doi, from = "entrez")
#> <fulltext links>
#> [Found] 5 
#> [IDs] ID_28166832 ID_28166755 ID_28007834 ID_27974440 ID_21935658 ...
```

## Get full text

`ft_get()` - get full or partial text of articles.


```r
ft_get('10.1371/journal.pone.0086169', from = 'plos')
#> <fulltext text>
#> [Docs] 1 
#> [Source] R session  
#> [IDs] 10.1371/journal.pone.0086169 ...
```

## Extract chunks


```r
library("rplos")
(dois <- searchplos(q = "*:*", fl = 'id',
   fq = list('doc_type:full',"article_type:\"research article\""), limit = 5)$data$id)
#> [1] "10.1371/journal.pone.0102138" "10.1371/journal.pone.0102147"
#> [3] "10.1371/journal.pone.0102132" "10.1371/journal.pone.0036392"
#> [5] "10.1371/journal.pone.0102240"
x <- ft_get(dois, from = "plos")
x %>% chunks("publisher") %>% tabularize()
#> $plos
#>                                     publisher
#> 1 Public Library of ScienceSan Francisco, USA
#> 2 Public Library of ScienceSan Francisco, USA
#> 3 Public Library of ScienceSan Francisco, USA
#> 4 Public Library of ScienceSan Francisco, USA
#> 5 Public Library of ScienceSan Francisco, USA
```


```r
x %>% chunks(c("doi","publisher")) %>% tabularize()
#> $plos
#>                            doi                                   publisher
#> 1 10.1371/journal.pone.0102138 Public Library of ScienceSan Francisco, USA
#> 2 10.1371/journal.pone.0102147 Public Library of ScienceSan Francisco, USA
#> 3 10.1371/journal.pone.0102132 Public Library of ScienceSan Francisco, USA
#> 4 10.1371/journal.pone.0036392 Public Library of ScienceSan Francisco, USA
#> 5 10.1371/journal.pone.0102240 Public Library of ScienceSan Francisco, USA
```

Use `dplyr` to data munge


```r
library("dplyr")
x %>%
 chunks(c("doi", "publisher", "permissions")) %>%
 tabularize() %>%
 .$plos %>%
 select(-permissions.license)
#>                            doi                                   publisher
#> 1 10.1371/journal.pone.0102138 Public Library of ScienceSan Francisco, USA
#> 2 10.1371/journal.pone.0102147 Public Library of ScienceSan Francisco, USA
#> 3 10.1371/journal.pone.0102132 Public Library of ScienceSan Francisco, USA
#> 4 10.1371/journal.pone.0036392 Public Library of ScienceSan Francisco, USA
#> 5 10.1371/journal.pone.0102240 Public Library of ScienceSan Francisco, USA
#>   permissions.copyright.year permissions.copyright.holder
#> 1                       2014                    Ong et al
#> 2                       2014               Songstad et al
#> 3                       2014                 Suzuki et al
#> 4                       2012                Doubeni et al
#> 5                       2014               Hirayama et al
#>                       permissions.license_url
#> 1 http://creativecommons.org/licenses/by/4.0/
#> 2 http://creativecommons.org/licenses/by/4.0/
#> 3 http://creativecommons.org/licenses/by/4.0/
#> 4                                        <NA>
#> 5 http://creativecommons.org/licenses/by/4.0/
```

## Supplementary materials

Grab supplementary materials for (re-)analysis of data

`ft_get_si()` accepts article identifiers, and output from `ft_search()`, `ft_get()`


```r
catching.crabs <- read.csv(ft_get_si("10.6084/m9.figshare.979288", 2))
head(catching.crabs)
#>   trap.no. length.deployed no..crabs
#> 1        1          10 sec         0
#> 2        2          10 sec         0
#> 3        3          10 sec         0
#> 4        4          10 sec         0
#> 5        5          10 sec         0
#> 6        1           1 min         0
```

## Cache

When dealing with full text data, you can get a lot quickly, and it can take a long time to get. That's where caching comes in. And after you pull down a bunch of data, if you do so within the R session, you don't want to lose that data if the session crashes, etc. When you search you _will be able to_ (i.e., not ready yet) optionally cache the raw JSON/XML/etc. of each request locally - when you do that exact search again we'll just give you the local data - unless of course you want new data, which you can do.


```r
ft_get('10.1371/journal.pone.0086169', from='plos', cache=TRUE)
```

## Extract text from PDFs

There are going to be cases in which some results you find in `ft_search()` have full text available in text, xml, or other machine readable formats, but some may be open access, but only in pdf format. We have a series of convenience functions in this package to help extract text from pdfs, both locally and remotely.

Locally, using code adapted from the package `tm`, and two pdf to text parsing backends


```r
pdf <- system.file("examples", "example2.pdf", package = "fulltext")
```


```r
(res <- ft_extract(pdf))
#> <document>/Library/Frameworks/R.framework/Versions/3.3/Resources/library/fulltext/examples/example2.pdf
#>   Title: pone.0107412 1..10
#>   Producer: Acrobat Distiller 9.0.0 (Windows); modified using iText 5.0.3 (c) 1T3XT BVBA
#>   Creation date: 2014-09-18
```

Or extract directly into a `tm` Corpus


```r
paths <- sapply(paste0("example", 2:5, ".pdf"), function(x) system.file("examples", x, package = "fulltext"))
(corpus <- ft_extract_corpus(paths))
#> $meta
#>           names                           class
#> 1 content, meta PlainTextDocument, TextDocument
#> 2 content, meta PlainTextDocument, TextDocument
#> 3 content, meta PlainTextDocument, TextDocument
#> 4 content, meta PlainTextDocument, TextDocument
#> 
#> $data
#> <<VCorpus>>
#> Metadata:  corpus specific: 0, document level (indexed): 0
#> Content:  documents: 4
#> 
#> attr(,"class")
#> [1] "ft_extract"
```

Extract pdf remotely on the web, using a service called `PDFX`


```r
pdf5 <- system.file("examples", "example5.pdf", package = "fulltext")
pdfx(file = pdf5)
```


```r
#> $meta
#> $meta$job
#> [1] "34b281c10730b9e777de8a29b2dbdcc19f7d025c71afe9d674f3c5311a1f2044"
#>
#> $meta$base_name
#> [1] "5kpp"
#>
#> $meta$doi
#> [1] "10.7554/eLife.03640"
#>
#>
#> $data
#> <?xml version="1.0" encoding="UTF-8"?>
#> <pdfx xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://pdfx.cs.man.ac.uk/static/article-schema.xsd">
#>   <meta>
#>     <job>34b281c10730b9e777de8a29b2dbdcc19f7d025c71afe9d674f3c5311a1f2044</job>
#>     <base_name>5kpp</base_name>
#>     <doi>10.7554/eLife.03640</doi>
#>   </meta>
#>    <article>
#>  .....
```

## Meta

* Please [report any issues or bugs](https://github.com/ropensci/fulltext/issues).
* License: MIT
* Get citation information for `fulltext`: `citation(package = 'fulltext')`
* Please note that this project is released with a [Contributor Code of Conduct](CONDUCT.md). By participating in this project you agree to abide by its terms.

[![rofooter](http://ropensci.org/public_images/github_footer.png)](http://ropensci.org)
