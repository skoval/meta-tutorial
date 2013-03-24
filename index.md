---
title       : Tutorial on Meta-Analysis 
subtitle    : useR! 2013
author      : Stephanie Kovalchik
job         : Research Fellow, National Cancer Institute
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : []            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
---

## What is Meta-Analysis?

> The analysis of analyses.
>
> -- Gene V. Glass, Primary, secondary and meta-analysis of research, Educational Researcher, 1976.

---

Conducting a meta-analysis

---

## Meta-Analysis

* Is a formal summary of *compatible* effects across multiple studies
* The summary method is typically some kind of weighted average
* All meta-analyses should be done within a systematic review
* Assessments of bias and heterogeneity are essential

--- .class #id 

## Why Meta-Analysis?

* Single summary of effect
* Increased power
* Increased generalizability
* Address questions that single studies could not
* Resolve controverisies

--- .class #id 

## When Not to Undertake a Meta-analysis

<center>
![Apples and Oranges](figure/apples_oranges.jpg)
</center>

--- 

### The Fall of Vioxx

<center>
![Nissen Forest Plot](figure/nissen.png)
</center>

---

<center>
![Nissen NY Times](figure/nissen_nytimes.jpg)
</center>

Drug Safety Critic Hurls Darts From the Inside, NY Times, 2007

---

### Key messages from tutorial

* Do not begin here!
* Meta-analyses are (mostly) about weighted averages of normal rvs


---

### Do not begin here!

- All meta-analyses should be part of a systematic review
	* Formulate question
	* Identify studies
	* Develop search strategy
	* Gather data
	* Summarize results (meta-analysis, if appropriate)
	* Evaluate potential biases
	
- The decision to undertake a meta-analysis comes after relevant studies are identified and assessed
- There are few and limited tools to help with early stages of systematic review


---

### Searching PubMed with <code>RISmed</code> package


```r
EUtilsSummary(query, type = "esearch", db = "pubmed", ...)

EUtilsGet(x, type = "efetch", db = "pubmed")
```

---

### Example: BCG Vaccine Trials



```r
library(RISmed)
query <- EUtilsSummary("BCG[tiab] trial*[tiab] placebo[tiab] controlled[tiab]")

substr(QueryTranslation(query), 1, 50)  # PubMed query
```

```
## [1] "BCG[tiab] AND (trial[tiab] OR trial/0[tiab] OR tri"
```

```r

QueryCount(query)  # Number of record returned
```

```
## [1] 43
```


---

### First 10 Records



```r
query <- EUtilsSummary("BCG[tiab] trial*[tiab] placebo[tiab] controlled[tiab]", 
    retmax = 10)
results <- EUtilsGet(query)  # Methods for all NCBI Fields
ArticleTitle(results)  # Method for article title
```

```
##  [1] "Treating BCG-induced disease in children."                                                                                                                                                                           
##  [2] "Safety and efficacy of MVA85A, a new tuberculosis vaccine, in infants previously vaccinated with BCG: a randomised, placebo-controlled phase 2b trial."                                                              
##  [3] "Impact of anthelminthic treatment in pregnancy and childhood on immunisations, infections and eczema in childhood: a randomised controlled trial."                                                                   
##  [4] "Proof-of-concept, randomized, controlled clinical trial of Bacillus-Calmette-Guerin for treatment of long-term type 1 diabetes."                                                                                     
##  [5] "Intravesical gemcitabine for non-muscle invasive bladder cancer."                                                                                                                                                    
##  [6] "Primary isoniazid prophylaxis against tuberculosis in HIV-exposed children."                                                                                                                                         
##  [7] "Prulifloxacin: a review focusing on its use beyond respiratory and urinary tract infections."                                                                                                                        
##  [8] "Effect of single-dose anthelmintic treatment during pregnancy on an infant's response to immunisation and on susceptibility to infectious diseases in infancy: a randomised, double-blind, placebo-controlled trial."
##  [9] "Vitamin A supplementation and BCG vaccination at birth in low birthweight neonates: two by two factorial randomised controlled trial."                                                                               
## [10] "Protective effect of the combination BCG vaccination and rifampicin prophylaxis in leprosy prevention."
```

```r
AbstractText(results)[1]  # Method for abstracts
```

```
## [1] "Bacillus Calmette-Guerín (BCG) is a live attenuated vaccine to prevent tuberculosis, routinely administered at birth as part of the World Health Organization global expanded immunisation programme. Given intradermally, it can cause adverse reactions, including local, regional, distant and disseminated manifestations that may cause parental distress. Rarely, it can cause serious illness and even death. Among those patients with immunocompromised conditions, such as the human immunodeficiency virus (HIV) infection, the complication rate is even higher."
```


---

### Example Data Set: Tuberculosis Vaccine Trials

* 13 vaccine trials of Bacillus Calmette–Guérin (BCG) vaccine against no vaccine (Sham?)
* Primary endpoint:  Tuberculosis infection
* Possible explanatory variables include latitude of study region, treatment allocation method, year published

Colditz GA, et al. Efficacy of BCG vaccine in the prevention of tuberculosis. JAMA 1994; 271:698 –702.

---

### Example Data Set: Continuous Outcome

---

### Notation

* $Y_i$ Trial effect size
* $V_i$ Effect size variance
* $W_i = 1/V_i$ Trial weight

---

### Fixed Effects

- Normal assumption: <math>Y_i ~ N(\mu, V_i)</math>

Summary treatment effect:

<math>
\hat{\mu} = \sum W_i Y_i /\sum W_i
</math>

Variance of treatment effect:

<math>
V(\hat{\mu}) = 1/\sum W_i
</math>

Trial contribution to weighted average:

<math>
\lambda_i = W_i/\sum W_i
</math>
---

### Load and inspect *smoking* data


```r

library(HSAUR)  # Source Package
data(BCG)
str(BCG)
```

```
## 'data.frame':	13 obs. of  7 variables:
##  $ Study   : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ BCGTB   : int  4 6 3 62 33 180 8 505 29 17 ...
##  $ BCGVacc : int  123 306 231 13598 5069 1541 2545 88391 7499 1716 ...
##  $ NoVaccTB: int  11 29 11 248 47 372 10 499 45 65 ...
##  $ NoVacc  : int  139 303 220 12867 5808 1451 629 88391 7277 1665 ...
##  $ Latitude: int  44 55 42 52 13 44 19 13 27 42 ...
##  $ Year    : int  1948 1949 1960 1977 1973 1953 1973 1980 1968 1961 ...
```


--- 

### Packages for performing meta-analysis



```r
library(meta)
library(metafor)
library(rmeta)
```



---

### Syntax for <code>meta.MH</code>


```r

meta.MH(ntrt, nctrl, ptrt, pctrl, conf.level = 0.95, data = NULL, ...)

```


- <code>ntrt</code>: Number in treatment group
- <code>nctrl</code>: Number in control group
- <code>ptrt</code>: Number of events in treatment group
- <code>pctrl</code>: Number of events in control group

---


```r
rmeta.BCG <- meta.MH(BCGVacc, NoVacc, BCGTB, NoVaccTB, data = BCG)
summary(rmeta.BCG)
```

```
## Fixed effects ( Mantel-Haenszel ) meta-analysis
## Call: meta.MH(ntrt = BCGVacc, nctrl = NoVacc, ptrt = BCGTB, pctrl = NoVaccTB, 
##     data = BCG)
## ------------------------------------
##         OR (lower  95% upper)
##  [1,] 0.39    0.12       1.26
##  [2,] 0.19    0.08       0.46
##  [3,] 0.25    0.07       0.91
##  [4,] 0.23    0.18       0.31
##  [5,] 0.80    0.51       1.26
##  [6,] 0.38    0.32       0.47
##  [7,] 0.20    0.08       0.50
##  [8,] 1.01    0.89       1.15
##  [9,] 0.62    0.39       1.00
## [10,] 0.25    0.14       0.42
## [11,] 0.71    0.57       0.89
## [12,] 1.56    0.37       6.55
## [13,] 0.98    0.58       1.66
## ------------------------------------
## Mantel-Haenszel OR =0.62 95% CI ( 0.57,0.68 )
## Test for heterogeneity: X^2( 12 ) = 163.94 ( p-value 0 )
```


---


```r
rma(yi, vi, sei, weights, ai, bi, ci, di, n1i, n2i, x1i, x2i, t1i, t2i, m1i, 
    m2i, sd1i, sd2i, xi, mi, ri, ti, sdi, ni, mods, measure = "GEN", intercept = TRUE, 
    data, slab, subset, add = 1/2, to = "only0", drop00 = FALSE, vtype = "LS", 
    method = "REML", weighted = TRUE, knha = FALSE, level = 95, digits = 4, 
    btt, tau2, verbose = FALSE, control)
```


---


```r
BCG$BCGNeg <- BCG$BCGVacc - BCG$BCGTB  # Not infected
BCG$NoVaccNeg <- BCG$NoVacc - BCG$NoVaccTB  # Not infected
metafor.BCG <- escalc(measure = "OR", ai = BCGTB, bi = BCGNeg, ci = NoVaccTB, 
    di = NoVaccNeg, data = BCG)
summary(metafor.BCG)
```

```
##    Study BCGTB BCGVacc NoVaccTB NoVacc Latitude Year BCGNeg NoVaccNeg
## 1      1     4     123       11    139       44 1948    119       128
## 2      2     6     306       29    303       55 1949    300       274
## 3      3     3     231       11    220       42 1960    228       209
## 4      4    62   13598      248  12867       52 1977  13536     12619
## 5      5    33    5069       47   5808       13 1973   5036      5761
## 6      6   180    1541      372   1451       44 1953   1361      1079
## 7      7     8    2545       10    629       19 1973   2537       619
## 8      8   505   88391      499  88391       13 1980  87886     87892
## 9      9    29    7499       45   7277       27 1968   7470      7232
## 10    10    17    1716       65   1665       42 1961   1699      1600
## 11    11   186   50634      141  27338       18 1974  50448     27197
## 12    12     5    2498        3   2341       33 1969   2493      2338
## 13    13    27   16913       29  17854       33 1976  16886     17825
##         yi     vi    sei       zi   ci.lb   ci.ub
## 1  -0.9387 0.3571 0.5976  -1.5708 -2.1100  0.2326
## 2  -1.6662 0.2081 0.4562  -3.6522 -2.5604 -0.7720
## 3  -1.3863 0.4334 0.6583  -2.1057 -2.6766 -0.0960
## 4  -1.4564 0.0203 0.1425 -10.2186 -1.7358 -1.1771
## 5  -0.2191 0.0520 0.2279  -0.9614 -0.6659  0.2276
## 6  -0.9581 0.0099 0.0995  -9.6269 -1.1532 -0.7631
## 7  -1.6338 0.2270 0.4765  -3.4290 -2.5676 -0.6999
## 8   0.0120 0.0040 0.0633   0.1899 -0.1120  0.1361
## 9  -0.4717 0.0570 0.2387  -1.9763 -0.9396 -0.0039
## 10 -1.4012 0.0754 0.2746  -5.1022 -1.9395 -0.8629
## 11 -0.3408 0.0125 0.1119  -3.0456 -0.5602 -0.1215
## 12  0.4466 0.5342 0.7309   0.6111 -0.9858  1.8791
## 13 -0.0173 0.0716 0.2676  -0.0648 -0.5419  0.5072
```


---

```r
metafor.BCG <- rma(method = "FE", measure = "OR", ai = BCGTB, bi = BCGNeg, ci = NoVaccTB, 
    di = NoVaccNeg, data = BCG)
summary(metafor.BCG)
```

```
## 
## Fixed-Effects Model (k = 13)
## 
##   logLik  deviance       AIC       BIC  
## -76.0290  163.1649  154.0580  154.6229  
## 
## Test for Heterogeneity: 
## Q(df = 12) = 163.1649, p-val < .0001
## 
## Model Results:
## 
## estimate       se     zval     pval    ci.lb    ci.ub          
##  -0.4361   0.0423 -10.3190   <.0001  -0.5190  -0.3533      *** 
## 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


---


```r
# CONVERT TO ODDS RATIO SCALE
exp(metafor.BCG$b)
```

```
##           [,1]
## intrcpt 0.6465
```

```r
exp(c(metafor.BCG$ci.lb, metafor.BCG$ci.ub))
```

```
## [1] 0.5951 0.7024
```

---


```r
metabin(event.e, n.e, event.c, n.c, studlab, data = NULL, subset = NULL, method = "MH", 
    sm = ifelse(!is.na(charmatch(method, c("Peto", "peto"), nomatch = NA)), 
        "OR", "RR"), incr = 0.5, allincr = FALSE, addincr = FALSE, allstudies = FALSE, 
    MH.exact = FALSE, RR.cochrane = FALSE, level = 0.95, level.comb = level, 
    comb.fixed = TRUE, comb.random = TRUE, hakn = FALSE, method.tau = "DL", 
    tau.preset = NULL, TE.tau = NULL, method.bias = NULL, title = "", complab = "", 
    outclab = "", label.e = "Experimental", label.c = "Control", label.left = "", 
    label.right = "", byvar, bylab, print.byvar = TRUE, print.CMH = FALSE, warn = TRUE)

```

---


```r
meta.BCG <- metabin(BCGTB, BCGVacc, NoVaccTB, NoVacc, data = BCG, method = "MH")
summary(meta.BCG)
```

```
## Number of studies combined: k=13
## 
##                         RR          95%-CI       z  p.value
## Fixed effect model   0.635  [0.588; 0.686] -11.534 < 0.0001
## Random effects model 0.490  [0.345; 0.695]  -3.992 < 0.0001
## 
## Quantifying heterogeneity:
## tau^2 = 0.3095; H = 3.57 [2.93; 4.34]; I^2 = 92.1% [88.3%; 94.7%]
## 
## Test of heterogeneity:
##       Q d.f.  p.value
##  152.57   12 < 0.0001
## 
## Details on meta-analytical method:
## - Mantel-Haenszel method
```

---

Resources
===

- http://cran.r-project.org/web/views/ClinicalTrials.html
- http://handbook.cochrane.org/


