---
layout: post
title: Testing randomForest and  H2O deep learning
---


I have been wondering what all the noise about deep learning is about.
Its still neural networks, right? I have had not so much experience with
NN because they're supposed to be hard to get right due to paramater
tuning, which is a downer if you're used to good alround performers like
random forests. Still I decided to set out on a series of blogposts
using h2o (R) and deeplearning4j (R) on biotech datasets.

We'll be working with the BreastCancer dataset from the mlbench package.
From the package description:

> The objective is to identify each of a number of benign or malignant
> classes. Samples arrive periodically as Dr. Wolberg reports his
> clinical cases. The database therefore reflects this chronological
> grouping of the data. This grouping information appears immediately
> below, having been removed from the data itself. Each variable except
> for the first was converted into 11 primitive numerical attributes
> with values ranging from 0 through 10. There are 16 missing attribute
> values. See cited below for more details.

{% highlight r %}

data("BreastCancer", package = "mlbench")

{% endhighlight %}


Let's do some data munging.

{% highlight r %}

BreastCancer %<>% as.data.table
# remove NA values for simplicity
BreastCancer %<>% na.omit

y <- BreastCancer$Class %>% as.character() %>% as.factor
x <- BreastCancer %>% select(2:(NCOL(.)-1))


# get all nominal values as numeric
x <- apply(x, 2, as.numeric) %>% data.table

{% endhighlight %}

Prepare test/training splits.

{% highlight r %}
# split the data in test/train
set.seed(10000)
splits <-
  sample(x = c("train","test"),
         size = NROW(x),
         replace = T,
         prob = c(0.7,0.3))
features <- split(x, splits)
response <- split(y, splits)
{% endhighlight %}

As a reference, how good does it get with minimum effort using random forest classification?
--------------------------------------------------------------------------------------------
{% highlight r %}
rf <- randomForest(x = features$train,
                   y = as.factor(response$train),
                   xtest = features$test,
                   ytest = response$test,
                   keep.forest = TRUE,
                   proximity = TRUE)
rf
{% endhighlight %}


    ##
    ## Call:
    ##  randomForest(x = features$train, y = as.factor(response$train),      xtest = features$test, ytest = response$test, proximity = TRUE,      keep.forest = TRUE)
    ##                Type of random forest: classification
    ##                      Number of trees: 500
    ## No. of variables tried at each split: 3
    ##
    ##         OOB estimate of  error rate: 3.21%
    ## Confusion matrix:
    ##           benign malignant class.error
    ## benign       294         8  0.02649007
    ## malignant      7       158  0.04242424
    ##                 Test set error rate: 2.31%
    ## Confusion matrix:
    ##           benign malignant class.error
    ## benign       139         3  0.02112676
    ## malignant      2        72  0.02702703



Test set error rate is at 2.31% without a lot of effort. There is not a
lot of room for improvement. (So maybe this is not the best dataset.)
One of the nice things about randomforests is that they're easy to understand by looking at the variable importance plot.

{% highlight r %}
varImpPlot(rf)
{% endhighlight %}

![](/assets/2015-06-04-hands-on-h2o-deeplearning_files/figure-markdown_strict/unnamed-chunk-6-1.png)

Demonstrates as expected that cell size and shape are most predictive
features for the breast cancer RF classifier. We can inspect per feature
decision surfaces, as plotted below where malignant weight increases
with higher value of cell size.

{% highlight r %}
randomForest::partialPlot(
  rf,
  response = features$train,
  features = "Cell.size",
  main = "",
  which.class = "malignant")
{% endhighlight %}

![](/assets/2015-06-04-hands-on-h2o-deeplearning_files/figure-markdown_strict/unnamed-chunk-7-1.png)



So how good does it get using h2o deeplearning without much finetuning?
-----------------------------------------------------------------------

Before this analysis, I had already setup the h2o R package. Instructions for running h2o are nicely summarized [here](http://tjo-en.hatenablog.com/entry/2015/02/15/194003). So I can
now simply fire up a local instance for testing with the following
command.

{% highlight r %}
require(h2o)
instance <- h2o.init()
{% endhighlight %}

Lets load the training and test data into h2o.

{% highlight r %}
h2orefs <- list()
h2orefs$train <- as.h2o(instance, cbind(features$train, Class=response$train))

h2orefs$test <- as.h2o(instance, cbind(features$test, Class=response$test))
{% endhighlight %}


Now build a model with default parameters. With h2o, you have to
specify predictor and response variables by column index or by column
name. Here, we are using column names. (Have a look at the magrittr R package if you're confused by the '%>%' operator.)

{% highlight r %}
h2orefs$model <-
  h2o.deeplearning(x = features$train %>% colnames,
                 y = "Class",
                 data = h2orefs$train,
                 validation = h2orefs$test,
                 classification = TRUE)


h2orefs$model
{% endhighlight %}

    ## Deep Learning Model Key: DeepLearning_824431d8d17d566b8e213c76233147cf
    ##
    ## Training classification error: 0.0235546
    ##
    ## Validation classification error: 0.01388889
    ##
    ## Confusion matrix:
    ## Reported on Last.value.1
    ##            Predicted
    ## Actual      benign malignant   Error
    ##   benign       139         3 0.02113
    ##   malignant      0        74 0.00000
    ##   Totals       139        77 0.01389
    ##
    ## AUC =  0.9942901 (on validation)

Clearly better than the RF, out of the box without many finetuning. The overall error rate dropped to 1.38%, but more importantly the sensitivity detecting malignant cases went up to 100%.

Lets try and turn some knobs and evaluate what happens.

### Adding more layers and neurons?

{% highlight r %}
h2orefs$model <-
  h2o.deeplearning(x = features$train %>% colnames,
                 y = "Class",
                 data = h2orefs$train,
                 validation = h2orefs$test,
                 classification = TRUE,
                 hidden = c(1000, 500, 300))

h2orefs$model
{% endhighlight %}

    ## Deep Learning Model Key: DeepLearning_aa147b54007becb9e8434eb8100a4ab9
    ##
    ## Training classification error: 0.02141328
    ##
    ## Validation classification error: 0.01388889
    ##
    ## Confusion matrix:
    ## Reported on Last.value.1
    ##            Predicted
    ## Actual      benign malignant   Error
    ##   benign       139         3 0.02113
    ##   malignant      0        74 0.00000
    ##   Totals       139        77 0.01389
    ##
    ## AUC =  0.9943852 (on validation)

Not much effect here.

### Decrease the number of neurons, and add some regularization?

{% highlight r %}
h2orefs$model <-
  h2o.deeplearning(x = features$train %>% colnames,
                 y = "Class",
                 data = h2orefs$train,
                 validation = h2orefs$test,
                 classification = TRUE,
                 hidden = c(10, 20, 30),
                 input_dropout_ratio = 0.3,
                 hidden_dropout_ratios = c(0.3,0.3,0.3),
                 epochs = 50,
                 l1 = 0.0005,
                 l2 = 0.0005)

h2orefs$model
{% endhighlight %}


    ## Deep Learning Model Key: DeepLearning_ae8df876b491d8bdbaf64926c7b64c59
    ##
    ## Training classification error: 0.0235546
    ##
    ## Validation classification error: 0.01851852
    ##
    ## Confusion matrix:
    ## Reported on Last.value.1
    ##            Predicted
    ## Actual      benign malignant   Error
    ##   benign       138         4 0.02817
    ##   malignant      0        74 0.00000
    ##   Totals       138        78 0.01852
    ##
    ## AUC =  0.9943852 (on validation)

Not much effect. It would be interesting to whether the false positives
evolved into malignant tissue after the data was collected for this
study.


Now lets have a look at the variable importance derived from the DL classifier.

{% highlight r %}
h2orefs$model <-
  h2o.deeplearning(x = features$train %>% colnames,
                 y = "Class",
                 data = h2orefs$train,
                 validation = h2orefs$test,
                 classification = TRUE,
                 variable_importances = TRUE)


varimp<-
  t(data.frame(h2orefs$model@model$varimp)) %>%
  data.table(feature = rownames(.), importance = .[,1]) %>%
  arrange(importance)

varimp$feature %<>%
  factor(., levels = unique(.))
{% endhighlight %}


{% highlight r %}
p <- ggplot(data = varimp, aes(x = feature, y = importance))
p <- p + geom_bar(stat = "identity")
p <- p + coord_flip()
p
{% endhighlight %}

![](/assets/2015-06-04-hands-on-h2o-deeplearning_files/figure-markdown_strict/unnamed-chunk-14-1.png)


So while cell shape was determining the RF a lot, its of minor
importance in the DL model. And while the mitoses did not contribute at
all to the RF, it has a lot of importance in the DL model. Actually all of the features seem to be used by the DL model, so is it making better use of all available information?
