---
layout: post
title: Building a simple word2vec model on OMIM database.
tags:
- deeplearning4j
- java
- omim

---

When reading about deep learning, I found the word2vec manuscript awesome. Its a relatively simple concept to transform a word by its context into a vector representation, but I was amazed that the mathematical distance between these vectors actually turned out to keep actual meaning.  

[word2vec manuscript](http://arxiv.org/pdf/1301.3781.pdf)

So, great. Now can we all agree its impressive that computers can learn how France->Paris is like Italy->Rome. but how useful is it if we give it a brief shot on medical genetic data?

I decided to make use of the NCBI OMIM database as a text corpus to build the word2vec model.  

> OMIM: Online Mendelian Inheritance In Man

> OMIM is a comprehensive, autorative compendium of human genes and genetic phenotypes that is freely available. Authored and edited by Institute of Genetic Medicine, Johns Hopkins University School of Medicine
>
> [www.ncbi.nlm.nih.gov/omim](www.ncbi.nlm.nih.gov/omim)

Willing to be productive as quick as possible, I decided to work with deeplearning4j as I am familiar with Java for the last 10 years. And I am pretty fond of spring-boot these days, so I could easily share the outcome of this experiment as a service in the future.


I first got up to speed with deeplearning4j by their tutorials on their home page, more specifically the one about [word2vec](http://deeplearning4j.org/word2vec.html).

Ok, so I downloaded the whole omim database, which is a 178MB txt file that I made available as a Java ClassPathResource which is fed into a SentenceIterator from deeplearning4j.

{% highlight sh %}
-rw-r--r--   1 kennyhelsens  staff   178M May 28 15:00 omim.txt
{% endhighlight %}

{% highlight java %}

ClassPathResource resource = new ClassPathResource("omim.txt");
SentenceIterator iter = new LineSentenceIterator(
        resource.getFile());
iter.setPreProcessor(new SentencePreProcessor() {
    @Override
    public String preProcess(String sentence) {
        return sentence.toLowerCase();
    }
});

{% endhighlight %}


Next, just following the instructions from deeplearning4j, I decided to make use of the default TokenizerFactory, and we're already fine to give it a first shot with a minimum configuration. (I'm running this during my daily train ride on my 3 year old macbook pro.)


{% highlight java %}

TokenizerFactory t = new DefaultTokenizerFactory();

Word2Vec vec = new Word2Vec.Builder().
    sampling(1e-5).
    minWordFrequency(20).
    batchSize(1000).
    useAdaGrad(false).
    layerSize(100).
    iterations(1).
    learningRate(0.05).
    minLearningRate(3e-2).
    negativeSample(10).
    iterate(iter).
    tokenizerFactory(t).
    build();

vec.fit();

SerializationUtils.saveObject(vec, new File("vec-omim.ser"));

{% endhighlight %}

So what does this configuration mean?

* We set minWordFrequency to 20 to leave out very sparse words.
* I've decreased iterations and increased the minLearningRate a little bit to make faster progress. I don't intend to write an academic paper here, just looking for some low hanging fruit.
* layerSize was also decreased to 100 instead of the 300 from the word2vec manuscript, also for time considerations. A 100 dimensional vector for a word still feels like a whole lot.


In the end, I'm serializing the Word2Vec object to disk, such that I can play a bit with it afterwards without retraining over and over again.


So in another Java class I deserialze the file, and make use of the wordsNearest methods on the Word2Vec instance.


{% highlight java %}
Word2Vec vec =
     SerializationUtils.readObject(new File("vec-omim.ser"));


 similar = vec.wordsNearest("alk");
 System.out.println("alk" + ":" + similar);
{% endhighlight %}


Term  | word2vec similar terms
------------- | -------------
alk  | nonsmall, carcinomas, carcinoma, mapdkd


Wow. This is pretty cool. Alk is a known oncogene in non-small cell lung carcinoma.


Lets try a few more.

Term  | word2vec similar terms
------------- | -------------
invasion  | metastasis, adhesion, invasiveness, migration, factor-alpha, colony, anchorage-independent, tumorigenicity, proliferation
angiogenesis | adhesion, apoptosis, proliferation, migration, invasion, tnf, healing, anchorage-independent, factor-alpha
signaling | mapkd, tgfbd, mapkdd, shh, erkd, cdknda, wnt, nfkb, erk
aneurysm | valve, arteriosu, ductus, dissection
telomere | break, fork, systolic, diastolic, clavicles, telomerase
lyme | erythematosus, rheum, allergy
brca | breast-ovarian, nonsmall, carcinoma, cancers, nonpolyposi
alzheimer | ad, pd, crohn, parkinson, celiac


* *signaling* is associated with a list of critical pathway genes
* *brca* gene has well known mutation driving breast and ovarian and cancer. As expected, but still very clever of the word2vec model.
* *telomere* is associated with systolic and diastolic, I do not fully grasp these associations. Telomerase is clear though.
* *angiogenesis* is associated with adhesion, migration, invasion, healing, tnf. Again very strong of the model.


**Conclusion I  
It seems like the word2vec model, far from optimally trained on my macbook, did learn to make quite a few good associations from the model.**



Lets do some negative testing with nonsense words.



Term  | word2vec similar terms
------------- | -------------
kenny  | harano, moo-penn, hamel, stevenson, male
university  | pp, press, ed.), (pub.)
the  | /
why  | /


So my first name is associated with some authors, and *university* is shared with press and other. As expected, *the* and *why* which occur at random, don't return any associtions. Great, its pretty good.


Finally, the word2vec examples are known for their analogies. France is to Paris, what Italy is to X. Word2vec can fill in Rome here by crunching Wikipedia. So can we try to find analogous terms for genotype-phenotype associations?



Positive Terms  | Negative Terms | Word2vec analogous Terms
------------- | ---------|----
+brca  +breast  | -alk | nonpolyposis, carcinoma, nonsmall, squamous, colorectal


Once again very impressive. The addition of the breast vector and negation of the alk vector, yields a vector nearby 'colorectal'. Indeed, in a cancer setting, brca means to breast what alk means to colorectal.



Any remarks or suggestions, let me know below!
