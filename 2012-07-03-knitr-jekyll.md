---
layout: post
title: "Blog with Knitr and Jekyll"
description: ""
category: r
tags: [knitr, jekyll, tutorial]
---
{% include JB/setup %}

The [knitr](http://yihui.name/knitr/) package provides an easy way to embed 
[R](http://www.r-project.org/) code in a [Jekyll-Bootstrap](http://jekyllbootstrap.com/) 
blog post. The only required input is an **R Markdown** source file. 
The name of the source file used to generate this post is *2012-07-03-knitr-jekyll.Rmd*, available
[here](https://github.com/jfisher-usgs/jfisher-usgs.github.com/blob/master/Rmd/2012-07-03-knitr-jekyll.Rmd).
Steps taken to build this post are as follows:

### Step 1

Create a Jekyll-Boostrap blog if you don't already have one. 
A brief tutorial on building this blog is available 
[here](/lessons/2012/05/30/jekyll-build-on-windows/).

### Step 2

Open the R Console and process the source file:


{% highlight r %}
KnitPost <- function(input, base.url = "/") {
    require(knitr)
    opts_knit$set(base.url = base.url)
    fig.path <- paste0("figs/", sub(".Rmd$", "", basename(input)), "/")
    opts_chunk$set(fig.path = fig.path)
    opts_chunk$set(fig.cap = "center")
    render_jekyll()
    knit(input, envir = parent.frame())
}
KnitPost("2012-07-03-knitr-jekyll.Rmd")
{% endhighlight %}




### Step 3

Move the resulting image folder *2012-07-03-knitr-jekyll* and **Markdown** file 
*2012-07-03-knitr-jekyll.md* to the local 
*jfisher-usgs.github.com* [git](http://git-scm.com/) repository.
The KnitPost function assumes that the image folder will be placed in a 
[figs](https://github.com/jfisher-usgs/jfisher-usgs.github.com/tree/master/figs) 
folder located at the root of the repository.

### Step 4

Add the following CSS code to the 
*/assets/themes/twitter-2.0/css/bootstrap.min.css* file to center images:

    [alt=center] {
      display: block;
      margin: auto;
    }

That's it.

***

Here are a few examples of embedding R code:


{% highlight r %}
summary(cars)
{% endhighlight %}



{% highlight text %}
##      speed           dist    
##  Min.   : 4.0   Min.   :  2  
##  1st Qu.:12.0   1st Qu.: 26  
##  Median :15.0   Median : 36  
##  Mean   :15.4   Mean   : 43  
##  3rd Qu.:19.0   3rd Qu.: 56  
##  Max.   :25.0   Max.   :120  
{% endhighlight %}






{% highlight r %}
par(mar = c(4, 4, 0.1, 0.1), omi = c(0, 0, 0, 0))
plot(cars)
{% endhighlight %}

![center](/figs/2012-07-03-knitr-jekyll/fig1.png) 

##### Figure 1: Caption



{% highlight r %}
par(mar = c(2.5, 2.5, 0.5, 0.1), omi = c(0, 0, 0, 0))
filled.contour(volcano)
{% endhighlight %}

![center](/figs/2012-07-03-knitr-jekyll/fig2.png) 

##### Figure 2: Caption

And don't forget your session information for proper reproducible research.


{% highlight r %}
sessionInfo()
{% endhighlight %}



{% highlight text %}
## R version 2.15.0 (2012-03-30)
## Platform: x86_64-apple-darwin9.8.0/x86_64 (64-bit)
## 
## locale:
## [1] C/en_US.UTF-8/C/C/C/C
## 
## attached base packages:
## [1] splines   grid      stats     graphics  grDevices utils     datasets 
## [8] methods   base     
## 
## other attached packages:
##  [1] knitr_0.6.2          xtable_1.7-0         entropy_1.1.7       
##  [4] Hmisc_3.9-3          survival_2.36-12     gplots_2.10.1       
##  [7] KernSmooth_2.23-7    caTools_1.13         bitops_1.0-4.1      
## [10] gdata_2.8.2          gtools_2.6.2         ggplot2_0.9.1       
## [13] reshape_0.8.4        plyr_1.7.1           rJava_0.9-3         
## [16] arulesViz_0.1-5      igraph_0.5.5-4       seriation_1.0-6     
## [19] gclus_1.3            TSP_1.0-6            cluster_1.14.2      
## [22] vcd_1.2-13           colorspace_1.1-1     scatterplot3d_0.3-33
## [25] MASS_7.3-17          arules_1.0-8         Matrix_1.0-6        
## [28] lattice_0.20-6       foreign_0.8-50      
## 
## loaded via a namespace (and not attached):
##  [1] RColorBrewer_1.0-5 Rcpp_0.9.10        codetools_0.2-8   
##  [4] dichromat_1.2-4    digest_0.5.2       evaluate_0.4.2    
##  [7] formatR_0.4        highlight_0.3.1    labeling_0.1      
## [10] memoise_0.1        munsell_0.3        parser_0.0-14     
## [13] proto_0.3-9.2      reshape2_1.2.1     scales_0.2.1      
## [16] stringr_0.6        tools_2.15.0      
{% endhighlight %}



