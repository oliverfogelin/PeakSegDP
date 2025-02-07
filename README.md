
<!-- README.md is generated from README.Rmd. Please edit that file -->

# PeakSegDP

<!-- badges: start -->

<!-- badges: end -->

PeakSeg is a constrained maximum Poisson likelihood segmentation model.
This is described in [PeakSeg: constrained optimal segmentation and
supervised penalty learning for peak detection in count
data](http://jmlr.org/proceedings/papers/v37/hocking15.html)
([source](https://github.com/tdhock/PeakSeg-paper)). We proposed a
[constrained Dynamic Programming Algorithm](file:src/cDPA.c) (cDPA) for
computing a model that satisfies the PeakSeg constraints.

## Installation

You can install the released version of PeakSegDP from
[CRAN](https://CRAN.R-project.org) with:

``` r
install.packages("PeakSegDP")
```

And the development version from [GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("tdhock/PeakSegDP")
```

## Usage

There are two main functions for computing the constrained segmentation
model:

  - [cDPA](file:R/cDPA.R) is a low-level interface to [the C
    solver](file:src/cDPA.c). Its inputs are N weighted data points and
    S, the maximum number of segments. It outputs a list with components
    named `loss`, `ends`, and `mean` (S x N matrices describing the
    solution).
  - [PeakSegDP](file:R/PeakSegDP.R) is a more user-friendly wrapper of
    the cDPA. Its input parameter is P, the maximum number of peaks,
    which implies S = P\*2+1. Its input data type is a data.frame with
    columns `count`, `chromStart`, `chromEnd`. It outputs a list of
    data.frames, `peaks`, `error`, `segments`, `breaks`.

## Example

This is an example plotting the fitting of PeakSegDP():

``` r
library(PeakSegDP)
library(ggplot2)
data(chr11ChIPseq, envir=environment())
one <- subset(chr11ChIPseq$coverage, sample.id=="McGill0002")[10000:12000,]
fit <- PeakSegDP(one, 3L)

ggplot()+
  geom_step(aes(chromStart/1e3, count), data=one)+
  geom_segment(aes(chromStart/1e3, mean,
                   xend=chromEnd/1e3, yend=mean),
               data=fit$segments, color="green")+
  geom_segment(aes(chromStart/1e3, 0,
                   xend=chromEnd/1e3, yend=0),
               data=subset(fit$segments, status=="peak"),
               size=3, color="deepskyblue")+
  theme_bw()+
  theme(panel.spacing=grid::unit(0, "cm"))+
  facet_grid(peaks ~ ., scales="free", labeller=function(df){
    s <- ifelse(df$peaks==1, "", "s")
    df$peaks <- paste0(df$peaks, " peak", s)
    df
  })
```

<img src="man/figures/README-unnamed-chunk-2-1.png" width="100%" />

## Related work

  - As explained in our ICML paper, the cDPA is a quadratic time
    algorithm that is not guaranteed to find the global optimum. For a
    linear time algorithm that recovers the global optimum, use the
    [coseg](https://github.com/tdhock/coseg) package.
  - For supervised peak detection in ChIP-seq data sets with several
    samples, see our newer method,
    [PeakSegJoint](https://github.com/tdhock/PeakSegJoint).
