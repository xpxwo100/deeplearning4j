---
title: 
layout: default
---

# Word2Vec

Contents

* <a href="#intro">Introduction</a>
* <a href="#embed">Neural Word Embeddings</a>
* <a href="#just">**Just Give Me the Code**</a>
* <a href="#anatomy">Anatomy of Word2Vec</a>
* <a href="#setup">Setup, Load and Train</a>
* <a href="#code">A Code Example</a>
* <a href="#trouble">Troubleshooting & Tuning Word2Vec</a>
* <a href="#use">Word2vec Use Cases</a>
* <a href="#next">Next Steps</a>
* <a href="#patent">Word2vec Patent</a>
* <a href="#foreign">Foreign Languages</a>

##<a name="intro">Introduction to Word2Vec</a>

Word2vec is a two-layer neural net that processes text. Its input is a text corpus and its output is a set of vectors: feature vectors for words in that corpus. While Word2vec is not a [deep neural network](../neuralnet-overview.html), it turns text into a numerical form that deep nets can understand. 

Word2vec's applications extend beyond parsing sentences in the wild. It can be applied just as well to playlists, social media graphs and other verbal series in which patterns may be discerned. [Deeplearning4j](http://deeplearning4j.org/quickstart.html) implements a distributed form of Word2vec for Java and Scala, which works on Spark with GPUs. 

The purpose and usefulness of Word2vec is to group the vectors of similar words together in vectorspace. That is, it detects similarities mathematically. Word2vec creates vectors that are distributed numerical representations of word features, features such as the context of individual words. It does so without human intervention. 

Given enough data, usage and contexts, Word2vec can make highly accurate guesses about a word’s meaning based on past appearances. Those guesses can be used to establish a word's association with other words (e.g. "man" is to "boy" what "woman" is to "girl"), or cluster documents and classify them by topic. Those clusters can form the basis of search, sentiment analysis and recommendations in such diverse fields as scientific research, legal discovery, e-commerce and customer relationship management. 

The output of the Word2vec neural net is a vocabulary in which each item has a vector attached to it, which can be fed into a deep-learning net or simply queried to detect relationships between words. 

Broadly speaking, we measure words' proximity to each other through their cosine similarity, which gauges the distance/dissimilarity between two word vectors. A perfect 90-degree angle represents identity; i.e. Sweden equals Sweden, while Norway has a cosine distance of 0.760124 from Sweden, the highest of any other country. 

Here's a list of words associated with "Sweden" using Word2vec, in order of proximity:

![Alt text](../img/sweden_cosine_distance.png) 

The nations of Scandinavia and several wealthy, northern European, Germanic countries are among the top nine. 

##<a name="embed">Neural Word Embeddings</a>

The vectors we use to represent words are called *neural word embeddings*, and representations are strange. One thing describes another, even though those two things are radically different. As Elvis Costello said: "Writing about music is like dancing about architecture." Word2vec "vectorizes" about words, and by doing so it makes natural language computer-readable -- we can start to perform powerful mathematical operations on words to detect their similarities. 

So a neural word embedding represents a word with numbers. It's a simple, yet unlikely, translation. Word2vec is similar to an autoencoder, encoding each word in a vector, but rather than training against the input words through reconstruction, as a [restricted Boltzmann machine](../restrictedboltzmannmachine.html) does, word2vec trains words against other words that neighbor them in the input corpus. 

Just as Van Gogh's painting of sunflowers is a two-dimensional mixture of oil on canvas that *represents* vegetable matter in a three-dimensional space in Paris in the late 1880s, so 500 numbers arranged in a vector can represent a word or group of words.

Those numbers locate each word as a point in 500-dimensional vectorspace. (Geoff Hinton, teaching people to imagine 13-dimensional space, suggests that students first picture 3-dimensional space and then say to themselves: "Thirteen, thirteen, thirteen." :) 

A well trained set of word vectors will place similar words close to each other in that space. The words *oak*, *elm* and *birch* might cluster in one corner, while *war*, *conflict* and *strife* huddle together in another. 

Similar things and ideas are shown to be "close". Their relative meanings have been translated to measurable distances. Qualities become quantities, and algorithms can do their work. But similarity is just the basis of many associations that Word2vec can learn. For example, it can gauge relations between words of one language, and map them to another.

![Alt text](../img/word2vec_translation.png) 

These vectors are the basis of a more comprehensive geometry of words. Not only will Rome, Paris, Berlin and Beijing cluster near each other, but they will each have similar distances in vectorspace to the countries whose capitals they are; i.e. Rome - Italy = Beijing - China. And if you only knew that Rome was the capital of Italy, and were wondering about the capital of China, then the equation Rome -Italy + China would return Beijing. No kidding. 

![Alt text](../img/countries_capitals.png) 

Let's imagine some other associations:

* Gender: *King - Queen = Man - Woman*
* Filiation: *George H. W. Bush - George W. Bush = John Adams - John Quincy Adams.*
* Geopolitics: *Iraq - Violence = Jordan*
* Distinction: *Human - Animal = Ethics*
* *President - Power = Prime Minister*
* *Library - Books = Hall*
* Analogy: *Stock Market ≈ Thermometer*

By building a sense of one word's proximity to other similar words, which do not necessarily contain the same letters, we have moved beyond hard tokens to a smoother and more general sense of meaning. 

![Alt text](../img/man_woman_king_queen.png) 

# <a name="just">Just Give Me the Code</a>

##<a name="anatomy">Anatomy of Word2vec in DL4J</a>

Here are Deeplearning4j's natural-language processing components:

* **SentenceIterator/DocumentIterator**: Used to iterate over a dataset. A SentenceIterator returns strings and a DocumentIterator works with inputstreams. Use the SentenceIterator wherever possible.
* **Tokenizer/TokenizerFactory**: Used in tokenizing the text. In NLP terms, a sentence is represented as a series of tokens. A TokenizerFactory creates an instance of a tokenizer for a "sentence." 
* **VocabCache**: Used for tracking metadata including word counts, document occurrences, the set of tokens (not vocab in this case, but rather tokens that have occurred), vocab (the features included in both bag of words as well as the word vector lookup table)
* **Inverted Index**: Stores metadata about where words occurred. Can be used for understanding the dataset. A Lucene index with the Lucene implementation[1] is automatically created. 

While Word2vec refers to a family of related algorithms, this implementation uses <a href="../glossary.html#skipgram">Skip-Gram</a> Negative Sampling.

## <a name="setup">Word2Vec Setup</a> 

Create a new project in IntelliJ using Maven. If you don't know how to do that, see our [Quickstart page](../quickstart.html). Then specify these properties and dependencies in the POM.xml file in your project's root directory.

                <properties>
                  <nd4j.version>0.0.3.5.5.3</nd4j.version> // check Maven Central for latest versions
                  <dl4j.version>0.0.3.3.3.alpha1</dl4j.version>
                </properties>
                
                <dependencies>
                  <dependency>
                     <groupId>org.deeplearning4j</groupId>
                     <artifactId>deeplearning4j-ui</artifactId>
                     <version>${dl4j.version}</version>
                   </dependency>
                   <dependency>
                     <groupId>org.deeplearning4j</groupId>
                     <artifactId>deeplearning4j-nlp</artifactId>
                     <version>${dl4j.version}</version>
                   </dependency>
                   <dependency>
                     <groupId>org.nd4j</groupId>
                     <artifactId>nd4j-jblas</artifactId> //you can choose different backends
                     <version>${nd4j.version}</version>
                   </dependency>
                </dependencies>

### Loading Data

Now create and name a new class in Java. After that, you'll take the raw sentences in your .txt file, traverse them with your iterator, and subject them to some sort of preprocessing, such as converting all words to lowercase. 

        log.info("Load data....");
        ClassPathResource resource = new ClassPathResource("raw_sentences.txt");
        SentenceIterator iter = new LineSentenceIterator(resource.getFile());
        iter.setPreProcessor(new SentencePreProcessor() {
            @Override
            public String preProcess(String sentence) {
                return sentence.toLowerCase();
            }
        });

If you want to load a text file besides the sentences provided in our example, you'd do this:

        log.info("Load data....");
        SentenceIterator iter = new LineSentenceIterator(new File("/Users/cvn/Desktop/file.txt"));
        iter.setPreProcessor(new SentencePreProcessor() {
            @Override
            public String preProcess(String sentence) {
                return sentence.toLowerCase();
            }
        });

That is, get rid of the `ClassPathResource` and feed the absolute path of your `.txt` file into the `LineSentenceIterator`. 

        SentenceIterator iter = new LineSentenceIterator(new File("/your/absolute/file/path/here.txt"));

In bash, you can find the absolute file path of any directory by typing `pwd` in your command line from within that same directory. To that path, you'll add the file name and *voila*. 

### Tokenizing the Data

Word2vec needs to be fed words rather than whole sentences, so the next step is to tokenize the data. To tokenize a text is to break it up into its atomic units, creating a new token each time you hit a white space, for example. 

        log.info("Tokenize data....");
        final EndingPreProcessor preProcessor = new EndingPreProcessor();
        TokenizerFactory tokenizer = new DefaultTokenizerFactory();
        tokenizer.setTokenPreProcessor(new TokenPreProcess() {
            @Override
            public String preProcess(String token) {
                token = token.toLowerCase();
                String base = preProcessor.preProcess(token);
                base = base.replaceAll("\\d", "d");
                if (base.endsWith("ly") || base.endsWith("ing"))
                    System.out.println();
                return base;
            }
        });

That should give you one word per line. 

### Training the Model

Now that the data is ready, you can configure the Word2vec neural net and feed in the tokens. 

        int batchSize = 1000;
        int iterations = 30;
        int layerSize = 300;
        
        log.info("Build model....");
        Word2Vec vec = new Word2Vec.Builder()
                .batchSize(batchSize) //# words per minibatch. 
                .sampling(1e-5) // negative sampling. drops words out
                .minWordFrequency(5) // 
                .useAdaGrad(false) //
                .layerSize(layerSize) // word feature vector size
                .iterations(iterations) // # iterations to train
                .learningRate(0.025) // 
                .minLearningRate(1e-2) // learning rate decays wrt # words. floor learning
                .negativeSample(10) // sample size 10 words
                .iterate(iter) //
                .tokenizerFactory(tokenizer)
                .build();
        vec.fit();

This configuration accepts a number of hyperparameters. A few require some explanation: 

* *batchSize* is the amount of words you process at a time. 
* *minWordFrequency* is the minimum number of times a word must appear in the corpus. Here, if it appears less than 5 times, it is not learned. Words must appear in multiple contexts to learn useful features about them. In very large corpora, it's reasonable to raise the minimum.
* *useAdaGrad* - Adagrad creates a different gradient for each feature. Here we are not concerned with that. 
* *layerSize* specifies the number of features in the word vector. This is equal to the number of dimensions in the featurespace. Words represented by 500 features become points in a 500-dimensional space.
* *iterations* this is the number of times you allow the net to update its coefficients for one batch of the data. Too few iterations mean it many not have time to learn all it can; too many will make the net's training longer.
* *learningRate* is the step size for each update of the coefficients, as words are repositioned in the feature space. 
* *minLearningRate* is the floor on the learning rate. Learning rate decays as the number of words you train on decreases. If learning rate shrinks too much, the net's learning is no longer efficient. This keeps the coefficients moving. 
* *iterate* tells the net what batch of the dataset it's training on. 
* *tokenizer* feeds it the words from the current batch. 
* *vec.fit()* tells the configured net to begin training. 

### Evaluating the Model

The next step is to evaluate the quality of your feature vectors. 

        log.info("Evaluate model....");
        double sim = vec.similarity("people", "money");
        log.info("Similarity between people and money: " + sim);
        Collection<String> similar = vec.wordsNearest("day", 10);
        log.info("Similar words to 'day' : " + similar);
        
        //output: [night, week, year, game, season, during, office, until, -]

The line `vec.similarity("word1","word2")` will return the cosine similarity of the two words you enter. The closer it is to 1, the more similar the net perceives those words to be (see the Sweden-Norway example above). For example:

        double cosSim = vec.similarity("day", "night");
        System.out.println(cosSim);
        //output: 0.7704452276229858

With `vec.wordsNearest("word1", numWordsNearest)`, the words printed to the screen allow you to eyeball whether the net has clustered semantically similar words. You can set the number of nearest words you want with the second parameter of wordsNearest. For example:

        Collection<String> lst3 = vec.wordsNearest("man", 10);
        System.out.println(lst3);
        //output: [director, company, program, former, university, family, group, such, general]

### Visualizing the Model

We rely on [TSNE](https://lvdmaaten.github.io/tsne/) to reduce the dimensionality of word feature vectors and project words into a two or three-dimensional space. 

        log.info("Plot TSNE....");
        BarnesHutTsne tsne = new BarnesHutTsne.Builder()
                .setMaxIter(1000)
                .stopLyingIteration(250)
                .learningRate(500)
                .useAdaGrad(false)
                .theta(0.5)
                .setMomentum(0.5)
                .normalize(true)
                .usePca(false)
                .build();
        vec.lookupTable().plotVocab(tsne);

### Saving, Reloading & Using the Model

You'll want to save the model. The normal way to save models in Deeplearning4j is via the serialization utils (Java serialization is akin to Python pickling, converting an object into a *series* of bytes).

        log.info("Save vectors....");
        WordVectorSerializer.writeWordVectors(vec, "words.txt");

This will save the vectors to a file called `words.txt` that will appear in the root of the directory where Word2vec is trained. The output in the file should one word per line, followed by a series of numbers that together are its vector representation.

To keep working with the vectors, simply call methods on `vec` like this:

        Collection<String> kingList = vec.wordsNearest(Arrays.asList("king", "woman"), Arrays.asList("queen"), 10);

The classic example of Word2vec's arithmetic of words is "king - queen = man - woman" and its logical extension "king - queen + woman = man". 

The example above will output the 10 nearest words to the vector `king - queen + woman`, which should include `man`. The first parameter for wordsNearest has to include the "positive" words `king` and `woman`; the second parameter includes the negative word `queen`; the third is the length of the list of nearest words you would like to see. Remember to add this to the top of the file: `import java.util.Arrays;`

Any number of combinations is possible, but they will only return sensible results if the words you query occurred with enough frequency in the corpus. Obviously, the ability to return similar words (or documents) is at the foundation of both search and recommendation engines. 

You can reload the vectors into memory like this:

        WordVectors wordVectors = WordVectorSerializer.loadTxtVectors(new File("words.txt"));

You can then use Word2vec as a lookup table:

        WeightLookupTable weightLookupTable = wordVectors.lookupTable();
        Iterator<INDArray> vectors = weightLookupTable.vectors();
        INDArray wordVector = vectors.getWordVectorMatrix("myword");
        double[] wordVector = vectors.getWordVector("myword");

If the word isn't in the vocabulary, Word2vec returns zeros.

### Importing Word2vec Models

The [Google News Corpus model](https://s3.amazonaws.com/dl4j-distribution/GoogleNews-vectors-negative300.bin.gz) we use to test the accuracy of our trained nets is hosted on S3. Users whose current hardware takes a long time to train on large corpora can simply download it to explore a Word2vec model without the prelude.

If you trained with the [C vectors](https://docs.google.com/file/d/0B7XkCwpI5KDYaDBDQm1tZGNDRHc/edit) or Gensimm, this line will import the model.

    File gModel = new File("/Developer/Vector Models/GoogleNews-vectors-negative300.bin.gz");
    Word2Vec vec = WordVectorSerializer.loadGoogleModel(gModel, true);

Remember to add `import java.io.File;` to your imported packages.

With large models, you may run into trouble with your heap space. The Google model may take as much as 10G of RAM, and the JVM only launches with 256 MB of RAM, so you have to adjust your heap space. You can do that either with a `bash_profile` file (see our [Troubleshooting section](../gettingstarted.html#trouble)), or through IntelliJ itself: 

    //Click:
    IntelliJ Preferences > Compiler > Command Line Options 
    //Then paste:
    -Xms1024m
    -Xmx10g
    -XX:MaxPermSize=2g

### <a name="grams">N-grams & Skip-grams</a>

Words are read into the vector one at a time, *and scanned back and forth within a certain range*. Those ranges are n-grams, and an n-gram is a contiguous sequence of *n* items from a given linguistic sequence; it is the nth version of unigram, bigram, trigram, four-gram or five-gram. A skip-gram simply drops items from the n-gram. 

The skip-gram representation popularized by Mikolov and used in the DL4J implementation has proven to be more accurate than other models, such as continuous bag of words, due to the more generalizable contexts generated. 

This n-gram is then fed into a neural network to learn the significance of a given word vector; i.e. significance is defined as its usefulness as an indicator of certain larger meanings, or labels. 

### <a name="code">A Working Example</a>

Now that you have a basic idea of how to set up Word2Vec, here's [one example](https://github.com/deeplearning4j/dl4j-0.0.3.3-examples/blob/master/src/main/java/org/deeplearning4j/word2vec/Word2VecRawTextExample.java) of how it can be used with DL4J's API:

<script src="http://gist-it.appspot.com/https://github.com/deeplearning4j/dl4j-0.0.3.3-examples/blob/master/src/main/java/org/deeplearning4j/word2vec/Word2VecRawTextExample.java?slice=28:97"></script>

After following the instructions in the [Quickstart](../quickstart.html), you can open this example in IntelliJ and hit run to see it work. If you query the Word2vec model with a word isn't contained in the training corpus, it will return null. 

### <a name="trouble">Troubleshooting & Tuning Word2Vec</a>

*Q: I get a lot of stack traces like this*

       java.lang.StackOverflowError: null
       at java.lang.ref.Reference.<init>(Reference.java:254) ~[na:1.8.0_11]
       at java.lang.ref.WeakReference.<init>(WeakReference.java:69) ~[na:1.8.0_11]
       at java.io.ObjectStreamClass$WeakClassKey.<init>(ObjectStreamClass.java:2306) [na:1.8.0_11]
       at java.io.ObjectStreamClass.lookup(ObjectStreamClass.java:322) ~[na:1.8.0_11]
       at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1134) ~[na:1.8.0_11]
       at java.io.ObjectOutputStream.defaultWriteFields(ObjectOutputStream.java:1548) ~[na:1.8.0_11]

*A:* Look inside the directory where you started your Word2vec application. This can, for example, be an IntelliJ project home directory or the directory where you typed Java at the command line. It should have some directories that look like:

       ehcache_auto_created2810726831714447871diskstore  
       ehcache_auto_created4727787669919058795diskstore
       ehcache_auto_created3883187579728988119diskstore  
       ehcache_auto_created9101229611634051478diskstore

You can shut down your Word2vec application and try to delete them.

*Q: Not all of the words from my raw text data are appearing in my Word2vec object…*

*A:* Try to raise the layer size via **.layerSize()** on your Word2Vec object like so

        Word2Vec vec = new Word2Vec.Builder().layerSize(300).windowSize(5)
                .layerSize(300).iterate(iter).tokenizerFactory(t).build();

*Q: I did everything you said and the results still don't look right.*

*A:* If you are using Ubuntu, the serialized data may not be getting loaded properly. This is a problem with Ubuntu. We recommend testing this version of Wordvec on another version of Linux.

###<a name="use">Use Cases</a>

Kenny Helsens, a data scientist based in Belgium, [applied Deeplearning4j's implementation of Word2vec](thinkdata.be/2015/06/10/word2vec-on-raw-omim-database/) to the NCBI's Online Mendelian Inheritance In Man (OMIM) database. He then looked for the words most similar to alk, a known oncogene of non-small cell lung carcinoma, and Word2vec returned: "nonsmall, carcinomas, carcinoma, mapdkd." From there, he established analogies between other cancer phenotypes and their genotypes. This is just one example of the associations Word2vec can learn on a large corpus. The potential for discovering new aspects of important diseases has only just begun, and outside of medicine, the opportunities are equally diverse.

Andreas Klintberg trained Deeplearning4j's implementation of Word2vec on Swedish, and wrote a [thorough walkthrough on Medium](https://medium.com/@klintcho/training-a-word2vec-model-for-swedish-e14b15be6cb). 

Word2Vec is especially useful in preparing text-based data for information retrieval and QA systems, which DL4J implements with [deep autoencoders](../deepautoencoder.html). 

Marketers might seek to establish relationships among products to build a recommendation engine. Investigators might analyze a social graph to surface members of a single group, or other relations they might have to location or financial sponsorship. 

### <a name="next">Next Steps</a>

(We are still testing our recent implementation. of Doc2vec. An [example of Global Vectors (GLoVE) is here](https://github.com/deeplearning4j/dl4j-0.0.3.3-examples/blob/master/src/main/java/org/deeplearning4j/glove/GloveRawSentenceExample.java).)

###<a name="patent">Google's Word2vec Patent</a>

Word2vec is [a method of computing vector representations of words](http://arxiv.org/pdf/1301.3781.pdf) introduced by a team of researchers at Google led by Tomas Mikolov. Google [hosts an open-source version of Word2vec](https://code.google.com/p/word2vec/) released under an Apache 2.0 license. In 2014, Mikolov left Google for Facebook, and in May 2015, [Google was granted a patent for the method](http://patft.uspto.gov/netacgi/nph-Parser?Sect1=PTO2&Sect2=HITOFF&p=1&u=%2Fnetahtml%2FPTO%2Fsearch-bool.html&r=1&f=G&l=50&co1=AND&d=PTXT&s1=9037464&OS=9037464&RS=9037464), which does not abrogate the Apache license under which it has been released. 

###<a name="foreign">Foreign Languages</a>

While words in all languages may be converted into vectors with Word2vec, and those vectors learned with Deeplearning4j, NLP preprocessing can be very language specific, and requires tools beyond our libraries. The [Stanford Natural Language Processing Group](http://nlp.stanford.edu/software/) has a number of Java-based tools for tokenization, part-of-speech tagging and  named-entity recognition for languages such as Mandarin Chinese, Arabic, French, German and Spanish. For Japanese, NLP tools like [Kuromoji](http://www.atilika.org/) are useful. Other foreign-language resources, including [text corpora, are available here](http://www-nlp.stanford.edu/links/statnlp.html).

### Resources

* [Quora: How Does Word2vec Work?](http://www.quora.com/How-does-word2vec-work)
* [Quora: What Are Some Interesting Word2Vec Results?](http://www.quora.com/Word2vec/What-are-some-interesting-Word2Vec-results/answer/Omer-Levy)
* [Word2Vec: an introduction](http://www.folgertkarsdorp.nl/word2vec-an-introduction/); Folgert Karsdorp
* [Mikolov's Original Word2vec Code @Google](https://code.google.com/p/word2vec/)
* [word2vec Explained: Deriving Mikolov et al.’s Negative-Sampling Word-Embedding Method](http://arxiv.org/pdf/1402.3722v1.pdf); Yoav Goldberg and Omer Levy

### <a name="doctorow">Word2Vec in Literature</a>

    It's like numbers are language, like all the letters in the language are turned into numbers, and so it's something that everyone understands the same way. You lose the sounds of the letters and whether they click or pop or touch the palate, or go ooh or aah, and anything that can be misread or con you with its music or the pictures it puts in your mind, all of that is gone, along with the accent, and you have a new understanding entirely, a language of numbers, and everything becomes as clear to everyone as the writing on the wall. So as I say there comes a certain time for the reading of the numbers.
        -- E.L. Doctorow, Billy Bathgate